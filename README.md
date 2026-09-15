# Module 4 workflow documents

This package brings together the documents used throughout the AI for QA
course. It contains unchanged instructions supplied in Modules 1–3, completed
examples of the documents learners wrote, and the coordinating skill for
Lesson 4.1. The files inside `repository-files/` use their installation paths
in the Kotlin practice repository.

## Use the package

1. Continue in your Kotlin practice repository after Module 3. Download this
   repository using **Code → Download ZIP**, then open `repository-files/`.
2. Copy its contents into the practice repository, retaining the directory
   structure, including the hidden `.agents/` and `.claude/` directories. Compare files you
   already edited before replacing them. Keep project-specific policy and
   source corrections that still apply to your checkout.
3. The package includes matching `.claude/skills/` copies of all six skills.
   Keep `.agents/skills/` as the canonical source. The original practice
   sync script covers only the two starter skills, so it is not an installer
   for the additional generation, repair or coordinating skills.
4. Read the example's source revision and case before using it. Verify the
   cited paths, symbols and commands against your current checkout. Reports
   describe their recorded runs; copying one does not execute your tests.

The package supplies documents, not the Android app, backend or test runners.
Keep the existing practice project and the repair runtime/hooks installed in
Module 3. Do not run the test commands from this document-only repository.

## Completed examples

These files fill the original coursework formats. Source-based examples refer
to [AI-for-Kotlin-practice at `cadca2fb`](https://github.com/nebius-academy-templates/AI-for-Kotlin-practice/tree/cadca2fb442682e0eda7df9a7199cf5ddfcf1beb).
The baseline identifies its executed source state and supporting reports.
Its recorded API run passed 4 of 4 tests; the final mobile run passed 5 of 6,
with one UiAutomator2 instrumentation failure. The report retains that failure
and the earlier emulator issue as observed results.

| File in `repository-files/` | Earlier lesson | Use in the workflow |
|---|---|---|
| [environment_notes.md](repository-files/environment_notes.md) | 1.3 | Recorded toolchain, emulator and build configuration |
| [agent_docs/building_the_project.md](repository-files/agent_docs/building_the_project.md) | 1.8 | Build, API/mobile execution and result locations |
| [agent_docs/test_architecture.md](repository-files/agent_docs/test_architecture.md) | 1.8 | Mobile test layers, synchronization and failure artifacts |
| [agent_docs/page_object_model.md](repository-files/agent_docs/page_object_model.md) | 1.8 | Pages, actions, locators and assertions |
| [baseline_report.md](repository-files/baseline_report.md) | 1.9 | An earlier suite result available for comparison |
| [AI_POLICY.md](repository-files/AI_POLICY.md) | 1.11 | Allowed changes, restrictions and evidence requirements |
| [automation_plan.md](repository-files/automation_plan.md) | 2.3 format | Filled API-2007 implementation plan |
| [task-automation-readiness.md](repository-files/task-automation-readiness.md) | 3.5 format | Source-based API-2007 readiness assessment |

The API-2007 examples use the complete case in the unchanged
[test-cases.xlsx](repository-files/test-cases.xlsx) workbook: `Case Summary`
and `Steps`. The plan describes proposed test code; it is not a
claim that the test has been generated or executed.

## Supplied files kept unchanged

The following files are copied byte for byte from the linked source revisions.
Completed `agent_docs/`, policy and report examples above are authored examples,
not unchanged copies of the original blank exercise scaffolds.

| Source revision | Files included under `repository-files/` |
|---|---|
| [Practice repository — `cadca2fb`](https://github.com/nebius-academy-templates/AI-for-Kotlin-practice/tree/cadca2fb442682e0eda7df9a7199cf5ddfcf1beb) | `AGENTS.md`, `CLAUDE.md`, `api-tests/README.md`, `appium-tests/README.md`, `AI_POLICY.md.template`, `baseline_report.md.template`, `.agents/skills/run-appium-suite/SKILL.md`, `.agents/skills/verify-sandbox-state/SKILL.md` |
| [Module 2 generation package — `22369580`](https://github.com/nebius-academy-templates/week-2-qa-kotlin-course-template/tree/223695806342b0b9e53b315ae8c120e5d27875bf) | `.agents/skills/gen-api-test/SKILL.md`, `.agents/skills/gen-mobile-test/SKILL.md`, `automation_plan.api.md.template`, `automation_plan.mobile.md.template` |
| [Module 2 review package — `d2343d39`](https://github.com/nebius-academy-templates/qa-foundation-agentic-template/tree/d2343d39a79160d32afa4a9df15ec7de5319ff27) | `review_template.md` |
| [Module 3 readiness package — `c2853522`](https://github.com/nebius-academy-templates/qa-template-week-3-agents/tree/c2853522386b627fadd2916164d25243f95ddcb9) | `test-cases.xlsx`; original root `task-automation-readiness-instructions.md` stored at `agent_docs/task-automation-readiness-instructions.md` |
| [Module 3 repair package — `daf8642c`](https://github.com/nebius-academy-templates/qa-template-week-3-python/tree/daf8642c091e29bb6b83dab04d8103e633340af8) | `.agents/skills/test-repair/SKILL.md` |

Only the location of the readiness instructions changes; their text remains
unchanged. Invoke them using their installed path under `agent_docs/`.
The five supplied canonical skills also have byte-identical copies under
`.claude/skills/`; the Module 4 coordinator has its own matching copy there.
The original policy and baseline templates are retained beside their completed
examples. Root READMEs from different source packages are linked above rather
than copied over one another.

## Workflow handoff

The [coordinating skill](repository-files/.agents/skills/automate-test-case/SKILL.md)
selects readiness, generation/execution and the final case check. An API case
uses the API generation instructions and relevant project documents. A mobile
case additionally uses the mobile architecture, page/action conventions,
mobile plan and execution procedure. Sandbox-state instructions apply when
the case requires that state. The final Strands review uses `review_template.md`.

The unchanged Module 2 generation skills require an `automation_plan.md` whose
automated validation has passed. They stop if that evidence is absent or the
plan belongs to another case. The filled example demonstrates the plan's
content; this package does not claim that the lesson checker accepted it.
Supply the actual validation result before asking those skills to generate.
The Module 4 coordinating skill in this package respects that handoff and
reports missing plan validation without changing the original generation skills.

Preserve the broader readiness report when producing a new selected-case
assessment. The coordinating skill creates its current assessment and workflow
record during the invocation; no pre-filled `qa-workflow.md` is supplied.
