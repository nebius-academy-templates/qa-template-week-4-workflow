# Baseline evidence

These results support the completed example in
[baseline_report.md](../../repository-files/baseline_report.md). They were
produced by the unchanged starter suites at
[AI-for-Kotlin-practice revision `cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`](https://github.com/nebius-academy-templates/AI-for-Kotlin-practice/tree/cadca2fb442682e0eda7df9a7199cf5ddfcf1beb)
on 2026-09-15.

| Directory | JUnit result | Allure result | Allure duration |
|---|---|---|---|
| `api/` | 4 passed, 0 failed, 0 skipped | 4 passed | 7.647 s |
| `mobile-initial/` | 0 passed, 6 failed, 0 skipped | 6 broken | 109.578 s |
| `mobile-final/` | 5 passed, 1 failed, 0 skipped | 5 passed, 1 broken | 54.549 s |

Each directory contains:

- `run.json`: recorded command, revision, timestamps and process exit code.
- `summary.json`: the generated Allure report's `widgets/summary.json`, copied
  without changing its statistics or timestamps.
- `junit/TEST-*.xml`: normalized copies of the Gradle JUnit results. Test and
  suite names, counts, timestamps, durations, failure types and the first line
  of each failure message are preserved. Hostname, properties, captured
  stdout/stderr, capabilities and stack traces are omitted. These copies are
  intentionally smaller than the original reports and do not replace their
  detailed diagnostic artifacts.

[mobile-environment-adjustment.txt](mobile-environment-adjustment.txt)
records the emulator checks and commands between the mobile executions. The
initial evidence remains separate from the final evidence. No combined pass
rate is calculated across retries.

The final mobile run is unsuccessful. Its remaining failure is an unavailable
UiAutomator2 instrumentation process, categorized as `broken` by Allure and
as a test failure by Gradle/JUnit. The cause of the process stopping is not
established by this compact package.

The complete local reports, attachments and server logs are not published
here. A student should retain the fresh outputs from their own environment
when investigating a failure.
