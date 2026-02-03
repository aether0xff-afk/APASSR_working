# 목표 정의 (아주 중요)

HTB 웹 박스에서,  
취약점 이름이나 해설지 없이  
에이전트가 스스로 발견한 행동 시퀀스로  
목표 상태(FLAG 또는 동등한 마일스톤)에 도달하는지 검증한다.

여기서 핵심은:

“FLAG를 땄다” ❌  
“어떤 상태 전이/관측 경로를 통해 도달했는가” ⭕

---

## 1️⃣ 실험 대상 범위 설정 (HTB용)

### 1. 사용 대상

HTB Web 중심

우선순위:

- 로컬 HTB-style 시나리오 (이미 있음)
- HTB Retired 웹 박스 (재현성 확보용)
- (나중에) Active 박스는 참고용만

❗ Active 박스를 RL 학습 대상으로 쓰는 건  
요청량/재현성/윤리 측면에서 추천하지 않음.

---

## 2️⃣ “FLAG 대신 쓸 목표”를 명확히 정의

HTB 실험에서 FLAG를 직접 보상으로 쓰지 않는다.  
대신 아래 마일스톤 목표를 둔다.

### HTB Web용 마일스톤 정의

(이게 B트랙의 핵심)

**Surface Expansion**

- 발견한 엔드포인트 수
- 발견한 폼/파라미터 수

**State Transition Discovery**

- 로그인 전/후 상태 분리 성공
- reachable URL 집합 변화
- redirect chain 확장

**High-Leverage Input Detection**

- 입력 변화 → 출력 구조 변화 감지
- 응답 길이/헤더/상태 코드 분산 증가

**Deep State Reach**

- 이전에는 접근 불가하던 상태 도달  
  (예: admin-like 페이지, 내부 API, vault 경로)

**Terminal Evidence**

- FLAG 또는  
  FLAG에 준하는 고엔트로피 토큰/내부 리소스 접근 증거

👉 이 중 3~5번을 달성하면 “성공”으로 판정

---

## 3️⃣ 실험 루프를 HTB에 맞게 고정

### 한 에피소드 정의

- Target: HTB 웹 박스 1개
- Max steps: 30~50

### Budget

- 요청 수 제한
- 시간 제한
- 도구 선택: TOOL→WHAT/HOW/WHERE (자동)

### 실험 종료 조건

- 정보이득 < ε (n step 연속)
- Budget 소진
- Terminal evidence 도달

---

## 4️⃣ 반드시 수집해야 할 로그/산출물 (논문+검증용)

HTB 실험에서 이 4개는 무조건 저장해야 해.

1) **Run Log (run.json)**

- step별:
  - 선택된 TOOL/WHAT/HOW/WHERE
  - 실행 비용
  - 새로 관측된 KK
  - reward

2) **Knowledge Snapshot (knowledge.json)**

- 누적 KK
- 아티팩트(엔드포인트/파라미터/쿠키)
- 상태 전이 그래프

3) **Transition Graph (graph.json)**

- 상태 노드
- 전이 엣지(action, cost, gain)

4) **Human-readable Report (report.md)**

- “무슨 취약점” ❌
- “이 서버에서 이런 상태 전이가 발견되었다” ⭕

---

## 5️⃣ HTB 실험의 평가 지표 (FLAG 없어도 성립)

논문/후속 연구/발표에서 바로 쓸 수 있는 지표야.

### 정량 지표

- IG/request (정보이득 대비 비용)
- discovered endpoints count
- discovered state transitions count
- steps-to-first-transition
- steps-to-deep-state

### 정성 지표 (아주 중요)

에이전트가 발견한 경로가  
HTB 공식 해설과 다른 순서/다른 경로인가?

해설에 없는 우회적 상태 전이를 발견했는가?

이게 바로 “자기 방법 발견”의 증거야.

---

## 6️⃣ HTB 실험에서 절대 하지 말 것 (B트랙 금기)

- ❌ HTB 해설지 참고
- ❌ 취약점 분류로 결과 설명
- ❌ “이건 SQLi다” 같은 문장
- ❌ FLAG만 보고 성공/실패 판정

이거 하나라도 깨지면 B트랙의 의미가 사라짐.

---

## 7️⃣ B트랙의 최종 성공 기준 (명확)

HTB 웹 박스 N개 중,  
에이전트가 인간 해설과 독립적인 행동 시퀀스로  
1개 이상에서 ‘깊은 상태’ 또는 FLAG에 도달하고,  
그 과정이 관측/전이/정보이득 관점에서 설명 가능할 것

이게 충족되면:

- A(후속 연구)로 넘어갈 실험 근거 확보 완료
- 논문 확장/대회/발표 전부 가능

---

## 8️⃣ 바로 다음 액션 (오늘/이번 주 기준)

### Step 1 (가장 먼저)

HTB-style 로컬 시나리오 1개를  
**“FLAG 없는 버전”**으로 실행  
마일스톤 1~4만으로 성공 판정

### Step 2

Retired HTB 웹 박스 1개 선정  
실험 프로토콜 고정(steps/budget)

### Step 3

결과 로그 + 그래프 + 리포트 생성  
“해설과 다른 경로인지”만 비교

---

## 한 문장으로 요약 (팀 공유용)

B트랙의 목적은 HTB를 푸는 게 아니라,  
HTB에서 ‘해설지 없이도 통하는 탐사 전략’을  
강화학습 에이전트가 스스로 만들어내는지를 검증하는 것이다.
