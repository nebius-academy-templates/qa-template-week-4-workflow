# Building the project

Course reference example, checked against practice revision
`cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`. Commands below are verified from
repository sources; no execution results are claimed. Run commands from the
practice repository root. Links resolve after this document is copied into
that repository's `agent_docs/` directory.

## Prerequisites

| Requirement | Repository evidence |
|---|---|
| JDK 17; use the checked-in Gradle wrapper | [AGENTS.md](../AGENTS.md), [app build configuration](../app/build.gradle.kts), and both test modules' `jvmToolchain(17)` declarations. The [wrapper properties](../gradle/wrapper/gradle-wrapper.properties) select Gradle 9.5.1. |
| Node.js 20 or newer and npm | [README prerequisites](../README.md#prerequisites) and [bootstrap.ps1](../scripts/bootstrap.ps1)/[bootstrap.sh](../scripts/bootstrap.sh) check the Node major version. |
| Android SDK platform 36, build-tools 36.0.0, platform-tools, Emulator and Command-line Tools | [setup-emulator.ps1](../scripts/setup-emulator.ps1) and [setup-emulator.sh](../scripts/setup-emulator.sh) resolve SDK tools and install missing packages. [app/build.gradle.kts](../app/build.gradle.kts) sets `compileSdk` and `targetSdk` to 36. |
| An API 36 Google APIs emulator for UI tests | The setup scripts select `arm64-v8a` on ARM hosts and `x86_64` otherwise, create or validate the `Pixel_6` AVD, and default to 2048 MB RAM. Starting the emulator also requires working hardware acceleration. |
| Repository-local Appium 2.16.2, UiAutomator2 driver 3.9.8 and Allure CLI 2.43.0 | Exact versions are pinned in [package.json](../package.json) and [package-lock.json](../package-lock.json). Install the lockfile with `npm ci`; do not substitute a global Appium installation. |

Initial preparation on Windows PowerShell:

```powershell
npm.cmd ci
.\scripts\setup-emulator.ps1
.\scripts\bootstrap.ps1
```

Initial preparation on macOS or Linux:

```bash
npm ci
./scripts/setup-emulator.sh
./scripts/bootstrap.sh
```

These commands come from the [root quick start](../README.md#quick-start).
The setup script provisions and starts the emulator; bootstrap checks the
local prerequisites. On Windows the scripts resolve `adb.exe` under the SDK
selected by `ANDROID_HOME`, `ANDROID_SDK_ROOT`, or the local Android SDK
directory. The bash suite runner invokes `adb` from `PATH` and uses `curl` for
service checks; those commands must be available in its shell. See the two
[suite](../scripts/run-suite.ps1) [runners](../scripts/run-suite.sh).

## Build commands

Build an explicit debug variant:

| Variant | Windows PowerShell | macOS/Linux | APK produced |
|---|---|---|---|
| `stable` | `.\gradlew.bat :app:assembleStableDebug` | `./gradlew :app:assembleStableDebug` | `app/build/outputs/apk/stable/debug/app-stable-debug.apk` |
| `redesign` | `.\gradlew.bat :app:assembleRedesignDebug` | `./gradlew :app:assembleRedesignDebug` | `app/build/outputs/apk/redesign/debug/app-redesign-debug.apk` |

The flavors and default `stable` selection are declared in
[app/build.gradle.kts](../app/build.gradle.kts). The commands are prescribed in
[AGENTS.md](../AGENTS.md#build-and-run), and the exact artifact paths are used by
the [PowerShell](../scripts/run-suite.ps1) and [bash](../scripts/run-suite.sh)
suite runners. An APK build compiles the application; it does not execute an
Appium scenario.

Before committing Kotlin changes, run `.\gradlew.bat ktlintCheck` on Windows or
`./gradlew ktlintCheck` on macOS/Linux, as required by
[AGENTS.md](../AGENTS.md#build-and-run). The root
[build.gradle.kts](../build.gradle.kts) applies the ktlint plugin to subprojects.

## Run the suite

For UI tests, keep the emulator booted and start these long-running processes
in separate terminals:

| Process | Windows PowerShell | macOS/Linux |
|---|---|---|
| Backend | `.\gradlew.bat :fake-api:run` | `./gradlew :fake-api:run` |
| Appium | `.\scripts\start-appium.ps1` | `./scripts/start-appium.sh` |

Use a third terminal for the suite:

| Selected UI | Windows PowerShell | macOS/Linux |
|---|---|---|
| Stable | `.\scripts\run-suite.ps1` | `./scripts/run-suite.sh` |
| Redesign | `.\scripts\run-suite.ps1 -Flavor redesign` | `FLAVOR=redesign ./scripts/run-suite.sh` |

These are the commands supplied in [appium-tests/README.md](../appium-tests/README.md#run).
The startup scripts bind Appium to `127.0.0.1:4723` by default and use the pinned
local installation. The suite runners require that address and check
`http://127.0.0.1:8080/swagger` for backend readiness. The app reaches the host
backend through `http://10.0.2.2:8080`, configured in
[app/build.gradle.kts](../app/build.gradle.kts).

The runners build the selected APK, select a booted device, disable animations,
request a sandbox reset, remove prior JUnit/Allure results and execute
`:appium-tests:test --rerun` with matching `app.apk`, `ui.variant` and
`appium.devices` properties. If several devices are connected, select one with
the PowerShell `-Device` parameter or bash `DEVICE` environment variable.
The [runner sources](../scripts/run-suite.ps1) and
[bash equivalent](../scripts/run-suite.sh) define these options. A direct Gradle
test invocation omits this preparation unless it is supplied separately.

For API tests, only the backend is needed; the emulator and Appium are not part
of API execution. Keep the backend terminal running, then use
`.\gradlew.bat :api-tests:test --rerun` or `./gradlew :api-tests:test --rerun`.
The [API README](../api-tests/README.md#run) documents this command and the
`-Dapi.url=http://host:port` override. Its default is `http://localhost:8080` in
[api-tests/build.gradle.kts](../api-tests/build.gradle.kts).

## Verify it worked

A verified run needs fresh results for the intended suite and variant, a
non-zero number of executed tests, and no failures or errors. Check skipped
tests as well: a skipped scenario has not verified its expected result.
`BUILD SUCCESSFUL`, an APK on disk, or an old report does not establish these
facts. This follows the evidence requirements in
[AGENTS.md](../AGENTS.md) and [AI_POLICY.md](../AI_POLICY.md).

| Evidence | Location and use |
|---|---|
| UI run log | Root `suite-run.log`; records Gradle execution. PowerShell also appends an `Appium result:` summary with passed, failed, error, skipped and total counts. |
| UI JUnit XML | `appium-tests/build/test-results/test/TEST-*.xml`; inspect the actual test names and counts for the fresh run. |
| UI Gradle HTML | `appium-tests/build/reports/tests/test/index.html`. |
| UI raw Allure results | `appium-tests/build/allure-results`; inspect steps and attachments. |
| UI static Allure report | `appium-tests/build/reports/allure-report/index.html`, when report generation succeeds. Report-generation failure does not remove the raw results. |
| UI failure details | `appium-tests/build/reports/digests` and `appium-tests/build/reports/failures`; see [test architecture](test_architecture.md#failure-artifacts). |
| API JUnit and Gradle HTML | `api-tests/build/test-results/test/TEST-*.xml` and `api-tests/build/reports/tests/test/index.html`. |
| API raw Allure results | `api-tests/build/allure-results`; the [shared request filter](../api-tests/src/test/kotlin/client/ApiSpec.kt) attaches requests and responses. |

The UI locations are listed in [appium-tests/README.md](../appium-tests/README.md#evidence)
and the suite scripts. The API task in
[api-tests/build.gradle.kts](../api-tests/build.gradle.kts) clears its raw Allure
results before execution. The API command does not create the UI runner's
`suite-run.log`; retain its own console output with the matching reports.

The [PowerShell runner](../scripts/run-suite.ps1) explicitly rejects zero tests.
The [bash runner](../scripts/run-suite.sh) returns Gradle's test-task exit code
without an independent zero-test check. On either OS, inspect fresh JUnit counts
before claiming that the requested coverage passed. The
[root CI description](../README.md#ci-scope) also states that CI compiles the
Appium module but does not run its UI suite.
