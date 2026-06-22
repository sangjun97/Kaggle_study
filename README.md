(dsstar) PS D:\python\etc\20260622_DS_STAR> echo $env:CORP_LLM_BASE_URL
http://10.150.6.47:8000/v1/chat/completions
(dsstar) PS D:\python\etc\20260622_DS_STAR> curl.exe -v $env:CORP_LLM_BASE_URL/models
*   Trying 10.150.6.47:8000...
* Connected to 10.150.6.47 (10.150.6.47) port 8000
> GET /v1/chat/completions/models HTTP/1.1
> Host: 10.150.6.47:8000
> User-Agent: curl/8.8.0
> Accept: */*
>
< HTTP/1.1 404 Not Found
< content-length: 0
< date: Mon, 22 Jun 2026 06:12:50 GMT
<
* Connection #0 to host 10.150.6.47 left intact
