# 📌 Developer Note

## HTB(Web) 타겟 다중 툴 강화학습 펜테스팅 에이전트 – 업그레이드 가이드

### 목적 요약

- **목표 플랫폼**: HTB(Web 중심)
- **목표 철학**:
  - 기존 해설지/취약점 분류를 학습하지 않음
  - 에이전트가 **스스로 탐색 전략을 발명**
  - 다중 도구 환경에서 **정보이득 대비 비용을 최적화**하는 강화학습 에이전트
- **장기 목표**:
  - 연구용 → 실전성 확보 → 상용화 가능 구조

---

## 1️⃣ 핵심 방향성 (반드시 지킬 것)

### ❌ 하지 말 것

- “이 툴은 SQLi용 / XSS용” 같은 의미적 분류
- 취약점 이름(Sqli, XSS, LFI 등)을 상태(KK)나 보상에 직접 사용
- 플래그만을 유일한 보상으로 사용

### ⭕ 반드시 할 것

- **툴 = 감각기관 + 근육**
- **관측 = 물리적 사실**
- **보상 = 정보이득 / 비용**
- **정책 = 실험 설계**

---

## 2️⃣ 아키텍처 유지 + 확장 원칙

### 기존 구조 (유지)

```
TOOL
 └─ WHAT / HOW / WHERE
     └─ Command Builder
         └─ Executor
             └─ JSON Output
                 └─ Parser
                     └─ Knowledge(KK)
                         └─ Reward
                             └─ Policy Update
```

### 확장 원칙

- nmap-only → **multi-tool**
- 하지만:
  - 모든 툴은 **동일한 JSON 출력 스키마**를 사용
  - Parser/KK/Reward는 툴 비의존적으로 유지

---

## 3️⃣ 필수 공통 JSON 출력 스키마 (모든 툴)

```json
{
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

> ⚠️ 중요
>
> - **“취약점”이라는 단어는 절대 JSON에 넣지 말 것**
> - 오직 “관측된 사실”만 기록

---

## 4️⃣ HTB 기준 최소 필수 툴 세트 (1차)

### TOOL-A: Stateful HTTP Executor

**역할**

- 쿠키 유지
- CSRF 토큰 추적
- 로그인 전/후 상태 분리

**관측 예**

- 인증 전/후 응답 변화
- 새로운 쿠키 등장
- 접근 가능한 URL 집합 변화

---

### TOOL-B: Parameter Influence Analyzer

**역할**

- 파라미터/헤더/바디 변형
- 입력 변화 → 출력 변화량 측정

**관측 예**

- 응답 길이 분산
- 에러 발생 여부
- 반사 위치(HTML/속성/JSON 등)

---

### TOOL-C: Crawler + JS Hint Collector

**역할**

- HTML 링크
- JS 번들 내 경로/엔드포인트 힌트 수집

**관측 예**

- API 경로 힌트
- 동적 라우트 후보
- 숨겨진 엔드포인트 문자열

---

## 5️⃣ Knowledge Key(KK) 확장 가이드 (HTB용)

### ❌ 나쁜 예

```yaml
SQLI_POSSIBLE: true
XSS_FOUND: true
```

### ⭕ 좋은 예

```yaml
INPUT_OUTPUT_COUPLING_DETECTED
RESPONSE_STRUCTURE_VARIANCE_HIGH
AUTH_BOUNDARY_SHIFT
STATE_TRANSITION_CHAIN_LENGTH_3
ERROR_DISCLOSURE_PATTERN
```

> KK는 **“현상”**만 기록
> 해석은 에이전트가 스스로 하게 둘 것

---

## 6️⃣ 보상 함수 업그레이드 (필수)

### 기존

```
Reward = FLAG + β * new_KK
```

### HTB용

```
Reward = (InformationGain) - λ * Cost
```

#### InformationGain 예시

- 새로운 KK
- 새로운 엔드포인트/파라미터
- 새로운 상태 전이(로그인 전 → 후)

#### Cost 예시

- 요청 수
- 시간
- 차단(403/429)

---

## 7️⃣ TOOL 정책의 의미 재정의

### 기존

- “어떤 툴을 쓸까?”

### 변경

> **“현재까지의 관측으로 세운 가설을
> 가장 싸게 검증할 툴은 무엇인가?”**

예:

- 리다이렉트 많음 → redirect graph 확장 툴
- 인증 경계 불명확 → stateful HTTP
- API 냄새 → 스키마/JS 분석

---

## 8️⃣ 훈련 전략 (HTB 대응)

### ❌ 바로 HTB 실전 박스

- 재현성 낮음
- 요청량 관리 어려움

### ⭕ 권장

1. 로컬 HTB-스타일 시나리오 N개
2. 난이도 커리큘럼
   - 단일 플래그
   - 멀티 스텝
   - 인증 포함
   - API/SPA 포함
3. 이후 HTB Retired/Academy 범위 확장

---

## 9️⃣ 절대 잊지 말 것 (상용/윤리)

- 모든 툴에 **scope check 필수**
- 요청/시간/부하 제한
- 감사 로그 남길 수 있게 설계
- “공격”이 아니라 **“탐사” 용어 사용**

---

## 10️⃣ 다음 스프린트 목표 (현실적인 TODO)

### Sprint 1

- [ ] 공통 JSON 스키마 확정
- [ ] Stateful HTTP Executor 추가
- [ ] KK 확장 (관측 중심)

### Sprint 2

- [ ] Parameter Influence Analyzer
- [ ] 정보이득/비용 보상 적용
- [ ] TOOL 정책을 “실험 설계”로 변경

### Sprint 3

- [ ] HTB 스타일 로컬 시나리오 5종
- [ ] 리포트 템플릿 초안

---

## 한 줄 정리 (개발팀 공유용)

> **이 프로젝트는 “자동 해킹 도구”가 아니라
> “웹 시스템을 스스로 이해하는 강화학습 탐사 에이전트”를 만든다.
> HTB는 목표이지만, 해설지는 절대 사용하지 않는다.**
