# Web Tool JSON Schema (webtool-v1)

모든 웹 계열 툴은 아래 공통 스키마를 사용한다.  
Parser/KK/Reward는 툴 비의존적으로 유지하며, **관측된 사실만 기록**한다.

## 버전

- 필드: `schema_version`
- 현재: `webtool-v1`
- Parser는 버전이 없거나 다를 경우 **graceful degrade** 해야 한다.

## 공통 스키마

```json
{
  "schema_version": "webtool-v1",
  "ok": true,
  "error": null,
  "observations": {
    "http_status": [200, 302],
    "headers_present": ["Location", "Set-Cookie"],
    "forms_detected": true,
    "params_detected": ["id", "file"],
    "redirect_depth": 2
  },
  "artifacts": {
    "urls": ["/login", "/admin"],
    "params": ["file"],
    "cookies": ["session"]
  },
  "cost": {
    "requests": 12,
    "time_ms": 840,
    "errors": 1
  },
  "safety": {
    "scope_violation": false,
    "blocked": false
  }
}
```

## 필드 규칙

- **취약점 이름 금지** (SQLi/XSS/LFI/RCE 등 직접 언급 금지)
- 오직 관측된 사실만 기록
- `observations`: 상태/결과/변화 등 관측값
- `artifacts`: 경로, 파라미터, 쿠키 등 수집 결과
- `cost`: 요청 수/시간/에러 수 (선형 패널티용)
- `safety`: scope 위반/차단 여부

## 확장 규칙

- 추가 필드가 필요하면 `observations` 또는 `artifacts` 아래에 추가
- 기존 키의 의미는 변경 금지
- Parser는 모르는 키를 무시 가능해야 한다
