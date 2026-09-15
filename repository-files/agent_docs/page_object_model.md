# Page object model

Course reference example, checked against practice revision
`cadca2fb442682e0eda7df9a7199cf5ddfcf1beb`. The examples below are copied from
the cited source declarations; they do not claim that a test was executed.
Links resolve from `agent_docs/` in the practice repository.

## Role of pages

Page objects are singleton element catalogs. For example,
[PhoneLoginPage](../appium-tests/src/test/kotlin/pages/PhoneLoginPage.kt) declares
the phone field and Continue button,
[MapPage](../appium-tests/src/test/kotlin/pages/MapPage.kt) declares ride controls
and per-ride factories, and
[OrderHistoryPage](../appium-tests/src/test/kotlin/pages/OrderHistoryPage.kt)
declares the history title and per-order fields. The catalog stores `Element`
descriptions rather than cached `WebElement` instances.

[Element](../appium-tests/src/test/kotlin/pages/Element.kt) stores a `testTag`
and default timeout. Each operation obtains the current thread's driver from
`DriverFactory.current()` and creates an `AppiumBy.id(testTag)` locator. The
app exposes Compose tags as resource IDs through
[MainActivity](../app/src/main/java/com/sandbox/qa/MainActivity.kt)'s
`testTagsAsResourceId = true` semantics. [AGENTS.md](../AGENTS.md#appium-test-architecture)
requires ID locators and forbids text XPath. The supplied
[DriverFactory](../appium-tests/src/test/kotlin/rule/DriverFactory.kt) disables
ID autocompletion so Appium does not prepend a package prefix to these tags.

| Element operation | How it finds or waits for the element |
|---|---|
| `waitFor`, `click`, `sendKeys`, `clear` | `locate(timeoutSec)` waits for `visibilityOfElementLocated`. `waitFor` returns that element; the other methods operate on it. |
| `text` | Uses `locate(defaultTimeoutSec)` before reading text. |
| `waitForGone` | Waits for `invisibilityOfElementLocated`, which accepts an invisible or absent element. |
| `isPresent` | Direct `findElements` probe, with no explicit wait and no visibility condition. |
| `retryClick` | Repeated direct `findElement(...).click()` calls, polling every 300 ms. Only missing, stale and intercepted-element exceptions are retried. |

The default timeout is 10 seconds unless a declaration or method argument
changes it. In particular, `retryClick` and `waitForGone` do not call the
ordinary visibility helper. This distinction comes from the method bodies
in [Element.kt](../appium-tests/src/test/kotlin/pages/Element.kt); a catalog
handle does not guarantee that the element currently exists or that a click
will complete the intended transition.

## No asserts rule

[AGENTS.md](../AGENTS.md#appium-test-architecture) and
[appium-tests/README.md](../appium-tests/README.md#architecture) place assertions,
interactions, waits and flows in actions. Pages declare elements; tests compose
the actions into named scenarios. The module's
[checkPageObjectBoundary task](../appium-tests/build.gradle.kts) rejects
`Element(...)` construction outside `pages/`.

For example, `OrderHistoryPage.orderPrice(id)` names a field, while
[OrderActions.assertHistoryPrices](../appium-tests/src/test/kotlin/actions/OrderActions.kt)
receives expected prices and compares them with the displayed text. This keeps
the same locator usable in different scenarios without embedding one scenario's
expected result in the catalog. Keep driver access in `Element` or the supplied
[Device](../appium-tests/src/test/kotlin/pages/Device.kt) operations rather than
repeating it in tests.

## Example from this repo

The following declarations are from
[OrderHistoryPage.kt](../appium-tests/src/test/kotlin/pages/OrderHistoryPage.kt):

```kotlin
object OrderHistoryPage {
    val title = Element("orders_title")
    val ordersList = Element("orders_list", defaultTimeoutSec = 15)
    val backButton = Element("orders_back_button")

    val errorLabel = Element("orders_error", defaultTimeoutSec = 15)

    fun orderPrice(orderId: Int) = Element("order_price_$orderId", defaultTimeoutSec = 15)

    fun orderRoute(orderId: Int) = Element("order_route_$orderId", defaultTimeoutSec = 15)
}
```

`orderPrice` and `orderRoute` build locators for a supplied order ID; they do
not select an order by list position or read expected data. The corresponding
product declarations are in
[OrderHistoryScreen.kt](../app/src/main/java/com/sandbox/qa/ui/OrderHistoryScreen.kt).

The catalog contains no `awaitReady` method. Its callers choose the evidence
needed for their operation: `OrderActions.assertHistoryPrices` first waits for
the title and then reads each required price through `Element.text`. The title
wait alone does not establish that the correct order values have loaded.

## How actions use pages

[OrderActions.kt](../appium-tests/src/test/kotlin/actions/OrderActions.kt)
contains this complete method:

```kotlin
fun assertHistoryPrices(expected: Map<Int, String>) {
    OrderHistoryPage.title.waitFor(15)
    expected.forEach { (id, price) ->
        assertEquals(price, OrderHistoryPage.orderPrice(id).text, "price of order $id")
    }
}
```

The action waits for the screen title, resolves the price associated with each
expected ID and compares its text with the caller's expected string. It verifies
the supplied prices; this method does not assert that the history contains no
additional orders. A scenario requiring an exact list needs evidence for that
additional expected result.

Navigation is also action-owned. `OrderActions.returnToRideForm` clicks the
history back button and waits for the map's destination and pull-to-refresh
elements. [DrawerActions.openOrders](../appium-tests/src/test/kotlin/actions/DrawerActions.kt)
opens the drawer and retries only the transition-sensitive menu click; the
history assertion provides the subsequent screen and value checks.

[RideAndHistoryE2ETest](../appium-tests/src/test/kotlin/tests/RideAndHistoryE2ETest.kt)
connects these operations in named steps: it calls `drawer.openOrders()`, then
`orders.assertHistoryPrices(TestData.PAST_ORDERS)`. Expected values come from
[TestData.kt](../appium-tests/src/test/kotlin/testdata/TestData.kt), rather than
being read from the same UI field that the assertion checks.
