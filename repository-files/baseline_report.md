# Suite baseline report

Date: 2026-09-15. Run timestamps below use UTC+02:00.

This completed example records the original starter suites at the revision
below. The API suite passed; the final mobile run had one execution failure.
It does not establish the state of a student's checkout after later modules.

## Environment

| Item | Value |
|---|---|
| OS and version | Windows 11 Home 25H2, build 26200.9457; x86_64 |
| Emulator: AVD name and API level | `Pixel_6`, API 36, Google APIs x86_64 image; `emulator-5554` |
| Appium server version | 2.16.2; UiAutomator2 driver 3.9.8 |
| JDK and Node.js | Oracle JDK 17.0.12; Node.js 22.13.0 |
| Allure CLI | 2.43.0 |

See [environment_notes.md](environment_notes.md) for the setup and the
disposable emulator's configuration. The backend was available at
`http://127.0.0.1:8080` and Appium at `http://127.0.0.1:4723`.

## Source state

| Item | Value |
|---|---|
| Repository revision | [AI-for-Kotlin-practice, `cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`](https://github.com/nebius-academy-templates/AI-for-Kotlin-practice/tree/cadca2fb442682e0eda7df9a7199cf5ddfcf1beb) |
| Working tree | Clean; no tracked product, test, configuration or script changes |
| Suite scope | Complete starter suites: six Appium tests and four API tests |

Build outputs and reports were generated during these runs. The only change
between the two mobile executions was the emulator adjustment described
under Known issues and flaky observations.

## App build variant

`stableDebug`, selected with `-Flavor stable` in both mobile executions.

## Command used

Run from the practice repository root in PowerShell. The backend and Appium
were started in separate processes using the supplied commands:

```powershell
.\gradlew.bat :fake-api:run --console=plain
.\scripts\start-appium.ps1
```

The complete Appium suite command was identical for both executions:

```powershell
.\scripts\run-suite.ps1 -Flavor stable -Device emulator-5554
```

The complete API suite command was:

```powershell
.\gradlew.bat :api-tests:test --rerun --console=plain
```

## Results: Appium suite

Final execution, 18:36:22–18:37:22, exit code **1**:

| Total | Passed | Failed | Skipped |
|---|---|---|---|
| 6 | 5 | 1 | 0 |

Duration: **54.549 seconds**, from the completed Appium Allure report.
The outer command took 59.760 seconds, including work outside test execution.

JUnit and the OS suite runner report one failure. Allure categorizes that
same execution exception as **broken**: 5 passed, 1 broken, 0 failed, 0 skipped.
These are different reporting categories for the same six tests.

The initial execution, 18:32:55–18:35:09, exited with code **1** and recorded
0 passed, 6 failed and 0 skipped. Allure categorized all six as broken and
reported a duration of **109.578 seconds**. This earlier result is preserved
separately; it is not combined with the final run.

## Results: API suite

Execution, 18:30:28–18:30:46, exit code **0**:

| Total | Passed | Failed | Skipped |
|---|---|---|---|
| 4 | 4 | 0 | 0 |

Duration: **7.647 seconds**, from the completed API Allure report.
The outer command took 18.184 seconds, including compilation and Gradle work.
Allure and JUnit agree that all four tests passed.

## Allure sources

Generated in the practice checkout:

- Appium report: `appium-tests/build/reports/allure-report/index.html`;
  input: `appium-tests/build/allure-results/`.
- API report: `api-tests/build/reports/allure-report/index.html`;
  input: `api-tests/build/allure-results/`.

The distribution includes the small
[baseline evidence package](https://github.com/nebius-academy-templates/qa-template-week-4-workflow/tree/main/evidence/baseline):
run metadata, normalized JUnit results and the generated Allure `summary.json`
for each of the three executions. Its README explains the normalization.
The full interactive reports and attachments are not included in the package.

## Cross-suite overview

- UI coverage visible in the report: full onboarding, wrong OTP, ride option
  prices, both passkey promo branches, and the ride search/order history
  scenario. The first five tests passed in the final run. The order history
  scenario failed during destination search, so that run did not verify its
  later history checks.
- API coverage visible in the report: phone/OTP authentication and access to
  data endpoints; a completed ride becoming the newest order; a cancelled
  ride staying out of history; and `driver_not_found` rejecting an order
  without changing history. All four scenarios passed.
- Failures, skips, or slow-test observations: no skips. The final mobile
  failure was `tests.RideAndHistoryE2ETest`, **Search a ride then check order
  history**. Appium reported that `POST /element` could not be forwarded
  because the UiAutomator2 instrumentation process was not running while
  looking for `rides_list`. Screenshot and page-source collection also
  failed after the session stopped responding. A logcat attachment was saved.
  The longest final-run test took 12.349 seconds in Allure; no performance
  threshold was defined for this baseline.

## Known issues and flaky observations

The first mobile run encountered Android's **Bluetooth keeps stopping**
dialog. Five tests timed out waiting for `phone_title`; the ride/history test
timed out waiting for `map_from_field`. The emulator's focused window was
confirmed as an application error for `com.google.android.bluetooth`.

Bluetooth was disabled only in the disposable emulator, that process was
stopped, and Android Back dismissed the dialog. Before the one full rerun,
`bluetooth_on` was `0` and the focused window was the launcher. No product,
locator, assertion, timeout or test code was changed. Five tests then passed;
one encountered the UiAutomator2 failure described above.

The remaining failure establishes an unavailable automation session at that
step. These results do not identify why the instrumentation process stopped,
or establish a product defect or a reproducible test flake.

## Next action

Inspect the Appium server log and the saved logcat around the final run's
instrumentation failure at 18:37:19 to determine why the session stopped.
After resolving that cause, rerun the unchanged complete mobile suite and
record a new baseline before treating all six mobile scenarios as verified.
