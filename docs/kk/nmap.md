# Nmap KK reference

## 목적
- `nmap` 실행 결과(XML)을 KK로 변환합니다.

## 입력(Policy/KK)
- 정책 슬롯 파라미터
  - WHAT/HOW/WHERE 조각은 `ToolAdapter.build_command()`에서 그대로 문자열로 조합됩니다.
- KK 입력(주요)
  - `SCRIPT_SET`: `--script` 선택 시 사용됩니다.
  - `PORT_LIST`, `TOP_PORTS`, `MAX_RETRIES`, `HOST_TIMEOUT`, `SCAN_DELAY`, `MIN_RATE`, `MAX_RATE`, `VER_INTENSITY`
  - `TARGET_IP`는 실제 커맨드 대상 IP로 사용됩니다.

## 출력(KK 업데이트)
- 포트/서비스
  - `Open_Ports`, `Closed_Ports`, `Filtered_Ports`
  - `Services`, `Service_Versions`
- 호스트 정보
  - `Target_IP`, `Host_Status`, `Host_OS`
- 스크립트/HTTP 힌트
  - `Script_Output`, `HTTP_ROBOTS_FOUND`, `HTTP_ROBOTS_HAS_DISALLOW`
  - `HTTP_STATUS_302`, `HTTP_HEADER_LOCATION_PRESENT`, `HTTP_HEADER_LOCATION`
  - `HTTP_HEADER_XNEXT_PRESENT`, `HTTP_HEADER_XNEXT`
  - `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- 플래그/오류
  - `Flag`, `FLAG_FOUND`, `Errors`, `RAW_XML`, `RAW_XML_KV`, `RAW_NOTES`

## 비고
- `ToolAdapter.update_knowledge_from_command()`는 `--script` 설정을 `Script_Set` 및 `SCRIPT_SET`에 누적합니다.
