# HTB-Style Local Scenarios (Draft)

아래 시나리오는 로컬에서 재현 가능한 HTB 웹 스타일을 목표로 한다.  
각 시나리오는 **관측 기반 탐사**만으로 풀 수 있어야 하며, 취약점 명칭을 사용하지 않는다.

## Scenario 1: Single Flag + Header Pivot

- 루트 페이지는 정상 HTML
- 플래그는 헤더에만 존재
- robots.txt에 1~2개 디스허용 경로

## Scenario 2: Multi-Step Redirect Chain

- /robots.txt → 302 → X-Next → Vault 경로
- 단계별로 관측값이 증가해야 탐색 효율이 올라감

## Scenario 3: Auth Boundary Shift

- 로그인 전/후 접근 가능한 URL 집합 변화
- 쿠키 세트 변경이 관측됨

## Scenario 4: API/SPA Surface

- HTML에 최소 링크
- JS 번들 내 숨겨진 API 경로 힌트
- JSON 응답 기반 탐사

## Scenario 5: Parameter Influence

- 특정 파라미터 값 변화에 따라 응답 길이/상태 변화
- “반응 민감도” 관측이 핵심 신호
