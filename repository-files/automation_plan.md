# Automation plan: API-2007

## Status

- Test Case: `API-2007` — `An empty destination is rejected without creating an active ride`
- Artifact status: `complete`
- Automated validation: recorded outside this file by the lesson checker. No checker result is supplied with this example.
- Generation gate: use this plan only after automated validation passes.
- Source review: practice revision `cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`; workbook `test-cases.xlsx`, `Case Summary` and `Steps`, row 19. No implementation or test execution occurred.

## Test contract

| Item | Decision |
|---|---|
| Objective | Reject an empty destination and leave the sandbox session without an active ride. |
| Preconditions | Fresh authorized session, default settings, no active ride; reuse `ApiTestCase.resetSandboxStates`, `obtainToken` and `releaseSandboxSession`. |
| Request | POST `/rides` with bearer token and body `{"from":"Oak Avenue","to":"","rideOptionId":1}`; then GET `/rides/active` with the same token and session. |
| Expected result | POST: HTTP 400, `error="Both from and to are required"`; GET: HTTP 404, `error="No active ride"`. |
| Contract source | `fake-api/openapi.yaml`: `/rides` POST 400, `/rides/active` GET 404, `CreateRideRequest` and `ErrorResponse`. Exact messages come from the supplied case; `fake-api/src/main/kotlin/com/sandbox/qa/fakeapi/Routes.kt` corroborates them. |
| Coverage check | Read both classes in `api-tests/src/test/kotlin/tests/`: `ApiSmokeTest` covers authentication; `RideLifecycleApiTest` covers completion, cancellation and no-driver behavior. None submits an empty destination. IDs 2000–2003 are occupied; 2007 is unused in this snapshot. |

## Implementation map

| Request or response concern | Reuse or planned addition | Repository and contract evidence |
|---|---|---|
| Session, authorization, cleanup | Reuse `api-tests/src/test/kotlin/rule/ApiTestCase.kt:resetSandboxStates`, `obtainToken`, `releaseSandboxSession`. | Reset isolates the case; `api-tests/src/test/kotlin/client/ApiSpec.kt:ApiSession` and `ApiSpec.request` retain the same `X-Sandbox-Session` for both calls. |
| Requests and decoding | Reuse `api-tests/src/test/kotlin/client/RidesApi.kt:create`, `active`; `api-tests/src/test/kotlin/model/RequestModels.kt:CreateRideRequest`; `api-tests/src/test/kotlin/model/ErrorResponse.kt:ErrorResponse`; `api-tests/src/test/kotlin/client/ApiResponse.kt:statusCode`, `error`. | Existing client methods accept empty strings and decode error responses. No client/model addition or product import is needed. |
| Test data | Reuse `api-tests/src/test/kotlin/testdata/ApiTestData.kt:FROM`, `YELLOW_TARIFF.id`, `NO_ACTIVE_RIDE_ERROR`; add `EMPTY_DESTINATION` and `EMPTY_ROUTE_ERROR`. | Existing values match the case; the empty value and validation-message constant are absent. New values are `""` and `"Both from and to are required"`, from `Steps` row 19. |
| Scenario | Add `api-tests/src/test/kotlin/tests/RideValidationApiTest.kt:testEmptyDestinationDoesNotCreateActiveRide`, extending `ApiTestCase`; use `@AllureId("2007")`, the case title as `@DisplayName`, and `@Feature("API: Ride validation")`. | Coverage comparison found no equivalent scenario. Keep the two requests and their assertions in consecutive named Allure steps. |

## Response assertions

| Expected result | Assertion | Expected-value source |
|---|---|---|
| Empty destination rejected | Compare POST `statusCode` with 400 and `error?.error` with `ApiTestData.EMPTY_ROUTE_ERROR`. | Case step 1; OpenAPI POST 400 and required `ErrorResponse.error`. |
| No active ride created | Compare subsequent GET `statusCode` with 404 and `error?.error` with `ApiTestData.NO_ACTIVE_RIDE_ERROR`. | Case step 2; OpenAPI GET 404 example. Preserve the token/session; do not reset or cancel between calls. |

- Excluded checks: order history, driver fields, prices, response timing, missing pickup and other invalid requests; the case does not require them.

## Allowed files

| Path | Change and reason |
|---|---|
| `api-tests/src/test/kotlin/tests/RideValidationApiTest.kt` | Add this one scenario with both expected outcomes. |
| `api-tests/src/test/kotlin/testdata/ApiTestData.kt` | Add the two case-sourced constants; preserve existing values. |

Any additional file requires a revised plan and another automated validation pass.

## Risks and stop conditions

- Risk: a reset or a different token/session between requests would hide an incorrectly created ride; retain the same session throughout both steps.
- Stop when: current coverage occupies ID 2007, case/contract evidence conflicts, the plan lacks validation, or execution prerequisites are unavailable.
- Do not compensate with backend edits, weaker assertions, retries or unrelated test changes.

## Verification

- Metadata inventory: `rg -n '@DisplayName|@AllureId' api-tests/src/test/kotlin/tests`.
- Format check: `./gradlew ktlintCheck`; use `.\gradlew.bat ktlintCheck` on Windows.
- Test run: start `./gradlew :fake-api:run` in one terminal; run `./gradlew :api-tests:test --rerun` in another. On Windows replace `./gradlew` with `.\gradlew.bat`.
- Required result: the generated method executes without skips, failures or errors; the full API suite passes.
- Evidence: fresh `api-tests/build/test-results/test/TEST-tests.RideValidationApiTest.xml`, matching results and both scenario HTTP request/response pairs under `api-tests/build/allure-results`, and actual command output. None is supplied by this source-only plan.
