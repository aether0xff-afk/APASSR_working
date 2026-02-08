# param-influence KK reference

## 목적
- 파라미터 입력 변화가 응답 구조에 영향을 주는지 측정합니다.

## 입력(Policy/KK)
- 파라미터
  - `MAX_PARAMS`, `TIMEOUT`, `PATH`
- KK 입력(주요)
  - `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 파라미터 영향
  - `INPUT_OUTPUT_COUPLING_DETECTED`, `RESPONSE_STRUCTURE_VARIANCE_HIGH`
  - `STATUS_VARIANCE_DETECTED`, `PARAM_DETECTED`
- HTTP/안전/플래그
  - `HTTP_STATUS_CODE`, `HTTP_BODY_SNIPPET`, `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`
  - `SCOPE_VIOLATION`, `BLOCKED_DETECTED`, `Flag`, `FLAG_FOUND`, `Errors`
  - `PATH_HINT`, `PATH_SEEN_BUCKET_*`
