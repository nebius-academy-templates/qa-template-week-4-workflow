# Module 4 QA workflow skills

Install this package into the Kotlin practice repository after Module 3. It
provides six skills for assessing a case, planning and generating its test,
executing it, repairing a supported test defect, and checking the result against
the original case.

The [completed practice repository](https://github.com/ai-qa-lab/AI-for-Kotlin-practice)
contains the application, tests, documents, case workbooks and installed hooks.
This package uses its [document layout at `6977a01`](https://github.com/ai-qa-lab/AI-for-Kotlin-practice/tree/6977a018a9b94b5c803ce7b1173f50e65b088b42/agent_docs).

## Install

1. Download this repository using **Code → Download ZIP** and open
   `repository-files/`.
2. Copy the contents of `repository-files/` into your practice repository,
   preserving the directory structure, including hidden directories. Compare
   personal edits before replacing
   the six named skills, `scripts/sync_agent_skills.py` and
   `.agents/hooks/test_repair.py` with this package's versions.
3. Use the instructions in `.agents/skills/` with your coding agent. Claude Code
   and Codex are not required to use the skill files.

The repair runtime requires Python 3.11 or newer. If you use Claude Code or
Codex, preserve the corresponding Module 3 hook configuration:
`.claude/settings.json` for Claude Code or `.codex/hooks.json` for Codex. It must
invoke `.agents/hooks/test_repair.py` with the appropriate adapter. Hook setup
depends on the coding agent; copying a `SKILL.md` alone does not install a hook.

### Optional: Claude Code copies

The package already includes `.claude/skills/` copies synchronized with the
canonical files in `.agents/skills/`. No synchronization command is needed
after copying the package unchanged.

If you later edit a canonical skill, regenerate its Claude copy from the
**practice repository root**:

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
| [automate-test-case](repository-files/.agents/skills/automate-test-case/SKILL.md) | Route one case through readiness, generation/execution, applicable repair and a final case-conformance check. |
| [gen-api-test](repository-files/.agents/skills/gen-api-test/SKILL.md) | Check coverage, create or validate the case's plan, generate one API test and verify it with a fresh test run. |
| [gen-mobile-test](repository-files/.agents/skills/gen-mobile-test/SKILL.md) | Check coverage, create or validate the case's plan, generate one Appium test and verify it with a fresh test run. |
| [run-appium-suite](repository-files/.agents/skills/run-appium-suite/SKILL.md) | Use the existing OS-specific runner and inspect actual execution results. |
| [verify-sandbox-state](repository-files/.agents/skills/verify-sandbox-state/SKILL.md) | Enable and verify a requested sandbox state when the case requires it. |
| [test-repair](repository-files/.agents/skills/test-repair/SKILL.md) | Diagnose one exact failing test and make an evidence-backed test-layer correction within the existing repair budgets. |

## Automation plans in Modules 2 and 4

The [Module 2 package](https://github.com/nebius-academy-templates/week-2-qa-kotlin-course-template)
keeps the lesson versions: students prepare a plan and submit it to the lesson
checker before generation. This Module 4 package installs the working versions,
which create and validate the plan against the case, contract and repository
sources as part of generation. This validation does not claim LMS acceptance.

A new case gets its own plan, for example
`agent_docs/automation-plans/API-2007.md`. Earlier plans such as `API-2004.md` and
`MOB-1006.md`, and the earlier `agent_docs/automation_plan.md`, remain available.
The generator checks existing coverage before creating a plan or test.

The two Module 4 templates have separate installation paths:

- [agent_docs/templates/automation_plan.api.workflow.md.template](repository-files/agent_docs/templates/automation_plan.api.workflow.md.template)
- [agent_docs/templates/automation_plan.mobile.workflow.md.template](repository-files/agent_docs/templates/automation_plan.mobile.workflow.md.template)

The original Module 2 templates keep their names and content. Each installed
generator points to the workflow template for its layer.

## Project documents used by the workflow

Verify these files in the practice repository before starting. Paths below are
relative to that repository's root; they are dependencies, not files to read
from this package checkout.

| Operation | Documents and inputs |
|---|---|
| Repository rules | `AGENTS.md`, `agent_docs/AI_POLICY.md` |
| Readiness | `agent_docs/task-automation-readiness-instructions.md`, the complete supplied case and relevant product/test sources |
| API planning and generation | `agent_docs/templates/automation_plan.api.workflow.md.template`, `agent_docs/building_the_project.md`, `api-tests/README.md`, `fake-api/openapi.yaml` and the relevant test-layer sources |
| Mobile planning and generation | `agent_docs/templates/automation_plan.mobile.workflow.md.template`, `agent_docs/building_the_project.md`, `agent_docs/test_architecture.md`, `agent_docs/page_object_model.md`, `appium-tests/README.md`, `agent_docs/baseline_report.md` when present, and the relevant test-layer sources |
| Execution and repair | Existing runners, `.agents/hooks/test_repair.py`, installed hook configuration and matching JUnit/Allure evidence |
| Final case check | Original case, final test and affected helpers, the case's plan when present, and matching execution evidence |

`agent_docs/environment_notes.md` and the earlier
`agent_docs/task-automation-readiness.md` remain useful context. They do not
replace current environment checks or the selected case's current assessment.
`agent_docs/review_template.md` belongs to the supplied GitHub PR reviewer;
the coordinator's final case check follows the original test case.

## Run one case

Use the full case from `test-cases/test-cases.xlsx`, including its preconditions,
actions and expected results. From your practice project, ask the agent:

```text
Automate API-2007 from test-cases/test-cases.xlsx using .agents/skills/automate-test-case/SKILL.md.
```

The coordinator writes the selected readiness assessment to
`.agent-state/automate-test-case/API-2007/readiness.md`. When new coverage is
needed, the API generator writes
`agent_docs/automation-plans/API-2007.md` before changing test code. The workflow
record goes to `agent_docs/qa-workflow.md`, or a case-specific filename when an
existing record belongs to another case.

The result records what ran, what evidence was reused, which case requirements
were checked and what remains unresolved. Missing prerequisites, ambiguous
requirements or a product defect can stop implementation. Completed source
files alone do not prove a passing run.

## Source and verification scope

The five operation skills, two workflow templates and repair runtime reuse the
[course source at `0fff3358`](https://github.com/nebius-academy-templates/ai-for-qa-kotlin/tree/0fff33586426ccc0f1fa1f1fdd14fe6dcc3ebfac).
The coordinator reuses the existing Module 4 skill. Document paths are adapted
for `agent_docs/`, plans use case-specific files, and coverage commands are
explicit for the practice checkout. The Appium version error retains the
wording checked by the existing repair tests. The synchronization script includes the
coordinator alongside the five operation skills.

This package provides agent instructions and their dependencies. It does not
include a Strands graph or execute a case when installed. Validate a workflow
run through its fresh test results and recorded evidence in the practice project.
