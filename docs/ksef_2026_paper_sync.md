# KSEF 2026 Paper Sync Notes

This document records how the `v2` branch is aligned with the submitted KSEF 2026 paper.

## 1. Paper Identity

- Korean title: `희소 보상 문제를 해결한 자동 펜테스팅 강화학습 에이전트 개발`
- English title: `Development of an Automated Pentesting Reinforcement Learning Agent with Solving the Sparse Reward Problem`
- Core keywords: Hierarchical Reinforcement Learning, Penetration Testing, Sparse Reward Problem
- Experimental target: local CTF-style web service
- Main tool: `nmap`
- Observation format: XML via `nmap -oX -`

## 2. Submitted Paper Scope

The submitted paper evaluates a controlled prototype, not a general offensive automation framework.

Paper scope:

1. Local CTF target only.
2. Single reconnaissance tool: `nmap`.
3. XML-only observation pipeline.
4. Knowledge Storage using KK/KV.
5. Action factorization into Policy A/B/C:
   - A = WHAT
   - B = HOW
   - C = WHERE
6. Reward design combining:
   - repeated-action decay
   - error penalty
   - KK update reward
   - FLAG discovery reward
   - prediction-based intrinsic reward
7. Prophecy Module for next-state prediction.
8. Imagination Cycle for candidate action evaluation before execution.
9. DMP closed-loop orchestration.
10. Ablation comparison between baseline, policy, and policy + imagination conditions.

## 3. Code Mapping

| Paper Component | Code File | Notes |
| --- | --- | --- |
| Knowledge Storage | `pentesting_rl/knowledge.py` | Fixed KK list and KV list storage |
| XML Parser | `pentesting_rl/parser.py` | Maps `nmap -oX` XML output into KK updates |
| Policy A/B/C | `pentesting_rl/policy.py` | WHAT/HOW/WHERE probability tables |
| Reward Module | `pentesting_rl/reward.py` | Extrinsic + intrinsic reward breakdown |
| Prophecy Module | `pentesting_rl/prophecy.py` | Lightweight online predictor implementation |
| Imagination Cycle | `pentesting_rl/dmp.py` | Candidate sampling and expected reward estimation |
| DMP | `pentesting_rl/dmp.py` | Main closed-loop decision process |
| Local CTF Demo | `pentesting_rl/demo.py` | Local target server and nmap execution loop |
| Run Logging | `pentesting_rl/run_logging.py`, `pentesting_rl/run_session.py` | Run artifacts and reports |

## 4. Paper Mode Execution

Use the following commands when reproducing the paper-style experiment.

```bash
python -m compileall pentesting_rl
```

```bash
python -m pentesting_rl.demo --scenario single-flag --steps 3
```

```bash
python -m pentesting_rl.demo --scenario multistep-single-flag --steps 5
```

```bash
python -m pentesting_rl --compare-random --steps 10 --report-dir runs/ksef_ablation
```

## 5. Ablation Mapping

| Paper Condition | Meaning | Implementation Note |
| --- | --- | --- |
| C0 / base | baseline exploration | random baseline / no prophecy / no imagination |
| C1 / p | policy-based exploration | A/B/C policy with reward-driven updates |
| C2 / p+i | policy + imagination | candidate actions are pre-evaluated before execution |

The current CLI directly supports random-vs-policy comparison. Full C0/C1/C2 reproduction should be run through GUI condition selection or explicit `DMPConfig` settings.

## 6. Important Difference: Paper vs Working Extension

The working repository contains code that goes beyond the paper:

- additional HTTP/Web tool adapters
- JSON schema based internal webtool outputs
- DVWA-oriented MVP notes
- stateful HTTP and parameter influence probes

These are **post-paper extensions**. They should not be described as part of the original KSEF submitted experiment unless explicitly marked as later work.

For paper, presentation, and portfolio explanations, use this wording:

> The submitted KSEF prototype evaluates the core APASSR loop in a controlled nmap-only/XML-only local CTF environment. The current repository additionally contains experimental multi-tool extensions developed after the paper submission.

## 7. Safety Boundary

This branch is intended for controlled local testing.

- Recommended target: `127.0.0.1`
- Do not scan public IPs or domains.
- Do not use the project against systems without explicit permission.
- The submitted paper does not include exploit execution, brute forcing, credential attacks, or real-world intrusion.

## 8. Next Sync Tasks

The documentation is now aligned with the paper scope. The next implementation-level sync tasks are:

1. Add a `--paper-mode` CLI flag.
2. In `--paper-mode`, force:
   - `tool_name="nmap"`
   - local demo target only
   - XML parser only
   - no external target input
3. Add a small script for C0/C1/C2 repeated evaluation.
4. Save summary metrics as CSV:
   - condition
   - seed
   - steps_to_flag
   - first_flag_time_sec
   - total_steps
   - unique_action_ratio
   - avg_reward
   - flag_found
5. Add plots matching the paper figures.
