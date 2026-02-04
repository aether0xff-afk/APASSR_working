# pentestingRL_KSEF

> **Reinforcement Learning 기반 "nmap-only" 자동 펜테스트 에이전트 프로토타입**
>
> 이 저장소는 학습 가능한 정책(WHAT/HOW/WHERE)과 XML-only 파서를 이용해
> **로컬 대상(127.0.0.1)** 에서 안전하게 동작하는 모듈형 데모를 제공합니다.
>
> ⚠️ **중요**: 실제 네트워크 대상에 대한 스캔은 법적·윤리적 책임이 따릅니다.
> 본 데모는 **로컬 루프백 환경**에서만 사용하세요.

---

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [핵심 아이디어](#핵심-아이디어)
- [전체 처리 흐름(End-to-End)](#전체-처리-흐름end-to-end)
- [모듈 구조 & 데이터 흐름](#모듈-구조--데이터-흐름)
- [지식 키(KK) 체계](#지식-키kk-체계)
- [정책(WHAT/HOW/WHERE) 구조](#정책whathowwhere-구조)
- [보상 설계](#보상-설계)
- [패키지/디렉터리 구조](#패키지디렉터리-구조)
- [설치 및 실행](#설치-및-실행)
  - [1) 빠른 실행 (더미 XML 모드)](#1-빠른-실행-더미-xml-모드)
  - [2) 로컬 nmap 데모 실행](#2-로컬-nmap-데모-실행)
  - [3) 한 줄 실행 (CLI)](#3-한-줄-실행-cli)
  - [4) GUI 실행 (Tkinter)](#4-gui-실행-tkinter)
  - [5) GUI 실행 (Streamlit)](#5-gui-실행-streamlit)
- [DVWA MVP 검증 절차](#dvwa-mvp-검증-절차)
- [커맨드 생성 규칙](#커맨드-생성-규칙)
- [로그/디버깅](#로그디버깅)
- [구성 및 커스터마이징](#구성-및-커스터마이징)
- [보안/윤리 가이드](#보안윤리-가이드)
- [FAQ](#faq)
- [개발 컨테이너 주의사항](#개발-컨테이너-주의사항)

---

## 프로젝트 개요

`pentestingRL_KSEF`는 "**nmap-only 자동 펜테스트 에이전트**"를
**강화학습 구조로 모사**하는 연구/실험용 프로토타입입니다.
실제 공격 프레임워크가 아니라, 다음의 목적을 위한 **구조적 데모**입니다.

- **WHAT/HOW/WHERE 슬롯 분리**: 스캔 명령의 선택을 세 정책으로 분리해
  각 정책의 탐색/학습 가능성을 확인
- **XML-only 파싱**: `nmap -oX -` 결과만 입력으로 사용하여
  구조화된 관측(knowledge keys)을 생성
- **예언/상상 모듈**: 비용이 큰 모델 대신 경량 predictor를 사용하여
  설계 문서의 핵심 아이디어를 재현

---

## 핵심 아이디어

1. **정책 분리(WHAT/HOW/WHERE)**
   - WHAT: 스캔 유형 및 스크립트 조합
   - HOW: 타이밍, 리트라이, 레이트 제한
   - WHERE: 포트 범위 및 대상 정의

2. **고정 관측 키(knowledge keys, KK)**
   - XML 파싱 결과를 KK 목록으로 매핑
   - 관측 벡터는 **multi-hot** 형태

3. **보상 구성**
   - 외재 보상: FLAG 탐지, 유의미한 서비스/포트 발견
   - 내재 보상: 새로운 KK 관측(탐색 가치)

4. **안전한 로컬 데모**
   - **127.0.0.1 대상**에서만 동작하도록 설계
   - 외부 네트워크 스캔 금지

---

## 전체 처리 흐름(End-to-End)

아래 흐름은 `DecisionMakingProcess`에서 수행됩니다.

1. **현재 KK 벡터 입력**
2. **WHAT/HOW/WHERE 정책에서 슬롯별 옵션 샘플링**
3. **커맨드 빌더**가 `nmap {WHAT} {HOW} {WHERE} {Target_IP} -oX -` 생성
4. **실행기(executor)**가 nmap 실행
5. **XML 파서**가 결과를 KK 업데이트로 변환
6. **보상 계산** (extrinsic + intrinsic)
7. **정책 업데이트 및 종료 조건 확인**

---

## 모듈 구조 & 데이터 흐름

```
┌──────────────────────────────┐
│ DecisionMakingProcess (dmp)  │
└──────────┬───────────────────┘
           │ KK vector
           ▼
┌──────────────────────────────┐
│ Policy A/B/C (WHAT/HOW/WHERE)│
└──────────┬───────────────────┘
           │ sampled slots
           ▼
┌──────────────────────────────┐
│ nmap Command Builder         │
└──────────┬───────────────────┘
           │ nmap command
           ▼
┌──────────────────────────────┐
│ Executor (nmap or dummy XML) │
└──────────┬───────────────────┘
           │ XML output
           ▼
┌──────────────────────────────┐
│ XML Parser                   │
└──────────┬───────────────────┘
           │ KK update
           ▼
┌──────────────────────────────┐
│ Reward + KnowledgeStorage    │
└──────────────────────────────┘
```

---

## 지식 키(KK) 체계

**KK(knowledge key)**는 관측 토큰의 고정 목록입니다.

- XML 파서는 `nmap -oX`의 결과를 **사전 정의된 KK에 매핑**
- 매핑되지 않은 정보는 `RAW_*` 형태로 저장하여
  후속 확장(정책 개선)에서 활용 가능
- 결과는 **multi-hot 벡터**로 변환되어 정책 입력으로 사용

### 예시 관측

- `PORT_OPEN_80`
- `SERVICE_HTTP`
- `OS_LINUX`
- `SCRIPT_HTTP_TITLE`
- `FLAG_FOUND`
- `HTTP_ROBOTS_FOUND`, `HTTP_ROBOTS_HAS_DISALLOW`
- `HTTP_STATUS_302`, `HTTP_HEADER_LOCATION_PRESENT`, `HTTP_HEADER_XNEXT_PRESENT`
- `PATH_HINT`, `PATH_SEEN_BUCKET_*`
- `RAW_BANNER_APACHE_2_4`

---

## 정책(WHAT/HOW/WHERE) 구조

### WHAT 정책

- 스캔 유형/스크립트 조합을 담당
- 예시 선택지:
  - `-sS` (SYN scan)
  - `-sV` (버전 탐지)
  - `-O` (OS fingerprint)
  - `--script http-headers,http-title`

### HOW 정책

- 실행 속도/안정성을 조정
- 예시 선택지:
  - `-T2`, `-T3`, `-T4`
  - `--max-retries 1`
  - `--min-rate 50`

### WHERE 정책

- 포트 범위/대상 정의
- 예시 선택지:
  - `-p 80,443,8080`
  - `-p 1-1024`
  - `--top-ports 100`

### 툴 계층(TOOL → WHAT/HOW/WHERE)

이제 정책은 **툴 선택**을 상위 계층으로 두고, 각 툴마다 WHAT/HOW/WHERE 테이블을
별도로 유지합니다. 즉, 실행 순서는 아래와 같습니다.

1. TOOL 정책이 사용할 도구를 선택 (`nmap`, `http-headers`, ...).
2. 선택된 TOOL에 대해 WHAT/HOW/WHERE 정책을 적용해 실제 커맨드 생성.

툴별 WHAT/HOW/WHERE가 동일한 의미일 필요는 없습니다. 예를 들어 `http-headers`
툴은 **curl 헤더 수집용 플래그(WHAT/HOW)** 와 **URL 경로(WHERE)** 로 구분하여
정책 테이블을 구성했습니다.

| TOOL | WHAT (A) | HOW (B) | WHERE (C) |
| --- | --- | --- | --- |
| nmap | 스캔 유형, NSE 스크립트 | 타이밍/리트라이/레이트 | 포트 범위/대상 |
| http-headers | curl 헤더/메서드 플래그 | 타임아웃/리트라이 플래그 | 요청 경로 |
| http-fetch | HTTP 메서드/헤더 | 타임아웃/리다이렉트 | 요청 경로 |
| robots-sitemap | robots/sitemap 모드 | 타임아웃 | base 경로 |
| html-crawler | HTML/JS 크롤링 모드 | 깊이/페이지 제한 | 시작 경로 |
| dir-enum | 워드리스트 크기 | 메서드/타임아웃 | base 경로 |
| hint-scanner | 힌트 스캔 모드 | 타임아웃 | 요청 경로 |
| stateful-http | 쿠키/상태 유지 | 타임아웃/리다이렉트 | 요청 경로 |
| param-influence | 파라미터 영향 측정 | 타임아웃/샘플 수 | 요청 경로 |

> 참고: `--tool auto`를 사용하면 TOOL 정책이 자동으로 도구를 선택합니다.

---

## 보상 설계

보상은 **extrinsic + intrinsic** 구조입니다.

- **Extrinsic** (외재):
  - FLAG 발견
  - 의미 있는 서비스/포트 탐지
- **Intrinsic** (내재):
  - 새로운 KK 관측

총 보상 합성:

```
Reward = Extrinsic + β * Intrinsic
β = 0.3
```

---

## 패키지/디렉터리 구조

```
.
├─ pentesting_rl/
│  ├─ __init__.py
│  ├─ __main__.py      # CLI 진입점
│  ├─ dmp.py            # DecisionMakingProcess 핵심 루프
│  ├─ knowledge.py      # KK 정의 및 KnowledgeStorage
│  ├─ parser.py         # nmap XML 파서
│  ├─ policy.py         # WHAT/HOW/WHERE 정책
│  ├─ prophecy.py       # 경량 predictor
│  ├─ reward.py         # 보상 계산
│  ├─ demo.py           # 로컬 데모 실행
│  ├─ gui.py            # Tkinter GUI
│  ├─ streamlit_app.py  # Streamlit GUI
│  ├─ run_logging.py    # 실행 로그/보고서 저장
│  ├─ run_session.py    # 실행 세션 헬퍼
│  ├─ target_utils.py   # 대상/URL 유틸
│  ├─ tools.py          # Tool 어댑터 레지스트리
│  └─ webtools.py       # HTTP 기반 Tool 어댑터
├─ docs/                # 관련 문서
└─ README.md
```

---

## 설치 및 실행

### 1) 빠른 실행 (더미 XML 모드)

`nmap`을 설치하지 않고도, 구조/루프를 검증할 수 있습니다.
기본 executor는 **더미 XML**을 반환하도록 구성 가능합니다.

```bash
python -m compileall pentesting_rl
```

```python
from pentesting_rl.dmp import DecisionMakingProcess

def run_nmap(command: str):
    # 실제 nmap 호출 대신 더미 XML 반환
    return "<nmaprun></nmaprun>", False

dmp = DecisionMakingProcess(executor=run_nmap)
dmp.run_episode()
```

---

### 2) 로컬 nmap 데모 실행

#### (1) nmap 설치

- **Windows**: [공식 설치 파일](https://nmap.org/download.html#windows)
  또는 `choco install nmap`
- **Ubuntu/Debian**:

```bash
sudo apt-get update && sudo apt-get install -y nmap
```

#### (2) 데모 실행

```bash
python -m pentesting_rl.demo
```

- 로컬 HTTP 대상이 여러 개 자동 생성
- 3 step 스캔 실행
- `http-headers`, `http-title` 등을 사용해
  서로 다른 위치(헤더/타이틀)의 FLAG를 검출

#### (3) 멀티스텝 단일 FLAG 시나리오

```bash
python -m pentesting_rl.demo --scenario multistep-single-flag --steps 5
```

- `/robots.txt` → 302 Redirect → `X-Next` 헤더 → 최종 Vault 순서로 진행
- Flask 서버는 `127.0.0.1`에만 바인딩되며, 관측은 XML-only로 처리됩니다.

---

### 3) 한 줄 실행 (CLI)

```bash
python -m pentesting_rl
```

옵션:

- `--steps N`: 에피소드 스텝 수 지정 (기본 3)
  - `--compare-random`: 랜덤 베이스라인과 정책 실행 결과 비교
  - `--target HOST`: 로컬 데모 대신 지정한 대상 IP/호스트 스캔
  - `--base-url http://HOST:PORT`: base_url 기준 대상 스캔
  - `--ports 80,443,8080`: `--target` 실행 시 사용할 포트 목록
  - `--tool nmap|http-headers|http-fetch|robots-sitemap|html-crawler|dir-enum|hint-scanner|stateful-http|param-influence|auto`: 대상 스캔에 사용할 도구 선택
  - `--report-dir PATH`: run.json/knowledge.json/graph.json/report.md 출력 경로 지정

---

### 4) GUI 실행 (Tkinter)

```bash
python -m pentesting_rl.gui
```

GUI 기능:

- 조건(Condition) 라디오 버튼: C0/C1/C2
- 시나리오 선택: `single-flag` / `multistep-single-flag`
- Target IP/base URL/포트, 에피소드 수, 스텝 수 설정
- 예언(Prophecy)/상상(Imagination) 토글
- Tool 선택, Compare-random, Report dir 출력 지원
- 실행 로그 및 요약 통계 표시

---

### 5) GUI 실행 (Streamlit)

Streamlit 기반 GUI로 **base_url 입력, 실행 옵션 선택, runlog/report 조회**를 제공합니다.

```bash
streamlit run pentesting_rl/streamlit_app.py
```

## DVWA MVP 검증 절차

DVWA Docker 환경에서 **runlog.jsonl + report.json 생성 여부**를 확인하는 절차는 아래 문서를 참고하세요.

- [docs/dvwa_mvp_test.md](docs/dvwa_mvp_test.md)

---

## 커맨드 생성 규칙

- 항상 `-oX -` 옵션 포함 (XML 출력만 파싱)
- 기본 포맷:

```
nmap {WHAT} {HOW} {WHERE} {Target_IP} -oX -
```

- Placeholder 값 기본 설정:
  - `{PORT_LIST}` → 유명 20개 포트
  - `{TOP_PORTS}` → [20, 50, 100, 200, 500, 1000]
  - `{SCRIPT_SET}` → `default/safe/http-*/banner/version/all/auth/brute/discovery/vuln/exploit/dos/broadcast/external/fuzzer/intrusive/malware`

---

## 로그/디버깅

콘솔 및 GUI 로그에는 다음 정보가 출력됩니다.

- 선택된 WHAT/HOW/WHERE 슬롯
- 실제 실행된 nmap 커맨드
- KK 업데이트 및 보상 값
- FLAG 탐지 여부
- 상상(imagination) 롤아웃 후보 및 선택된 추정 보상
- 정책 테이블(슬롯별 가중치)
- 지식 저장소(누적 KK 값)

추가 디버깅 방법:

- `print()` 로 정책 선택 및 XML 파싱 결과 확인
- `knowledge.py`에서 KK 목록 확장 시 로그 출력

---

## 구성 및 커스터마이징

### 0. 입력 하이퍼 파라미터 전체 목록 (모두 출력됨)

아래 하이퍼 파라미터는 **실행 시작 시 로그에 모두 출력**되도록 구성되어 있습니다.
CLI 실행 시에는 `[입력 하이퍼 파라미터]`와 `[DMP config]`가,
GUI 실행 시에는 `[RUN]` 로그가 각 값을 표시합니다.

#### CLI 입력값 (`python -m pentesting_rl`)

| 파라미터 | 기본값 | 설명 |
| --- | --- | --- |
| `--steps` | `3` | DMP 실행 스텝 수 |
| `--target` | `""` (빈 값) | 로컬 데모 대신 스캔할 대상 IP/호스트 |
| `--base-url` | `""` (빈 값) | base URL 기준 대상 스캔 |
| `--ports` | `80,443,8080` | `--target` 사용 시 포트 리스트 |
| `--compare-random` | `False` | 랜덤 베이스라인과 정책 비교 실행 |
| `--tool` | `nmap` | 대상 스캔에 사용할 도구 선택 |
| `--report-dir` | `""` (빈 값) | run.json/knowledge.json/graph.json/report.md 출력 경로 |

#### GUI 입력값 (`python -m pentesting_rl.gui`)

| 입력 필드 | 기본값 | 설명 |
| --- | --- | --- |
| Episodes | `3` | 실행할 에피소드 수 |
| Max steps/episode | `50` | 에피소드당 최대 스텝 수 |
| Target IP | `127.0.0.1` | 스캔 대상 |
| Rollouts | `3` | 상상(imagination) 롤아웃 횟수 |
| Condition | `C2` | C0/C1/C2 조건 선택 |
| Prophecy | `ON` | 예언 모델 사용 여부 |
| Imagination | `ON` | 상상 롤아웃 사용 여부 |

> 참고: GUI에서 Condition 선택은 예언/상상 토글 값을 자동으로 덮어씁니다.

#### DMPConfig 하이퍼 파라미터 (코드 기본값)

| 파라미터 | 기본값 | 설명 |
| --- | --- | --- |
| `max_steps` | `50` | 에피소드당 최대 스텝 |
| `beta` | `0.3` | intrinsic 보상 스케일 |
| `lambda_err` | `0.5` | 예언 모델의 에러 패널티 계수 |
| `target_ip` | `127.0.0.1` | 기본 대상 IP |
| `prophecy_enabled` | `True` | 예언 모델 사용 여부 |
| `imagination_enabled` | `False` | 상상 롤아웃 사용 여부 |
| `imagination_rollouts` | `3` | 상상 롤아웃 횟수 |
| `learning_enabled` | `True` | 정책 업데이트 활성화 여부 |

### 1. KK 확장

- `pentesting_rl/knowledge.py`에서
  `KK_LIST` 혹은 관련 상수를 수정
- 새로운 관측 항목 추가 가능

### 2. 정책 확장

- `pentesting_rl/policy.py`의
  WHAT/HOW/WHERE 행동 공간 수정

### 3. 보상 재설계

- `pentesting_rl/reward.py`에서
  extrinsic/intrinsic 비율 조정

---

## 보안/윤리 가이드

- 이 프로젝트는 **학습/연구용** 데모입니다.
- 외부 네트워크 대상 스캔은 **법적 허가** 없이 절대 실행하지 마세요.
- 로컬 환경에서만 재현 가능한 구조로 설계되어 있습니다.

---

## FAQ

**Q. 실제 외부 IP를 대상으로 스캔해도 되나요?**

A. 법적·윤리적 책임이 따르므로 권장하지 않습니다.
데모는 로컬 대상만 사용하세요.

**Q. nmap이 PATH에 없으면 어떻게 되나요?**

A. Windows에서는 기본 설치 경로를 자동 탐지하며,
필요 시 `NMAP_PATH` 환경변수로 지정 가능합니다.
찾지 못하면 더미 실행 모드로 전환됩니다.

**Q. Windows 방화벽 팝업이 뜹니다.**

A. 로컬 서버가 127.0.0.1에 바인딩되므로
`python`에 대해 허용해주세요.

---

## 개발 컨테이너 주의사항

이 개발 컨테이너는 Linux 기반이며
Windows VM을 중첩 실행할 수 없습니다.
Windows 검증은 직접 Windows 호스트에서 실행해주세요.

---

## 라이선스

별도 명시가 없다면, 본 저장소는 연구/학습 목적의 예제 코드로 제공됩니다.

---

## 참고

- nmap 공식 문서: https://nmap.org/book/man.html
- XML 출력 설명: https://nmap.org/book/output-formats-output-to-xml.html
