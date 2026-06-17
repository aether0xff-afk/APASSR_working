# Paper Evaluation Runner

`pentesting_rl.paper_eval` runs the KSEF paper-style local ablation evaluation and exports numeric metrics.

## Scope

The runner intentionally uses the paper-aligned scope:

- local target only (`127.0.0.1`)
- `nmap` only
- XML observation pipeline through `nmap -oX -`
- C0/C1/C2 ablation conditions
- no external target input
- no multi-tool adapter evaluation

## Conditions

| Condition | Label | Meaning |
| --- | --- | --- |
| C0 | `base` | uniform random baseline, no learning, no prophecy, no imagination |
| C1 | `policy_prophecy` | policy learning + prophecy-based intrinsic signal, no imagination |
| C2 | `policy_prophecy_imagination` | policy learning + prophecy + imagination candidate selection |

## Run

```bash
python -m pentesting_rl.paper_eval --runs 5 --steps 10 --out-dir runs/paper_eval
```

Multistep scenario:

```bash
python -m pentesting_rl.paper_eval \
  --scenario multistep-single-flag \
  --runs 5 \
  --steps 10 \
  --out-dir runs/paper_eval_multistep
```

Specific seeds:

```bash
python -m pentesting_rl.paper_eval --seeds 7,13,21,42,100 --runs 5 --steps 10
```

Run only C2:

```bash
python -m pentesting_rl.paper_eval --conditions C2 --runs 5 --steps 10
```

Continue after first FLAG until the full step budget is consumed:

```bash
python -m pentesting_rl.paper_eval --continue-after-flag --runs 5 --steps 50
```

## Output Files

The output directory contains:

- `paper_eval_runs.csv`: per-run raw metrics
- `paper_eval_summary.csv`: condition-level mean/std/min/max table
- `paper_eval_summary.json`: condition-level summary in JSON

## Metrics

Per run:

| Metric | Meaning |
| --- | --- |
| `flag_found` | Whether at least one FLAG was found |
| `flag_count` | Number of FLAG events observed during the run |
| `steps_to_first_flag` | 1-indexed step count until first FLAG; blank if not found |
| `first_flag_time_sec` | Wall-clock seconds until first FLAG; blank if not found |
| `total_steps` | Executed steps in the run |
| `step_efficiency` | `flag_count / total_steps` |
| `unique_action_count` | Number of unique A/B/C combinations used |
| `unique_action_ratio` | `unique_action_count / total_steps` |
| `unique_kk_count` | Number of unique KK names updated during the run |
| `avg_reward` | Mean reward over executed steps |
| `total_reward` | Sum of rewards over executed steps |
| `total_requests` | Sum of request cost reported by parser/tool output |
| `error_count` | Parser/tool error count |

## Interpretation Note

This runner makes the repository numerically evaluable in the same direction as the paper, but it does not make the current lightweight `ProphecyModel` identical to the Transformer-style model described in the text. The current default Prophecy implementation is an online lightweight predictor. The evaluation therefore reports the behavior of the current v2 codebase under paper-style constraints.
