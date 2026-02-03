## DVWA Docker 기반 MVP 검증 절차

본 문서는 **runlog.jsonl + report.json 생성**을 확인하는 최소 성공 루프(MVP) 검증 절차입니다.

### 1) DVWA 컨테이너 실행

예시 (포트 8080 → 80):

```bash
docker run --rm -it -p 8080:80 vulnerables/web-dvwa
```

### 2) 에이전트 실행 (base_url 기준)

호스트에서 실행할 경우:

```bash
python -m pentesting_rl --base-url http://127.0.0.1:8080 --steps 5 --tool http-headers
```

같은 Docker 네트워크에서 실행할 경우:

```bash
python -m pentesting_rl --base-url http://dvwa:80 --steps 5 --tool http-headers
```

### 3) 산출물 확인

- `runs/<run_id>/runlog.jsonl`
- `runs/<run_id>/report.json`

### 4) 합격 기준 체크리스트

1. **run_id 포맷**
   - `YYYYMMDDTHHMMSSZ_<rand6>` (예: `20260203T104233Z_k3p9xq`)
2. **runlog 인덱스 규칙**
   - 첫 줄 `runlog_line=0`, `step_index=0`
   - 모든 라인에 `step_index` 존재
3. **unique_paths_top 규칙**
   - `/dvwa/login.php` 방문 시 `/dvwa` 포함
4. **artifact 실패 내구성**
   - 저장 실패 시에도 런 지속, `*_ref=null`, `errors` 기록, report.json 생성 유지
