---
name: gen-api-test
description: Create or validate a case-specific automation plan and generate one Kotlin/JUnit 5 REST Assured API test from one API Test Case and the relevant OpenAPI operation in this repository. Use when asked to plan, generate or cover one API scenario. Follow the rule/client/model/tests/testdata architecture and verify exact response behavior; never use for Appium tests, backend fixes or open-ended repair.
---

# Generate an API test

## Purpose

Create or validate one repository-specific automation plan, then create one
verified black-box API test from the supplied scenario. Treat OpenAPI as the
specified contract and a fresh response as runtime evidence. Report any
disagreement; do not rewrite either source.

## Required inputs

| Input | Requirement |
|---|---|
| Test Case | One API behavior with explicit preconditions, request data and expected results |
| Plan | Required working artifact after the coverage preflight confirms a gap; create it from `agent_docs/templates/automation_plan.api.workflow.md.template` when missing, or validate the existing `agent_docs/automation-plans/<case-id>.md` for this Test Case |
| Contract | Relevant operation from `fake-api/openapi.yaml` |

Use the complete Test Case text supplied by the task. Do not assume that a case
exists under `fixtures/`.

## Coverage preflight

Run this preflight before validating the plan or loading implementation-specific
context:

1. Search current API coverage with
   `rg -n '@DisplayName|@AllureId' api-tests/src/test/kotlin/tests`; there is no registry file.
2. Compare the Test Case behavior and expected result with current tests.
3. If current coverage already proves the complete Test Case behavior, stop.
   Cite the existing file, class and test method, retain the coverage
   comparison, and report that no files changed.
4. If the assigned `@AllureId` is occupied but the existing test does not prove
   equivalent behavior, stop. Report the conflicting target and the unmet Test
   Case behavior. Do not generate a duplicate, remap the ID or claim coverage.

Neither stop requires an automation plan. Only equivalent existing behavior is
coverage. When the preflight confirms a coverage gap and the assigned ID is
available, create or validate the plan before editing test code.

## Plan creation and validation

Use `agent_docs/templates/automation_plan.api.workflow.md.template` as the required schema. Do not add
sections or leave placeholders. Replace `<case-id>` in the plan path with the
supplied case ID, for example `API-2007` or `MOB-1007`.

1. If `agent_docs/automation-plans/<case-id>.md` is missing, create it for the supplied Test Case.
2. If it already describes the same Test Case, validate every claim against
   the relevant OpenAPI operation and current repository sources. Correct a
   stale path, symbol, value or command only when current evidence supports one
   unambiguous replacement.
3. Use a fresh response when the plan depends on runtime behavior that OpenAPI
   does not specify. Stop on contract drift instead of rewriting the Test Case,
   contract or backend.
4. Mark the plan validated only after its Test Case mapping, contract and
   repository citations, allowed files, stop conditions and verification
   commands are complete and internally consistent.
5. If an existing plan describes another Test Case, do not overwrite it unless
   the current request explicitly authorizes replacement.
6. If evidence is missing or contradictory, stop before test-code changes and
   report the exact plan item that cannot be validated.

For a planning-only request, stop after writing and validating
`agent_docs/automation-plans/<case-id>.md`. For a generation request, continue with the validated
plan.

## Required context

`AGENTS.md` is already loaded; use its repository rules and source-routing
table without reading the file again. Before planning or editing, read:

1. `agent_docs/AI_POLICY.md`;
2. `api-tests/README.md`;
3. `agent_docs/building_the_project.md`;
4. the relevant operation in `fake-api/openapi.yaml`;
5. the nearest API test, endpoint client, models and test data.

Use these sources for layer ownership, black-box boundaries, test metadata,
formatting and execution commands. Keep those rules in their source documents;
do not restate them in this skill.

## Pre-generation checks

1. Verify every `agent_docs/automation-plans/<case-id>.md` claim against the contract and current
   repository files.
2. Confirm that every planned file is permitted by the validated plan and
   repository policy.
3. Apply the assertion rules below.

Use the `@AllureId` assigned by the Test Case and verify that it is not already
present. Do not invent or remap the ID during generation.

Abort generation before editing when any condition applies:

- expected behavior is ambiguous or conflicts with verified runtime behavior;
- the scenario contains more than one independent objective;
- an exact value has no support in the scenario or contract;
- the oracle can pass on pre-existing state;

## Assertion rules

| Data or behavior | Required assertion |
|---|---|
| Contract field | Assert the field and the value or invariant required by the scenario |
| Example or test-data value | Assert exact identity only when the scenario or contract requires it |
| Generated resource | Capture its ID or response identity and correlate later lifecycle or collection reads |
| Error outcome | Assert the documented status and meaningful error body |

Reject status-only and list-size-only checks when the scenario names response
values. Reject full-object equality when unrelated response fields are outside
the expected result. Keep expected values independent from the system under
test.

## Verification and result

1. Start `fake-api` with the setup documented in `api-tests/README.md` and
   `agent_docs/building_the_project.md`.
2. Run only the generated test's exact `package.Class.method` with the API test
   task, `--tests package.Class.method` and `--rerun` (or `--rerun-tasks`). Do
   not run the whole API suite as verification for this one-case workflow.
3. Require fresh JUnit XML containing exactly one matching, passing,
   non-skipped test. A cached task, zero matching tests, or another passing
   target does not verify the generated test.
4. Inspect the matching Allure scenario step and attached HTTP request and
   response.

On success, report the Test Case ID, exact target, changed files, assertions,
command and observed result for the generated test. On failure, report the
smallest relevant evidence and stop. Test repair and backend modification
require separate authorization.
