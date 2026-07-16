import cv2
import numpy as np
from typing import List, Tuple
from PIL import Image
from scipy import stats

def exists(x):
    return x is not None

def open_image(path: str) -> np.ndarray:
    x = Image.open(path)
    x = np.array(x)
    if x.ndim == 2:
        return x
    elif x.ndim == 3:
        return x[:, :, 0]
    elif x.ndim == 4:
        return x[:, :, 0, 0]
    else:
        raise Exception(f"image dimension error! the ndim is {x.ndim}")  

class GrowingDipImageProcessor:
    def __init__(self, x_margin = 0.1, y_margin = 0.1, max_width = None):
        """
        x_margin: x-direction margin of the crop image.
        y_margin: y-direction margin of the crop image.
        """       
        assert x_margin >= 0
        assert y_margin >= 0
        self.x_margin = x_margin
        self.y_margin = y_margin
        self.max_width = max_width
        
    @staticmethod
    def _bbox(image, x_margin = 0.1, y_margin = 0.1, max_width = None) -> Tuple[List[int], List[int]]:
        """
        return bounding boxes
        bbox type: [x_min, x_max, y_min, y_max]
        """

        # copy image
        image = image.copy()
        h, w = image.shape

        # resize when max_width is not None
        if exists(max_width):
            ratio = np.clip(max_width / w, 0, 1)
            image = cv2.resize(
                image, 
                dsize = (int(w * ratio), int(h * ratio)), 
                interpolation = cv2.INTER_AREA
            )
        
        # canny edge
        image = cv2.normalize(image, image, 0, 255, cv2.NORM_MINMAX)
        _, image = cv2.threshold(image, 200, 255, cv2.THRESH_BINARY)
        image = cv2.Canny(image, 50, 250, apertureSize=3, L2gradient=False)

        # bounding box without margin (tight)
        index = np.where(image > 0)
        x_min = np.min(index[1])
        x_max = np.max(index[1])
        y_min = np.min(index[0])
        y_max = np.max(index[0])

        # rescale
        if exists(max_width):
            x_min = int(x_min / ratio)
            x_max = int(x_max / ratio)
            y_min = int(y_min / ratio)
            y_max = int(y_max / ratio)

        bbox_tight = [x_min, x_max, y_min, y_max]

        # bounding box with margin
        x_margin_pixel = int(x_margin*(x_max-x_min))
        y_margin_pixel = int(y_margin*(y_max-y_min))

        x_margin_min = np.maximum(0, x_min-x_margin_pixel)
        x_margin_max = np.minimum(x_max+x_margin_pixel, w)
        y_margin_min = np.maximum(0, y_min-y_margin_pixel)
        y_margin_max = np.minimum(y_max+y_margin_pixel, h)

        bbox_margin = [x_margin_min, x_margin_max, y_margin_min, y_margin_max]
        return bbox_margin, bbox_tight
    
    @staticmethod
    def _crop(image, bbox):
        x_min, x_max, y_min, y_max = bbox
        return image[y_min:y_max, x_min:x_max]
        
    @staticmethod
    def _shape(crop_image):
        original = crop_image.copy()
        image = cv2.normalize(original, original, 0, 255, cv2.NORM_MINMAX)
        image = cv2.fastNlMeansDenoising(image, None, h=9, templateWindowSize=7, searchWindowSize=21)
        image = cv2.adaptiveThreshold(image, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)
        image = image * ((original / 255.)**2)
        image = cv2.normalize(image, image, 0, 255, cv2.NORM_MINMAX)
        _, image = cv2.threshold(image, 100, 255, cv2.THRESH_BINARY)
        return image.astype(np.uint8)
    
    @staticmethod
    def _mask(shape_image, bbox_tight):
        image = shape_image.copy()
        
        x_min, x_max, y_min, y_max = bbox_tight
        kernel_size = int(np.maximum(x_max-x_min, y_max-y_min)/15)
        kernel_size = kernel_size+1 if kernel_size%2==0 else kernel_size
        
        image = cv2.blur(image, (kernel_size, kernel_size))
        _, image = cv2.threshold(image, kernel_size, 255, cv2.THRESH_BINARY)
        return image.astype(np.uint8)
    
    @staticmethod
    def _segment(crop_image, mask_image):
        return (mask_image/255. * crop_image.astype(float)).astype(np.uint8)
    
    @staticmethod
    def _hist(segment_image):
        image = segment_image.flatten()
        image = image[np.where(image>0)]

        estimator = stats.gaussian_kde(image, bw_method="silverman")
        x = np.linspace(0, 255, num=256)
        y = estimator(x)
        return dict(x=x, y=y)
    
    def __call__(
        self, 
        image: np.ndarray, 
        only_crop = False, 
        return_hist = True
    ) -> dict:
        assert isinstance(image, np.ndarray)
        assert image.ndim == 2, "only support 1 channel image. the shape of image should be (h, w)."

        bbox, bbox_tight = self._bbox(
            image = image, 
            x_margin = self.x_margin, 
            y_margin = self.y_margin, 
            max_width = self.max_width
        )
        crop_image = self._crop(image, bbox)

        if only_crop:
            return dict(
                image = dict(
                    crop = crop_image
                ), 
                info = dict(
                    bbox = bbox, 
                    size = crop_image.shape
                )
            )

        shape_image = self._shape(crop_image)
        mask_image = self._mask(shape_image, bbox_tight)
        segment_image = self._segment(crop_image, mask_image)
        hist = self._hist(segment_image) if return_hist else None

        return dict(
            image = dict(
                crop = crop_image,
                shape = shape_image,
                mask = mask_image,
                segment = segment_image
            ),
            info = dict(
                bbox = bbox,
                size = crop_image.shape,
                hist = hist
            )
        )
