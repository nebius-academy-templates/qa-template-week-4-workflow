# Environment notes

- OS and version: Windows 11 Home 25H2, build 26200.9457.
- Host CPU architecture: x86_64 (AMD64).
- JDK: Oracle JDK 17.0.12; `java version "17.0.12" 2024-07-16 LTS`, runtime build `17.0.12+8-LTS-286`.
- Android platform: `android-36`, platform package revision 2.
- Build tools: 36.0.0.
- System image: Android API 36, Google APIs, x86_64, system-image package revision 7.
- AVD name: `Pixel_6`; 2048 MB RAM; connected as `emulator-5554`.
- Node.js: `v22.13.0`.
- Appium: `2.16.2`.
- UiAutomator2 driver: `uiautomator2@3.9.8 [installed (npm)]`.
- APK build variant: `stableDebug`, selected explicitly with `-Flavor stable`.
- Appium smoke command: `.\scripts\run-suite.ps1 -Flavor stable -Device emulator-5554`.
- API smoke command: `.\gradlew.bat :api-tests:test --rerun --console=plain`.
- Allure CLI: `2.43.0`.
- Date of setup: 2026-09-15; execution timestamps use UTC+02:00.

The two test commands executed the complete starter suites, including their
smoke tests. Their separate results and durations are recorded in
[baseline_report.md](baseline_report.md).

## Setup verification

The source was
[AI-for-Kotlin-practice at `cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`](https://github.com/nebius-academy-templates/AI-for-Kotlin-practice/tree/cadca2fb442682e0eda7df9a7199cf5ddfcf1beb).
The original `npm.cmd ci` installed the lockfile's dependencies. The version
commands confirmed the JDK, Node.js, Appium, driver and Allure values above.
Android package metadata confirmed the platform, build-tools and system image.
The booted device reported API 36 and `x86_64`.

The original `.\scripts\bootstrap.ps1` completed with **14 passed, 0 failed**.
The original backend command was `.\gradlew.bat :fake-api:run --console=plain`.
The original `.\scripts\start-appium.ps1` started Appium on `127.0.0.1:4723`;
the backend served `127.0.0.1:8080`.

## Recorded environment adjustments

The existing AVD could not start because its drive lacked free space. A
disposable copy of its configuration used fresh emulator data on another
drive, selected through the process-local `ANDROID_AVD_HOME`. It was started
with `emulator.exe -avd Pixel_6 -no-window -no-audio -no-snapshot`. The original
AVD data was preserved.

The first mobile run encountered an Android **Bluetooth keeps stopping**
dialog. Bluetooth was disabled only inside the disposable emulator, the
Bluetooth process was stopped, and Android Back dismissed the dialog. Before
the second mobile run, `bluetooth_on` was `0` and the focused window was the
launcher. Both mobile executions are distinguished in the baseline report.

The [baseline evidence package](https://github.com/nebius-academy-templates/qa-template-week-4-workflow/tree/main/evidence/baseline)
retains the source revision, run metadata, JUnit results and Allure summaries.
These notes describe this example's environment; verify your own installed
versions and device before running the practice project.
