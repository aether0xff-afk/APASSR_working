# Paper Evaluation Run Request

This file is intentionally small. It exists to create a fresh commit on `v2` after adding `.github/workflows/paper-eval.yml`, so the pull request workflow can run against the current paper evaluation runner.

Generated run target:

```bash
python -m pentesting_rl.paper_eval --runs 5 --steps 10 --out-dir runs/paper_eval
```
