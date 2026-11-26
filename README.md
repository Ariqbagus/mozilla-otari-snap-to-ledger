# snap-to-ledger — Otari Fight Night (Mozilla Jam Sesh #1)

A two-stage local-model team for the Otari spreadsheet-repair fight. One small
model, two disciplined passes, and the win comes from the prompts and the stage
structure — not from a bigger model.

**Result: net repair 73.9% vs the stock team's 48.9% — a delta of +25.0 points**,
on the same messy file, the same machine, the same model. Full official record:
[fight-card.md](fight-card.md) (and [fight-card.json](fight-card.json)).
Show-and-tell video: [docs/demo.mp4](docs/demo.mp4). Long-form write-up,
including the friction log: [FIGHT-REPORT.md](FIGHT-REPORT.md). The filled
submission form from the starter kit: [SUBMISSION.md](SUBMISSION.md).

Hackathon: [Mozilla Otari Jam Sesh #1 on DoraHacks](https://dorahacks.io/hackathon/mozilla-jam-sesh-1) ·
Starter kit: [mozilla-ai/otari-fight-night](https://github.com/mozilla-ai/otari-fight-night) ·
Gateway: [mozilla-ai/otari](https://github.com/mozilla-ai/otari)

## The fight

Both teams repair the same 50-row expense sheet (63 messy rows, 88 problems),
`PRESET=medium`, `ROWS=50`, `SEED=20261120`, every request served locally
through the Otari gateway to Ollama. Net repair = (dirty cells fixed − clean
cells broken) / dirty cells.

| | Stock | snap-to-ledger |
|---|---|---|
| Net repair | 48.9% | **73.9%** (+25.0) |
| Problems fixed | 50 / 88 | **70 / 88** |
| Clean cells broken | 7 | **5** |
| Cells exactly right | 82.4% | **90.8%** |
| Duplicates kept | 0 | 0 |
| Real rows dropped | 0 | 0 |
| Junk rows kept | 1 | **0** |
| Rows lost or garbled | 26 | **2** |
| Failed requests | 0 | 0 |
| Requests | 14 | 28 |
| Request seconds | 983.9 | 1180.1 |
| Model weights | 1.93 GB | 1.93 GB |
| Net repair per GB | 25.3 | **38.3** |
| Requests served locally | 100% | **100%** |

Repair rate per column (stock → ours): date 79.2% → 75.0% · vendor 52.4% → 66.7% ·
amount 72.2% → **94.4%** · category 26.3% → **89.5%** · note 33.3% → 66.7%.

One honest loss: the date column finished slightly behind stock. Everything
else won, with category (+63.2 points) the biggest jump — the direct effect of
the synonym table and the vendor→category map in the prompt. A preliminary
smoke fight (ROWS=20, same seed) also won: 72.2% vs 58.3% (+13.9).

## Architecture: why this split

One model for every stage: plain `qwen2.5:3b` (1.93 GB, far under the 16 GB
team limit). That is a pound-for-pound decision — the denominator is gigabytes
of model weights, so a second model has to pay for itself. Here it didn't need
to: the edge came from the prompts and the stage structure.

1. **clean** (`map`, batch 5) — the main repair pass, with the full lookup
   tables inside the prompt: the canonical list of 31 vendors grouped by
   category (which doubles as the vendor→category map), the complete category
   synonym table, all seven date formats the generator produces with the
   slash-date reading rule (day-first only when the first number is > 12;
   two-digit years are always 2026), the amount normalisation rules (strip
   symbols/currency/commas; 12.3 → 12.30; 12.340 → 12.34), the amount/category
   swap rule, and a narrow drop rule (empty id, or one copy of a duplicate).
   Every table is knowledge *about* the data that the kit README explicitly
   allows — no dataset ids, no golden rows.
2. **check** (`verify`, batch 5, `retry: 1`) — a verify stage that actually
   verifies: it re-reads every row against the same tables and the clean
   sample the referee attaches, fixes what is still wrong, and sets
   `flag: true` on rows it had to guess — flagged rows are resent once by the
   referee's retry mechanism. This stage processed 51 rows in 15 requests
   including the retry pass and closed the fight with 0 garbled rows.

Why not route/triage or vote? The kit's own worked example shows a
triage-and-route team losing because a 3B model marks dirty rows `ok` and
those rows then escape every model. And voting with the referee's temperature
0.1 on the same model produces near-identical answers, so a vote only makes
sense with a different model — which doubles the gigabytes and eats the
pound-for-pound score.

Batch 5 (instead of the stock 10) is the other half of the story: replies per
request are shorter, so rows lost or garbled fell from 26 (stock) to 2.

Stage table from the card:

| Stage | Model | Requests | Rows | Seconds | Rows lost or garbled |
|---|---|---|---|---|---|
| clean | qwen2.5:3b | 13 | 63 | 535.4 | 2 |
| check | qwen2.5:3b | 15 | 51 | 644.8 | 0 |

## Integrity

All of this is on the card, which is what the organizers check against their
rerun:

- Requests served locally: **100%** for both teams.
- Stock team sha256 on the card: `13571dac3037` — the official value.
- Model digest for `qwen2.5:3b`: `357c53fb659c` — the genuine public digest, no retag.
- Answer-key check: "no dataset ids or golden rows found in prompts".
- Team file sha256: `b596ddf68f76`. Messy file sha256: `f244b6558da5` — the
  same dataset as the kit README's official example (same seed), so these
  numbers sit next to the organizers' own reference run.
- The referee ran from a fresh clone of the official kit; `fightnight/`,
  `teams/stock.yml`, `otari.yml` and `docker-compose.yml` were not modified.
  The only files written are the entrant files the kit allows: `team.yml`,
  `prompts/clean.md`, `prompts/check.md`, plus `.env` (`OLLAMA_URL` and
  `OLLAMA_URL_FROM_DOCKER` pointed at `127.0.0.1:11434` per the kit's own
  troubleshooting docs; `TIMEOUT_SECONDS` 120 → 300 because the Kaggle CPU is
  slow, a knob the referee's own warning message suggests; `SEED` pinned for
  reproducibility).
- Machine: Kaggle CPU (AMD EPYC 7B12, 31 GB RAM, no GPU). Docker cannot run
  there, so the Otari gateway ran from the official source tag v0.54.0 —
  exactly what the Docker image runs — via `uv run otari serve --config otari.yml`.

## Reproduce

In a clone of the starter kit, drop `team.yml` and `prompts/` from this repo
into place, then:

```bash
TEAM=team.yml SEED=20261120 PRESET=medium ROWS=50 make fight
```

## Files

| Path | What it is |
|---|---|
| `team.yml` | The team: two stages, one model, batch 5 |
| `prompts/clean.md` | Repair prompt with the full lookup tables |
| `prompts/check.md` | Verify prompt (re-check, fix, flag guesses) |
| `fight-card.md` / `fight-card.json` | The official fight card, full scored fight |
| `FIGHT-REPORT.md` | Full report: scores, architecture, friction log, video notes |
| `SUBMISSION.md` | The kit's submission form, filled in |
| `docs/demo.mp4` | Show-and-tell video (under 3 minutes, narrated) |

## Friction log (summary)

Docker cannot run on Kaggle, so the official Docker Compose path was replaced
by running the gateway from the official v0.54.0 source — config untouched,
no kit code changed. The dashboard does not appear on a source-run gateway
(it must be built first), so the per-key usage API became the proof surface —
and it worked exactly as promised: the card is built from gateway logs,
cleanly split per team key. Full answers to the three feedback questions are
in [SUBMISSION.md](SUBMISSION.md) and [FIGHT-REPORT.md](FIGHT-REPORT.md).

Would we put Otari in front of our own local models after this? Yes — for the
per-key accounting. One gateway in front of Ollama makes every request
attributable to the key, model and provider that actually served it, which is
exactly the layer you would otherwise build by hand for multi-agent work.
