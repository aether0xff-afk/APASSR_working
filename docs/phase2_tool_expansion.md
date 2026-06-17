# v2 Smoke Checkpoint and Phase 2 Expansion Plan

## Current checkpoint

The paper-aligned v2 smoke stage is complete enough to move forward.

Confirmed:

- Local-only nmap paper mode works.
- nmap XML output can contain the demo FLAG when HTTP service detection is enabled.
- Parser maps observed FLAG strings into `Flag` and `FLAG_FOUND` KK entries.
- Reward loop can receive sparse FLAG reward.
- Step-level trace exporter exists for checking selected command, KK updates, Prophecy output, Imagination candidates, grammar metadata, and FLAG observability.

Important caveat:

- The observable smoke target is intentionally easy. It verifies the loop, not final performance.
- It does not prove that C2 is better than C0/C1 yet.

## Phase 2 goal

Move from a single-tool nmap proof-of-loop to a harder local web security lab where the agent must combine multiple safe reconnaissance tools.

The goal is still not real-world exploitation. It is controlled local CTF-style reasoning:

1. observe surface
2. update KK/KV state
3. choose next tool/action
4. predict next state with Prophecy
5. compare candidate futures with Imagination
6. find FLAG in fewer attempts

## Tool expansion plan

Add safe local-lab tool adapters in this order.

### Tool 1: HTTP header probe

Purpose:

- Fetch headers from a known path.
- Observe status code, redirects, auth prompts, custom headers, and FLAG headers.

Possible KK updates:

- `HTTP_STATUS_302`
- `HTTP_HEADER_LOCATION_PRESENT`
- `HTTP_HEADER_LOCATION`
- `HTTP_HEADER_XNEXT_PRESENT`
- `HTTP_HEADER_XNEXT`
- `Flag`
- `FLAG_FOUND`
- `PATH_HINT`

### Tool 2: robots.txt probe

Purpose:

- Request `/robots.txt`.
- Extract `Disallow` paths as path hints.

Possible KK updates:

- `HTTP_ROBOTS_FOUND`
- `HTTP_ROBOTS_HAS_DISALLOW`
- `PATH_HINT`
- `PATH_SEEN_BUCKET_*`

### Tool 3: path probe

Purpose:

- Request one known or predicted path.
- Follow simple local hints such as redirect and `X-Next`.

Possible KK updates:

- HTTP status categories
- `PATH_HINT`
- redirect targets
- FLAG headers

### Tool 4: HTML link/title probe

Purpose:

- Fetch a page and parse title, links, forms, and visible path hints.

Possible KK updates:

- `Script_Output`
- `PATH_HINT`
- `RAW_NOTES`
- `Flag`

### Tool 5: local wordlist path explorer

Purpose:

- Try a tiny built-in safe wordlist against the local target only.
- This should be rate-limited and bounded.

Allowed scope:

- `127.0.0.1`
- localhost lab ports started by the demo environment
- no external targets

## Harder web lab plan

The next target should require more than one observation step.

Suggested staged local challenge:

1. `/robots.txt` reveals a decoy path and one useful path.
2. Useful path returns a redirect.
3. Redirect response contains `X-Next` header.
4. `X-Next` points to a gate path.
5. Gate path reveals another path or token in headers.
6. Final path returns the FLAG in a header.

This makes the sparse reward real because the agent must chain observations.

## Metrics for Phase 2

Primary:

- `steps_to_first_flag`
- `tool_sequence_to_flag`
- `unique_action_count`
- `failed_command_count`
- `path_hints_discovered`

Secondary:

- wall-clock time
- nmap runtime
- total requests
- Prophecy loss trend
- Imagination candidate diversity

## Required implementation checks

Before claiming performance improvements:

- `paper_eval_steps.csv` must show the actual tool/action sequence.
- `output_has_flag=True` must appear before or at `flag_found=True`.
- C2 must use Prophecy/Imagination metadata, not a fixed shortcut.
- C2 must not be locked to one trivial command in the harder lab.
- All tools must enforce local-only scope.

## Current research position

The v2 smoke result supports this limited claim:

> The APASSR loop can execute local tool actions, parse observations into KK/KV state, receive sparse FLAG reward, and expose enough trace data to evaluate Prophecy and Imagination behavior.

It does not yet support this stronger claim:

> Prophecy and Imagination outperform simpler policies on hard web tasks.

Phase 2 exists to test that stronger claim.
