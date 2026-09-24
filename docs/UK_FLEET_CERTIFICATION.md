# UK fleet certification — 2026-09-24

State of the UK fleet after the closing round. Every PASS below corresponds to
a command actually executed in this round; nothing is carried over from an
earlier certification.

## Environment used

- Python 3.11.15, Linux
- PostgreSQL 16.13 (real server; the SQLite stand-in was not accepted as a substitute)
- Spark 4.1.1 with OpenJDK 21 (real parser for the Databricks SQL grammar gate)
- Live public internet for every public-source gate

## Fleet result

| Repository | Initial SHA | Certified SHA | PR | Entrypoint | Live source | PostgreSQL | PIT | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| collector_boe_dmp_uk | 2ecd28c | c59c0c6 | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_boe_fx_uk | d0585bc | 6c72ccd | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_brc_uk | a377c5e | b8b9759 | #8 | PASS | GATED | PASS | PASS | READY_WITH_ENVIRONMENT_GATE |
| collector_cbi_uk | 9f7db3c | c51441c | #9 | PASS | GATED | PASS | PASS | READY_WITH_ENVIRONMENT_GATE |
| collector_defra_uk | df2e22d | d390745 | #11 | PASS | PASS | PASS | PASS | PR open |
| collector_desnz_uk | 54e5c49 | 17cc8ea | #10 | PASS | PASS | PASS | PASS | PR open |
| collector_dft_uk | 8749600 | 1481d6e | #10 | PASS | PASS | PASS | PASS | PR open |
| collector_elexon_uk | 3286948 | e56f3fd | #11 | PASS | PASS | PASS | PASS | PR open |
| collector_hmrc_uk | f965bca | 8f654c1 | #10 | PASS | PASS | PASS | PASS | PR open |
| collector_ofgem_uk | 91bfb55 | 2b8ef66 | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_ons_awe_uk | 096e6cd | 64a4e9b | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_ons_bics_uk | 38e7b58 | 4309e83 | #4 | FIXED | FIXED | PASS | PASS | BLOCKED — methodology decision |
| collector_ons_business_prices_uk | 2c88a6d | f2e7f2e | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_ons_cpi | 6687e56 | 33a1228 | #18 | PASS | PASS | PASS | PASS | PR open |
| collector_ons_ex_cpi | 23d61f9 | **b29915c (merged)** | #15 | PASS | PASS | PASS | PASS | **MERGED** |
| collector_ons_housing_uk | 8a06dd6 | 2a736bc | #4 | FIXED | PASS | PASS | PASS | PR open |
| collector_orr_uk | e40876f | **f99d045 (merged)** | #4 | FIXED | PASS | PASS | PASS | **MERGED** |
| collector_predictor_template | 9017bfe | d9469ab | #7 | PASS | n/a | PASS | PASS | PR open |

## The defect that defined this round

`python main.py` raised `AttributeError: 'Namespace' object has no attribute
'start_date'` in **8 of 17** UK collectors. `_parse_args` executed
`return parser.parse_args(argv)` above a later `add_argument("--start-date", ...)`,
so the flag was declared but unreachable, while `run()` read `args.start_date`.

pytest, ruff, mypy and compileall were all green in every affected repository.
The 9 collectors that were **not** broken were exactly the 9 that shipped
`tests/test_cli_contract.py`, which invokes the real entrypoint.

Both guards are now in all 18 repositories.

## Live source results (2026-09-24, real publications)

| Source | Observations | Series | Range |
| --- | ---: | ---: | --- |
| BoE DMP | 799 | 11 | 2017-01-01 .. 2026-08-01 |
| BoE FX | 27,846 | 3 | 1990-01-02 .. 2026-09-23 |
| DEFRA (4 sources) | 56,084 | 266 | 1970-01-01 .. 2026-09-14 |
| DESNZ | 7,296 | 6 | 2003-06-09 .. 2026-09-21 |
| DfT | 1,376 | 16 | 2005-03-01 .. 2026-06-01 |
| Elexon | 346,768 | 196 | 2016-09-12 .. 2026-09-24 |
| HMRC (2 sources) | 5,922 | 58 | 1991-01-01 .. 2026-07-01 |
| Ofgem | 2,264 | 346 | 2017-04-01 .. 2026-10-01 |
| ONS AWE | 5,104 | 16 | 2000-01-01 .. 2026-07-01 |
| ONS BICS | 436 | 40 | 2026-03-29 .. 2026-09-20 |
| ONS Business Prices | 5,965 | 27 | 1957-01-01 .. 2026-08-01 |
| ONS Housing | 6,114 | 45 | 2015-01-01 .. 2026-08-01 |
| ORR | 1,348 | 54 | 1995-03-01 .. 2026-03-01 |
| ONS CPI | live gate passed against the current release | | |
| ONS EX-CPI | live source and live history gates passed | | |

BRC and CBI raise `PendingVendorDiscoveryError` naming their own remediation —
the correct outcome on an unentitled machine, not a code failure.

## Post-merge verification

`collector_ons_ex_cpi#15` and `collector_orr_uk#4` merged on 2026-09-24. Both
were re-verified **from a fresh clone of merged `main`**, not from the branch:

| Repository | main | Result |
| --- | --- | --- |
| collector_orr_uk | f99d045 | 347 passed, ruff and mypy clean; `--start-date` reaches the namespace |
| collector_ons_ex_cpi | b29915c | 169 passed, ruff and mypy clean |

Both merges landed complete: no branch commit was left behind, and both fleet
guards (CLI contract, standalone architecture) are present on `main`.

## Gates executed

- **Tests:** 21,317 passing fleet-wide, **0 skipped**. Every opt-in gate
  (`ONS_LIVE_TEST`, `DATABRICKS_SQL_PARSE_TEST`, `POSTGRES_TEST_URL`) was
  enabled and executed rather than left dormant.
- **Lint/type:** `ruff check`, `ruff format --check`, `mypy`, `compileall`
  clean in all 18. The mypy errors seen with a partial environment were venv
  artifacts and disappeared once the full declared dependency set was
  installed — no configuration was changed on the strength of them.
- **VERBATIM:** 17 paths byte-identical across all 18 repositories, proven by
  Git blob hash. `scripts/check_verbatim_drift.py` enforces this and was
  negative-tested: a single added newline makes it exit 1.
- **Standalone:** no `from collector_`, `import collector_`, `../collector_`,
  `sys.path` manipulation, submodule, path dependency or `file://` dependency
  in runtime code.
- **Secrets:** no `.env`, token, credential, key material or raw vendor payload
  tracked in any repository's history.

## Remaining external gates

1. **Upstream authority.** `guimasuko/collector_template` is not reachable from
   this environment — denied at the authorization layer, not merely
   unauthenticated. VERBATIM equality is therefore asserted against the fleet's
   own agreed generation, and `VERBATIM_MANIFEST.md` records no upstream SHA.
   **Close this before starting NZ/AU.**
2. **Databricks.** Grammar validated against a real Spark parser; no workspace
   reachable, so no execution is claimed. SKIP, not PASS.
3. **Bloomberg/LSEG.** Entitlement, vendor identifier discovery and a live
   vendor smoke remain for BRC and CBI.
4. **Windows corporate.** Clean-room equivalence was proved on Linux with a
   fresh venv and no siblings; the PowerShell path itself was not executed.

## Open decision

`collector_ons_bics_uk` collects live but persists nothing: the latest wave's
~5.5-month window is shorter than `MIN_HISTORY_YEARS=2`, so every series is
dropped. It now exits 1 rather than exiting 0 with an empty database. Choose
one: stitch history across archived waves, judge usability against stored
history rather than only the fresh batch, or set a source-appropriate
`MIN_HISTORY_YEARS`.
