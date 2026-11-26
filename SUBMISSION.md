# Submission — snap-to-ledger (Otari Fight Night, Jam Sesh #1)

## 1. Fight card

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

## 2. Team

One model (`qwen2.5:3b`, 1.93 GB), two stages. **clean** (map, batch 5) does the repair with the
full lookup tables inside its prompt: the canonical vendor list grouped by category (which doubles
as the vendor→category map), the complete category synonym table, all seven date formats with the
slash-date reading rule, and the amount normalisation rules. **check** (verify, batch 5, retry 1)
re-reads every row against the same tables and the referee's clean sample, fixes what is still
wrong, and flags rows it had to guess so the retry pass takes one more look at exactly those rows.

Why this split: the scoring divides by gigabytes of model weights, so a second model has to pay
for itself — here the edge was in the prompts, not in more weights. Triage/route was rejected
because a 3B router marks dirty rows `ok` and they escape every later stage (the kit's own worked
example loses this way), and voting on one model at temperature 0.1 just repeats the same answer.
The stage that made the difference is **check**: it took the clean stage's output from good to
70 of 88 problems fixed with only 5 clean cells broken, and closed with 0 rows lost or garbled —
while batch 5 (vs the stock 10) is why rows lost mid-flight fell from 26 to 2.

## 3. Feedback (the friction log)

**Where did setup or the fight slow you down or break?**

- Docker cannot run on Kaggle (unshare is denied, no CAP_SYS_ADMIN), so the official `make fight`
  path — which runs the gateway through Docker Compose — was not available. The fix: run the Otari
  gateway from the official source tag v0.54.0, exactly what the Docker image runs, with
  `uv run otari serve --config otari.yml` and the `.env` variables exported into its process. The
  kit's stock `otari.yml` worked as-is; the referee only speaks HTTP to the gateway and Ollama, so
  not one line of kit code was changed.
- The card lists the Otari version as "unknown": the referee reads the gateway version from
  `docker compose logs`, which does not exist without Docker. Cosmetic only — the version is
  0.54.0 by construction (source tag), but the card cannot prove it from logs.
- Kaggle's CPU is slow for a 3B model: average gateway latency was 42 s (our team) to 70 s (stock)
  per request, and one full fight used about 1,180 request-seconds for our team. With the default
  `TIMEOUT_SECONDS` of 120, slow batch replies risked being thrown away as failed requests; the
  knob exists and the referee's own warning message suggests raising it, so we set it to 300.
- Putting `OLLAMA_MODELS` in `/kaggle/working` made the kernel output include the 1.9 GB model
  blob, so the output download was heavy and dropped midway once. The fight artifacts themselves
  are tiny; the lesson is to keep the model folder outside the output folder.

**What did you look for in Otari (dashboard, logs, config) and not find?**

- The dashboard at `/` does not appear on a source-run gateway: you get the get-started tutorial
  page, and the startup log itself explains the dashboard has to be built first (`make dashboard`).
  The API was unaffected, but for a jam that sells "watch your dashboard", the source-only path is
  surprising. What we ended up using to prove the flow was the per-key usage API — and it worked
  exactly as promised: the card is built from the gateway logs, cleanly separated per team key vs
  stock key.
- Startup from source floods the log with warnings about optional variables
  (`OTARI_PROVIDER_USER_IDENTIFIER_KEY`, `OTARI_PROVIDER_ACCOUNT_PEPPER`) that are irrelevant to a
  single-machine local setup; you have to read carefully to find the "running at
  http://localhost:8000" line among them.

**Would you put Otari in front of your own local models after this? Why or why not?**

Yes — for one reason proven during the fight: per-key accounting. One gateway in front of Ollama
makes every request attributable to the key, the model, and the provider that actually served it
(the `otari-provider` header), and the per-key usage summary is strong enough to be the single
auditable source for a results card. For multi-team or multi-agent experiments on your own
machine, that is exactly the layer you would otherwise have to build by hand. The caveat: the
value shows when there is more than one consumer; for a single solo script, talking to Ollama
directly is simpler.

## 4. Video

Screen recording with narration (under 3 minutes): the result, the fight card, and what we
changed in `team.yml` and why —
[docs/demo.mp4](https://github.com/Ariqbagus/mozilla-otari-snap-to-ledger/blob/main/docs/demo.mp4)
(also the video link in the DoraHacks submission).
