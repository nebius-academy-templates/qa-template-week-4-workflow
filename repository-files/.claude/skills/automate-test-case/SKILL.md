---
name: automate-test-case
description: Coordinate readiness, generation and execution for one supplied API or mobile test case, then check the resulting test against the case. Retain its requirements and evidence across operations. Use for an end-to-end case workflow; use the individual instructions for assessment-only, generation-only or repair-only requests.
---

# Automate one test case

## Inputs and scope

Use one complete supplied case, the current repository, and any earlier workflow
record or execution evidence the user provides. Preserve the case ID, layer,
preconditions and every expected result. Resolve multiple cases or missing case
details before implementation; do not create a backlog or invent a new case.

Follow repository rules, `agent_docs/AI_POLICY.md` and the current execution and repair
permissions. Each operation's instructions remain authoritative for its scope.
This skill does not authorize a push, external write, new framework or product change.
Use the case and its contract for expected behavior, not observed product output.

Update the requested workflow output for this invocation as operations finish,
using the fields in `Workflow result` below. Default to `agent_docs/qa-workflow.md`; if it belongs to another
case, preserve it and use `agent_docs/qa-workflow.<case-id>.md`. Preserve an earlier record's
original case and useful sections; it is context, not execution proof or a
replacement for the repair queue or a persisted execution engine.
If an instruction or required source is missing, report the dependency rather
than silently substituting another tool.

## 1. Assess the case from sources

Follow `agent_docs/task-automation-readiness-instructions.md`, or the supplied
alternative, including its source-only scope and status definitions. Recheck
the decisive sources of any supplied earlier assessment. Write the selected-case
report to the requested readiness path or
`.agent-state/automate-test-case/<case-id>/readiness.md`, separately from the
workflow output; preserve any broader report. Only after assessment finishes,
record its outcome and continue:

| Assessment result | Next action |
|---|---|
| `READY` | Continue to the case's generation skill. |
| `BLOCKED` | Record the confirmed missing capability and evidence; stop implementation. |
| `NEEDS_CLARIFICATION` | Record the question that changes the decision; stop implementation. |

## 2. Delegate generation; interpret its result

Follow `.agents/skills/gen-api-test/SKILL.md` for API or
`.agents/skills/gen-mobile-test/SKILL.md` for mobile, passing the original case
and selected readiness report. Use the current session or an optional subagent
when supported.

The selected skill owns coverage preflight, plan creation/validation, generation
and execution. Load its requested context; do not add another planning stage,
immediately repeat a successful run, or load unrelated layer documentation.

| Generation outcome | Coordinator action |
|---|---|
| Equivalent existing coverage | Retain the target and coverage comparison; establish execution evidence in step 3. Apply step 4's conditions for another case check. No duplicate test or new plan. |
| Occupied ID without equivalent behavior | Record the conflicting target and unmet expected result. Stop; do not rename the case or claim coverage. |
| New test with a valid passing result | Continue to step 4 with the plan, changes and matching evidence. |
| Unsupported or contradictory plan | Preserve the specific conflict and stop implementation. |
| Failed or unverified execution | Preserve the actual failing target or missing evidence; use the failure route below. |

## 3. Establish the execution result

Inspect JUnit, relevant Allure steps/attachments and the command/log for the
intended non-skipped test with its original assertions and the suite outcome.
Coverage, compilation or a shell exit alone cannot establish passing execution;
missing XML leaves it unverified.

Match earlier evidence to the current target, revision and relevant local changes,
including untracked test sources; a name or timestamp alone is insufficient.
Reuse matching evidence unless a new run is requested, recording the original
run time and explicitly stating when no test ran in this invocation.

When evidence is absent or mismatched and execution is authorized, run the
existing target without re-entering the generator. For API, follow
`api-tests/README.md` and `agent_docs/building_the_project.md` for backend setup
and a fresh full suite; for mobile, use `run-appium-suite` and its supported
filter. Retain the revision and relevant local source changes before the run
to match the resulting evidence.

If execution is outside scope or unavailable, record `NOT_VERIFIED` and the
missing proof. Source conformance may still be inspected; it does not establish
execution or readiness for integration.

### Failure route

Distinguish failures of the target, other suite tests and execution/setup;
retain both the target result and unresolved suite result.

For authorized repair of that failure, follow `.agents/skills/test-repair/SKILL.md`
with the exact target and decisive evidence, retaining its classification and
outcome. Use the installed queue, locks and both budgets without clearing or
restarting them or automatically dispatching unrelated work. An incompatible lock blocks
execution until resolved through that procedure; never run a broad suite
through a repair lock.

After a verified correction, continue to step 4 with matching new evidence;
earlier green evidence no longer verifies changed code.

## 4. Check the test against the test case

Check every test created or changed by this workflow, including changes to
helpers that affect the scenario. For an unchanged existing test, reuse the
current coverage comparison. Repeat the check only when the user requests it,
the case, relevant contract or test behavior has changed since that comparison,
or the comparison leaves full coverage unclear.

When a check is needed, read the original case, final test and relevant helpers,
the plan when one exists, and matching execution evidence. Verify that the test
prepares the required preconditions, performs the case's actions in the required order and
asserts every expected result, including any resulting state. Check the test's
behavior beyond changed lines. Keep this check read-only.

If corrections are authorized, stay within the original allowed test scope,
revalidate the affected plan, verify outside this check, and check the corrected
version. Otherwise retain the gaps as remaining work.
Do not add unrequested repairs, additional tests or product changes.

## Workflow result

Include in the workflow record and final summary:

- case source, working revision and relevant local changes;
- selected route, operations, instructions and observed outcomes;
- target `Class.method`, plan and changed paths when applicable;
- target/suite results, evidence paths and source versions, reused versus new evidence;
- case check performed or coverage comparison reused; supported gaps with the case
  requirement, test location and consequence;
- remaining work and next action, with supporting evidence.

Product bugs, exhausted budgets and unresolved investigations remain unresolved.
A supported stop is a workflow result, not a completed passing test. A `skipped`
queue outcome or passing test with missing case assertions does not establish
successful test completion. Keep unresolved suite failures, missing proof and
stale evidence visible.
