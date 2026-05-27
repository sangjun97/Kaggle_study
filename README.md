"""
문서 전처리 파이프라인
- 암호화 문서 복호화
- PPT → PDF → 이미지 변환 (고해상도)
- 카탈로그 업데이트 및 저장
- 폴더 구조 복제
"""

import os
import sys
from pathlib import Path
import pandas as pd
import requests
import logging
from typing import Tuple, List, Dict, Optional
from datetime import datetime
from tqdm import tqdm
import comtypes.client
from pdf2image import convert_from_path


# ----------------------------------------------------------------------
# 🔧 전역 설정
# ----------------------------------------------------------------------

# 복호화 서버 URL
DECODE_URL = "http://10.150.6.47:9001/decrypt"

# 경로 설정
ROOT_DATA = "D:/python/MI/20260211_datapipeline"
SOURCE_DOC_ROOT = Path(f"{ROOT_DATA}/TI_DOCS")
DECRYPTED_DOC_ROOT = Path(f"{ROOT_DATA}/TI_DOCS_dec")
CATALOG_DIR = Path(f"{ROOT_DATA}/TI_DOCS_catalog")
IMAGE_OUTPUT_BASE = Path(f"{ROOT_DATA}/TI_DOCS_IMGS")
TEMP_PDF_DIR = Path(r"D:\python\MI\20260211_datapipeline\temp_pdfs")

# 이미지 변환 설정
IMAGE_CONFIG = {
    "dpi": 300,
    "quality": 95,
}

# 제외 확장자
EXCLUDE_EXTS = {'.xlsx', '.xlsm', '.csv'}

# 로그 설정
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("decryption_log.txt", encoding='utf-8'),
        logging.StreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger(__name__)


# ----------------------------------------------------------------------
# 1. 복호화 서버 연결 테스트
# ----------------------------------------------------------------------
def test_decrypt_api():
    """복호화 API 서버 연결 상태 확인"""
    try:
        response = requests.post(DECODE_URL, timeout=10)
        print(f"✅ 연결 성공: HTTP {response.status_code}")
        print(f"응답 헤더: {response.headers.get('Content-Type')}")
        content = response.content
        print(f"응답 본문 (앞 100자): {content[:100]}" if len(content) > 100 else f"응답 본문: {content}")
    except requests.exceptions.ConnectionError:
        print("❌ 연결 실패: 서버에 도달할 수 없습니다. 네트워크 또는 서버 확인 필요.")
        sys.exit(1)
    except requests.exceptions.Timeout:
        print("❌ 요청 타임아웃: 서버 응답 없음 (10초 초과)")
        sys.exit(1)
    except requests.exceptions.RequestException as e:
        print(f"❌ 기타 오류: {e}")
        sys.exit(1)


# ----------------------------------------------------------------------
# 2. 폴더 구조 복제 (파일 없이 디렉터리만 생성)
# ----------------------------------------------------------------------
def create_folder_structure_only():
    """원본 폴더 구조를 MI_DOCS_dec에 동일하게 생성"""
    if not SOURCE_DOC_ROOT.exists():
        print(f"❌ 원본 폴더 없음: {SOURCE_DOC_ROOT}")
        return False

    DECRYPTED_DOC_ROOT.mkdir(parents=True, exist_ok=True)

    for current_dir, dirs, files in os.walk(SOURCE_DOC_ROOT):
        current_path = Path(current_dir)
        try:
            relative_path = current_path.relative_to(SOURCE_DOC_ROOT)
        except ValueError:
            continue
        target_dir = DECRYPTED_DOC_ROOT / relative_path
        target_dir.mkdir(parents=True, exist_ok=True)
        logger.info(f"📁 생성됨: {target_dir}")

    print(f"✅ 폴더 구조 복제 완료: {SOURCE_DOC_ROOT} → {DECRYPTED_DOC_ROOT}")
    return True


# ----------------------------------------------------------------------
# 3. 동기 복호화 함수
# ----------------------------------------------------------------------
def decrypt_any_sync(file_path: str) -> Tuple[Optional[str], int, str]:
    """
    단일 파일 복호화 (동기 방식)
    :param file_path: 원본 암호화 파일 경로
    :return: (복호화된 경로, 상태 코드, 메시지)
    """
    original_path = Path(file_path)
    if not original_path.exists():
        return None, 404, f"파일 없음: {original_path}"

    # # MIME 타입 설정
    # content_type_map = {
    #     '.pdf': 'application/pdf',
    #     '.xlsx': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    #     '.pptx': 'application/vnd.openxmlformats-officedocument.presentationml.presentation',
    #     '.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    # }
    # content_type = content_type_map.get(original_path.suffix.lower(), 'application/octet-stream')

    # 타임아웃 설정
    timeout = 3600 if original_path.suffix.lower() == '.xlsx' else 600

    # 파일 읽기
    try:
        with open(original_path, 'rb') as f:
            # file_data = f.read()
            files = {'file': (file_path, f, 'application/octet-stream')} ## 변경필요
            response = requests.post(DECODE_URL, files=files, timeout=timeout)

    except Exception as e:
        return None, 500, f"파일 읽기 실패: {e}"

    # 복호화 요청
    # files = {'file': (original_path.name, file_data, 'application/octet-stream')}
    try:
        if response.status_code == 200:
            relative_path = original_path.relative_to(SOURCE_DOC_ROOT)
            output_path = DECRYPTED_DOC_ROOT / relative_path
            output_path.parent.mkdir(parents=True, exist_ok=True)
            with open(output_path, 'wb') as f:
                f.write(response.content)
            return str(output_path), 200, "복호화 성공"
        else:
            error_text = response.text[:100]
            return None, response.status_code, f"서버 오류: {error_text}"
    except requests.Timeout:
        return None, 504, "타임아웃"
    except requests.RequestException as e:
        return None, 500, f"요청 실패: {e}"
    except Exception as e:
        return None, 500, f"기타 오류: {e}"


# ----------------------------------------------------------------------
# 4. 복호화 진행 (tqdm + 로그)
# ----------------------------------------------------------------------
def decrypt_files_with_progress(df: pd.DataFrame) -> List[Dict]:
    """
    DataFrame에 포함된 파일들을 복호화하고 결과 기록
    """
    results = []
    success_count = 0
    failure_count = 0

    logger.info("🟢 복호화 작업 시작...")
    for _, row in tqdm(df.iterrows(), total=len(df), desc="복호화 진행 중", unit="file"):
        file_path = row['AbsolutePath']
        output_path, status, message = decrypt_any_sync(file_path)

        results.append({
            'original_path': file_path,
            'decrypted_path': output_path,
            'status': status,
            'message': message
        })

        if status == 200:
            success_count += 1
            logger.info(f"✅ 성공 | {output_path}")
        else:
            failure_count += 1
            logger.error(f"❌ 실패 | {file_path} | {status} | {message}")

    logger.info(f"🏁 작업 완료 | 성공: {success_count}, 실패: {failure_count}")
    return results


def unlock_pdf(file_path):
    """PDF 복호화 요청 후 파일 덮어쓰기"""
    DECRYPT_URL = 'http://10.150.6.47:9001/decrypt'
    EXCEL_MIME_MAP = {
        '.pdf': 'application/pdf'
    }

    with open(file_path, "rb") as f:
        files = {'file': (file_path, f, 'application/octet-stream')}
        response = requests.post(DECRYPT_URL, files=files)

    output_path = "unlock_" + os.path.basename(file_path)
    with open(output_path, "wb") as out_file:
        out_file.write(response.content)

    os.remove(file_path)
    os.rename(output_path, file_path)


# ----------------------------------------------------------------------
# 5. PPT → PDF 변환
# ----------------------------------------------------------------------
def ppt_to_pdfs(ppt_path: str, temp_folder: str) -> Optional[str]:
    """PPT/PPTX 파일을 PDF로 변환"""
    try:
        powerpoint = comtypes.client.CreateObject("Powerpoint.Application")
        powerpoint.DisplayAlerts = 0
        powerpoint.Visible = True
        presentation = powerpoint.Presentations.Open(str(ppt_path))
        base_name = Path(ppt_path).stem
        output_pdf = Path(temp_folder) / f"{base_name}.pdf"

        presentation.SaveAs(str(output_pdf), 32)  # 32 = ppSaveAsPDF
        presentation.Close()
        powerpoint.Quit()
        unlock_pdf(str(output_pdf))
        print(f"PPT 변환 성공 : {output_pdf}")
        return str(output_pdf)
    except Exception as e:
        print(f"[ERROR] PPT 변환 실패: {ppt_path}, 오류: {e}")
        return None


# ----------------------------------------------------------------------
# 6. PDF → 이미지 변환 (고해상도)
# ----------------------------------------------------------------------
def pdf_to_images(pdf_path: str, output_dir: str, dpi: int = 300):
    """PDF를 페이지별 고해상도 이미지로 변환"""
    try:
        images = convert_from_path(pdf_path, dpi=dpi, fmt='jpeg')
        base_name = Path(pdf_path).stem
        for i, image in enumerate(images):
            image_path = Path(output_dir) / f"{base_name}_{i+1:03d}.jpg"
            image.save(image_path, "JPEG", quality=IMAGE_CONFIG["quality"])
        logger.info(f"[SUCCESS] {len(images)}페이지 추출 (DPI={dpi}): {pdf_path}")
    except Exception as e:
        logger.error(f"[ERROR] PDF → 이미지 변환 실패: {pdf_path}, 오류: {e}")


# ----------------------------------------------------------------------
# 7. 문서 → 이미지 변환 실행
# ----------------------------------------------------------------------
def convert_documents_to_images(df: pd.DataFrame):
    """복호화된 문서를 이미지로 변환 (PDF/PPT)"""
    TEMP_PDF_DIR.mkdir(exist_ok=True)
    IMAGE_OUTPUT_BASE.mkdir(exist_ok=True)

    df_filtered = df[~df['Extension'].isin(EXCLUDE_EXTS)].copy()
    logger.info(f"처리 대상 파일 수: {len(df_filtered)}")

    for idx, row in tqdm(df_filtered.iterrows(), total=len(df_filtered), desc="문서 → 이미지 변환 중"):
        file_path = row['Dec_AbsolutePath']
        pk_id = row['PK_1']
        if pk_id!= 'Customer Meeting_0002':
            continue
        ext = Path(file_path).suffix.lower()

        output_dir = IMAGE_OUTPUT_BASE / pk_id
        output_dir.mkdir(parents=True, exist_ok=True)

        if not Path(file_path).exists():
            logger.warning(f"[SKIP] 파일 없음: {file_path}")
            continue

        try:
            if ext == '.pdf':
                pdf_to_images(file_path, output_dir, dpi=IMAGE_CONFIG["dpi"])

            elif ext in ['.ppt', '.pptx']:
                logger.info(f"[PPT] 처리 중: {file_path}")
                pdf_path = ppt_to_pdfs(file_path, str(TEMP_PDF_DIR))
                if pdf_path and Path(pdf_path).exists():
                    unlock_pdf(pdf_path)
                    pdf_to_images(pdf_path, output_dir, dpi=IMAGE_CONFIG["dpi"])
                    Path(pdf_path).unlink(missing_ok=True)  # 임시 PDF 삭제
                else:
                    logger.error(f"[FAIL] PPT → PDF 변환 실패: {file_path}")

            else:
                logger.warning(f"[SKIP] 지원하지 않는 확장자: {ext} - {file_path}")

        except Exception as e:
            logger.error(f"[ERROR] 처리 중 예외 발생 ({file_path}): {e}")

    logger.info(f"✅ 모든 문서 처리 완료.")
    logger.info(f"고해상도 이미지 저장 위치: {IMAGE_OUTPUT_BASE}")
    logger.info(f"출력 해상도: {IMAGE_CONFIG['dpi']} DPI, JPEG 품질 {IMAGE_CONFIG['quality']}")


# ----------------------------------------------------------------------
# 8. 카탈로그 전처리 및 저장
# ----------------------------------------------------------------------
def update_catalog(df: pd.DataFrame) -> pd.DataFrame:
    """카탈로그 내 확장자 및 경로 업데이트"""
    df['FileExists'] = df['AbsolutePath'].apply(lambda x: Path(x).exists() if pd.notna(x) else False)
    return df


def save_catalog(df: pd.DataFrame):
    """업데이트된 카탈로그를 CSV 및 Parquet로 저장"""
    CATALOG_DIR.mkdir(exist_ok=True)
    # today = datetime.now().strftime("%y%m%d")
    fake_date = datetime(2026, 2, 23)
    today = fake_date.strftime("%y%m%d")
    csv_path = CATALOG_DIR / f"TI_DOCS_catalog_{today}.csv"
    parquet_path = CATALOG_DIR / f"TI_DOCS_catalog_{today}.parquet"

    df.to_csv(csv_path, index=False, encoding='utf-8-sig')
    df.to_parquet(parquet_path, index=False)

    logger.info(f"✅ {csv_path} 파일이 성공적으로 저장되었습니다.")
    logger.info(f"✅ {parquet_path} 파일이 성공적으로 저장되었습니다.")


# ----------------------------------------------------------------------
# 🚀 메인 실행 함수 (수정됨: 추가된 파일만 처리)
# ----------------------------------------------------------------------
def main():
    """전체 전처리 파이프라인 실행"""
    print("🚀 문서 전처리 파이프라인 시작")

    # 1. 복호화 서버 연결 테스트
    test_decrypt_api()

    # 2. 카탈로그 로드
    # today = datetime.now().strftime("%y%m%d")
    fake_date = datetime(2026, 2, 23)
    today = fake_date.strftime("%y%m%d")
    catalog_path = CATALOG_DIR / f"TI_DOCS_catalog_{today}.parquet"
    if not catalog_path.exists():
        logger.error(f"❌ 카탈로그 파일 없음: {catalog_path}")
        sys.exit(1)
    df = pd.read_parquet(catalog_path)
    logger.info(f"📄 카탈로그 로드 완료: {len(df)}건")

    # ✅ [핵심 수정] 추가된 파일만 필터링: Dec_AbsolutePath가 비어 있거나 NaN인 경우
    df_to_process = df[
        # df['Dec_AbsolutePath'].isna() |
        # (df['Dec_AbsolutePath'] == '') |
        (df['FileExists'].isna()) |
        (df['FileExists'] == False)
    ].copy()

    logger.info(f"📌 추가된 파일 수: {len(df_to_process)}건 (총 {len(df)}건 중)")

    if len(df_to_process) == 0:
        logger.info("✅ 모든 파일이 이미 처리되었습니다. 작업 종료.")
        print("🔄 추가 작업 없음. 프로세스 종료.")
        return

    # 3. 폴더 구조 생성
    create_folder_structure_only()

    # 4. 카탈로그 전처리
    df = update_catalog(df)

    # 5. 복호화 수행: 추가된 파일만
    df_to_process['Dec_AbsolutePath'] = df_to_process['AbsolutePath'].str.replace('TI_DOCS', 'TI_DOCS_dec', n=1)
    results = decrypt_files_with_progress(df_to_process)

    # 6. 문서 → 이미지 변환: 추가된 파일만
    convert_documents_to_images(df_to_process)

    # 7. 카탈로그 업데이트: 원본 df에 결과 반영
    for result in results:
        original_path = result['original_path']
        decrypted_path = result['decrypted_path']
        status = result['status']

        mask = df['AbsolutePath'] == original_path
        if mask.any():
            df.loc[mask, 'Dec_AbsolutePath'] = decrypted_path
            df.loc[mask, 'FileExists'] = (status == 200) and (decrypted_path is not None)

    # 8. 카탈로그 저장
    save_catalog(df)

    print("🎉 전처리 파이프라인 완료!")


# ----------------------------------------------------------------------
# 실행
# ----------------------------------------------------------------------
if __name__ == "__main__":
    main()
