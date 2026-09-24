# International fleet bootstrap

How to start a new country fleet using the UK fleet as the baseline, without
copying British hardcodes into it.

This document is a checklist, not a licence to generate collectors. Nothing
here approves a series, a source or a repository: each still needs its own
source research and intake entry before any code exists.

## Why the UK fleet is the baseline

The UK fleet is the only fleet that has been proved end to end: entrypoint,
live public sources, PostgreSQL point-in-time behaviour, VERBATIM equality by
hash, standalone isolation, and an honest environment gate for the licensed
sources. Start from it because it is proved, not because it is British.

## What the UK round taught, and what that means for you

These are the defects the UK fleet actually shipped. Each one is cheap to
prevent in a new fleet and expensive to find later.

| What happened in UK | What to do from day one |
| --- | --- |
| `_parse_args` returned before registering `--start-date`, so `python main.py` raised `AttributeError` in 8 of 17 collectors while pytest, ruff and mypy stayed green | Ship `tests/test_cli_contract.py` in the first collector. It runs the real entrypoint through `subprocess` and fails on any `add_argument()` declared after the parser returns |
| The fleet split into two generations of VERBATIM assets; one carried the predictor guidelines under the canonical `copilot-instructions.md` filename | Record a `VERBATIM_MANIFEST.md` at fleet creation and run `scripts/check_verbatim_drift.py` in CI |
| A collector that dropped every series exited 0 with an empty database | Fail loudly on an all-dropped load. An empty load must be a decision, never a silent default |
| The point-in-time suite ran only on SQLite, which accepts types and constraints PostgreSQL rejects | Make the test engine honour `POSTGRES_TEST_URL` from the first collector |
| `_raw/` was written into the working tree and was not ignored — for licensed sources those are raw vendor payloads | Put `_raw/` in `.gitignore` in the template, before the first licensed collector exists |
| Governance said `implemented_verified` for collectors whose entrypoint was broken | Never let a status outrank its evidence. Record the certified SHA and the date beside every claim |

## Bootstrap checklist

### Country and currency vocabulary
- [ ] ISO 3166-1 alpha-3 country code fixed once, in config, never inlined at a call site.
- [ ] Currency code fixed once; no assumption that the currency symbol is `£`.
- [ ] Unit vocabulary reviewed against the canonical list. If a source publishes a diffusion index or a balance and the vocabulary has no term for it, use the canonical fallback and carry the meaning in description/provenance. **Do not invent a local unit**, and do not label a balance `percent`.
- [ ] Date conventions confirmed: fiscal year start, quarter labelling, and week numbering are country-specific and are a common source of silent off-by-one errors.

### Repository and schema naming
- [ ] `collector_<source>_<cc>` — one repository per publisher family, never one per dataset.
- [ ] Database schema name equals the repository name, asserted by `tests/test_architecture.py`.
- [ ] No shared library, no `common` package, no monorepo, no submodule. Standardization comes from the contract and the template; each collector stays self-contained.

### Source registry
- [ ] Every source has a fiche before any code: publisher, licence, URL, cadence, revision policy, release calendar.
- [ ] Public vs licensed classified explicitly. A licensed source gets an environment gate, never a fake PASS and never a scraped substitute.
- [ ] The economic publisher is recorded separately from the delivery provider. A vendor terminal is how data arrives, not who published it.

### Predictor registry and forecast targets
- [ ] Target defined before predictors, with its own collector.
- [ ] Target collector must not import a predictor collector, and vice versa; the architecture guard enforces this.
- [ ] Exclusion/aggregate ownership assigned once, with no duplicate ownership across repositories.

### Point-in-time and vintages
- [ ] Same-day correction updates the current vintage.
- [ ] Later-day revision preserves the prior vintage and creates a new one.
- [ ] An unchanged rerun performs zero writes to the data tables.
- [ ] No historical vintage is ever destroyed to simplify idempotency.
- [ ] `series_id` is deterministic, stable and round-trippable, with a migration guard.

### Release calendars
- [ ] Release dates modelled per source; do not infer availability from file mtime.
- [ ] A mutable current file does not prove a value existed in an older release — record the witnessed version.

### PostgreSQL
- [ ] `init_db` runs against an empty database.
- [ ] Fresh build, unchanged rerun, same-day revision, later-day vintage and rollback all exercised on a real server.
- [ ] Primary keys and unique constraints verified on PostgreSQL, not on the SQLite stand-in.
- [ ] `CREATE TABLE IF NOT EXISTS` is not a migration. A migration that could lose a vintage fails loudly with a runbook instead.

### Databricks
- [ ] DDL validated against a real Spark parser as the minimum gate.
- [ ] No PostgreSQL-only syntax. Note that PostgreSQL wants `DOUBLE PRECISION` and Spark wants `DOUBLE`, and a bare `FLOAT` silently means 8 bytes on one and 4 on the other.
- [ ] Without a workspace the status is SKIP with the exact missing variable named — never PASS.

### Windows corporate and clean room
- [ ] `httpx[socks]` for the corporate proxy, in both `requirements.txt` and `pyproject.toml`.
- [ ] Fresh clone, no siblings, new venv, `pip install -r requirements.txt`, no custom `PYTHONPATH`.
- [ ] Commands documented for PowerShell as well; nothing bash-only on the main path.

### Metadata and source snapshots
- [ ] Metadata matches the real publication: name, description, frequency, unit, country, native id, first/last date, publication date, source URL.
- [ ] Source snapshot retained per run, with a content digest.
- [ ] Source-specific sidecars stay out of the canonical tables.

### Filtering
- [ ] Thresholds chosen for the source's actual published window, not copied. **The UK BICS collector was configured with a 2-year minimum against a source that publishes a ~5.5-month window, so it dropped every series and persisted nothing.**
- [ ] Valid series are never removed; retired and stale series are.
- [ ] Trailing nulls must not make a dead series look fresh.
- [ ] Rotating-module surveys are normal: a question absent from one wave is not drift. Distinguish "not asked this wave" from "the layout changed".

### Research integration
- [ ] `get_predictor_as_of()` and `get_target_as_of()` respect strict point-in-time.
- [ ] Cutoffs, release dates and monthly alignment verified before any benchmark is quoted.
- [ ] Leakage tests run before pseudo-out-of-sample results are believed.

## Australia — candidate authorities

Starting points for source research. **None is approved, and no series is
specified here.** Each needs a fiche, a licence review and a cadence/revision
audit before any collector exists.

- Australian Bureau of Statistics (ABS) — CPI and the forecast target; labour, wages, producer prices.
- Reserve Bank of Australia (RBA) — policy rates, FX, statistical tables, business surveys.
- Australian Energy Market Operator (AEMO) — wholesale electricity and gas.
- Australian Energy Regulator (AER) — retail price determinations.
- Department of Climate Change, Energy, the Environment and Water — fuel and energy prices.
- Bureau of Infrastructure and Transport Research Economics (BITRE) — transport costs.
- Australian Competition and Consumer Commission (ACCC) — fuel price monitoring.
- Fair Work Commission — award wage decisions.

## New Zealand — candidate authorities

Same caveat: candidates for research, not approved sources.

- Stats NZ — CPI and the forecast target; labour cost index, producer prices, food price index.
- Reserve Bank of New Zealand (RBNZ) — policy rates, FX, surveys of expectations.
- Ministry of Business, Innovation and Employment (MBIE) — energy and fuel prices, rental bond data.
- Electricity Authority / EMI — wholesale electricity.
- New Zealand Transport Agency (Waka Kotahi) — transport costs.
- Commerce Commission — regulated price determinations.

## Order of work for a new fleet

1. Close the UK authority gate first: confirm VERBATIM equality against
   `guimasuko/collector_template` and record the upstream SHA in
   `VERBATIM_MANIFEST.md`. Do not branch a new fleet from an unconfirmed baseline.
2. Fork the template, strip UK specifics, and prove the empty template passes
   every gate before adding a source.
3. Build the forecast-target collector first.
4. Add predictor collectors one publisher family at a time, each with a fiche.
5. Wire the research layer only once the target and its point-in-time contract are proved.
