# Repository Instructions

This repository coordinates a fleet of standalone macroeconomic data collectors.

Before making changes, read `/MASTER_MACRO_COLLECTOR_GUIDELINES.md` completely. It is the authoritative consolidated contract and overrides stale examples in individual skills.

## Repository role

- Maintain collector standards, intake, and workflow.
- Track one row per requested source/dataset in `/intake/collector_demands.csv`.
- Use `/templates/COLLECTOR_REQUEST.md` to gather mandatory inputs.
- Keep each actual collector in its own `collector_<source>_<dataset>` repository.
- Do not create a shared collector core, base class, plugin framework, ORM layer, or undocumented schema extension.

## Required workflow

1. Normalize the request into the intake backlog.
2. Identify missing mandatory inputs.
3. Research the official source and current documentation.
4. Curate series using the series-selection skill.
5. Build from the live pilot using build-collector.
6. Verify values and metadata against the exact source.
7. Run idempotency, security, review, and Phase 8 checks.
8. Update status and handoff evidence.

Use the relevant skill under `.github/skills/` for each phase.
