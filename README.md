# APASSR — KSEF 2026 Paper-Synced Branch

> **Development of an Automated Pentesting Reinforcement Learning Agent with Solving the Sparse Reward Problem**  
> 희소 보상 문제를 해결한 자동 펜테스팅 강화학습 에이전트 개발

이 브랜치는 KSEF 2026 제출 논문 기준으로 저장소 설명과 실험 범위를 정리한 **paper-synced v2** 브랜치입니다.

논문 제출판의 핵심 범위는 다음과 같습니다.

- **단일 정찰 도구**: `nmap`
- **표준화된 관측 형식**: `nmap -oX -` XML 출력
- **통제된 로컬 CTF 환경**: `127.0.0.1` / loopback 기반 실험
- **상태 표현**: KK/KV Knowledge Storage와 업데이트된 KK 집합
- **행동 공간**: WHAT/HOW/WHERE, 즉 Policy A/B/C로 분해
- **희소 보상 완화**: FLAG 발견 전에도 KK 업데이트와 예측 오차를 학습 신호로 사용
- **폐루프 의사결정**: DMP가 행동 선택, 명령 실행, XML 파싱, 지식 갱신, 보상 계산, 정책 갱신, 로그 기록을 연결

> ⚠️ 이 저장소는 연구/학습용 로컬 실험 코드입니다. 허가받지 않은 실제 네트워크 대상에 대한 스캔이나 공격 실험을 목적으로 하지 않습니다.

---

## 1. Paper Scope vs Working Extensions

현재 저장소에는 논문 이후 확장 실험을 위해 여러 HTTP/Web tool adapter가 포함되어 있습니다. 하지만 **KSEF 논문 제출판의 실험 범위는 nmap-only / XML-only / local CTF** 입니다.

| 구분 | 논문 제출판 범위 | 현재 working 코드의 확장 |
| --- | --- | --- |
| 도구 | `nmap` 단일 도구 | `http-fetch`, `robots-sitemap`, `html-crawler`, `dir-enum`, `hint-scanner`, `stateful-http`, `param-influence` 등 추가 |
| 관측 | `nmap -oX -` XML | XML + 내부 JSON schema 기반 webtool 출력 |
| 대상 | 로컬 CTF 서버 | 로컬/사설망 실험용 확장 가능 구조 |
| 목적 | 희소 보상 완화 구조 검증 | 다중 도구 정책 확장 가능성 탐색 |

따라서 논문 재현 또는 발표용 설명에서는 **paper mode = nmap-only** 로 설명하는 것이 맞습니다.

---

## 2. Core Idea

APASSR의 핵심은 자동화된 명령 실행 자체가 아니라, 펜테스팅 과정을 다음과 같은 순차적 의사결정 문제로 재정의한 것입니다.

```text
관측(XML) → 지식화(KK/KV) → 상태화(ΔKK, error) → 보상 → 정책 갱신
```

기존 체크리스트 기반 점검은 사람이 미리 생각한 경로만 반복하기 쉽습니다. 이 연구는 로컬 CTF 환경에서 에이전트가 직접 행동을 선택하고, 실행 결과에서 새 지식을 얻고, 희소한 FLAG 보상 이전에도 KK 업데이트와 예측 오차를 활용해 탐색을 이어가도록 설계했습니다.

---

## 3. System Modules

| 논문 모듈 | 역할 | 코드 위치 |
| --- | --- | --- |
| Knowledge Storage | KK/KV 형태로 관측 정보 누적 | `pentesting_rl/knowledge.py` |
| Policy A/B/C | WHAT/HOW/WHERE 행동 공간 분해 | `pentesting_rl/policy.py` |
| Reward Module | 반복 감쇠, 에러 패널티, KK 업데이트, FLAG 보상, 예측 기반 보상 | `pentesting_rl/reward.py` |
| Prophecy Module | 다음 상태의 KK 업데이트 및 에러를 예측하는 모듈 | `pentesting_rl/prophecy.py` |
| Imagination Cycle | 실행 전 후보 행동을 예측 미래로 비교 | `pentesting_rl/dmp.py` |
| DMP | 행동 선택→실행→파싱→지식 갱신→보상→학습→로그 폐루프 | `pentesting_rl/dmp.py` |
| XML Parser | `nmap -oX` 결과를 KK 업데이트로 변환 | `pentesting_rl/parser.py` |
| Local CTF Demo | 통제된 로컬 실험 서버 및 실행 루프 | `pentesting_rl/demo.py` |

---

## 4. Knowledge Storage: KK/KV

Knowledge Storage는 에이전트가 실행 결과에서 얻은 정보를 구조적으로 저장하기 위한 딕셔너리 기반 저장소입니다.

- **KK(Knowledge Key)**: 명령어 생성과 상태 표현에 쓰이는 고정 키
- **KV(Knowledge Value)**: 각 KK에 대응하는 실제 관측값 리스트
- **업데이트된 KK(ΔKK)**: 한 스텝에서 새 값이 추가된 KK 집합

예시:

```text
Target_IP      → ["127.0.0.1"]
Open_Ports     → ["tcp/8080"]
Services       → ["tcp/8080:http"]
Script_Output  → ["..."]
Flag           → ["FLAG{...}"]
```

이 구조를 통해 에이전트는 단순 로그 문자열이 아니라, “무엇을 새롭게 알게 되었는가”를 상태로 사용할 수 있습니다.

---

## 5. Policy A/B/C

행동은 하나의 거대한 명령어 공간으로 두지 않고 세 축으로 분해합니다.

| 정책 | 의미 | 예시 |
| --- | --- | --- |
| Policy A / WHAT | 무엇을 할 것인가 | 스캔 유형, NSE script 조합 |
| Policy B / HOW | 어떻게 수행할 것인가 | timing, retry, rate, timeout |
| Policy C / WHERE | 어디를 대상으로 할 것인가 | port range, target port set |

명령 생성 형식:

```bash
nmap {WHAT} {HOW} {WHERE} {Target_IP} -oX -
```

이 분해는 행동 공간을 해석 가능하게 만들고, 옵션을 축별로 확장할 수 있게 합니다.

---

## 6. Reward Design

보상은 외재 보상과 내재 보상을 결합합니다.

### 6.1 Extrinsic Reward

- 동일 행동 조합 재실행 감쇠
- syntax/runtime error penalty
- KK 업데이트 발생 보상
- FLAG 발견 보상

### 6.2 Intrinsic / Prediction-Based Reward

FLAG는 희소하게만 등장하므로, 매 스텝 더 자주 관측되는 신호가 필요합니다.

이 연구에서는 Prophecy Module이 다음 상태의 KK 업데이트와 오류 발생을 예측하고, 실제 관측과의 차이를 보상 신호로 사용합니다. 즉, FLAG 이전에도 “새로운 정보가 발생할 가능성”과 “예측 불확실성”을 통해 탐색을 유도합니다.

---

## 7. Prophecy and Imagination

### Prophecy Module

최근 전이 데이터와 행동 시퀀스를 바탕으로 다음 상태를 예측합니다.

```text
aseq_t = {s_{t-1}, a_t, s_t}
예측 대상 = {ΔKK_{t+1}, e_{t+1}}
```

현재 working 구현의 `ProphecyModel`은 논문 구조를 재현하기 위한 경량 online predictor 인터페이스입니다. 논문에서 설명한 구조처럼 Transformer/RNN/1D-CNN 등 다른 시계열 모델로 교체할 수 있도록 모듈 경계를 분리해 두었습니다.

### Imagination Cycle

실제 명령을 실행하기 전에 여러 후보 행동을 만들고, Prophecy를 이용해 후보별 미래를 예측합니다. 그중 기대 누적 보상이 큰 행동을 실제 실행합니다.

---

## 8. Paper Reproduction Mode

### 8.1 준비

Python 환경에서 코드 문법 검사를 먼저 수행합니다.

```bash
python -m compileall pentesting_rl
```

`nmap`이 필요합니다.

Windows:

```powershell
choco install nmap
```

Ubuntu/Debian:

```bash
sudo apt-get update && sudo apt-get install -y nmap
```

### 8.2 로컬 CTF 데모 실행

```bash
python -m pentesting_rl.demo --scenario single-flag --steps 3
```

멀티스텝 로컬 시나리오:

```bash
python -m pentesting_rl.demo --scenario multistep-single-flag --steps 5
```

### 8.3 기준선 비교 실행

```bash
python -m pentesting_rl --compare-random --steps 10 --report-dir runs/ksef_ablation
```

현재 CLI 비교는 random baseline과 policy run을 중심으로 동작합니다. 논문식 C0/C1/C2 전체 비교는 GUI 조건 선택 또는 config-level 실행으로 맞추는 것이 가장 정확합니다.

---

## 9. Ablation Conditions

논문 설명 기준 조건은 다음처럼 정리합니다.

| 조건 | 의미 | 설명 |
| --- | --- | --- |
| C0 / base | 기준선 | 정책 학습, 예언, 상상 없이 기본 탐색 |
| C1 / p | 정책 + 예언 | A/B/C 정책과 Prophecy 기반 보상 사용 |
| C2 / p+i | 정책 + 예언 + 상상 | 실행 전 후보 행동을 비교한 뒤 선택 |

평가 지표:

- 총 스텝 수
- FLAG 최초 발견 시간
- 스텝 효율
- 전략 다양성
- 고유 행동 조합 수
- 반복 실행 감소 여부

---

## 10. Output Logs and Reports

실행 시 다음 정보를 기록합니다.

- 선택된 A/B/C option
- 생성된 nmap command
- XML parser가 추출한 KK update
- reward breakdown
- FLAG 발견 여부
- Prophecy/Imagination 관련 정보
- policy table snapshot
- knowledge storage snapshot

`--report-dir`를 지정하면 run log와 report artifact를 저장할 수 있습니다.

---

## 11. Safety and Ethics

이 브랜치의 논문 재현 범위는 로컬 CTF / loopback 실험입니다.

- 허가받지 않은 외부 네트워크 스캔 금지
- 실제 서비스 대상 실험 금지
- exploit, brute force, credential attack을 목표로 하지 않음
- 논문 실험은 방어적 연구와 통제된 환경 검증을 목적으로 함

---

## 12. Known Differences from the Submitted Paper

이 저장소는 논문 제출 후 계속 확장된 working repository입니다. 따라서 다음 차이가 존재합니다.

1. 논문 제출판은 `nmap-only`였지만, working code에는 multi-tool adapter가 추가되어 있습니다.
2. 논문은 Prophecy를 시계열 예측 모델 구조로 설명하지만, 현재 기본 구현은 의존성을 줄인 lightweight online predictor입니다.
3. 논문 실험은 로컬 CTF 환경 중심이며, working code에는 DVWA 및 web reconnaissance 확장 문서가 포함되어 있습니다.
4. 논문 재현을 위해서는 `v2` 브랜치에서 nmap/local demo 중심으로 실행하는 것을 권장합니다.

---

## 13. Citation / Project Identity

Research title:

```text
희소 보상 문제를 해결한 자동 펜테스팅 강화학습 에이전트 개발
Development of an Automated Pentesting Reinforcement Learning Agent with Solving the Sparse Reward Problem
```

Author:

```text
이은세, 충남과학고등학교
Lee-Eun Se, Chungnam Science High School
```

Keywords:

```text
Hierarchical Reinforcement Learning, Penetration Testing, Sparse Reward Problem, Knowledge Storage, Prophecy, Imagination Cycle
```
