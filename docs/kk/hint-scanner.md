# hint-scanner KK reference

## 목적
- 응답 본문에서 힌트/키워드를 탐지합니다.

## 입력(Policy/KK)
- 파라미터
  - `METHOD`, `TIMEOUT`, `PATH`
- KK 입력(주요)
  - `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 힌트 탐지
  - `HINT_KEYWORD`, `HINT_BASE64`, `HINT_JWT`, `HINT_COMMENT`
  - `HTTP_BODY_SNIPPET`, `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- 플래그/오류
  - `Flag`, `FLAG_FOUND`, `Errors`, `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`
