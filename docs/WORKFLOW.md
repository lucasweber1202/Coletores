# Collector Fleet Workflow

## Architecture decision

`Coletores` is the governance and demand-management hub. Each source/dataset implementation remains a separate repository named `collector_<source>_<dataset>`.

## Status flow

| Status | Meaning | Exit criterion |
|---|---|---|
| `backlog` | Demand recorded | Mandatory intake reviewed |
| `blocked_input` | Required information missing | Missing fields supplied |
| `research` | Official docs and source IDs being verified | Endpoint, auth, scope, frequency and sample values confirmed |
| `series_selection` | Coverage is being curated | Selection checklist approved |
| `planned` | Build plan ready | Plan approved |
| `building` | Standalone collector under implementation | Fresh run succeeds |
| `audit` | Metadata and values checked against source | No blocker/major mismatch |
| `verification` | Quality gates and rerun behavior checked | All applicable Phase 8 items pass |
| `ready` | Ready for use/PR | Handoff evidence recorded |
| `paused` | Intentionally deferred | Owner resumes it |
| `rejected` | Source/series unsuitable | Reason documented |

## Intake gate

A demand cannot leave `backlog` until the following are known:

- country and currency/country code;
- official agency/source;
- dataset/release;
- official documentation or source page;
- intended modelling frequency;
- target series or authorization to curate them;
- authentication method;
- collector kind: realized, forecast/projection, or forecast target.

## Research gate

Confirm the official endpoint/file, current documentation, native IDs, publication cadence, scope, unit, available history, revision behavior, rate limits and authentication. Verify at least one observation before implementation.

## Build gate

Build from the live pilot. Preserve the standardized schema, structured IDs, source-specific extraction, vintage behavior, batch writes, logging and self-containment rules.

## Audit gate

Audit against the same official API/page/file used by the collector. An audit reports findings and does not silently fix them.

## Verification gate

Minimum evidence:

- fresh database initialization;
- successful end-to-end run;
- populated standardized tables;
- immediate second run with zero metadata and observation writes;
- one new success log on the second run;
- forced failure produces an error log with traceback;
- official value samples match;
- no duplicate vintage key;
- no NaN/infinity/secret;
- diff and dependency review complete.

## Handoff

Record repository URL, number of series, observation range, authentication requirements, special semantics, verification result, remaining external limitation and next operational action in the backlog.
