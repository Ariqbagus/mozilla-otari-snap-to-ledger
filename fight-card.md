# Fight card: Otari Fight Night

**snap-to-ledger** beat the stock team by **+25.0 points**: net repair 73.9% vs 48.9% on 50 rows (63 messy, 88 problems), preset `medium`. Generated 2026-11-26T02:11:00+00:00.

Net repair = (dirty cells fixed - clean cells broken) / dirty cells. An untouched file scores 0. Removing duplicates and junk earns nothing (the referee can do that); keeping them costs.

## Machine

- OS: Linux 6.18.55+ (x86_64)
- CPU: AMD EPYC 7B12
- RAM: 31 GB
- GPU: none detected
- Ollama 0.40.18, Otari unknown at http://localhost:8000

## Stock vs your team

| | stock | yours |
|---|---|---|
| Team file | `teams/stock.yml` | `team.yml` |
| Models | `qwen2.5:3b` | `qwen2.5:3b` |
| Net repair | 48.9% | **73.9%** (+25.0) |
| Problems fixed | 50 / 88 | 70 / 88 |
| Clean cells broken | 7 | 5 |
| Cells exactly right (reference) | 82.4% | 90.8% |
| Duplicates kept | 0 | 0 |
| Real rows dropped | 0 | 0 |
| Junk rows kept | 1 | 0 |
| Rows a model lost or garbled (kept as-is) | 26 | 2 |
| Failed requests (timeouts, gateway errors) | 0 | 0 |
| Real rows a model tried to delete (refused) | 32 | 37 |
| Request seconds (including failures) | 983.9 | 1180.1 |
| Requests | 14 | 28 |
| Model weights loaded | 1.93 GB | 1.93 GB |
| Net repair per GB | 25.3 | 38.3 |
| Requests served locally | 100% | 100% |

Repair rate per column, stock: date 79.2%, vendor 52.4%, amount 72.2%, category 26.3%, note 33.3%

Repair rate per column, yours: date 75.0%, vendor 66.7%, amount 94.4%, category 89.5%, note 66.7%

## Otari usage (from the gateway's own log, per team key)

| | stock | yours |
|---|---|---|
| Requests | 14 | 28 |
| Errors | 0 | 0 |
| Prompt tokens | 19868 | 47917 |
| Completion tokens | 6934 | 8142 |
| Avg gateway latency (ms) | 70242 | 42110 |
| Cost | $0.00 | $0.00 |

## Your team, stage by stage

| Stage | Model | Requests | Rows | Seconds | Rows lost or garbled | Failed requests |
|---|---|---|---|---|---|---|
| clean | qwen2.5:3b | 13 | 63 | 535.4 | 2 | 0 |
| check | qwen2.5:3b | 15 | 51 | 644.8 | 0 | 0 |

### Where the work went

| Stage | Prompt tokens | Completion tokens | Token share | Request time share | Providers (requests) |
|---|---|---|---|---|---|
| clean | 24445 | 3613 | 50.1% | 45.4% | ollama: 13 |
| check | 23472 | 4529 | 49.9% | 54.6% | ollama: 15 |

Most request time: **check** (54.6%). Most tokens: **clean** (50.1%).

These shares measure cost, not quality: to see whether a stage earns its place, remove or swap it and rerun on the same seed. Request time sums model calls, failed ones included; it is not wall-clock time. Providers come from Otari's response headers; a failed request has none and shows as unknown.

## Integrity (what the organizers check against the rerun)

- Referee code sha256 `f8ffb651e804`, git `c90c3df`
- Model digests from Ollama: `qwen2.5:3b` 357c53fb659c
- Otari reached Ollama at `http://127.0.0.1:11434`
- Prompt bytes: stock 1746, yours 5776
- Answer-key check: no dataset ids or golden rows found in prompts

## Reproduce

`TEAM=team.yml SEED=20261120 PRESET=medium ROWS=50 make fight`. Messy file sha256 `f244b6558da5`. Team file sha256 `b596ddf68f76`, stock `13571dac3037`.
