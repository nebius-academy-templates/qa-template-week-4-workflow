# Test architecture

Course reference example, checked against practice revision
`cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`. This is a source-based map of the
suite, not a record of a successful run. Links resolve from `agent_docs/` in
the practice repository.

## Layers

The mobile suite separates element declarations from interactions and scenario
composition. [AGENTS.md](../AGENTS.md#appium-test-architecture) and
[appium-tests/README.md](../appium-tests/README.md#architecture) define the
boundaries; the classes below show their implementation.

| Layer | Responsibility and example | Boundary |
|---|---|---|
| `rule/` | [AppiumTestCase](../appium-tests/src/test/kotlin/rule/AppiumTestCase.kt) starts and closes sessions, resets sandbox state and exposes action objects. [DriverFactory](../appium-tests/src/test/kotlin/rule/DriverFactory.kt) configures the driver and device ownership. JUnit extensions collect failure evidence. | Keep scenario interactions in actions; element waits belong to `Element`. |
| `pages/` | Singleton catalogs such as [MapPage](../appium-tests/src/test/kotlin/pages/MapPage.kt) hold stateless `Element` values and factories. [Element](../appium-tests/src/test/kotlin/pages/Element.kt) resolves and operates on the current driver's elements; [Device](../appium-tests/src/test/kotlin/pages/Device.kt) provides device-level UI operations. | Page catalogs contain no assertions or flows. Construct `Element` only in this layer. |
| `actions/` | [MapActions](../appium-tests/src/test/kotlin/actions/MapActions.kt) composes typing, keyboard dismissal, waits and value assertions. | Reuse page catalogs instead of constructing locators or `Element` instances. |
| `tests/` | [RideAndHistoryE2ETest](../appium-tests/src/test/kotlin/tests/RideAndHistoryE2ETest.kt) composes actions in named Allure `step` blocks and passes expected test data. | Keep interaction implementation in actions and locator declarations in pages. |
| `testdata/` | [TestData](../appium-tests/src/test/kotlin/testdata/TestData.kt) supplies shared scenario values and expected results. | Use the data from scenarios and actions; it does not own driver operations. |

The `checkPageObjectBoundary` task in
[appium-tests/build.gradle.kts](../appium-tests/build.gradle.kts) rejects
`Element(...)` construction outside `pages/` when the module's `check` task
runs. This checks that specific boundary; it is not a complete architecture
review.

`AppiumTestCase.startAuthorized` defaults to `true`; onboarding tests override
it when authentication is the scenario under test. `startSession` creates the
driver, then resets state; `endSession` attempts another reset, closes the
driver and releases the device. [DriverFactory](../appium-tests/src/test/kotlin/rule/DriverFactory.kt)
forces APK installation on the first session per device in a JVM run and sets
`appium:disableIdLocatorAutocompletion=true`. Preserve these supplied safeguards
as required by [AGENTS.md](../AGENTS.md#safety-boundaries).

The API suite has a separate arrangement:
[ApiTestCase](../api-tests/src/test/kotlin/rule/ApiTestCase.kt) starts an isolated
sandbox session, resets it and offers token and Allure-step helpers;
`client/` builds REST Assured requests without assertions; `model/` contains
test-owned wire DTOs; `tests/` holds assertions; `testdata/` contains shared
values. [ApiSpec](../api-tests/src/test/kotlin/client/ApiSpec.kt) adds the
`X-Sandbox-Session` header and request/response Allure filter. Tests treat the
API as a black box and do not import backend domain classes. These boundaries
are documented in [api-tests/README.md](../api-tests/README.md#architecture).

## Where things live

| Content | Repository directory |
|---|---|
| Mobile lifecycle and evidence | [appium-tests/src/test/kotlin/rule](../appium-tests/src/test/kotlin/rule/) |
| Mobile elements and device operations | [appium-tests/src/test/kotlin/pages](../appium-tests/src/test/kotlin/pages/) |
| Mobile actions | [appium-tests/src/test/kotlin/actions](../appium-tests/src/test/kotlin/actions/) |
| Mobile scenarios | [appium-tests/src/test/kotlin/tests](../appium-tests/src/test/kotlin/tests/) |
| Mobile data | [appium-tests/src/test/kotlin/testdata](../appium-tests/src/test/kotlin/testdata/) |
| API layers | [api-tests/src/test/kotlin](../api-tests/src/test/kotlin/) |
| API contract | [fake-api/openapi.yaml](../fake-api/openapi.yaml) |

Read the current `@Test`, `@DisplayName` and `@AllureId` declarations in the two
test directories for the coverage inventory. Do not use this document as a
fixed class list. New tests need unused Allure IDs and must not duplicate
coverage, as required by [AGENTS.md](../AGENTS.md#starter-tests).

## Waits and retries

[Element.kt](../appium-tests/src/test/kotlin/pages/Element.kt) implements the
synchronization operations:

| Operation | Condition and intended use |
|---|---|
| `waitFor(timeoutSec)` | Waits for `visibilityOfElementLocated` and returns the located `WebElement`; use it for a visible screen element. |
| `click`, `sendKeys`, `clear`, `text` | Resolve an element through the same visibility wait before acting or reading. `text` uses the element's default timeout. |
| `waitForGone(timeoutSec)` | Waits for `invisibilityOfElementLocated`; absence or invisibility satisfies it. It does not wait for visibility first. |
| `isPresent()` | Calls `findElements(...).isNotEmpty()` directly, with no explicit wait or visibility check. It is an immediate presence probe, not proof of a completed transition. |
| `retryClick(timeoutSec, message)` | Tries a fresh `findElement(...).click()` every 300 ms until success or timeout. It retries only `NoSuchElementException`, `StaleElementReferenceException` and `ElementClickInterceptedException`. It does not use the ordinary visibility-wait helper. |

The default `Element` timeout is 10 seconds; individual declarations can
specify another value. [MapActions.awaitReady](../appium-tests/src/test/kotlin/actions/MapActions.kt)
waits for the pickup field, destination field and pull-to-refresh surface.
[DrawerActions.openOrders](../appium-tests/src/test/kotlin/actions/DrawerActions.kt)
uses a narrow `retryClick` for a drawer transition. Neither a successful click
nor a retry replaces an assertion of the scenario's result.

[AGENTS.md](../AGENTS.md#appium-test-architecture) forbids `Thread.sleep` and
text XPath locators. Its safety boundary requires evidence before adding a
retry or increasing a timeout: the expectation must be correct and
synchronization must be the defect. Do not use retries to suppress assertion
failures or a dead session. `retryClick` does not catch those failures.

## Failure artifacts

[AppiumTestCase](../appium-tests/src/test/kotlin/rule/AppiumTestCase.kt) registers
`FailureDigest` and `ArtifactsOnFailure`. The exception handler captures
evidence before teardown closes the session and then rethrows the original
failure. Capture is best effort: an unavailable session or an individual
capture failure can leave a missing artifact.

| Evidence | Producer and location |
|---|---|
| Screenshot, device logcat and UI XML | [ArtifactsOnFailure](../appium-tests/src/test/kotlin/rule/ArtifactsOnFailure.kt) writes `appium-tests/build/reports/failures/<sanitized-display-name>-<timestamp>.png`, `.logcat` and `.xml`, and attempts to attach the same bytes to Allure. |
| Compact failure digest | [FailureDigest](../appium-tests/src/test/kotlin/rule/FailureDigest.kt) writes `appium-tests/build/reports/digests/test_log_<method>_<try>.txt`. It includes the exception, project stack frames, raw-artifact paths, resource IDs extracted from XML and a focused logcat slice. |
| Screen after a successful Allure step | `AppiumTestCase.step` attaches a screenshot after the step body completes. These attachments live with the raw Allure results; failure screenshots belong to the exception handler. |
| JUnit, console and reports | [Building the project](building_the_project.md#verify-it-worked) lists current run and report paths and the checks needed to establish execution. |

Read the digest first, then use the linked raw artifacts when it does not
resolve the cause. The digest's `try` suffix preserves successive files; it
is not a repair-attempt budget. Its resource-ID list is extracted from the
captured XML, so an ID's inclusion is not itself a visibility assertion.
Failure directories can contain earlier runs: correlate the method, timestamp
and run output before using an artifact as current evidence.
