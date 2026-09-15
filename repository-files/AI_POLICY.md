# AI Policy

Version: 0.1, course reference example. Owner: Course reference example.
Scope: this repository, including the Appium and API test suites. Applies to
any AI agent and to any human applying an AI-generated diff.

Reference revision: `cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`. This example fills
the five sections in [AI_POLICY.md.template](AI_POLICY.md.template) using the
existing rules in [AGENTS.md](AGENTS.md) and
[scripts/protected-paths.txt](scripts/protected-paths.txt). Recheck the cited
sources when using it with another revision. No successful run is claimed here.

Ground rule: agent output is unverified until source code, a fresh run or a log
supports the claim. Every rule below identifies evidence that can expose a
violation.

## Allowed changes

| Allowed change | Condition: proof required | How a violation is detected |
|---|---|---|
| New tests, page objects and actions under `appium-tests/src`, excluding protected files | Follow the supplied page/action/test boundaries and attach fresh output from `scripts/run-suite.ps1` or `scripts/run-suite.sh` for the intended variant and scope. The executed count must be non-zero, with no failed or errored required tests. | Review the diff, `suite-run.log` and matching JUnit results; a skipped required case, cached task, missing report or zero-test run cannot establish that coverage passed. Sources: [architecture and evidence](appium-tests/README.md), [page boundary check](appium-tests/build.gradle.kts). |
| Clients, models, data and tests under `api-tests/src` | Follow the API layers, keep the backend as a black box, and provide a fresh `:api-tests:test --rerun` result against the running backend. Required tests must execute and pass. | Review imports, assertions, JUnit results and matching request/response attachments. Connection failures, skipped required cases and zero tests leave the result unverified. Sources: [API README](api-tests/README.md), [ApiSpec](api-tests/src/test/kotlin/client/ApiSpec.kt). |
| Repository documentation and explanations of test code | Keep prose in English and check paths, commands and behavior claims against their cited sources. Distinguish a source-based explanation from execution evidence. | Compare statements and links with the current source, diff and run artifacts. Source: [AGENTS.md](AGENTS.md) and the drafting instructions in the original `agent_docs/` scaffolds at the reference revision. |

New coverage must use an unused Allure ID and must not duplicate an existing
scenario; inspect current test sources, as required by
[AGENTS.md](AGENTS.md#starter-tests). Run the required `ktlintCheck` before
committing Kotlin changes. Test success does not replace diff review.

## Forbidden changes

| Forbidden change | Why it is forbidden | How a violation is detected |
|---|---|---|
| Editing `app/` or `fake-api/` to make a test pass | A repair would change the observed product and could mask a regression. | Diff review exposes product changes in a test-repair task. Source: [AGENTS.md safety boundaries](AGENTS.md#safety-boundaries). |
| Editing `.github/workflows/` to remove or weaken a failing check | These files are supplied evaluation infrastructure; changing the evaluator does not repair the tested behavior. | Compare changed paths and workflow logic with the protected-path list and the trusted revision. Source: [scripts/protected-paths.txt](scripts/protected-paths.txt). |
| Unapproved changes to the pinned toolchain, `DriverFactory.kt`, `VariantLocator.kt`, fixtures or other protected paths | The supplied environment, APK-selection safeguards and exercise evidence must remain reliable. Locator fallback attempts can conceal a wrong UI variant. | Review the diff against [scripts/protected-paths.txt](scripts/protected-paths.txt) and the explicit scope of any approval. [AGENTS.md](AGENTS.md#safety-boundaries) requires preservation of these safeguards. |
| Weakening assertions or adding retries/timeouts without evidence that the expectation is correct and synchronization is the defect | Such changes can hide a product failure instead of correcting test automation. `Thread.sleep` and text XPath are prohibited. | Inspect assertions, waits and locators in the diff; compare the proposed cause with captured UI/API evidence. Sources: [AGENTS.md](AGENTS.md#safety-boundaries), [Element.kt](appium-tests/src/test/kotlin/pages/Element.kt). |
| Direct edits to the generated `.claude/skills` mirror | The canonical source is `.agents/skills`; editing only a copy creates drift. | Inspect changed skill paths and the synchronization check described in [README.md](README.md). Source: [AGENTS.md](AGENTS.md#safety-boundaries). |

## Human gates

- Before merging a PR, a human reviews its complete diff and attached local run
  output. This gate is supplied by [AI_POLICY.md.template](AI_POLICY.md.template).
- Before any `git push`, obtain explicit approval for that specific push in
  the current session. Check the requested action against the recorded
  approval. Source: [AGENTS.md](AGENTS.md#safety-boundaries).
- Before modifying a protected path, obtain explicit approval covering the
  named change. Product work requires an explicit human request; test repair
  remains limited to test layers. Compare the proposed diff with the approved
  scope and [scripts/protected-paths.txt](scripts/protected-paths.txt).

## Stop conditions

- If a test fix requires changing `app/` or `fake-api/`, stop that repair and
  report the suspected product bug with evidence. This is the existing boundary
  in [AGENTS.md](AGENTS.md#safety-boundaries) and
  [AI_POLICY.md.template](AI_POLICY.md.template).
- If the proposed change would weaken an expectation, add a prohibited pause
  or locator, or change synchronization without supporting evidence, do not
  apply it. Report the failing assertion and the evidence still needed to
  explain it. Source: [AGENTS.md](AGENTS.md#safety-boundaries).
- If the relevant runner cannot execute required tests, or results are missing,
  stale, skipped or zero-test, do not claim completion or a verified fix. Record
  the failed prerequisite or missing proof. Sources:
  [AI_POLICY.md.template](AI_POLICY.md.template) and
  [building_the_project.md](agent_docs/building_the_project.md#verify-it-worked).
- If the installed or selected UI variant cannot be established, do not call
  its result verified. Compare the run's APK and `ui.variant` arguments with
  the requested variant; preserve the supplied
  [DriverFactory](appium-tests/src/test/kotlin/rule/DriverFactory.kt) and
  [VariantLocator](appium-tests/src/test/kotlin/pages/VariantLocator.kt) safeguards.

## Safe failure modes

- If the emulator, Appium or backend required by the selected suite is
  unavailable, report `blocked` with the failed prerequisite and command
  output; do not report `done`. This is the supplied failure mode in
  [AI_POLICY.md.template](AI_POLICY.md.template).
- If tests fail, retain the fresh result counts, test names and relevant
  failure artifacts. Report what failed and what remains unverified; do not
  use the existence of an HTML report as a pass result. Sources:
  [suite runners](scripts/run-suite.ps1), [bash runner](scripts/run-suite.sh)
  and [failure evidence](appium-tests/README.md#evidence).
- If an artifact is missing, state which conclusion it cannot support. Capture
  is best effort in [ArtifactsOnFailure](appium-tests/src/test/kotlin/rule/ArtifactsOnFailure.kt);
  missing screenshot or log evidence must not be replaced with an invented
  observation.
- If a sandbox-state command returns successfully, verify its visible or
  API-level effect before claiming the state applied. Enable controlled states
  after session startup, which relaunches the app. Source:
  [AGENTS.md sandbox states](AGENTS.md#sandbox-states).
