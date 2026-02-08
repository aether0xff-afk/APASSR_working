# html-crawler KK reference

## 목적
- HTML/JS를 크롤링해 링크, 폼, 스크립트, JS 엔드포인트를 수집합니다.

## 입력(Policy/KK)
- 파라미터
  - `FETCH_SCRIPTS`, `MAX_SCRIPTS`, `MAX_PAGES`, `MAX_DEPTH`, `TIMEOUT`
  - `PATH`
- KK 입력(주요)
  - `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 크롤링 결과
  - `LINK_FOUND`, `FORM_ACTION`, `SCRIPT_SRC`, `JS_ENDPOINT`
  - `FORMS_DETECTED`, `PARAM_DETECTED`, `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- HTTP/안전/플래그
  - `HTTP_STATUS_CODE`
  - `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`
  - `SCOPE_VIOLATION`, `BLOCKED_DETECTED`, `Flag`, `FLAG_FOUND`, `Errors`
