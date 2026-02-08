# http-headers KK reference

## 목적
- `curl -I` 기반의 HTTP 헤더를 수집하고 KK로 변환합니다.

## 입력(Policy/KK)
- 파라미터
  - `CURL_FLAGS`: `-I`, `-L`, 사용자 에이전트 등 헤더 수집 옵션
  - `PATH`/`PATH_HINT`: 요청 경로
- KK 입력(주요)
  - `PORT_LIST`: 첫 번째 포트를 대상 포트로 사용
  - `PATH_HINT`: 기본 경로 후보
  - `TARGET_IP`

## 출력(KK 업데이트)
- HTTP 헤더/리다이렉트
  - `Script_Output`, `HTTP_STATUS_302`, `HTTP_HEADER_LOCATION_PRESENT`, `HTTP_HEADER_LOCATION`
  - `HTTP_HEADER_XNEXT_PRESENT`, `HTTP_HEADER_XNEXT`, `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- 플래그/오류
  - `Flag`, `FLAG_FOUND`, `Errors`
