# Instructions for Coding Agents

Read `MASTER_MACRO_COLLECTOR_GUIDELINES.md` completely before planning or modifying any collector-related file.

This repository is the fleet governance and intake hub. It is not itself a collector. Unless the user explicitly changes the architecture, every implementation belongs in its own standalone repository named `collector_<source>_<dataset>`.

Instruction precedence:

1. Current explicit user request.
2. `MASTER_MACRO_COLLECTOR_GUIDELINES.md`.
3. The live pilot implementation identified for the task.
4. Relevant skill under `.github/skills/`.
5. Generic agent defaults.

For new demands, update `intake/collector_demands.csv`, identify missing mandatory inputs, and do not invent endpoints, source IDs, metadata, units, frequency, or authentication.

Before declaring a collector ready, complete source verification, two-run idempotency, failure-path logging, security review, diff review, and the full applicable Phase 8 checklist.
