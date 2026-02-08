# dir-enum KK reference

## 목적
- 워드리스트 기반 경로 탐색으로 유효 경로를 수집합니다.

## 입력(Policy/KK)
- 파라미터
  - `WORDLIST`(small/medium/large)
  - `METHOD`, `TIMEOUT`
  - `BASE_PATH`
- KK 입력(주요)
  - `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 탐색 결과
  - `PATH_HINT`, `PATH_SEEN_BUCKET_*`, `HTTP_STATUS_CODE`
  - `LINK_FOUND`
- HTTP/안전/플래그
  - `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`
  - `SCOPE_VIOLATION`, `BLOCKED_DETECTED`, `Flag`, `FLAG_FOUND`, `Errors`
