# robots-sitemap KK reference

## 목적
- robots.txt 및 sitemap.xml을 확인하고 경로 힌트를 KK로 적재합니다.

## 입력(Policy/KK)
- 파라미터
  - `MODE`(ROBOTS_ONLY/ROBOTS_SITEMAP)
  - `TIMEOUT`
  - `BASE_PATH`
- KK 입력(주요)
  - `TARGET_IP`, `PORT_LIST`, `PATH_HINT`

## 출력(KK 업데이트)
- 로봇/사이트맵
  - `HTTP_ROBOTS_FOUND`, `HTTP_ROBOTS_HAS_DISALLOW`
  - `ROBOTS_DISALLOW`, `ROBOTS_ALLOW`, `SITEMAP_URL`
  - `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- HTTP 응답/안전/플래그
  - `HTTP_STATUS_CODE`, `Errors`, `REQUEST_COUNT`, `TIME_MS`, `ERROR_COUNT`
  - `SCOPE_VIOLATION`, `BLOCKED_DETECTED`, `Flag`, `FLAG_FOUND`
