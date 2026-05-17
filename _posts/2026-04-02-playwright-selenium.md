---
title: "Playwright-Selenium"
date: 2026-04-02 16:01:09 
author: shajeebhameed
layout: page
slug: playwright-selenium
status: publish
---

## Playwright & Selenium — Complete Technical Interview Q&A

\---First, here's the full question navigator so you can see everything covered at a glance — click any topic to get that specific deep-dive answer.

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-3.png?w=1024)

Click any button above to get the deep-dive answer on that topic. Now here is the complete written Q&A for every section.

* * *

### 🔷 SECTION 1: Architecture & Internals

* * *

**Q1: How does Playwright communicate with the browser internally compared to Selenium?**

This is the most important architectural question and the root of most practical differences between the two tools.

Selenium uses the WebDriver protocol — an HTTP-based, external process model. Your test code sends HTTP requests to a WebDriver server (ChromeDriver, GeckoDriver, etc.), which translates them into browser commands. Every action is a separate HTTP round-trip: find element, click element, get text — each is a distinct request-response cycle. This introduces latency per action and means the test process and browser are completely separate processes communicating over a network socket.

Playwright uses the Chrome DevTools Protocol (CDP) for Chromium-based browsers and equivalent protocols for Firefox and WebKit, communicating over a persistent WebSocket connection. Instead of separate HTTP requests per action, commands are sent over a single persistent connection as JSON messages. Playwright also runs in-process with the browser — it directly controls the browser engine rather than going through an intermediary driver.

The practical consequences of this architectural difference are significant:

Speed: Playwright's WebSocket connection and in-process execution is substantially faster than Selenium's HTTP round-trip model. A Playwright test that does 50 interactions completes faster than an equivalent Selenium test because there's no per-action HTTP overhead.

Auto-waiting: because Playwright has direct access to the browser's internal state, it can subscribe to browser events and wait for specific conditions (element visible, network idle, animation complete) without polling. Selenium has no native access to these events, which is why explicit waits in Selenium are polling loops.

Multi-browser: Selenium uses browser-specific drivers maintained separately from the test framework. Playwright ships with its own patched browser binaries, giving it consistent behaviour across Chromium, Firefox, and WebKit without driver compatibility issues.

The architecture diagram worth drawing in your head: Selenium is `test → HTTP → ChromeDriver → Chrome`; Playwright is `test ↔ WebSocket ↔ Browser engine directly`.

* * *

**Q2: Explain Playwright browser contexts — what are they and how do they differ from Selenium sessions?**

A browser context in Playwright is an isolated browser session within a single browser instance. It has its own cookies, localStorage, sessionStorage, cache, and permissions — completely isolated from other contexts — but it shares the underlying browser process. You can have 20 contexts running in parallel inside a single Chromium instance, each completely isolated from the others.

In Selenium, isolation is achieved by spinning up entirely separate WebDriver instances, each with their own browser process. Running 20 parallel tests in Selenium means 20 browser processes. This is significantly more resource-intensive.

The practical architecture for test isolation in Playwright: one browser instance per test worker, a new context per test. Creating a context is cheap (milliseconds) compared to launching a browser (seconds). Playwright's default test runner creates a fresh context for every test automatically, giving you perfect isolation without the overhead of browser restarts.

Within a context, you create pages — equivalent to browser tabs. A single context can have multiple pages open simultaneously, which is how you test multi-tab flows.
    
    
    // Playwright — one browser, two isolated contexts
    const browser = await chromium.launch();
    const context1 = await browser.newContext(); // User A's session
    const context2 = await browser.newContext(); // User B's session
    const page1 = await context1.newPage();
    const page2 = await context2.newPage();
    // page1 and page2 have completely isolated cookies and storage
    
    
    

This architecture is also why Playwright's `storageState` feature is powerful for authentication: you authenticate once, save the context state to a JSON file, and restore it in subsequent tests — avoiding a full login flow per test.

* * *

**Q3: Explain the Selenium WebDriver W3C protocol — how do commands travel from test to browser?**

When you call `driver.findElement(By.id("submit")).click()` in Selenium, here's the exact sequence:

Your test code calls the Selenium client library (Java, Python, C#, etc.). The client library serialises the command into an HTTP request in the W3C WebDriver JSON format. For `findElement`, the request body is `{"using": "id", "value": "submit"}` sent as a POST to `http://localhost:4444/session/{session-id}/element`. ChromeDriver receives this HTTP request, translates it into a Chrome DevTools Protocol command, sends it to the Chrome browser over a local socket, receives the element reference back, and returns the element ID in an HTTP response. Your client library receives the HTTP response, deserialises the element reference, and stores it. Then the `click()` call makes a second HTTP round-trip: POST to `/session/{session-id}/element/{element-id}/click`.

Every single action is a round-trip. Finding an element, clicking it, typing text, getting its text value — each is a separate HTTP request. This is the fundamental source of Selenium's latency relative to Playwright.

The W3C WebDriver protocol replaced the older JSON Wire Protocol (which was ChromeDriver's predecessor). The W3C protocol is now the official standard, meaning all modern browser drivers implement the same API surface, which fixed many cross-browser consistency issues that plagued older Selenium suites.

One important implication: because Selenium uses HTTP, you can run tests against a remote browser just by pointing the driver URL at a remote Selenium Grid endpoint. The test code doesn't change — only the connection target. This is how Selenium Grid and cloud testing services like BrowserStack and Sauce Labs work.

* * *

**Q4: How does Playwright's auto-waiting work internally?**

This is one of the most commonly misunderstood aspects of Playwright. Auto-waiting is not a retry loop — it's an event-driven system built on the browser's native event model.

When you call `page.click('button#submit')`, Playwright performs a sequence of actionability checks before executing the click. In order: it verifies the element exists in the DOM, it verifies the element is visible (not `display: none`, `visibility: hidden`, or has zero dimensions), it verifies the element is stable (not currently animating — it waits for the element's bounding box to stop changing), it verifies the element is enabled (not disabled), it verifies the element can receive pointer events (not covered by another element), and only then does it execute the click.

These checks are performed by Playwright injecting a predicate into the browser context and waiting for it to resolve. Playwright subscribes to DOM mutation events and layout events — so instead of polling every 100ms, it's notified by the browser when the state changes. This is what makes Playwright's waiting both reliable and fast.

The `timeout` you set (default 30 seconds) is the maximum wait before giving up. If all actionability checks pass immediately, the action executes immediately — there's no artificial delay.

You can also explicitly wait for specific conditions:
    
    
    // Wait for element to be visible
    await page.waitForSelector('.modal', { state: 'visible' });
    
    // Wait for network to be idle (no requests for 500ms)
    await page.waitForLoadState('networkidle');
    
    // Wait for a specific API response
    const response = await page.waitForResponse('**/api/users');
    
    // Wait for arbitrary condition
    await page.waitForFunction(() => document.title === 'Loaded');
    
    
    

The key distinction from Selenium's explicit waits: Playwright's waits are event-driven and stop as soon as the condition is met. Selenium's explicit waits poll at a configurable interval (default 500ms), meaning there's always a delay between when the condition becomes true and when the wait returns.

* * *

### 🔷 SECTION 2: Locators & Element Interaction

* * *

**Q5: Playwright locator strategies — which to use and when?**

Playwright's recommended locator priority from most to least preferred:

`page.getByRole()` is the gold standard. It locates elements by their ARIA role and accessible name, which mirrors how assistive technologies and users actually perceive the page. It's the most resilient to implementation changes — a button stays a button even if its class or ID changes.
    
    
    await page.getByRole('button', { name: 'Submit' }).click();
    await page.getByRole('textbox', { name: 'Email' }).fill('test@example.com');
    await page.getByRole('checkbox', { name: 'Accept terms' }).check();
    
    
    

`page.getByLabel()` locates form inputs by their associated label text. Clean, semantic, and resilient.

`page.getByPlaceholder()` for inputs with placeholder text when no label is available.

`page.getByText()` for locating elements by their text content. Use `exact: true` when you need exact match rather than substring.

`page.getByTestId()` uses a `data-testid` attribute. Requires the development team to add test IDs, but produces the most stable locators since `data-testid` exists purely for testing and won't change with UI redesigns.

`page.locator('css=...')` and `page.locator('xpath=...')` for when semantic locators aren't feasible. CSS is preferred over XPath for performance and readability.

The senior-level answer here includes the tradeoff: `getByRole` and `getByText` are fragile to text changes (if "Submit" becomes "Send", the locator breaks), while `getByTestId` is fragile to the team's discipline in maintaining test IDs. In practice, I use `getByRole` for interactive elements, `getByTestId` for complex components, and CSS selectors as a last resort.

* * *

**Q6: Selenium locator strategies — priority order and when to use each?**

Priority order from most to least preferred in Selenium:

ID (`By.id("submit")`) — fastest, most specific. Use whenever a stable, unique ID exists. The caveat: auto-generated IDs (like React's root div) change between renders and are worse than useless.

Name (`By.name("username")`) — good for form inputs. Stable when the form structure is stable.

CSS selector (`By.cssSelector(".login-form button[type='submit']")`) — powerful, readable, fast. The preferred choice for most scenarios when ID is unavailable. CSS selectors are evaluated natively by the browser engine, making them faster than XPath.

XPath (`By.xpath("//button[@data-action='submit']")`) — necessary for traversals that CSS can't express (parent element selection, text content matching). Slower than CSS and more brittle to DOM structure changes. Use only when CSS genuinely can't do the job.

Link text (`By.linkText("Click here")`) — brittle to text changes. Avoid.

Class name (`By.className("btn-primary")`) — problematic because multiple elements often share the same class. Use CSS selector instead for more specificity.

The rule I follow: try ID first, fall back to CSS selector with enough context for uniqueness, reach for XPath only for cases CSS can't handle (like "the div that contains text 'Error'").

* * *

**Q7: How do you handle StaleElementReferenceException in Selenium?**

`StaleElementReferenceException` occurs when the element reference held by your test becomes invalid because the DOM has been updated since you found the element. Common causes: a page reload, a JavaScript framework re-rendering the component, or an AJAX update replacing the DOM subtree.

The three solutions in order of preference:

Re-find the element before each interaction: instead of storing `WebElement loginBtn = driver.findElement(...)` and reusing it, call `driver.findElement(...)` immediately before each action. This is the most reliable approach but verbose.

Use a retry wrapper:
    
    
    public void clickWithRetry(By locator) {
        int attempts = 0;
        while (attempts < 3) {
            try {
                driver.findElement(locator).click();
                return;
            } catch (StaleElementReferenceException e) {
                attempts++;
                if (attempts == 3) throw e;
            }
        }
    }
    
    
    

Use `ExpectedConditions.refreshed()` in an explicit wait, which handles the staleness internally:
    
    
    WebElement element = wait.until(
        ExpectedConditions.refreshed(
            ExpectedConditions.elementToBeClickable(By.id("submit"))
        )
    );
    element.click();
    
    
    

The senior-level insight: `StaleElementReferenceException` is often a symptom of a test design problem. If you're regularly hitting stale elements, it usually means the test is racing against the UI's rendering cycle. The fix is better wait conditions, not just retry logic.

Playwright essentially eliminates this problem by design — locators in Playwright are lazy and re-evaluated on every action, so you never hold a stale element reference.

* * *

**Q8: How does Playwright handle Shadow DOM, iframes, and web components?**

These are three distinct DOM isolation mechanisms that cause pain in Selenium but are handled cleanly in Playwright.

Shadow DOM: Playwright's `locator()` method pierces Shadow DOM boundaries automatically by default. You don't need any special handling — `page.locator('.my-custom-element input')` will find an input inside a shadow root. In Selenium, you have to explicitly execute JavaScript to access shadow roots: `((JavascriptExecutor)driver).executeScript("return arguments[0].shadowRoot", hostElement)`, then interact with the returned shadow root object.

iframes: In Playwright, use `frameLocator()` to scope queries inside an iframe:
    
    
    const frame = page.frameLocator('#payment-iframe');
    await frame.getByLabel('Card number').fill('4111111111111111');
    
    
    

In Selenium, you switch the driver's context: `driver.switchTo().frame("payment-iframe")`, interact with the frame's elements, then `driver.switchTo().defaultContent()` to return. Forgetting to switch back is a classic source of test bugs.

Web components: since Playwright pierces Shadow DOM automatically, web components built with Shadow DOM work transparently. In Selenium you need custom JavaScript executor calls to traverse into component shadow roots, and these break easily with component library updates.

* * *

### 🔷 SECTION 3: Waits & Synchronisation

* * *

**Q9: Selenium wait types — implicit, explicit, and fluent. How do they work?**

Implicit wait: a global setting that tells Selenium how long to wait when trying to find an element before throwing `NoSuchElementException`. Set once, applies to every `findElement` call.
    
    
    driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    
    
    

The problem: implicit waits interact badly with explicit waits, creating unpredictable combined timeout behaviour. They also slow down negative tests — a test asserting an element does NOT exist will wait the full implicit timeout before confirming the element isn't there. Never mix implicit and explicit waits.

Explicit wait: waits for a specific condition to be true before proceeding, with a configurable timeout and polling interval.
    
    
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement element = wait.until(
        ExpectedConditions.elementToBeClickable(By.id("submit"))
    );
    
    
    

Explicit waits are the correct approach. They're precise, they fail fast when conditions become impossible, and they don't interfere with other waits.

Fluent wait: an explicit wait with configurable polling interval and the ability to ignore specific exception types during polling.
    
    
    Wait<WebDriver> fluentWait = new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(30))
        .pollingEvery(Duration.ofMillis(500))
        .ignoring(NoSuchElementException.class);
    
    
    

Use fluent waits when you need fine-grained control over polling behaviour, or when you want to ignore specific transient exceptions during the wait period.

The definitive rule: always use explicit waits, never use implicit waits in production test code, and never use `Thread.sleep()`.

* * *

**Q10: Playwright wait methods — which to use and when?**

`page.waitForSelector(selector, { state })` waits for an element to reach a specific state: `'attached'` (in DOM), `'detached'` (removed from DOM), `'visible'`, or `'hidden'`. Use this when you need to wait for a specific element state before proceeding.

`page.waitForLoadState(state)` waits for the page to reach a load state: `'load'` (the load event fired), `'domcontentloaded'` (DOMContentLoaded fired), or `'networkidle'` (no network requests for 500ms). `networkidle` is useful but fragile on pages with continuous background polling.

`page.waitForResponse(urlPattern)` waits for a specific network response. Invaluable for testing flows that depend on API calls:
    
    
    const [response] = await Promise.all([
        page.waitForResponse('**/api/checkout'),
        page.click('#pay-now')
    ]);
    const body = await response.json();
    expect(body.status).toBe('success');
    
    
    

`page.waitForFunction(fn)` evaluates an arbitrary JavaScript expression in the browser context and waits until it returns truthy:
    
    
    await page.waitForFunction(
        () => document.querySelectorAll('.product-card').length > 10
    );
    
    
    

`expect(locator).toBeVisible()` with Playwright's built-in assertion auto-retries automatically — it polls until the assertion passes or times out. This is often the cleanest approach for simple visibility checks.

The senior insight: most flaky tests in Playwright come from developers not using the right wait for the job. Using `waitForLoadState('networkidle')` on a page with WebSocket connections or polling APIs will cause random timeouts. Prefer `waitForResponse` for API-triggered state changes and `waitForSelector` for DOM-driven state changes.

* * *

### 🔷 SECTION 4: Framework Design & Patterns

* * *

**Q11: Page Object Model — design, implementation, and anti-patterns**

The Page Object Model separates element locators and page interactions from test logic. Each page (or significant component) gets its own class that exposes business-level actions, not low-level element interactions.

Playwright implementation:
    
    
    // pages/LoginPage.ts
    export class LoginPage {
        constructor(private page: Page) {}
    
        async navigate() {
            await this.page.goto('/login');
        }
    
        async login(email: string, password: string) {
            await this.page.getByLabel('Email').fill(email);
            await this.page.getByLabel('Password').fill(password);
            await this.page.getByRole('button', { name: 'Sign in' }).click();
        }
    
        async getErrorMessage(): Promise<string> {
            return this.page.getByRole('alert').textContent() ?? '';
        }
    }
    
    // tests/login.spec.ts
    test('invalid credentials show error', async ({ page }) => {
        const loginPage = new LoginPage(page);
        await loginPage.navigate();
        await loginPage.login('bad@email.com', 'wrongpassword');
        expect(await loginPage.getErrorMessage()).toContain('Invalid credentials');
    });
    
    
    

Selenium implementation (Java):
    
    
    public class LoginPage {
        private WebDriver driver;
        private By emailField = By.id("email");
        private By passwordField = By.id("password");
        private By submitButton = By.cssSelector("button[type='submit']");
        private By errorMessage = By.className("error-alert");
    
        public LoginPage(WebDriver driver) {
            this.driver = driver;
            PageFactory.initElements(driver, this);
        }
    
        public void login(String email, String password) {
            driver.findElement(emailField).sendKeys(email);
            driver.findElement(passwordField).sendKeys(password);
            driver.findElement(submitButton).click();
        }
    
        public String getErrorMessage() {
            return driver.findElement(errorMessage).getText();
        }
    }
    
    
    

The five POM anti-patterns to name in interviews: (1) assertions inside page objects — page objects should return data, not assert on it; (2) test logic inside page objects — navigation and state setup belongs in tests, not pages; (3) one massive page object for the entire application — split by feature area; (4) storing WebElement references as instance fields — they go stale; (5) page objects that expose raw locators to tests — tests should call methods, not access locators directly.

* * *

**Q12: Playwright fixtures — what they are and how they differ from hooks**

Fixtures are Playwright's dependency injection system for tests. They provide reusable, composable setup and teardown without the coupling problems of `beforeEach`/`afterEach` hooks.

A hook is implicitly shared across all tests in a describe block. A fixture is explicitly declared as a parameter — only tests that request it get it, and the fixture is created fresh for each test that needs it.
    
    
    // fixtures.ts
    import { test as base, expect } from '@playwright/test';
    import { LoginPage } from './pages/LoginPage';
    import { DashboardPage } from './pages/DashboardPage';
    
    type MyFixtures = {
        loginPage: LoginPage;
        authenticatedPage: Page; // pre-authenticated page
    };
    
    export const test = base.extend<MyFixtures>({
        loginPage: async ({ page }, use) => {
            const loginPage = new LoginPage(page);
            await loginPage.navigate();
            await use(loginPage);
            // teardown here if needed
        },
    
        authenticatedPage: async ({ browser }, use) => {
            // Load saved auth state — no login flow needed
            const context = await browser.newContext({
                storageState: 'auth.json'
            });
            const page = await context.newPage();
            await use(page);
            await context.close();
        }
    });
    
    // tests/dashboard.spec.ts
    import { test, expect } from '../fixtures';
    
    test('dashboard shows user name', async ({ authenticatedPage }) => {
        await authenticatedPage.goto('/dashboard');
        await expect(authenticatedPage.getByRole('heading')).toContainText('Welcome');
    });
    
    
    

The key advantages over hooks: fixtures are composable (a fixture can depend on other fixtures), they're lazy (only created when a test requests them), and they make dependencies explicit (reading a test tells you exactly what it needs). With hooks, the setup is implicit — you have to read the `beforeEach` to understand what state a test starts in.

* * *

**Q13: Test data management — isolation strategies**

Test isolation is one of the hardest problems in E2E automation. There are four strategies in order of robustness:

Factory pattern with teardown: each test creates its own data via API calls or database seeding, and deletes it after. Fully isolated but slow and requires cleanup logic.
    
    
    test.beforeEach(async ({ request }) => {
        testUser = await request.post('/api/test/users', {
            data: { email: `test-${Date.now()}@example.com` }
        });
    });
    
    test.afterEach(async ({ request }) => {
        await request.delete(`/api/test/users/${testUser.id}`);
    });
    
    
    

Database transactions: wrap each test in a database transaction that's rolled back after the test. Perfect isolation with zero cleanup overhead — but only works for tests that run against a directly accessible database.

Unique data per test: use timestamped or UUID-suffixed data so each test run creates globally unique data that doesn't conflict with parallel runs. Requires periodic cleanup of accumulated test data.

Snapshot/restore: take a database snapshot before the test suite, restore it after. Fast but not suitable for parallel runs.

The CI-specific consideration: parallel tests require strategies 1, 3, or 4. Strategy 2 only works for serial execution. The senior answer acknowledges this constraint and explains why test data management is more complex in parallel CI environments.

* * *

### 🔷 SECTION 5: Parallelism & CI/CD

* * *

**Q14: Playwright parallelism and sharding — how does it work?**

Playwright's test runner parallelises at the file level by default — each spec file runs in its own worker process with its own browser context. Tests within a single file run serially by default (to preserve the ability to share state within a describe block), but you can enable full parallel within files using `test.describe.configure({ mode: 'parallel' })`.

Worker count is configured in `playwright.config.ts`:
    
    
    export default defineConfig({
        workers: process.env.CI ? 4 : undefined, // 4 in CI, CPU count locally
        fullyParallel: true, // run all tests in parallel, even within files
    });
    
    
    

Sharding splits the test suite across multiple CI machines. Each shard runs an independent subset:
    
    
    # Machine 1
    npx playwright test --shard=1/4
    
    # Machine 2
    npx playwright test --shard=2/4
    
    # Machine 3
    npx playwright test --shard=3/4
    
    # Machine 4
    npx playwright test --shard=4/4
    
    
    

After all shards complete, merge the reports:
    
    
    npx playwright merge-reports --reporter html ./blob-reports
    
    
    

This is how you scale a 2,000-test suite from 40 minutes to 10 minutes by running across 4 CI agents. Each agent runs its shard independently; the merge step combines results for a unified report.

The important constraint: sharding assumes tests are independent. Any test that depends on state left by another test will be unreliable when sharded, because shard order is not guaranteed.

* * *

**Q15: Selenium Grid — architecture and how to distribute tests**

Selenium Grid has a hub-and-node architecture. The hub is a central router that receives test requests and dispatches them to nodes based on the browser and version requested. Nodes register themselves with the hub and report their available capabilities (browser type, version, OS, maximum concurrent sessions).

Modern Selenium Grid 4 uses a different topology: Router, Distributor, Session Map, and Node components can run as separate microservices or as a combined Standalone deployment.

Docker Grid — the practical CI approach:
    
    
    # docker-compose.yml
    services:
      selenium-hub:
        image: selenium/hub:4
        ports:
          - "4442:4442"
          - "4443:4443"
          - "4444:4444"
    
      chrome-node:
        image: selenium/node-chrome:4
        environment:
          - SE_EVENT_BUS_HOST=selenium-hub
        depends_on:
          - selenium-hub
        deploy:
          replicas: 4  # 4 parallel Chrome instances
    
      firefox-node:
        image: selenium/node-firefox:4
        environment:
          - SE_EVENT_BUS_HOST=selenium-hub
        depends_on:
          - selenium-hub
    
    
    

Tests connect to the grid by pointing `RemoteWebDriver` at the hub URL:
    
    
    WebDriver driver = new RemoteWebDriver(
        new URL("http://selenium-hub:4444"),
        new ChromeOptions()
    );
    
    
    

The Grid handles dispatching the request to an available Chrome node. If all nodes are busy, the request queues until a node is free.

* * *

**Q16: How do you integrate Playwright or Selenium into a CI/CD pipeline?**

The pattern is the same for both tools, with tool-specific configuration differences.

GitHub Actions example for Playwright:
    
    
    name: Playwright Tests
    on: [push, pull_request]
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-node@v4
            with:
              node-version: 20
          - name: Install dependencies
            run: npm ci
          - name: Install Playwright browsers
            run: npx playwright install --with-deps chromium
          - name: Run tests
            run: npx playwright test --shard=${{ matrix.shard }}/4
          - name: Upload test results
            if: always()
            uses: actions/upload-artifact@v4
            with:
              name: playwright-report-${{ matrix.shard }}
              path: playwright-report/
        strategy:
          matrix:
            shard: [1, 2, 3, 4]
    
    
    

Key CI-specific configuration decisions: always run headless (`--headed` is for local debugging only), set workers to a fixed number rather than CPU-auto-detect (CI VMs have variable CPU availability), always upload test artifacts on failure (`if: always()`), cache browser binaries to avoid reinstalling on every run, and set an explicit timeout to prevent hung tests from blocking the pipeline indefinitely.

For Selenium in CI: use Docker Grid or Selenium Manager (included in Selenium 4.6+, which automatically downloads the correct driver version). Set `--headless` flag in browser options. Consider Selenoid (a Docker-based Selenium alternative) for better resource management.

* * *

### 🔷 SECTION 6: Debugging & Flakiness

* * *

**Q17: Playwright debugging tools — Trace Viewer, UI Mode, and Inspector**

Playwright ships with three debugging tools that are genuinely transformative for diagnosing failures.

Trace Viewer captures a complete recording of a test run: every action, every network request and response, every console message, and a screenshot timeline. Enable it in config:
    
    
    use: {
        trace: 'on-first-retry', // capture trace only when test fails and retries
    }
    
    
    

Open a trace file: `npx playwright show-trace trace.zip`. You see a timeline of the test, can click any action to see the DOM state at that moment, inspect network traffic, view console logs, and replay the test visually. This eliminates "I can't reproduce it locally" as an excuse for CI failures.

UI Mode is an interactive test runner that shows your tests in a panel with real-time browser preview as they run. You can step through tests, filter by test name, and see the Playwright actions being executed. Run with `npx playwright test --ui`.

The Inspector (`PWDEBUG=1 npx playwright test`) pauses execution and opens an interactive browser with a Playwright toolbar. You can step through actions one at a time, inspect the page, and use the element picker to generate locators — invaluable when a locator isn't finding the right element.

For Selenium the equivalent debugging toolkit is: WebDriver's `getPageSource()` and `getScreenshotAs()` for post-failure capture, browser DevTools logging via ChromeOptions logging preferences, and for structured failure analysis, Allure Reports with screenshot attachments on failure.

* * *

**Q18: Diagnosing and fixing flaky tests — root causes and strategies**

Flaky tests fail intermittently without code changes. There are five root cause categories, each with distinct fixes.

Timing issues — the most common cause. The test proceeds before the UI is in the expected state. Fix: replace hardcoded sleeps with proper wait conditions. In Playwright, use `waitForResponse`, `waitForSelector`, or `expect(locator).toBeVisible()`. In Selenium, use explicit waits with `ExpectedConditions`.

Test data conflicts — parallel tests interfere because they share test data. Test A creates a user, Test B looks for users and gets confused by Test A's user. Fix: use unique, timestamped data per test. Never share mutable test data across parallel tests.

Environment instability — the test environment itself is unreliable (flapping services, network timeouts, database connection pool exhaustion). Fix: distinguish test flakiness from environment flakiness by tracking which tests flake and whether they correlate with specific time periods or CI load.

Order dependencies — tests pass in sequence but fail when run in a different order or in parallel. Fix: each test must be completely self-sufficient — create its own data, not rely on data left by previous tests.

Browser-specific rendering — an element renders slightly differently at specific viewport sizes or browser versions, causing a locator or assertion to fail intermittently. Fix: use semantic locators (`getByRole`) rather than visual locators, and pin browser versions in CI.

My quarantine process: tag flaky tests with `@flaky`, exclude them from the main CI gate but run them in a separate nightly job. Create a ticket for each flaky test with the failure trace. Review the quarantine queue weekly. A test stays quarantined until the root cause is identified and fixed — it doesn't get silently deleted.

* * *

### 🔷 SECTION 7: Advanced Scenarios

* * *

**Q19: Playwright network interception — route, fulfill, and abort**

`page.route()` intercepts network requests matching a URL pattern and lets you fulfil, modify, or abort them. This is one of Playwright's most powerful features for test isolation.

Mocking an API response:
    
    
    await page.route('**/api/products', async route => {
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify([
                { id: 1, name: 'Test Product', price: 29.99 }
            ])
        });
    });
    await page.goto('/products');
    // The products page renders with mock data, no real API call
    
    
    

Testing error handling:
    
    
    await page.route('**/api/checkout', route => {
        route.fulfill({ status: 503, body: 'Service Unavailable' });
    });
    // Now test that the UI handles the 503 gracefully
    
    
    

Modifying a real response:
    
    
    await page.route('**/api/user', async route => {
        const response = await route.fetch(); // make the real request
        const json = await response.json();
        json.role = 'admin'; // inject admin role
        await route.fulfill({ json }); // return modified response
    });
    
    
    

Blocking specific requests (e.g. analytics, ads) to speed up tests:
    
    
    await page.route('**/*.{png,jpg,gif,webp}', route => route.abort());
    await page.route('**/analytics/**', route => route.abort());
    
    
    

This is a capability Selenium doesn't have natively — you'd need a separate proxy tool like BrowserMob Proxy to intercept network requests in Selenium.

* * *

**Q20: Authentication strategies — storing and reusing auth state**

Repeated login flows are one of the biggest sources of test suite slowness. The solution is to authenticate once and reuse the session state.

Playwright's `storageState` approach:
    
    
    // auth.setup.ts — runs once before all tests
    import { test as setup } from '@playwright/test';
    
    setup('authenticate', async ({ page }) => {
        await page.goto('/login');
        await page.getByLabel('Email').fill(process.env.TEST_EMAIL!);
        await page.getByLabel('Password').fill(process.env.TEST_PASSWORD!);
        await page.getByRole('button', { name: 'Sign in' }).click();
        await page.waitForURL('/dashboard');
        // Save cookies and localStorage to file
        await page.context().storageState({ path: 'auth.json' });
    });
    
    // playwright.config.ts
    projects: [
        { name: 'setup', testMatch: /auth.setup.ts/ },
        {
            name: 'authenticated tests',
            use: { storageState: 'auth.json' }, // all tests start authenticated
            dependencies: ['setup']
        }
    ]
    
    
    

For token-based auth, inject the token directly into the browser's localStorage:
    
    
    await page.context().addInitScript(token => {
        localStorage.setItem('auth_token', token);
    }, process.env.TEST_TOKEN);
    
    
    

For Selenium, the equivalent approach uses browser cookies: log in once, extract all cookies using `driver.manage().getCookies()`, and restore them in subsequent tests using `driver.manage().addCookie()`. Less robust than Playwright's `storageState` because it only persists cookies, not localStorage or sessionStorage.

* * *

### Quick Reference Cheat Sheet

Topic| Playwright| Selenium  
---|---|---  
Protocol| CDP over WebSocket| HTTP WebDriver protocol  
Auto-waiting| Built-in, event-driven| Manual explicit waits required  
Browser processes| One process, multiple contexts| One process per driver instance  
Flaky test resistance| High — auto-wait prevents most timing issues| Lower — manual waits introduce timing gaps  
Network interception| Native `page.route()`| Needs external proxy (BrowserMob)  
Parallel execution| Worker-based, built-in sharding| Requires Grid infrastructure  
Shadow DOM| Auto-piercing| Manual JS executor required  
iframes| `frameLocator()`| `driver.switchTo().frame()`  
Auth state| `storageState` (cookies + localStorage)| Cookie injection only  
Debugging| Trace Viewer, UI Mode, Inspector| Screenshot on failure, logs  
Language support| JS/TS, Python, Java, C#| Java, Python, C#, Ruby, JS — broadest  
Mobile| Experimental| Via Appium (separate tool)  
  
* * *

Would you like me to now run a mock technical interview on Playwright or Selenium — asking questions one at a time and giving you feedback on your answers? Or go deeper on any specific topic?
