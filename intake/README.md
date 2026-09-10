# Demand Intake

Use `collector_demands.csv` as the fleet backlog. Add one row per unique source/dataset combination, not one row per country when several countries share the same multi-country dataset.

## Required before build

The following fields cannot remain blank when status advances to `research` or beyond:

- `country`
- `country_or_currency_code`
- `source`
- `dataset`
- `source_name`
- `release_name`
- `collector_kind`
- `intended_frequency`
- `docs_url`
- `target_series` or an explicit `CURATE` instruction
- `auth`

Use semicolons inside a CSV cell containing multiple series.

## Status values

`backlog`, `blocked_input`, `research`, `series_selection`, `planned`, `building`, `audit`, `verification`, `ready`, `paused`, or `rejected`.

## Priority values

`P0` urgent/blocking, `P1` high, `P2` normal, `P3` low.
