# QA workflow skills

Install this package into your Kotlin project repository. It
provides the complete 18-case workbook and six skills for assessing a case,
planning and generating its test, executing it, repairing a supported test
defect, and checking the result against the original case.

## Install

1. Download this repository using **Code → Download ZIP** and open
   `repository-files/`.
2. Copy the contents of `repository-files/` into your project repository,
   preserving the directory structure, including hidden directories. Compare
   personal edits before replacing
   the six named skills, `scripts/sync_agent_skills.py`, `test-cases/test-cases.xlsx` and
   `.agents/hooks/test_repair.py` with this package's versions.
3. Use the instructions in `.agents/skills/` with your coding agent. Claude Code
   and Codex are not required to use the skill files.

The repair runtime requires Python 3.11 or newer. If you use Claude Code or
Codex, preserve the corresponding existing hook configuration:
`.claude/settings.json` for Claude Code or `.codex/hooks.json` for Codex. It must
invoke `.agents/hooks/test_repair.py` with the appropriate adapter. Hook setup
depends on the coding agent; copying a `SKILL.md` alone does not install a hook.

### Read the Excel workbook

Install `openpyxl` in the Python environment used by your coding agent. Activate
your virtual environment first if you use one, then install and verify:

```shell
python -m pip install openpyxl
python -c "import openpyxl; print(openpyxl.__version__)"
```

Use `python3` instead of `python` if that is your Python command. Run the agent's
workbook-reading commands with the same interpreter.

### Optional: Claude Code copies

The package already includes `.claude/skills/` copies synchronized with the
canonical files in `.agents/skills/`. No synchronization command is needed
after copying the package unchanged.

If you later edit a canonical skill, regenerate its Claude copy from the
**project repository root**:

```shell
python scripts/sync_agent_skills.py
```

To check whether the copies match without changing them:

```shell
python scripts/sync_agent_skills.py --check
```

Use `python3` on macOS/Linux. Skip this section if you do not use Claude Code.
The script synchronizes all six `SKILL.md` files. The optional
`agents/openai.yaml` files remain with the canonical skills.

## Included skills

| Skill | Responsibility |
|---|---|
| [automate-test-case](repository-files/.agents/skills/automate-test-case/SKILL.md) | Reuse a saved readiness status or assess the case when no status exists, then coordinate test generation, execution, applicable repair and a final case-conformance check. |
| [gen-api-test](repository-files/.agents/skills/gen-api-test/SKILL.md) | Check coverage, create or validate the case's plan, generate one API test and verify only its exact target with a fresh test run. |
| [gen-mobile-test](repository-files/.agents/skills/gen-mobile-test/SKILL.md) | Check coverage, create or validate the case's plan, generate one Appium test and verify it with a fresh test run. |
| [run-appium-suite](repository-files/.agents/skills/run-appium-suite/SKILL.md) | Use the existing OS-specific runner and inspect actual execution results. |
| [verify-sandbox-state](repository-files/.agents/skills/verify-sandbox-state/SKILL.md) | Enable and verify a requested sandbox state when the case requires it. |
| [test-repair](repository-files/.agents/skills/test-repair/SKILL.md) | Diagnose one exact failing test and make an evidence-backed test-layer correction within the existing repair budgets. |

## Included test cases

[test-cases.xlsx](repository-files/test-cases/test-cases.xlsx) contains all 18
cases from the [Module 3 workbook](https://github.com/nebius-academy-templates/qa-template-week-3-agents/blob/main/test-cases.xlsx):
10 mobile and 8 API cases. Match `Case Summary` and `Steps` by `Case ID` to read
the complete preconditions, actions and expected results. `Endpoints` is a
column in `Case Summary`.

The `Automated Test` column contains 13 expected test-source references after
Modules 1-3. The other five cases have a status of `READY FOR AUTOMATION`,
`BLOCKED` or `NEEDS_CLARIFICATION`, based on the Lesson 3.5 assessment.
`READY FOR AUTOMATION` is the workbook label for the assessment result `READY`.
Verify test references in your own checkout, where generated class or method
names may differ. Neither a source reference nor a readiness status proves a
passing run. The coordinator reuses one of these three readiness statuses and
does not reassess it unless the current request explicitly asks for a new
assessment. `TODO`, a blank cell and a test-source reference are not readiness
statuses; they do not skip assessment by themselves. An application may expose
an explicit prepared-batch mode for course-curated cases. In that mode, it may
replace the model assessment with deterministic workbook validation only after
checking the selected case rows and required non-empty fields, and it must
record that the decision came from the prepared preflight rather than a
readiness assessment. Preserve a completed assessment report with its evidence
so a reused status retains its original context.

## Project documents used by the workflow

Verify these files in the project repository before starting. Paths below are
relative to that repository's root; they are dependencies, not files to read
from this package checkout.

| Operation | Documents and inputs |
|---|---|
| Repository rules | `AGENTS.md`, `agent_docs/AI_POLICY.md` |
| Can this test case be automated? | `agent_docs/task-automation-readiness-instructions.md`, the complete supplied case and relevant product/test sources |
| API planning and generation | `agent_docs/templates/automation_plan.api.workflow.md.template`, `agent_docs/building_the_project.md`, `api-tests/README.md`, `fake-api/openapi.yaml` and the relevant test-layer sources |
| Mobile planning and generation | `agent_docs/templates/automation_plan.mobile.workflow.md.template`, `agent_docs/building_the_project.md`, `agent_docs/test_architecture.md`, `agent_docs/page_object_model.md`, `appium-tests/README.md`, `agent_docs/baseline_report.md` when present, and the relevant test-layer sources |
| Execution and repair | Existing runners, `.agents/hooks/test_repair.py`, installed hook configuration and matching exact-target JUnit/Allure evidence |
| Final case check | One host-prepared packet containing the original case, line-numbered final test and required helpers, the plan when present, and matching exact-target JUnit/Allure/HTTP evidence |

`agent_docs/environment_notes.md` and the earlier
`agent_docs/task-automation-readiness.md` remain useful context. They do not
replace current environment checks or the selected case's current assessment.
`agent_docs/review_template.md` belongs to the supplied GitHub PR reviewer;
the coordinator's final case check follows the original test case.
The coordinator validates and assembles this packet before invoking a separate
reviewer. The reviewer receives no general repository discovery tools and does
not repeat list, search or read operations already completed by the workflow.

## Optional direct use

Installing the package does not require running a case. Use the coordinating
skill when a lesson or task asks you to automate a named case. Supply the full
case from the packaged `test-cases/test-cases.xlsx`, including its
preconditions, actions and expected results:

```text
Automate <case-id> from test-cases/test-cases.xlsx using .agents/skills/automate-test-case/SKILL.md.
```

When no saved readiness status or explicit prepared preflight exists, or when
the request explicitly asks for reassessment, the coordinator writes the case
automation assessment to
`.agent-state/automate-test-case/<case-id>/readiness.md`. Otherwise it records
the reused status or prepared preflight and its source in the workflow record
without rerunning the assessment. When new coverage is needed, the selected
generator writes `agent_docs/automation-plans/<case-id>.md` before changing test
code. The workflow record goes to `agent_docs/qa-workflow.md`, or a case-specific
filename when an existing record belongs to another case.

The result records what ran, what evidence was reused, which case requirements
were checked and what remains unresolved. Missing prerequisites, ambiguous
requirements or a product defect can stop implementation. Completed source
files alone do not prove a passing run.
