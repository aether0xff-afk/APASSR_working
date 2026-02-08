# stateful-http KK reference

## 목적
- 쿠키를 유지하면서 HTTP 요청을 반복하여 상태 변화를 관측합니다.

## 입력(Policy/KK)
- 파라미터
  - `METHOD`, `TIMEOUT`, `FOLLOW`, `MAX_REDIRECTS`, `PATH`
- KK 입력(주요)
  - `HTTP_SET_COOKIE`(쿠키 jar로 변환), `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 상태/세션 관측
  - `HTTP_SET_COOKIE`, `COOKIE_SEEN`, `REDIRECT_CHAIN_DEPTH`
- HTTP/안전/플래그
  - `HTTP_STATUS_CODE`, `HTTP_HEADER_LOCATION_PRESENT`, `HTTP_HEADER_XNEXT_PRESENT`
  - `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`, `SCOPE_VIOLATION`, `BLOCKED_DETECTED`
  - `Flag`, `FLAG_FOUND`, `Errors`, `PATH_HINT`, `PATH_SEEN_BUCKET_*`
