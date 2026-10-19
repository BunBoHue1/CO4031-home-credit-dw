# Repository Guidelines

## Read first and ownership

Read README.md, TASKS.md, CONTRIBUTING.md, dataset_guide.md and schema_doc.md, then AGENTS.md in your code area. Identify task ID, owner, dependencies and acceptance criteria before editing.

Phúc owns business/KPIs, database, Python ETL/CLI, warehouse validation and OLAP/reporting. Kiên owns Express backend, React frontend, DSS, integration, evaluation and delivery. Both review contracts and contribute report/slides/presentation.

## Architecture and scope

Local PostgreSQL analytical core, Python ETL, JavaScript Express and React. No Docker or Power BI. schema_doc.md is the chosen five-dimension/four-fact baseline. Validate against it; do not redesign without team agreement and documentation updates. Frontend calls Express; credentials stay server-side.

Code goes in src/db, src/etl, src/be, src/fe; tests holds verification; docs holds contracts/demo/business/report/slides. Preserve raw data_set/ CSVs. Current repository contains documentation/placeholders only; no runnable application, manifests or test framework.

ETL/backend explicitly read root .env copied from .env.example. Resolve DATA_DIR from repository root regardless of working directory. Python orchestrates chunked input/bulk loading and SQL transformations; pin tool versions and dependencies when implemented. Commit manifests/lockfiles, not environments, output or secrets.

DSS is required but its form is undecided; screening-policy simulator is only a candidate. Define business questions and metric denominators first. TARGET is an observed payment-difficulty label, never a screening/model input.

## Data invariants

Restrict history to application_train customers. Preserve source IDs and relative dates. Validate grain, uniqueness, foreign keys, reconciliation and repeat-run idempotency. Aggregate installments by (SK_ID_PREV, NUM_INSTALMENT_VERSION, NUM_INSTALMENT_NUMBER); count scheduled amount once. Completion is first cumulative payment reaching scheduled amount, only observed payments through day 0. Quarantine conflicts. Combine monthly sources with UNION ALL. Preserve Unknown, Not applicable and NULL. Do not sum snapshot balances across months or join facts in a way that multiplies rows.

Two demo batches partition sorted unique train IDs at floor(N/2); include all related history. See docs/demo.md for manifest, CLI, reset and publication rules. Operational batch tracking stays outside the five-dimension/four-fact model. Reporting exposes only completed batches.

## Commands and style

No build/ETL/test commands implemented. Read-only PowerShell inspection:

```powershell
Get-Content -Encoding UTF8 schema_doc.md
Get-ChildItem -Recurse data_set -Filter *.csv
```

Preserve UTF-8 Vietnamese. Warehouse snake_case, source identifiers unchanged. JavaScript two spaces, camelCase functions/variables, PascalCase React components. Add actual setup/run/check commands to relevant README as code is added.

## Handoff and Git

Use feat/fix/docs branches and PRs to main. Stay within task scope; do not silently change schema, KPI or contract. Update TASKS/docs. Handoff states behavior, affected mappings/interfaces, commands actually run, results/evidence, limitations and remaining work. Mark unrun checks unverified with reasons. DONE requires acceptance evidence; BLOCKED records the cause. Never claim placeholder functionality works.

Deadline November 15, 2026, 23:59 Vietnam time. Deliver report/slides PDFs, src/, sample data/ and video link on report cover if applicable; all members present. Preserve assignment PDF and source documentation.
