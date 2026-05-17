---
title: "Aptitude-100-C# Automation"
date: 2026-04-18 21:08:27 
author: shajeebhameed
layout: page
slug: aptitude-100-c-automation
status: publish
---

## C# Playwright & Selenium — Automation Aptitude

## 100 Interview Questions & Answers (STAR Format)

**Senior QA Automation Engineer**

> All answers follow the **STAR format** : **S** ituation → **T** ask → **A** ction → **R** esult

* * *

## TABLE OF CONTENTS

  1. [C# Playwright — Core & Setup (Q1–Q10)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-1>)
  2. [C# Playwright — Locators & Selectors (Q11–Q20)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-2>)
  3. [C# Playwright — Advanced Interactions (Q21–Q30)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-3>)
  4. [C# Playwright — API Testing (Q31–Q36)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-4>)
  5. [C# Selenium WebDriver — Core & Setup (Q37–Q46)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-5>)
  6. [C# Selenium — Advanced Techniques (Q47–Q56)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-6>)
  7. [Framework Design — Page Object Model (Q57–Q64)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-7>)
  8. [BDD with SpecFlow (Q65–Q72)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-8>)
  9. [CI/CD, Parallel Execution & Reporting (Q73–Q82)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-9>)
  10. [Debugging, Stability & Best Practices (Q83–Q100)](<https://claude.ai/chat/6261b5ec-b3cc-4546-965c-dfc02ce04a57#section-10>)



* * *

<a name="section-1"></a>

## Section 1: C# Playwright — Core & Setup (Q1–Q10)

* * *

### Q1. How did you set up a Playwright automation project in C# from scratch?

**Situation:** On the AXA Insurance project, there was no existing automation infrastructure. The application was a complex .NET-based insurance portal requiring end-to-end automation.

**Task:** I needed to initialise a production-ready Playwright C# project that integrated with our CI pipeline from day one.

**Action:** I created a new .NET project using `dotnet new nunit` and installed the `Microsoft.Playwright` NuGet package. I ran `playwright install` to download browser binaries and structured the project with a base test class for shared browser setup:
    
    
    // Base test class
    [TestFixture]
    public class BaseTest
    {
        protected IPlaywright Playwright;
        protected IBrowser Browser;
        protected IPage Page;
    
        [SetUp]
        public async Task Setup()
        {
            Playwright = await Microsoft.Playwright.Playwright.CreateAsync();
            Browser = await Playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions
            {
                Headless = true,
                SlowMo = 50
            });
            Page = await Browser.NewPageAsync();
        }
    
        [TearDown]
        public async Task TearDown()
        {
            await Browser.CloseAsync();
            Playwright.Dispose();
        }
    }
    
    

**Result:** The project was CI-ready from sprint one. Headless execution in Jenkins worked without modification, and the base class pattern meant all 300+ test classes inherited consistent browser lifecycle management with zero duplication.

* * *

### Q2. How have you configured Playwright to run tests across multiple browsers in C#?

**Situation:** On the AXA portal project, the client required validation across Chromium, Firefox, and WebKit, as UK and Ireland policyholders used a variety of browsers.

**Task:** I needed to execute the same test suite across three browser engines without duplicating test code.

**Action:** I used NUnit's `[TestFixtureSource]` to parameterise browser type at the fixture level:
    
    
    [TestFixtureSource(typeof(BrowserProvider))]
    public class CrossBrowserTests : BaseTest
    {
        private readonly string _browserType;
    
        public CrossBrowserTests(string browserType)
        {
            _browserType = browserType;
        }
    
        protected override async Task<IBrowser> LaunchBrowserAsync()
        {
            return _browserType switch
            {
                "firefox"  => await Playwright.Firefox.LaunchAsync(new() { Headless = true }),
                "webkit"   => await Playwright.Webkit.LaunchAsync(new() { Headless = true }),
                _          => await Playwright.Chromium.LaunchAsync(new() { Headless = true })
            };
        }
    }
    
    public class BrowserProvider : IEnumerable
    {
        public IEnumerator GetEnumerator()
        {
            yield return "chromium";
            yield return "firefox";
            yield return "webkit";
        }
    }
    
    

**Result:** The same 300+ tests ran across all three browsers in parallel on Jenkins using a matrix strategy. Cross-browser defects — particularly WebKit-specific rendering issues — were caught automatically that would have been missed with single-browser execution.

* * *

### Q3. How did you implement browser context isolation between tests in Playwright C#?

**Situation:** On the AXA Insurance project, tests were sharing browser state (cookies, localStorage) between runs, causing intermittent failures due to authentication bleed-over between scenarios.

**Task:** I needed to ensure every test ran in a completely isolated browser context to eliminate state interference.

**Action:** I implemented per-test `IBrowserContext` creation, which gives each test a clean browser session:
    
    
    [SetUp]
    public async Task Setup()
    {
        _context = await Browser.NewContextAsync(new BrowserNewContextOptions
        {
            ViewportSize = new ViewportSize { Width = 1920, Height = 1080 },
            Locale = "en-GB",
            TimezoneId = "Europe/London",
            RecordVideoDir = "videos/",
            RecordVideoSize = new RecordVideoSize { Width = 1280, Height = 720 }
        });
        Page = await _context.NewPageAsync();
    }
    
    [TearDown]
    public async Task TearDown()
    {
        await _context.CloseAsync(); // Automatically saves video
    }
    
    

**Result:** Test isolation was achieved completely. Authentication bleed-over disappeared, and the video recording per context made debugging failures significantly faster by providing a visual replay of exactly what happened in each test run.

* * *

### Q4. How did you handle async/await patterns correctly in C# Playwright tests?

**Situation:** When junior engineers joined the AXA project, they were mixing synchronous and asynchronous calls, causing deadlocks and unpredictable test behaviour.

**Task:** I needed to establish correct async/await patterns and help the team understand why they mattered in Playwright's async model.

**Action:** I established framework conventions and code review standards around async usage:
    
    
    // WRONG - Blocking async with .Result causes deadlocks
    public void WrongApproach()
    {
        var element = Page.QuerySelectorAsync(".policy-number").Result; // DEADLOCK RISK
    }
    
    // CORRECT - Full async chain
    public async Task CorrectApproach()
    {
        // All Playwright operations are async - always await them
        await Page.GotoAsync("https://portal.axa.ie/policy");
        await Page.FillAsync("#policyNumber", "POL-123456");
        await Page.ClickAsync("#searchBtn");
        
        // Wait for navigation after action
        await Page.WaitForURLAsync("**/policy/details**");
        
        // Assertion with await
        await Expect(Page.Locator(".policy-status")).ToHaveTextAsync("Active");
    }
    
    

**Result:** After establishing these conventions and running a team workshop on async/await in the context of Playwright, deadlock-related failures disappeared entirely. Code reviews enforced the pattern consistently, and new engineers adopted correct async habits from their first PR.

* * *

### Q5. How did you implement global test configuration and settings management in Playwright C#?

**Situation:** On the AXA project, hardcoded URLs and credentials in test classes made it impossible to run the same suite against different environments (UAT, Preprod, Production) without code changes.

**Task:** I needed a flexible configuration system that allowed environment switching via build parameters.

**Action:** I implemented a configuration class backed by `appsettings.json` with environment variable overrides:
    
    
    // appsettings.json
    {
      "TestSettings": {
        "BaseUrl": "https://uat.portal.axa.ie",
        "Username": "test.user@axa.ie",
        "DefaultTimeout": 30000,
        "Headless": true
      }
    }
    
    // TestSettings.cs
    public class TestSettings
    {
        public string BaseUrl { get; set; }
        public string Username { get; set; }
        public int DefaultTimeout { get; set; }
        public bool Headless { get; set; }
    
        public static TestSettings Load()
        {
            var config = new ConfigurationBuilder()
                .AddJsonFile("appsettings.json")
                .AddEnvironmentVariables() // Overrides JSON for CI
                .Build();
            return config.GetSection("TestSettings").Get<TestSettings>();
        }
    }
    
    // Usage in base class
    protected static readonly TestSettings Settings = TestSettings.Load();
    
    

**Result:** Environment switching required only a single environment variable change — no code modifications. Jenkins pipelines could target UAT, Preprod, or Production environments using the same artifact, which was essential for the In-Sprint automation model across multiple environments.

* * *

### Q6. How did you implement Playwright's Trace Viewer for debugging failures in CI?

**Situation:** On the AXA project, some test failures occurred only in the CI headless environment and were impossible to reproduce locally, making debugging extremely time-consuming.

**Task:** I needed a debugging mechanism that provided full test context even for CI-only failures without requiring interactive debugging.

**Action:** I implemented Playwright trace capture on failure using a custom test observer:
    
    
    [SetUp]
    public async Task StartTracing()
    {
        await _context.Tracing.StartAsync(new TracingStartOptions
        {
            Screenshots = true,
            Snapshots = true,  // Captures DOM at each step
            Sources = true     // Includes source code
        });
    }
    
    [TearDown]
    public async Task StopTracing()
    {
        var testFailed = TestContext.CurrentContext.Result.Outcome.Status 
                         == NUnit.Framework.Interfaces.TestStatus.Failed;
        
        if (testFailed)
        {
            var tracePath = $"traces/{TestContext.CurrentContext.Test.Name}.zip";
            await _context.Tracing.StopAsync(new TracingStopOptions { Path = tracePath });
            TestContext.AddTestAttachment(tracePath, "Playwright Trace");
        }
        else
        {
            await _context.Tracing.StopAsync(); // Discard if passed
        }
    }
    
    

**Result:** CI-only failures could be replayed step-by-step in Playwright's Trace Viewer, showing DOM snapshots, network requests, and console logs at every action. Debugging time for CI-specific failures dropped from hours to minutes.

* * *

### Q7. How did you manage Playwright browser installation in a CI/CD pipeline?

**Situation:** On the AXA project, the Jenkins build agents didn't have Playwright browser binaries pre-installed, and every build was failing at the browser launch step.

**Task:** I needed to ensure browser binaries were available reliably in the CI environment without requiring manual agent setup.

**Action:** I added browser installation as a step in the Jenkins pipeline and used Playwright's environment variable to control browser caching:
    
    
    # Jenkinsfile stage
    stage('Install Playwright Browsers') {
        steps {
            sh 'dotnet build'
            sh 'pwsh bin/Debug/net8.0/playwright.ps1 install --with-deps chromium'
        }
    }
    
    
    
    
    // In project file - ensure playwright CLI is available
    <ItemGroup>
      <PackageReference Include="Microsoft.Playwright" Version="1.44.0" />
      <PackageReference Include="Microsoft.Playwright.NUnit" Version="1.44.0" />
    </ItemGroup>
    
    
    
    
    <!-- .runsettings for CI -->
    <RunSettings>
      <Playwright>
        <BrowserName>chromium</BrowserName>
        <LaunchOptions>
          <Headless>true</Headless>
        </LaunchOptions>
      </Playwright>
    </RunSettings>
    
    

**Result:** Browser installation became a reliable, repeatable CI step. Using `--with-deps` ensured all OS-level dependencies were installed on Linux agents, eliminating the browser launch failures completely.

* * *

### Q8. How did you implement Playwright's built-in assertions (Expect API) in C#?

**Situation:** On the AXA project, junior engineers were using basic `Assert.AreEqual` for UI element assertions, which didn't have auto-waiting and produced cryptic failure messages.

**Task:** I needed to migrate the team to Playwright's Expect API for reliable, auto-waiting assertions with meaningful failure output.

**Action:** I established Playwright's `Expect` as the team standard and documented the key patterns:
    
    
    // OLD approach - no auto-waiting, brittle
    Assert.AreEqual("Active", await Page.InnerTextAsync(".policy-status"));
    
    // CORRECT - Playwright Expect with auto-waiting and retries
    // Element visibility
    await Expect(Page.Locator(".policy-status")).ToBeVisibleAsync();
    
    // Text content
    await Expect(Page.Locator(".policy-status")).ToHaveTextAsync("Active");
    
    // Attribute value
    await Expect(Page.Locator("#claimRef")).ToHaveValueAsync("CLM-2024-001");
    
    // URL assertion
    await Expect(Page).ToHaveURLAsync("**/claims/confirmation**");
    
    // Element count
    await Expect(Page.Locator(".policy-row")).ToHaveCountAsync(5);
    
    // Element state
    await Expect(Page.Locator("#submitBtn")).ToBeEnabledAsync();
    await Expect(Page.Locator(".error-message")).ToBeHiddenAsync();
    
    // Custom timeout override
    await Expect(Page.Locator(".report-table"))
        .ToBeVisibleAsync(new LocatorAssertionsToBeVisibleOptions { Timeout = 60000 });
    
    

**Result:** Test stability improved immediately after migration. The Expect API's built-in retry mechanism (polling until assertion passes or timeout) eliminated timing-related false failures. Failure messages became detailed and actionable, significantly reducing debugging time.

* * *

### Q9. How did you implement screenshot capture on test failure in C# Playwright?

**Situation:** On the AXA project, when a test failed in CI, the team had no visual evidence of what the UI looked like at the point of failure, making diagnosis very difficult.

**Task:** I needed automatic screenshot capture triggered on test failure without manual intervention.

**Action:** I implemented screenshot capture in the test teardown using NUnit's test context:
    
    
    [TearDown]
    public async Task CaptureOnFailure()
    {
        if (TestContext.CurrentContext.Result.Outcome.Status == 
            NUnit.Framework.Interfaces.TestStatus.Failed)
        {
            // Full page screenshot
            var screenshotPath = Path.Combine(
                "screenshots",
                $"{TestContext.CurrentContext.Test.Name}_{DateTime.Now:yyyyMMdd_HHmmss}.png"
            );
            
            await Page.ScreenshotAsync(new PageScreenshotOptions
            {
                Path = screenshotPath,
                FullPage = true   // Captures scrollable content
            });
            
            // Attach to NUnit report
            TestContext.AddTestAttachment(screenshotPath, "Failure Screenshot");
            
            // Also capture element-level screenshot if locator is known
            // await Page.Locator(".error-container").ScreenshotAsync(new() { Path = "error.png" });
        }
    }
    
    

**Result:** Every CI failure had an immediately available screenshot attached to the NUnit report. The team's average time-to-diagnose for UI failures dropped significantly, and stakeholders reviewing test reports had visual context for every failure.

* * *

### Q10. How did you implement reusable page navigation helpers in Playwright C#?

**Situation:** On the AXA project, each test was independently navigating to its starting page with repeated navigation code spread across hundreds of test files.

**Task:** I needed to centralise navigation logic and make it reusable across the entire test suite.

**Action:** I created a `NavigationHelper` class integrated with the base page object:
    
    
    public class NavigationHelper
    {
        private readonly IPage _page;
        private readonly TestSettings _settings;
    
        public NavigationHelper(IPage page, TestSettings settings)
        {
            _page = page;
            _settings = settings;
        }
    
        public async Task GoToAsync(string relativePath = "")
        {
            var url = $"{_settings.BaseUrl}/{relativePath.TrimStart('/')}";
            await _page.GotoAsync(url, new PageGotoOptions
            {
                WaitUntil = WaitUntilState.NetworkIdle,
                Timeout = _settings.DefaultTimeout
            });
        }
    
        public async Task GoToPolicyAsync(string policyNumber)
        {
            await GoToAsync($"policy/{policyNumber}");
            await Expect(_page.Locator(".policy-header")).ToBeVisibleAsync();
        }
    
        public async Task GoToClaimsAsync()
        {
            await GoToAsync("claims");
            await Expect(_page.Locator("h1")).ToHaveTextAsync("Claims");
        }
    }
    
    

**Result:** Navigation was centralised and consistent across all 300+ tests. When the application routing changed, updates were made in one place only. The `WaitUntil = NetworkIdle` strategy also eliminated premature interactions with pages that hadn't fully loaded.

* * *

<a name="section-2"></a>

## Section 2: C# Playwright — Locators & Selectors (Q11–Q20)

* * *

### Q11. What is Playwright's Locator API and how is it different from ElementHandle?

**Situation:** On the AXA project, junior engineers were using `Page.QuerySelectorAsync()` which returned `IElementHandle` — a snapshot of an element that could become stale.

**Task:** I needed to migrate the team to Playwright's Locator API and explain the key differences.

**Action:** I ran a team session demonstrating the difference with a practical example:
    
    
    // ElementHandle approach - STALE REFERENCE RISK
    IElementHandle element = await Page.QuerySelectorAsync(".policy-status");
    await Page.ReloadAsync(); // DOM re-renders
    await element.ClickAsync(); // ERROR: element is stale
    
    // Locator approach - ALWAYS FRESH, AUTO-RETRYING
    ILocator locator = Page.Locator(".policy-status");
    await Page.ReloadAsync(); // DOM re-renders
    await locator.ClickAsync(); // WORKS: locates element fresh each time
    
    // Locator chains (preferred for complex structures)
    ILocator policyRow = Page.Locator(".policy-table")
                             .Locator("tr")
                             .Filter(new LocatorFilterOptions { HasText = "POL-2024-001" });
    
    await policyRow.Locator(".status-cell").ClickAsync();
    
    // Locator with nth
    ILocator firstPolicy = Page.Locator(".policy-row").First;
    ILocator lastPolicy  = Page.Locator(".policy-row").Last;
    ILocator thirdPolicy = Page.Locator(".policy-row").Nth(2);
    
    

**Result:** After migration to the Locator API, stale element exceptions disappeared completely. The auto-retrying behaviour of Locators also eliminated most timing-related explicit waits, improving both stability and test clarity.

* * *

### Q12. How did you implement role-based and accessibility locators in Playwright C#?

**Situation:** On the AXA project, CSS and XPath locators were breaking frequently as the front-end team refactored styling and class names during rapid development cycles.

**Task:** I needed locator strategies that were resilient to CSS/styling changes and aligned with accessibility standards.

**Action:** I introduced Playwright's built-in ARIA/role-based locators as the primary strategy:
    
    
    // By ARIA role (most resilient to styling changes)
    ILocator submitBtn    = Page.GetByRole(AriaRole.Button, new() { Name = "Submit Claim" });
    ILocator policyInput  = Page.GetByRole(AriaRole.Textbox, new() { Name = "Policy Number" });
    ILocator claimsNav    = Page.GetByRole(AriaRole.Link, new() { Name = "Claims" });
    ILocator errorAlert   = Page.GetByRole(AriaRole.Alert);
    
    // By label (form fields)
    ILocator emailField   = Page.GetByLabel("Email Address");
    ILocator dobField     = Page.GetByLabel("Date of Birth");
    
    // By placeholder
    ILocator searchBox    = Page.GetByPlaceholder("Search policies...");
    
    // By text
    ILocator heading      = Page.GetByText("Your Claims", new() { Exact = true });
    
    // By test ID (agreed with dev team)
    ILocator claimStatus  = Page.GetByTestId("claim-status-badge");
    
    await submitBtn.ClickAsync();
    await Expect(claimStatus).ToHaveTextAsync("Submitted");
    
    

**Result:** Locator breakage from CSS refactoring dropped to near zero. Role-based locators also had the side benefit of flagging missing ARIA attributes in the application, improving overall accessibility compliance.

* * *

### Q13. How did you handle dynamic elements and lists in Playwright C#?

**Situation:** On the AXA policy management portal, the policy list was dynamically rendered with an unknown number of rows, and tests needed to interact with specific rows based on policy data rather than fixed indices.

**Task:** I needed to implement dynamic element handling that could filter and interact with list items based on content.

**Action:** I used Playwright's `Filter` method for content-based row selection:
    
    
    public class PolicyListPage : BasePage
    {
        private ILocator PolicyRows => Page.Locator(".policy-table tbody tr");
    
        // Filter by text content
        public ILocator GetPolicyRow(string policyNumber)
            => PolicyRows.Filter(new LocatorFilterOptions { HasText = policyNumber });
    
        // Get cell within filtered row
        public async Task<string> GetPolicyStatus(string policyNumber)
        {
            var row = GetPolicyRow(policyNumber);
            await Expect(row).ToBeVisibleAsync();
            return await row.Locator(".status-badge").InnerTextAsync();
        }
    
        // Iterate all rows
        public async Task<List<string>> GetAllPolicyNumbers()
        {
            var rows = PolicyRows;
            var count = await rows.CountAsync();
            var policyNumbers = new List<string>();
    
            for (int i = 0; i < count; i++)
            {
                var policyNum = await rows.Nth(i).Locator(".policy-number").InnerTextAsync();
                policyNumbers.Add(policyNum);
            }
            return policyNumbers;
        }
    
        // Assert specific count
        public async Task AssertPolicyCount(int expected)
            => await Expect(PolicyRows).ToHaveCountAsync(expected);
    }
    
    

**Result:** Dynamic list interactions worked reliably regardless of list length or order. The content-based filtering eliminated the fragile index-based approach junior engineers had been using, and tests remained stable even when server-side sorting changed the display order.

* * *

### Q14. How did you implement self-healing locators in C# Playwright?

**Situation:** On the AXA Insurance portal, the front-end team used a component library that occasionally changed element structures during upgrades, causing locator failures even when the functional behaviour was unchanged.

**Task:** I needed a fallback locator mechanism that could recover from single-locator failures without test execution interruption.

**Action:** I implemented a self-healing locator utility with primary, secondary, and tertiary strategies:
    
    
    public static class SelfHealingLocator
    {
        public static async Task<ILocator> FindAsync(IPage page, 
            string primarySelector,
            string secondarySelector = null,
            string tertiarySelector = null)
        {
            // Try primary
            var primary = page.Locator(primarySelector);
            if (await primary.CountAsync() > 0)
                return primary;
    
            Console.WriteLine($"[WARN] Primary locator '{primarySelector}' not found. Trying secondary.");
    
            // Try secondary
            if (secondarySelector != null)
            {
                var secondary = page.Locator(secondarySelector);
                if (await secondary.CountAsync() > 0)
                    return secondary;
            }
    
            // Try tertiary
            if (tertiarySelector != null)
            {
                var tertiary = page.Locator(tertiarySelector);
                if (await tertiary.CountAsync() > 0)
                    return tertiary;
            }
    
            throw new InvalidOperationException(
                $"All locator strategies failed. Primary: '{primarySelector}'");
        }
    }
    
    // Usage
    var claimBtn = await SelfHealingLocator.FindAsync(
        page,
        primarySelector: "[data-testid='submit-claim']",
        secondarySelector: "button:has-text('Submit Claim')",
        tertiarySelector: ".claim-submit-btn"
    );
    await claimBtn.ClickAsync();
    
    

**Result:** The self-healing strategy reduced locator-related test failures by over 80% during UI upgrade cycles. Warning logs flagged which primary locators were degraded, giving the team advance notice to update selectors proactively without blocking test execution.

* * *

### Q15. How did you work with iframes and shadow DOM in Playwright C#?

**Situation:** On the AXA portal, a third-party payment processing widget was embedded in an iframe, and the standard Playwright locators couldn't reach inside it.

**Task:** I needed to interact with elements inside the payment iframe as reliably as with the main document.

**Action:** I implemented iframe frame locator support:
    
    
    public class PaymentPage : BasePage
    {
        // FrameLocator for iframe interaction
        private IFrameLocator PaymentFrame 
            => Page.FrameLocator("#payment-widget-iframe");
    
        public async Task EnterCardDetails(string cardNumber, string expiry, string cvv)
        {
            // All interactions scoped to the iframe context
            await PaymentFrame.Locator("#card-number").FillAsync(cardNumber);
            await PaymentFrame.Locator("#expiry-date").FillAsync(expiry);
            await PaymentFrame.Locator("#cvv").FillAsync(cvv);
        }
    
        public async Task ConfirmPayment()
        {
            await PaymentFrame.Locator("[data-testid='pay-now-btn']").ClickAsync();
            
            // Return to main frame for confirmation
            await Expect(Page.Locator(".payment-confirmation")).ToBeVisibleAsync();
        }
    }
    
    // Shadow DOM - pierce:// prefix (Playwright pierces shadow DOM automatically)
    var shadowElement = Page.Locator("my-custom-component >> .inner-element");
    // OR use the explicit pierce
    var explicit = Page.Locator("pierce/.inner-element");
    
    

**Result:** Payment flow automation worked reliably through the third-party iframe. The FrameLocator API handled the iframe context switching transparently, and the automation covered the full end-to-end payment journey without any manual workarounds.

* * *

### Q16. How did you locate and interact with dropdown and select elements in Playwright C#?

**Situation:** On the AXA portal, policy type and coverage level dropdowns were a mix of native HTML `<select>` elements and custom JavaScript-driven dropdowns, requiring different interaction strategies.

**Task:** I needed a reliable approach for both native and custom dropdowns within the same page object.

**Action:** I implemented handlers for both types:
    
    
    public class PolicyFormPage : BasePage
    {
        // Native <select> element
        public async Task SelectPolicyTypeAsync(string value)
        {
            await Page.Locator("#policy-type-select").SelectOptionAsync(value);
            // Can also select by label or index
            // await locator.SelectOptionAsync(new SelectOptionValue { Label = "Comprehensive" });
            // await locator.SelectOptionAsync(new SelectOptionValue { Index = 2 });
        }
    
        // Custom dropdown (not <select>)
        public async Task SelectCoverageLevel(string level)
        {
            // Open the dropdown
            await Page.Locator(".coverage-dropdown-trigger").ClickAsync();
            
            // Wait for options to appear
            await Page.Locator(".dropdown-options").WaitForAsync(
                new LocatorWaitForOptions { State = WaitForSelectorState.Visible });
            
            // Click the specific option
            await Page.Locator(".dropdown-options li")
                      .Filter(new() { HasText = level })
                      .ClickAsync();
            
            // Verify selection applied
            await Expect(Page.Locator(".coverage-dropdown-trigger"))
                  .ToHaveTextAsync(level);
        }
        
        // Multi-select
        public async Task SelectMultiplePolicies(string[] values)
        {
            await Page.Locator("#policy-multi-select")
                      .SelectOptionAsync(values);
        }
    }
    
    

**Result:** Both native and custom dropdown interactions worked reliably. The pattern was documented in the team's coding standards, ensuring consistency across all 300+ test cases that involved form interactions.

* * *

### Q17. How did you handle file upload scenarios in Playwright C#?

**Situation:** On the AXA claims portal, policyholders could attach supporting documents to claims — receipts, medical reports, and photos. This file upload workflow needed automation.

**Task:** I needed to automate file upload interactions for the FNOL claims journey automation suite.

**Action:** I implemented file upload using Playwright's `SetInputFilesAsync`:
    
    
    public class ClaimsDocumentPage : BasePage
    {
        // Simple single file upload
        public async Task UploadSupportingDocument(string filePath)
        {
            await Page.Locator("#document-upload-input").SetInputFilesAsync(filePath);
            await Expect(Page.Locator(".upload-success-message")).ToBeVisibleAsync();
        }
    
        // Multiple file upload
        public async Task UploadMultipleDocuments(string[] filePaths)
        {
            await Page.Locator("#document-upload-input").SetInputFilesAsync(filePaths);
            await Expect(Page.Locator(".upload-count"))
                  .ToHaveTextAsync($"{filePaths.Length} files uploaded");
        }
    
        // Handle upload triggered by drag-and-drop zone (not <input>)
        public async Task UploadViaDragDropZone(string filePath)
        {
            // Create a fake file input for drag-and-drop zones
            await Page.Locator("#drop-zone").SetInputFilesAsync(filePath);
        }
    
        // Upload from test data directory
        public async Task UploadTestDocument(string fileName)
        {
            var testDataPath = Path.Combine("TestData", "Documents", fileName);
            await UploadSupportingDocument(testDataPath);
        }
    }
    
    

**Result:** File upload automation worked reliably for all document types in the FNOL journey. Test files were maintained in a versioned `TestData/Documents` directory, making the automation self-contained and reproducible across environments.

* * *

### Q18. How did you handle date picker interactions in Playwright C#?

**Situation:** On the AXA Insurance portal, claim dates and policy effective dates were entered through a custom JavaScript date picker that didn't respond to simple input typing.

**Task:** I needed to automate date selection reliably without relying on brittle date picker UI interactions.

**Action:** I implemented two approaches — direct input override and full UI interaction — depending on the date picker type:
    
    
    public class DatePickerHelper
    {
        private readonly IPage _page;
        
        // Approach 1: Override via JavaScript (most reliable)
        public async Task SetDateViaJS(string selector, DateTime date)
        {
            await _page.EvalOnSelectorAsync(selector,
                @"(el, dateStr) => {
                    var nativeInputValueSetter = Object.getOwnPropertyDescriptor(
                        window.HTMLInputElement.prototype, 'value').set;
                    nativeInputValueSetter.call(el, dateStr);
                    el.dispatchEvent(new Event('input', { bubbles: true }));
                    el.dispatchEvent(new Event('change', { bubbles: true }));
                }",
                date.ToString("yyyy-MM-dd"));
        }
    
        // Approach 2: UI-based date picker navigation
        public async Task SelectDateFromPicker(DateTime targetDate)
        {
            await _page.Locator(".datepicker-trigger").ClickAsync();
            
            // Navigate to correct month/year
            while (true)
            {
                var displayedMonth = await _page.Locator(".datepicker-month").InnerTextAsync();
                if (displayedMonth == targetDate.ToString("MMMM yyyy")) break;
                await _page.Locator(".datepicker-next").ClickAsync();
            }
            
            // Select the day
            await _page.Locator(".datepicker-day")
                       .Filter(new() { HasText = targetDate.Day.ToString() })
                       .First.ClickAsync();
        }
    }
    
    

**Result:** Date interactions were reliable across both standard HTML date inputs and the custom date picker component. The JavaScript approach was used for simple form dates, and the UI approach for the visual calendar where business rule validation was tied to the picker interaction sequence.

* * *

### Q19. How did you implement locator filtering with `HasText` and `Has` options in C# Playwright?

**Situation:** On the AXA claims portal, multiple claim records were displayed in a table and tests needed to target a specific claim by both policy number and claim status simultaneously.

**Task:** I needed complex filtering logic to identify unique rows from multi-column data without using fragile XPath.

**Action:** I used Playwright's chainable Filter options for multi-condition row selection:
    
    
    public class ClaimsListPage : BasePage
    {
        private ILocator ClaimRows => Page.Locator(".claims-table tbody tr");
    
        // Filter by text in specific column
        public ILocator GetClaimByReference(string claimRef)
            => ClaimRows.Filter(new() { HasText = claimRef });
    
        // Compound filter: row containing specific text AND having a child with specific text
        public ILocator GetClaimByRefAndStatus(string claimRef, string status)
            => ClaimRows
                .Filter(new() { HasText = claimRef })
                .Filter(new() { 
                    Has = Page.Locator(".status-badge", new() { HasTextString = status }) 
                });
    
        // Assert specific claim exists with expected status
        public async Task AssertClaimStatus(string claimRef, string expectedStatus)
        {
            var claimRow = GetClaimByRefAndStatus(claimRef, expectedStatus);
            await Expect(claimRow).ToBeVisibleAsync();
            await Expect(claimRow).ToHaveCountAsync(1);
        }
    
        // Get all claims in a specific status
        public async Task<int> GetClaimCountByStatus(string status)
        {
            var filtered = ClaimRows.Filter(new() { 
                Has = Page.Locator($".status-badge:has-text('{status}')") 
            });
            return await filtered.CountAsync();
        }
    }
    
    

**Result:** Complex row targeting worked reliably without any XPath, making tests readable by non-technical reviewers. The compound filter approach replaced brittle coordinate-based or index-based selection that junior engineers had originally implemented.

* * *

### Q20. How did you handle elements inside pop-ups and modals in Playwright C#?

**Situation:** On the AXA portal, policy cancellation and claim submission triggered confirmation modal dialogs that required interaction before the underlying page responded.

**Task:** I needed to automate modal interactions including confirmation, dismissal, and data entry within modals.

**Action:** I implemented modal interaction patterns in a reusable helper:
    
    
    public class ModalHelper
    {
        private readonly IPage _page;
        private ILocator Modal => _page.Locator(".modal-overlay");
        private ILocator ModalTitle => Modal.Locator(".modal-header h2");
        private ILocator ConfirmBtn => Modal.Locator("[data-testid='modal-confirm']");
        private ILocator CancelBtn  => Modal.Locator("[data-testid='modal-cancel']");
    
        public async Task WaitForModalAsync(string expectedTitle = null)
        {
            await Expect(Modal).ToBeVisibleAsync();
            if (expectedTitle != null)
                await Expect(ModalTitle).ToHaveTextAsync(expectedTitle);
        }
    
        public async Task ConfirmAsync()
        {
            await WaitForModalAsync();
            await ConfirmBtn.ClickAsync();
            await Expect(Modal).ToBeHiddenAsync(); // Modal should close after confirm
        }
    
        public async Task DismissAsync()
        {
            await WaitForModalAsync();
            await CancelBtn.ClickAsync();
            await Expect(Modal).ToBeHiddenAsync();
        }
    
        // Handle browser-level dialogs (alert/confirm/prompt)
        public void AcceptNextDialog()
        {
            _page.Dialog += async (_, dialog) => await dialog.AcceptAsync();
        }
    
        public void DismissNextDialog()
        {
            _page.Dialog += async (_, dialog) => await dialog.DismissAsync();
        }
    }
    
    

**Result:** Modal interactions were encapsulated in a single reusable helper used across 40+ test cases on AXA. Both UI-rendered modals and browser-native dialogs were handled consistently, and the auto-wait for modal visibility/invisibility eliminated timing issues around modal animations.

* * *

<a name="section-3"></a>

## Section 3: C# Playwright — Advanced Interactions (Q21–Q30)

* * *

### Q21. How did you implement network interception and mocking in Playwright C#?

**Situation:** On the AXA project, some test scenarios required testing specific error states from backend APIs — such as pricing engine timeouts — that were difficult to trigger reliably in the test environment.

**Task:** I needed to intercept network requests and return controlled mock responses to reliably test error handling scenarios.

**Action:** I used Playwright's `RouteAsync` for request interception:
    
    
    [Test]
    public async Task Should_Display_Error_When_PricingEngine_Times_Out()
    {
        // Intercept and mock the pricing API
        await Page.RouteAsync("**/api/pricing/calculate**", async route =>
        {
            await route.FulfillAsync(new RouteFulfillOptions
            {
                Status = 504,
                ContentType = "application/json",
                Body = JsonSerializer.Serialize(new { error = "Gateway Timeout", code = "PRICING_TIMEOUT" })
            });
        });
    
        await Page.GotoAsync($"{Settings.BaseUrl}/policy/quote");
        await Page.Locator("#calculateBtn").ClickAsync();
    
        // Assert error handling
        await Expect(Page.Locator(".pricing-error")).ToBeVisibleAsync();
        await Expect(Page.Locator(".pricing-error")).ToContainTextAsync("pricing service is temporarily unavailable");
    
        // Unroute after test
        await Page.UnrouteAsync("**/api/pricing/calculate**");
    }
    
    // Abort specific requests (simulate no-network for assets)
    await Page.RouteAsync("**/*.{png,jpg,css}", route => route.AbortAsync());
    
    // Modify request headers
    await Page.RouteAsync("**/api/**", async route =>
    {
        var headers = route.Request.Headers.ToDictionary(h => h.Key, h => h.Value);
        headers["X-Test-Flag"] = "automation";
        await route.ContinueAsync(new RouteContinueOptions { Headers = headers });
    });
    
    

**Result:** Error state scenarios that previously required production incidents or backend team cooperation could be tested reliably and repeatedly in automation. Coverage for error handling paths increased significantly, and a critical timeout handling defect was caught before UAT.

* * *

### Q22. How did you handle authentication flows in Playwright C# without repeating login in every test?

**Situation:** On the AXA portal, every test started with a login flow that took 4-6 seconds, adding significant cumulative overhead to the 300+ test suite.

**Task:** I needed to eliminate repeated login overhead while maintaining security isolation between tests.

**Action:** I implemented Playwright's storage state for session reuse:
    
    
    // GlobalSetup.cs - runs once before all tests
    public class GlobalSetup
    {
        [OneTimeSetUp]
        public async Task SetupAuth()
        {
            using var playwright = await Playwright.CreateAsync();
            var browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });
            var context = await browser.NewContextAsync();
            var page = await context.NewPageAsync();
    
            // Perform login once
            await page.GotoAsync($"{Settings.BaseUrl}/login");
            await page.FillAsync("#username", Settings.Username);
            await page.FillAsync("#password", Settings.Password);
            await page.ClickAsync("#loginBtn");
            await page.WaitForURLAsync("**/dashboard**");
    
            // Save authenticated state to file
            await context.StorageStateAsync(new() { Path = "auth-state.json" });
            await browser.CloseAsync();
        }
    }
    
    // BaseTest.cs - reuse auth state per test context
    [SetUp]
    public async Task Setup()
    {
        _context = await Browser.NewContextAsync(new BrowserNewContextOptions
        {
            StorageStatePath = "auth-state.json" // Pre-authenticated
        });
        Page = await _context.NewPageAsync();
    }
    
    

**Result:** Test suite execution time dropped by approximately 25% by eliminating 300+ repeated login flows. Each test still got a fresh browser context (isolated cookies/storage), but started pre-authenticated, maintaining both isolation and efficiency.

* * *

### Q23. How did you implement keyboard and mouse actions in Playwright C#?

**Situation:** On the AXA portal, some policy comparison features used keyboard shortcuts for accessibility, and a drag-and-drop interface was used for document ordering in the claims journey.

**Task:** I needed to automate keyboard shortcut interactions and drag-and-drop operations as part of the regression suite.

**Action:** I implemented both using Playwright's keyboard and mouse APIs:
    
    
    public class AdvancedInteractionHelper
    {
        private readonly IPage _page;
    
        // Keyboard shortcuts
        public async Task SelectAllAndCopy()
        {
            await _page.Keyboard.PressAsync("Control+A");
            await _page.Keyboard.PressAsync("Control+C");
        }
    
        public async Task TypeWithTabNavigation(Dictionary<string, string> fieldValues)
        {
            foreach (var (value, _) in fieldValues)
            {
                await _page.Keyboard.TypeAsync(value);
                await _page.Keyboard.PressAsync("Tab");
            }
        }
    
        // Drag and drop
        public async Task DragDocumentToPosition(string sourceName, string targetName)
        {
            var source = _page.Locator(".document-item").Filter(new() { HasText = sourceName });
            var target = _page.Locator(".document-item").Filter(new() { HasText = targetName });
    
            await source.DragToAsync(target);
            
            // Verify order changed
            var items = await _page.Locator(".document-item").AllInnerTextsAsync();
            var sourceIndex = Array.IndexOf(items.ToArray(), sourceName);
            var targetIndex = Array.IndexOf(items.ToArray(), targetName);
            Assert.That(sourceIndex, Is.LessThan(targetIndex));
        }
    
        // Hover to reveal tooltip
        public async Task<string> GetTooltipText(ILocator triggerElement)
        {
            await triggerElement.HoverAsync();
            var tooltip = _page.Locator(".tooltip-content");
            await Expect(tooltip).ToBeVisibleAsync();
            return await tooltip.InnerTextAsync();
        }
    }
    
    

**Result:** Keyboard navigation and drag-and-drop automation covered accessibility-critical workflows that manual testers had been skipping due to complexity. Several keyboard shortcut defects and a drag-and-drop ordering bug were found through automated tests.

* * *

### Q24. How did you handle multiple tabs and new windows in Playwright C#?

**Situation:** On the AXA portal, clicking certain policy document links opened new browser tabs for PDF viewing, and the test needed to validate content in the new tab before returning to the main page.

**Task:** I needed to automate cross-tab interactions without losing the original page context.

**Action:** I used Playwright's page event handling for new tab detection:
    
    
    public async Task ValidatePolicyDocumentInNewTab()
    {
        // Listen for new page before triggering the action
        var newPageTask = Page.Context.WaitForPageAsync();
        
        // Click that opens a new tab
        await Page.Locator(".view-policy-doc-btn").ClickAsync();
        
        // Await the new page
        var newTab = await newPageTask;
        await newTab.WaitForLoadStateAsync(LoadState.NetworkIdle);
        
        // Interact with new tab
        await Expect(newTab.Locator(".pdf-viewer")).ToBeVisibleAsync();
        var documentTitle = await newTab.TitleAsync();
        Assert.That(documentTitle, Does.Contain("Policy Schedule"));
        
        // Close new tab and return to original
        await newTab.CloseAsync();
        
        // Original page context still active
        await Expect(Page.Locator(".policy-details-panel")).ToBeVisibleAsync();
    }
    
    

**Result:** New tab interactions were automated reliably without requiring manual test execution. The `WaitForPageAsync` pattern eliminated race conditions that occurred when trying to get the new page after it had already opened.

* * *

### Q25. How did you implement JavaScript execution within Playwright C# tests?

**Situation:** On the AXA portal, some UI state changes (like scrolling to a lazy-loaded element or resetting a complex form component) couldn't be achieved through standard Playwright interactions.

**Task:** I needed to use JavaScript execution as a controlled escape hatch for scenarios where the Playwright API was insufficient.

**Action:** I implemented targeted JavaScript execution with typed return values:
    
    
    public class JavaScriptHelper
    {
        private readonly IPage _page;
    
        // Scroll element into view
        public async Task ScrollToElement(string selector)
        {
            await _page.EvalOnSelectorAsync(selector, 
                "el => el.scrollIntoView({ behavior: 'smooth', block: 'center' })");
        }
    
        // Get computed style (not accessible via standard API)
        public async Task<string> GetComputedColor(string selector)
        {
            return await _page.EvalOnSelectorAsync<string>(selector,
                "el => window.getComputedStyle(el).getPropertyValue('color')");
        }
    
        // Manipulate localStorage
        public async Task SetLocalStorageItem(string key, string value)
        {
            await _page.EvaluateAsync($"localStorage.setItem('{key}', '{value}')");
        }
    
        // Force element property (for read-only inputs)
        public async Task SetReadOnlyInputValue(string selector, string value)
        {
            await _page.EvalOnSelectorAsync(selector,
                @"(el, val) => {
                    var setter = Object.getOwnPropertyDescriptor(
                        HTMLInputElement.prototype, 'value').set;
                    setter.call(el, val);
                    el.dispatchEvent(new Event('change', { bubbles: true }));
                }", value);
        }
    
        // Check if element is in viewport
        public async Task<bool> IsInViewport(string selector)
        {
            return await _page.EvalOnSelectorAsync<bool>(selector,
                @"el => {
                    const rect = el.getBoundingClientRect();
                    return rect.top >= 0 && rect.bottom <= window.innerHeight;
                }");
        }
    }
    
    

**Result:** JavaScript execution enabled automation of edge cases that the standard Playwright API didn't support. The typed return values from `EvalOnSelectorAsync<T>` provided type safety, and wrapping all JS execution in helper methods kept test code clean and auditable.

* * *

### Q26. How did you handle page load states and network idle conditions in Playwright C#?

**Situation:** On the AXA claims portal, some pages made multiple sequential AJAX calls after the initial page load, and tests were interacting with elements before the page was fully ready.

**Task:** I needed to implement reliable page readiness checks that accounted for asynchronous content loading.

**Action:** I implemented a layered readiness strategy:
    
    
    public class PageReadinessHelper
    {
        private readonly IPage _page;
    
        // Wait for complete network idle (all requests settled)
        public async Task WaitForNetworkIdle()
        {
            await _page.WaitForLoadStateAsync(LoadState.NetworkIdle);
        }
    
        // Wait for specific API response
        public async Task WaitForApiResponse(string urlPattern)
        {
            await _page.WaitForResponseAsync(response => 
                response.Url.Contains(urlPattern) && response.Status == 200);
        }
    
        // Custom readiness: wait for loading spinner to disappear
        public async Task WaitForLoadingSpinner()
        {
            var spinner = _page.Locator(".loading-spinner");
            
            // If spinner is present, wait for it to disappear
            if (await spinner.CountAsync() > 0)
            {
                await Expect(spinner).ToBeHiddenAsync(
                    new LocatorAssertionsToBeHiddenOptions { Timeout = 30000 });
            }
        }
    
        // Comprehensive page ready check
        public async Task WaitForPageReady()
        {
            await _page.WaitForLoadStateAsync(LoadState.DOMContentLoaded);
            await WaitForLoadingSpinner();
            await _page.WaitForFunctionAsync("() => document.readyState === 'complete'");
        }
    }
    
    

**Result:** Premature interaction failures were eliminated. The layered approach handled the three distinct page states on the AXA portal — initial HTML load, spinner while AJAX calls ran, and final rendered state — correctly.

* * *

### Q27. How did you implement soft assertions for validating multiple conditions in Playwright C#?

**Situation:** On the AXA claims portal, the FNOL (First Notification of Loss) confirmation page had 15+ data fields that needed validation. A single assertion failure was stopping the test and hiding other failures.

**Task:** I needed to validate all fields in a single test run and report all failures together rather than stopping at the first failure.

**Action:** I implemented soft assertions using a custom collector pattern:
    
    
    public class SoftAssertions
    {
        private readonly List<string> _failures = new();
        private readonly IPage _page;
    
        public async Task AssertTextAsync(ILocator locator, string expected, string fieldName)
        {
            try
            {
                await Expect(locator).ToHaveTextAsync(expected);
            }
            catch (Exception ex)
            {
                _failures.Add($"[{fieldName}] Expected: '{expected}' | Error: {ex.Message}");
            }
        }
    
        public async Task AssertVisibleAsync(ILocator locator, string fieldName)
        {
            try
            {
                await Expect(locator).ToBeVisibleAsync();
            }
            catch
            {
                _failures.Add($"[{fieldName}] Expected to be visible but was not found.");
            }
        }
    
        public void AssertAll()
        {
            if (_failures.Any())
            {
                Assert.Fail($"Soft assertion failures:\n{string.Join("\n", _failures)}");
            }
        }
    }
    
    // Usage in FNOL confirmation test
    [Test]
    public async Task Should_Display_All_FNOL_Confirmation_Details()
    {
        var soft = new SoftAssertions(Page);
        await soft.AssertTextAsync(Page.Locator("#claim-ref"), "CLM-2024-001", "Claim Reference");
        await soft.AssertTextAsync(Page.Locator("#policy-number"), "POL-12345", "Policy Number");
        await soft.AssertVisibleAsync(Page.Locator(".claim-confirmation-badge"), "Confirmation Badge");
        // ... 12 more assertions
        soft.AssertAll(); // Reports ALL failures at once
    }
    
    

**Result:** All 15+ field validations now reported in a single test run. Instead of discovering failures one-at-a-time across multiple runs, the team could see the full picture of what was broken in a single execution.

* * *

### Q28. How did you implement data-driven testing in C# Playwright with NUnit?

**Situation:** On the AXA project, the same claim submission flow needed to be validated with 40+ different policy/claim data combinations representing different insurance product lines.

**Task:** I needed to implement data-driven test execution without duplicating test logic.

**Action:** I used NUnit's `[TestCaseSource]` with JSON-based test data:
    
    
    // TestData/ClaimScenarios.json
    [
      { "PolicyType": "Comprehensive", "ClaimType": "Theft", "ExpectedOutcome": "Approved" },
      { "PolicyType": "ThirdParty",    "ClaimType": "Damage", "ExpectedOutcome": "Pending Review" },
      { "PolicyType": "Comprehensive", "ClaimType": "FireDamage", "ExpectedOutcome": "Approved" }
    ]
    
    // ClaimTestDataProvider.cs
    public class ClaimTestDataProvider
    {
        public static IEnumerable<ClaimScenario> GetScenarios()
        {
            var json = File.ReadAllText("TestData/ClaimScenarios.json");
            return JsonSerializer.Deserialize<List<ClaimScenario>>(json);
        }
    }
    
    // Test class
    public class ClaimSubmissionTests : BaseTest
    {
        [Test, TestCaseSource(typeof(ClaimTestDataProvider), nameof(ClaimTestDataProvider.GetScenarios))]
        public async Task Should_Process_Claim_Correctly(ClaimScenario scenario)
        {
            var claimsPage = new ClaimsPage(Page);
            await claimsPage.SubmitClaim(scenario.PolicyType, scenario.ClaimType);
            await Expect(Page.Locator(".claim-outcome"))
                  .ToHaveTextAsync(scenario.ExpectedOutcome);
        }
    }
    
    

**Result:** 40+ claim scenarios ran from a single test method. Business analysts maintained the JSON test data independently of the framework code — exactly the model established for AXA's 400+ policy permutations. Adding new scenarios required no code changes.

* * *

### Q29. How did you implement Playwright's Request/Response interception for security testing?

**Situation:** On the AXA portal, part of the QA brief included validating that sensitive API endpoints enforced authentication — i.e., unauthenticated requests were rejected with 401.

**Task:** I needed to test API security behaviours without separate API testing tooling, within the Playwright framework.

**Action:** I used Playwright's request interception and response inspection:
    
    
    [Test]
    public async Task Should_Return_401_For_Unauthenticated_API_Calls()
    {
        // Create unauthenticated context (no storage state)
        var unauthContext = await Browser.NewContextAsync();
        var unauthPage = await unauthContext.NewPageAsync();
        
        // Capture API responses
        var responses = new List<(string Url, int Status)>();
        unauthPage.Response += (_, response) =>
        {
            if (response.Url.Contains("/api/"))
                responses.Add((response.Url, response.Status));
        };
    
        // Attempt to access protected resource
        await unauthPage.GotoAsync($"{Settings.BaseUrl}/dashboard");
    
        // Assert all API calls were rejected
        Assert.That(responses.All(r => r.Status == 401 || r.Status == 302),
            $"Unexpected authenticated API responses: {string.Join(", ", responses.Where(r => r.Status != 401))}");
    
        await unauthContext.CloseAsync();
    }
    
    

**Result:** Security validation was integrated into the standard regression suite. Two endpoints that were missing authentication middleware were caught through this automated approach before the release, preventing a potential data exposure vulnerability.

* * *

### Q30. How did you implement visual regression testing with Playwright C#?

**Situation:** On the AXA portal, the front-end team's CSS changes occasionally introduced unexpected visual regressions — layout shifts, colour changes — that functional assertions didn't catch.

**Task:** I needed to add visual regression testing to the suite to catch UI-level visual defects alongside functional ones.

**Action:** I implemented screenshot comparison using Playwright's snapshot API:
    
    
    // playwright.config in .runsettings
    // Set update snapshots mode for first run, then compare
    
    [Test]
    public async Task Policy_Dashboard_Should_Match_Baseline()
    {
        await Page.GotoAsync($"{Settings.BaseUrl}/dashboard");
        await Page.WaitForLoadStateAsync(LoadState.NetworkIdle);
        
        // Hide dynamic content (timestamps, notifications) before comparison
        await Page.Locator(".live-notification").EvaluateAllAsync(
            "elements => elements.forEach(e => e.style.visibility = 'hidden')");
    
        // Take and compare screenshot
        // First run: saves baseline. Subsequent runs: compares.
        await Expect(Page).ToHaveScreenshotAsync("dashboard-baseline.png", 
            new PageAssertionsToHaveScreenshotOptions
            {
                MaxDiffPixelRatio = 0.02f, // Allow 2% pixel difference
                FullPage = true
            });
    }
    
    // Element-level visual comparison
    [Test]  
    public async Task PolicyCard_Should_Match_Design()
    {
        var policyCard = Page.Locator(".policy-card").First;
        await Expect(policyCard).ToHaveScreenshotAsync("policy-card-baseline.png");
    }
    
    

**Result:** Visual regressions were caught automatically on three separate occasions when the CSS framework was upgraded. The `MaxDiffPixelRatio` threshold prevented false positives from minor anti-aliasing differences while still catching meaningful visual defects.

* * *

<a name="section-4"></a>

## Section 4: C# Playwright — API Testing (Q31–Q36)

* * *

### Q31. How did you integrate API testing directly into your Playwright C# framework?

**Situation:** On the AXA project, I needed to validate REST API endpoints for pricing engines and payment systems alongside UI automation flows within the same test execution pipeline.

**Task:** I needed to build an API testing layer that shared the same authentication context as the UI tests.

**Action:** I used Playwright's `APIRequestContext` for integrated API testing:
    
    
    public class ApiTestBase : BaseTest
    {
        protected IAPIRequestContext ApiContext;
    
        [SetUp]
        public async Task SetupApiContext()
        {
            ApiContext = await Playwright.APIRequest.NewContextAsync(new()
            {
                BaseURL = Settings.ApiBaseUrl,
                StorageStatePath = "auth-state.json", // Reuse UI auth tokens
                ExtraHTTPHeaders = new Dictionary<string, string>
                {
                    { "Content-Type", "application/json" },
                    { "X-Api-Version", "v2" }
                }
            });
        }
    
        [TearDown]
        public async Task DisposeApiContext() => await ApiContext.DisposeAsync();
    }
    
    // API test using shared context
    public class PricingApiTests : ApiTestBase
    {
        [Test]
        public async Task Should_Return_Valid_Quote_For_Comprehensive_Policy()
        {
            var requestBody = new { PolicyType = "Comprehensive", CoverageLevel = "Premium" };
            
            var response = await ApiContext.PostAsync("/api/pricing/calculate",
                new APIRequestContextOptions
                {
                    DataObject = requestBody
                });
    
            Assert.That(response.Status, Is.EqualTo(200));
            
            var body = await response.JsonAsync<PricingResponse>();
            Assert.That(body.QuoteAmount, Is.GreaterThan(0));
            Assert.That(body.Currency, Is.EqualTo("EUR"));
            Assert.That(body.ValidUntil, Is.GreaterThan(DateTime.UtcNow));
        }
    }
    
    

**Result:** API and UI tests ran in the same pipeline with shared authentication, eliminating the need for separate API testing tools for regression scenarios. The integrated approach caught API contract changes the same day they were introduced, before they caused UI test failures.

* * *

### Q32. How did you implement API response schema validation in C# Playwright?

**Situation:** On the J&J Salesforce project, Salesforce API response schemas changed during platform upgrades, causing downstream UI failures that were hard to trace to their API root cause.

**Task:** I needed to implement schema validation that would catch breaking API changes before they propagated to UI test failures.

**Action:** I implemented JSON schema validation within the API layer:
    
    
    using Newtonsoft.Json.Schema;
    using Newtonsoft.Json.Linq;
    
    public class SchemaValidator
    {
        public static async Task ValidateResponseSchema(IAPIResponse response, string schemaFile)
        {
            var schemaJson = await File.ReadAllTextAsync($"TestData/Schemas/{schemaFile}");
            var schema = JSchema.Parse(schemaJson);
    
            var responseBody = await response.TextAsync();
            var jsonResponse = JToken.Parse(responseBody);
    
            var isValid = jsonResponse.IsValid(schema, out IList<string> errors);
            
            if (!isValid)
            {
                Assert.Fail($"API response schema validation failed:\n{string.Join("\n", errors)}");
            }
        }
    }
    
    // Schema file: TestData/Schemas/case-response.json
    // {
    //   "type": "object",
    //   "required": ["caseId", "status", "createdDate"],
    //   "properties": {
    //     "caseId": { "type": "string" },
    //     "status": { "type": "string", "enum": ["Open", "InProgress", "Resolved"] }
    //   }
    // }
    
    [Test]
    public async Task Case_API_Response_Should_Match_Schema()
    {
        var response = await ApiContext.GetAsync("/api/cases/CASE-001");
        await SchemaValidator.ValidateResponseSchema(response, "case-response.json");
    }
    
    

**Result:** API schema validation caught three breaking changes introduced during Salesforce seasonal updates before they caused UI test failures. Schema violations were reported with specific field-level detail, making root cause analysis immediate.

* * *

### Q33. How did you implement end-to-end API + UI validation in a single Playwright C# test?

**Situation:** On AXA, the claims submission involved both UI interaction (form filling) and API verification (backend state confirmation). Testing each layer separately left gaps in end-to-end coverage.

**Task:** I needed to combine UI and API validation in a single cohesive test scenario.

**Action:** I implemented a combined test using both `IPage` and `IAPIRequestContext`:
    
    
    [Test]
    public async Task Claim_Submission_Should_Persist_Correctly_In_Backend()
    {
        // STEP 1: Submit claim via UI
        var claimsPage = new ClaimsPage(Page);
        var claimReference = await claimsPage.SubmitClaim(new ClaimData
        {
            PolicyNumber = "POL-12345",
            ClaimType = "Theft",
            IncidentDate = DateTime.Today.AddDays(-3),
            Description = "Vehicle theft - automated test scenario"
        });
    
        // Verify UI confirmation
        await Expect(Page.Locator(".confirmation-ref")).ToHaveTextAsync(claimReference);
    
        // STEP 2: Verify API reflects the same state
        var apiResponse = await ApiContext.GetAsync($"/api/claims/{claimReference}");
        Assert.That(apiResponse.Status, Is.EqualTo(200));
    
        var claim = await apiResponse.JsonAsync<ClaimResponse>();
        Assert.That(claim.Reference, Is.EqualTo(claimReference));
        Assert.That(claim.Status, Is.EqualTo("Submitted"));
        Assert.That(claim.PolicyNumber, Is.EqualTo("POL-12345"));
    
        // STEP 3: Verify database via API (indirect)
        var auditResponse = await ApiContext.GetAsync($"/api/claims/{claimReference}/audit");
        var audit = await auditResponse.JsonAsync<AuditResponse>();
        Assert.That(audit.Events.Any(e => e.Type == "CLAIM_SUBMITTED"), Is.True);
    }
    
    

**Result:** End-to-end validation caught a critical defect where the UI showed submission success but the backend status was stuck in "Processing" due to a message queue issue. This was invisible to UI-only testing and API-only testing but visible to the combined approach.

* * *

### Q34. How did you handle authentication tokens in API tests within Playwright C#?

**Situation:** On the AXA project, the API used JWT bearer tokens that expired after 30 minutes. Long-running API test suites were failing mid-run due to token expiry.

**Task:** I needed to implement automatic token refresh to ensure API tests remained authenticated throughout long execution sessions.

**Action:** I implemented a token manager with automatic refresh:
    
    
    public class TokenManager
    {
        private string _currentToken;
        private DateTime _tokenExpiry;
        private readonly IAPIRequestContext _authContext;
    
        public async Task<string> GetValidTokenAsync()
        {
            if (_currentToken == null || DateTime.UtcNow >= _tokenExpiry.AddMinutes(-2))
            {
                await RefreshTokenAsync();
            }
            return _currentToken;
        }
    
        private async Task RefreshTokenAsync()
        {
            var response = await _authContext.PostAsync("/auth/token", new()
            {
                DataObject = new { 
                    Username = Settings.Username, 
                    Password = Settings.Password,
                    GrantType = "client_credentials"
                }
            });
            
            var tokenResponse = await response.JsonAsync<TokenResponse>();
            _currentToken = tokenResponse.AccessToken;
            _tokenExpiry = DateTime.UtcNow.AddSeconds(tokenResponse.ExpiresIn);
        }
    }
    
    // In ApiTestBase - inject fresh token per request
    protected async Task<IAPIResponse> GetWithAuth(string endpoint)
    {
        var token = await _tokenManager.GetValidTokenAsync();
        return await ApiContext.GetAsync(endpoint, new()
        {
            Headers = new Dictionary<string, string> { { "Authorization", $"Bearer {token}" } }
        });
    }
    
    

**Result:** Long-running API test suites completed without authentication failures. The 2-minute buffer before expiry ensured no test started with an about-to-expire token, eliminating mid-test auth failures entirely.

* * *

### Q35. How did you test paginated API responses in C# Playwright?

**Situation:** On the Google project, BigQuery result sets and geospatial data APIs returned paginated responses. Validating that all pages contained correct data was a manual bottleneck.

**Task:** I needed to automate pagination traversal and aggregate data validation across all pages.

**Action:** I implemented a recursive pagination handler:
    
    
    public class PaginationHelper
    {
        public static async Task<List<T>> GetAllPagesAsync<T>(
            IAPIRequestContext context,
            string endpoint,
            Func<JsonElement, List<T>> dataExtractor,
            int pageSize = 100)
        {
            var allItems = new List<T>();
            int page = 1;
            bool hasMore = true;
    
            while (hasMore)
            {
                var response = await context.GetAsync(
                    $"{endpoint}?page={page}&pageSize={pageSize}");
                
                Assert.That(response.Status, Is.EqualTo(200), 
                    $"Pagination failed at page {page}");
                
                var json = await response.JsonAsync<JsonElement>();
                var items = dataExtractor(json);
                allItems.AddRange(items);
    
                // Check if more pages exist
                hasMore = json.TryGetProperty("hasNextPage", out var next) && next.GetBoolean();
                page++;
            }
    
            return allItems;
        }
    }
    
    // Usage
    var allDataPoints = await PaginationHelper.GetAllPagesAsync<GeospatialRecord>(
        ApiContext,
        "/api/geospatial/data",
        json => JsonSerializer.Deserialize<List<GeospatialRecord>>(
            json.GetProperty("data").GetRawText())
    );
    
    Assert.That(allDataPoints.Count, Is.GreaterThan(10000));
    Assert.That(allDataPoints.All(d => d.Latitude != 0 && d.Longitude != 0), Is.True);
    
    

**Result:** Full dataset validation across paginated APIs was automated for the first time on the Google project. Data integrity issues affecting specific page ranges — which had been completely invisible to single-page manual testing — were caught automatically.

* * *

### Q36. How did you implement contract testing in your API automation suite?

**Situation:** On the J&J Salesforce project, the automation team and the Salesforce integration team had ongoing disputes about whether API changes were backward-compatible. Each team had different assumptions about the contract.

**Task:** I needed to formalise API contracts and automate contract validation to eliminate ambiguity.

**Action:** I implemented consumer-driven contract validation within the Playwright API testing layer:
    
    
    public class ContractValidator
    {
        // Define the consumer's contract expectations
        public static readonly ApiContract CaseApiContract = new ApiContract
        {
            Endpoint = "/api/cases/{id}",
            Method = "GET",
            RequiredFields = new[] { "caseId", "status", "createdDate", "assignedTo" },
            StatusCodes = new[] { 200, 404 },
            ResponseTimeMax = 2000 // ms
        };
    
        public static async Task ValidateContract(IAPIResponse response, ApiContract contract)
        {
            // Status code
            Assert.That(contract.StatusCodes.Contains(response.Status), 
                $"Unexpected status: {response.Status}");
            
            // Response time
            // Measured via performance timing wrapper
            
            // Required fields
            var body = JsonDocument.Parse(await response.TextAsync());
            foreach (var field in contract.RequiredFields)
            {
                Assert.That(body.RootElement.TryGetProperty(field, out _),
                    $"Missing required field: '{field}'");
            }
        }
    }
    
    

**Result:** Contract validation tests were added to the CI pipeline and ran on every deployment. When the Salesforce team removed a field that the automation consumer expected, the contract test failed immediately and unambiguously, resolving the dispute with evidence rather than discussion.

* * *

<a name="section-5"></a>

## Section 5: C# Selenium WebDriver — Core & Setup (Q37–Q46)

* * *

### Q37. How did you set up a Selenium WebDriver project in C# from scratch?

**Situation:** On the Thomson Reuters Aumentum Tax project, I joined with no existing automation infrastructure and needed to establish a C# Selenium framework quickly.

**Task:** I needed to create a maintainable, extensible Selenium WebDriver framework in C# with WebDriverManager for driver handling.

**Action:** I set up the framework using NuGet packages and WebDriverManager for automatic driver management:
    
    
    // Install: Selenium.WebDriver, WebDriverManager, NUnit
    
    // BaseTest.cs
    [TestFixture]
    public class BaseTest
    {
        protected IWebDriver Driver;
    
        [SetUp]
        public void Setup()
        {
            // Auto-manage ChromeDriver version
            new DriverManager().SetUpDriver(new ChromeConfig());
    
            var options = new ChromeOptions();
            options.AddArgument("--headless=new");
            options.AddArgument("--no-sandbox");
            options.AddArgument("--disable-dev-shm-usage");
            options.AddArgument("--window-size=1920,1080");
            options.AddArgument("--disable-gpu");
    
            Driver = new ChromeDriver(options);
            Driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
            Driver.Manage().Timeouts().PageLoad = TimeSpan.FromSeconds(30);
        }
    
        [TearDown]
        public void TearDown()
        {
            Driver?.Quit();
            Driver?.Dispose();
        }
    }
    
    

**Result:** The framework was operational within a day. WebDriverManager's automatic driver version management eliminated the maintenance overhead of manually updating ChromeDriver to match Chrome browser versions — a common pain point on projects where browser updates occurred without warning.

* * *

### Q38. How did you implement the WebDriverWait and ExpectedConditions patterns in C#?

**Situation:** On the Thomson Reuters Tax project, implicit waits were causing tests to either wait too long for elements that appeared quickly, or fail on slow-loading elements when the implicit timeout was too short.

**Task:** I needed to replace implicit waits with explicit, condition-based waiting for reliable element interaction.

**Action:** I implemented WebDriverWait with custom expected conditions:
    
    
    public class WaitHelper
    {
        private readonly IWebDriver _driver;
        private readonly WebDriverWait _wait;
    
        public WaitHelper(IWebDriver driver, int timeoutSeconds = 30)
        {
            _driver = driver;
            _wait = new WebDriverWait(driver, TimeSpan.FromSeconds(timeoutSeconds));
            _wait.IgnoreExceptionTypes(typeof(NoSuchElementException), 
                                        typeof(StaleElementReferenceException));
        }
    
        public IWebElement WaitForElement(By locator)
            => _wait.Until(ExpectedConditions.ElementIsVisible(locator));
    
        public IWebElement WaitForClickable(By locator)
            => _wait.Until(ExpectedConditions.ElementToBeClickable(locator));
    
        public bool WaitForTextInElement(By locator, string text)
            => _wait.Until(ExpectedConditions.TextToBePresentInElementLocated(locator, text));
    
        public bool WaitForUrl(string urlFragment)
            => _wait.Until(ExpectedConditions.UrlContains(urlFragment));
    
        // Custom condition
        public IWebElement WaitForElementWithAttribute(By locator, string attribute, string value)
            => _wait.Until(driver =>
            {
                var element = driver.FindElement(locator);
                return element.GetAttribute(attribute) == value ? element : null;
            });
    
        // Alert handling
        public IAlert WaitForAlert()
            => _wait.Until(ExpectedConditions.AlertIsPresent());
    }
    
    

**Result:** Eliminating implicit waits in favour of explicit WebDriverWait reduced both false timeouts on fast elements and genuine failures on slow elements. Test execution time also decreased because waits were bounded to actual element states rather than fixed sleep periods.

* * *

### Q39. How did you handle StaleElementReferenceException in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, the application frequently re-rendered DOM sections after AJAX calls, causing `StaleElementReferenceException` on elements that had been located before the re-render.

**Task:** I needed to implement a reliable mechanism to recover from stale element references without cascading failures.

**Action:** I implemented a retry wrapper that re-locates the element on stale reference:
    
    
    public static class StaleElementHandler
    {
        public static T RetryOnStale<T>(Func<T> action, int maxRetries = 3)
        {
            for (int attempt = 0; attempt < maxRetries; attempt++)
            {
                try
                {
                    return action();
                }
                catch (StaleElementReferenceException)
                {
                    if (attempt == maxRetries - 1) throw;
                    Console.WriteLine($"[WARN] StaleElementReferenceException - retry {attempt + 1}/{maxRetries}");
                    Thread.Sleep(500); // Brief pause for DOM to settle
                }
            }
            throw new InvalidOperationException("Max retries exceeded for stale element.");
        }
    
        // Extension method on IWebDriver for stale-safe find
        public static IWebElement FindElementSafe(this IWebDriver driver, By locator, int maxRetries = 3)
        {
            return RetryOnStale(() => driver.FindElement(locator), maxRetries);
        }
    }
    
    // Usage in page object
    public void ClickSubmitButton()
    {
        StaleElementHandler.RetryOnStale(() =>
        {
            var btn = Driver.FindElement(By.Id("submitBtn"));
            btn.Click();
        });
    }
    
    

**Result:** `StaleElementReferenceException` failures dropped to near zero. The retry mechanism handled AJAX re-renders gracefully, and the warning log provided visibility when retries were occurring — useful for identifying pages with particularly aggressive re-rendering that warranted additional investigation.

* * *

### Q40. How did you implement cross-browser testing with Selenium in C#?

**Situation:** On the Cisco TelePresence Management Suite project, the enterprise portal needed to support Internet Explorer, Firefox, and Chrome across different customer environments.

**Task:** I needed a single test suite that could run against multiple browsers via a runtime parameter rather than separate test codebases.

**Action:** I implemented a factory pattern for browser instantiation:
    
    
    public enum BrowserType { Chrome, Firefox, Edge, InternetExplorer }
    
    public static class WebDriverFactory
    {
        public static IWebDriver Create(BrowserType browserType)
        {
            return browserType switch
            {
                BrowserType.Firefox => CreateFirefox(),
                BrowserType.Edge    => CreateEdge(),
                BrowserType.InternetExplorer => CreateIE(),
                _                   => CreateChrome() // Default
            };
        }
    
        private static IWebDriver CreateChrome()
        {
            new DriverManager().SetUpDriver(new ChromeConfig());
            var options = new ChromeOptions();
            options.AddArgument("--headless=new");
            return new ChromeDriver(options);
        }
    
        private static IWebDriver CreateFirefox()
        {
            new DriverManager().SetUpDriver(new FirefoxConfig());
            var options = new FirefoxOptions();
            options.AddArgument("--headless");
            return new FirefoxDriver(options);
        }
    
        private static IWebDriver CreateEdge()
        {
            new DriverManager().SetUpDriver(new EdgeConfig());
            return new EdgeDriver(new EdgeOptions());
        }
    }
    
    // In BaseTest - read from environment variable or config
    protected static BrowserType Browser = Enum.Parse<BrowserType>(
        Environment.GetEnvironmentVariable("BROWSER") ?? "Chrome");
    
    [SetUp]
    public void Setup() => Driver = WebDriverFactory.Create(Browser);
    
    

**Result:** The same test suite ran against Chrome, Firefox, and Edge via a single Jenkins environment variable. Browser-specific rendering defects on the Cisco TMS portal — particularly around IE's handling of video conference scheduling modals — were caught that would have been missed with single-browser execution.

* * *

### Q41. How did you manage WebDriver timeouts effectively in C# Selenium?

**Situation:** On the Refinitiv FX Trading project, different parts of the trading application had vastly different loading characteristics — some pages loaded in under a second, while complex trade blotter screens could take 8-10 seconds.

**Task:** I needed timeout strategies that were appropriate per page/action rather than a single global value.

**Action:** I implemented a context-specific timeout configuration:
    
    
    public class TimeoutManager
    {
        private readonly IWebDriver _driver;
        
        // Default timeouts
        private const int StandardTimeout = 10;
        private const int HeavyPageTimeout = 30;
        private const int ApiWaitTimeout = 60;
    
        public IDisposable TemporarilySetTimeout(int seconds)
        {
            var original = _driver.Manage().Timeouts().ImplicitWait;
            _driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(seconds);
            return new RestoreTimeout(_driver, original);
        }
        
        private class RestoreTimeout : IDisposable
        {
            private readonly IWebDriver _driver;
            private readonly TimeSpan _original;
    
            public RestoreTimeout(IWebDriver driver, TimeSpan original)
            {
                _driver = driver;
                _original = original;
            }
            
            public void Dispose()
            {
                _driver.Manage().Timeouts().ImplicitWait = _original;
            }
        }
    }
    
    // Usage - temporarily extend timeout for heavy page
    using (timeoutManager.TemporarilySetTimeout(30))
    {
        Driver.FindElement(By.Id("tradeBlotter")); // Needs longer wait
    }
    // Reverts to standard timeout automatically
    
    

**Result:** Test execution time was optimised by using tight timeouts for fast-loading elements and extended timeouts only where genuinely needed. False timeout failures on trade blotter screens were eliminated while not impacting suite speed for lighter pages.

* * *

### Q42. How did you implement element highlighting for debugging in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, debugging complex form interactions was difficult without visual indicators of which elements were being acted upon, particularly in headless runs.

**Task:** I needed a debugging utility that could visually highlight elements during test execution for non-headless debugging sessions.

**Action:** I implemented a JavaScript-based element highlighter toggle:
    
    
    public class ElementHighlighter
    {
        private readonly IJavaScriptExecutor _js;
        private readonly bool _debugMode;
    
        public ElementHighlighter(IWebDriver driver)
        {
            _js = (IJavaScriptExecutor)driver;
            _debugMode = bool.TryParse(
                Environment.GetEnvironmentVariable("DEBUG_HIGHLIGHT"), out var d) && d;
        }
    
        public IWebElement Highlight(IWebElement element, string color = "yellow")
        {
            if (!_debugMode) return element; // No-op in CI
    
            var original = element.GetCssValue("background-color");
            _js.ExecuteScript($"arguments[0].style.backgroundColor='{color}'", element);
            Thread.Sleep(300);
            _js.ExecuteScript($"arguments[0].style.backgroundColor='{original}'", element);
            return element;
        }
    }
    
    // Usage in page object
    public void ClickSubmit()
    {
        var btn = Driver.FindElement(By.Id("submitBtn"));
        _highlighter.Highlight(btn, "lightgreen");
        btn.Click();
    }
    
    

**Result:** The highlighter was invaluable for debugging complex multi-step form interactions locally. Setting `DEBUG_HIGHLIGHT=false` in CI meant zero performance impact in pipeline runs.

* * *

### Q43. How did you handle browser cookies and session management in C# Selenium?

**Situation:** On the Cisco TMS project, some tests needed to simulate different session states — fresh sessions, remembered sessions, and expired sessions — to cover session management scenarios.

**Task:** I needed to manage browser cookies programmatically within test scenarios.

**Action:** I implemented a cookie management utility:
    
    
    public class CookieHelper
    {
        private readonly IWebDriver _driver;
    
        // Add authentication cookie (bypass login for faster tests)
        public void AddAuthCookie(string token)
        {
            _driver.Navigate().GoToUrl(Settings.BaseUrl); // Must be on the domain first
            _driver.Manage().Cookies.AddCookie(new Cookie(
                "auth_token", token,
                domain: new Uri(Settings.BaseUrl).Host,
                path: "/",
                expiry: DateTime.UtcNow.AddHours(8)
            ));
        }
    
        // Simulate expired session
        public void ExpireAuthSession()
        {
            _driver.Manage().Cookies.DeleteCookieNamed("auth_token");
            _driver.Manage().Cookies.DeleteCookieNamed("session_id");
        }
    
        // Get all current cookies
        public IReadOnlyCollection<Cookie> GetAllCookies()
            => _driver.Manage().Cookies.AllCookies;
    
        // Clear all cookies (clean state)
        public void ClearAllCookies()
            => _driver.Manage().Cookies.DeleteAllCookies();
    
        // Save cookie state for reuse
        public string SerializeCookies()
            => JsonSerializer.Serialize(GetAllCookies()
                .Select(c => new { c.Name, c.Value, c.Domain, c.Path }));
    }
    
    

**Result:** Session management scenarios — including session timeout handling and "remember me" functionality — were fully automated. Adding authentication cookies directly bypassed the login UI for applicable tests, reducing suite execution time.

* * *

### Q44. How did you implement JavaScript executor for complex interactions in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, some UI components didn't respond to standard Selenium `Click()` due to JavaScript event binding that intercepted browser-native events.

**Task:** I needed to trigger JavaScript events directly as an alternative to Selenium's native actions.

**Action:** I implemented targeted JavaScript execution helpers:
    
    
    public class JavaScriptExecutorHelper
    {
        private readonly IJavaScriptExecutor _js;
    
        // JS click for stubborn elements
        public void JsClick(IWebElement element)
        {
            _js.ExecuteScript("arguments[0].click();", element);
        }
    
        // Scroll element into view before interacting
        public void ScrollIntoView(IWebElement element)
        {
            _js.ExecuteScript("arguments[0].scrollIntoView({block: 'center'});", element);
        }
    
        // Set value on readonly or framework-controlled inputs
        public void SetValue(IWebElement element, string value)
        {
            _js.ExecuteScript($"arguments[0].value='{value}';", element);
            _js.ExecuteScript("arguments[0].dispatchEvent(new Event('change'));", element);
        }
    
        // Get inner text (handles hidden elements too)
        public string GetText(IWebElement element)
            => _js.ExecuteScript("return arguments[0].textContent;", element)?.ToString();
    
        // Trigger custom DOM events
        public void TriggerEvent(IWebElement element, string eventName)
        {
            _js.ExecuteScript(
                $"arguments[0].dispatchEvent(new Event('{eventName}', {{bubbles: true}}));",
                element);
        }
    }
    
    

**Result:** All previously unresponsive UI elements in the Thomson Reuters Tax application became reliably automatable. The JS-based interactions handled the complex JavaScript event model of the tax management system, enabling coverage of workflows that were previously considered "un-automatable."

* * *

### Q45. How did you implement screenshot capture in C# Selenium on test failure?

**Situation:** On multiple projects using Selenium — Cisco, Thomson Reuters, and AXA — failure diagnosis in CI was hampered by lack of visual evidence.

**Task:** I needed automatic screenshot capture on test failure integrated with the NUnit reporting pipeline.

**Action:** I implemented a standard teardown screenshot pattern:
    
    
    [TearDown]
    public void TearDown()
    {
        try
        {
            if (TestContext.CurrentContext.Result.Outcome.Status == 
                NUnit.Framework.Interfaces.TestStatus.Failed)
            {
                var screenshot = ((ITakesScreenshot)Driver).GetScreenshot();
                var fileName = $"screenshots/{TestContext.CurrentContext.Test.Name}_{DateTime.Now:HHmmss}.png";
                
                Directory.CreateDirectory("screenshots");
                screenshot.SaveAsFile(fileName);
                
                // Attach to NUnit report
                TestContext.AddTestAttachment(fileName, "Failure Screenshot");
                
                // Also log page source for DOM analysis
                var pageSourcePath = $"pagesource/{TestContext.CurrentContext.Test.Name}.html";
                Directory.CreateDirectory("pagesource");
                File.WriteAllText(pageSourcePath, Driver.PageSource);
                TestContext.AddTestAttachment(pageSourcePath, "Page Source");
            }
        }
        finally
        {
            Driver?.Quit();
        }
    }
    
    

**Result:** Every CI failure had an immediately accessible screenshot and page source attached to the NUnit results. The combined visual + DOM evidence reduced average debugging time and enabled remote diagnosis without access to the test environment.

* * *

### Q46. How did you implement retry logic for flaky tests in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, a subset of tests were intermittently failing due to timing issues in the AJAX-heavy application, causing noise in CI results.

**Task:** I needed controlled retry logic that distinguished genuine failures from transient environmental flakiness.

**Action:** I implemented a retry attribute for NUnit:
    
    
    // Custom retry attribute
    public class RetryAttribute : NUnitAttribute, IRepeatableAttribute
    {
        private readonly int _maxRetries;
        
        public RetryAttribute(int maxRetries = 3)
        {
            _maxRetries = maxRetries;
        }
        
        public TestCommand Wrap(TestCommand command)
            => new RetryCommand(command, _maxRetries);
    }
    
    public class RetryCommand : DelegatingTestCommand
    {
        private readonly int _maxRetries;
    
        public RetryCommand(TestCommand innerCommand, int maxRetries) 
            : base(innerCommand)
        {
            _maxRetries = maxRetries;
        }
    
        public override TestResult Execute(TestExecutionContext context)
        {
            for (int attempt = 1; attempt <= _maxRetries; attempt++)
            {
                context.CurrentResult = innerCommand.Execute(context);
                if (context.CurrentResult.ResultState != ResultState.Error &&
                    context.CurrentResult.ResultState != ResultState.Failure)
                    break;
    
                Console.WriteLine($"[RETRY] Attempt {attempt}/{_maxRetries} failed. Retrying...");
            }
            return context.CurrentResult;
        }
    }
    
    // Usage
    [Test, Retry(3)]
    public void Should_Load_TaxReport_Successfully()
    {
        // Test that occasionally fails due to slow rendering
    }
    
    

**Result:** Retry logic reduced flaky test noise in CI by absorbing genuine transient failures. The console logging provided visibility into how often retries were needed, identifying the highest-frequency retry tests as candidates for root cause investigation and permanent fixing.

* * *

<a name="section-6"></a>

## Section 6: C# Selenium — Advanced Techniques (Q47–Q56)

* * *

### Q47. How did you implement the Actions class for complex user interactions in C# Selenium?

**Situation:** On the Cisco TelePresence Management Suite project, the scheduling interface required drag-and-drop to set meeting times on a calendar grid, and right-click context menus were used for conference room management.

**Task:** I needed to automate complex mouse-based interactions that weren't achievable through standard `Click()` and `SendKeys()`.

**Action:** I implemented Actions chain-based interactions:
    
    
    public class AdvancedActionsHelper
    {
        private readonly IWebDriver _driver;
        private readonly Actions _actions;
    
        public AdvancedActionsHelper(IWebDriver driver)
        {
            _driver = driver;
            _actions = new Actions(driver);
        }
    
        // Drag and drop between elements
        public void DragAndDrop(IWebElement source, IWebElement target)
        {
            _actions.DragAndDrop(source, target).Perform();
        }
    
        // Drag by offset (for calendar time slots)
        public void DragByOffset(IWebElement source, int xOffset, int yOffset)
        {
            _actions.ClickAndHold(source)
                    .MoveByOffset(xOffset, yOffset)
                    .Release()
                    .Perform();
        }
    
        // Right-click context menu
        public void RightClick(IWebElement element)
        {
            _actions.ContextClick(element).Perform();
        }
    
        // Double-click
        public void DoubleClick(IWebElement element)
        {
            _actions.DoubleClick(element).Perform();
        }
    
        // Hover (tooltip trigger)
        public void Hover(IWebElement element)
        {
            _actions.MoveToElement(element).Perform();
        }
    
        // Multi-select with Ctrl+Click
        public void MultiSelect(IEnumerable<IWebElement> elements)
        {
            _actions.KeyDown(Keys.Control);
            foreach (var element in elements)
                _actions.Click(element);
            _actions.KeyUp(Keys.Control).Perform();
        }
    }
    
    

**Result:** Complex mouse interactions including drag-to-schedule, right-click menus, and multi-element selection were fully automated on the Cisco TMS project. Regression of the scheduling interface — previously manual — was brought under automated coverage for the first time.

* * *

### Q48. How did you handle Select/dropdown elements in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, the application had both native HTML `<select>` dropdowns and JavaScript-powered custom dropdowns for tax period and jurisdiction selection.

**Task:** I needed a consistent pattern for both types that abstracted the complexity from test authors.

**Action:** I implemented a `SelectHelper` with strategies for each type:
    
    
    public class SelectHelper
    {
        private readonly IWebDriver _driver;
        private readonly WaitHelper _wait;
    
        // Native <select> using Selenium's SelectElement
        public void SelectByText(IWebElement selectElement, string text)
        {
            var select = new SelectElement(selectElement);
            select.SelectByText(text);
        }
    
        public void SelectByValue(IWebElement selectElement, string value)
        {
            new SelectElement(selectElement).SelectByValue(value);
        }
    
        public string GetSelectedOption(IWebElement selectElement)
            => new SelectElement(selectElement).SelectedOption.Text;
    
        public List<string> GetAllOptions(IWebElement selectElement)
            => new SelectElement(selectElement).Options.Select(o => o.Text).ToList();
    
        // Custom JS dropdown
        public void SelectCustomDropdown(By trigger, By optionsContainer, string optionText)
        {
            _wait.WaitForClickable(trigger).Click();
            _wait.WaitForElement(optionsContainer);
            
            var options = _driver.FindElements(By.CssSelector($"{optionsContainer} li"));
            var target = options.FirstOrDefault(o => o.Text.Trim() == optionText)
                         ?? throw new Exception($"Option '{optionText}' not found in dropdown.");
            target.Click();
        }
    
        // Multi-select native
        public void MultiSelectByTexts(IWebElement selectElement, string[] texts)
        {
            var select = new SelectElement(selectElement);
            foreach (var text in texts)
                select.SelectByText(text);
        }
    }
    
    

**Result:** All dropdown interactions — native and custom — were handled through a consistent interface. Test authors used the same API regardless of the underlying dropdown implementation, making tests readable and reducing the learning curve for new team members.

* * *

### Q49. How did you handle alerts and pop-up windows in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, some destructive actions (deleting tax records) triggered browser-native `alert()` and `confirm()` dialogs that blocked test execution.

**Task:** I needed to handle these browser dialogs reliably, including both acceptance and dismissal paths.

**Action:** I implemented a comprehensive alert handler:
    
    
    public class AlertHandler
    {
        private readonly IWebDriver _driver;
        private readonly WebDriverWait _wait;
    
        public IAlert WaitForAlert(int timeoutSeconds = 10)
        {
            _wait.Timeout = TimeSpan.FromSeconds(timeoutSeconds);
            return _wait.Until(ExpectedConditions.AlertIsPresent());
        }
    
        public void AcceptAlert()
        {
            var alert = WaitForAlert();
            Console.WriteLine($"[Alert] Accepting: '{alert.Text}'");
            alert.Accept();
        }
    
        public void DismissAlert()
        {
            var alert = WaitForAlert();
            Console.WriteLine($"[Alert] Dismissing: '{alert.Text}'");
            alert.Dismiss();
        }
    
        public string GetAlertText()
        {
            return WaitForAlert().Text;
        }
    
        public void TypeInPrompt(string text)
        {
            var alert = WaitForAlert();
            alert.SendKeys(text);
            alert.Accept();
        }
    
        // Safe dismiss - only if alert is present
        public bool TryDismissIfPresent()
        {
            try
            {
                _driver.SwitchTo().Alert().Dismiss();
                return true;
            }
            catch (NoAlertPresentException)
            {
                return false;
            }
        }
    }
    
    

**Result:** Browser dialog handling became reliable and logged transparently. Tests covering delete operations and bulk data changes on the Thomson Reuters Tax platform could now include accurate dialog handling, improving coverage of data management workflows.

* * *

### Q50. How did you implement parallel test execution in C# Selenium with NUnit?

**Situation:** On the Thomson Reuters Tax project, the full regression suite was taking 4+ hours. Parallel execution was the primary lever to reduce execution time without reducing coverage.

**Task:** I needed to implement parallel execution that was both safe (no shared state) and practical (minimal refactoring of existing tests).

**Action:** I configured NUnit parallel execution with thread-safe WebDriver management:
    
    
    // AssemblyInfo.cs

[assembly: Parallelizable(ParallelScope.All)] 

[assembly: LevelOfParallelism(4)] // 4 concurrent threads // BaseTest.cs - ThreadLocal for thread-safe driver isolation [TestFixture] public class BaseTest { private static readonly ThreadLocal<IWebDriver> _threadLocalDriver = new ThreadLocal<IWebDriver>(); protected IWebDriver Driver => _threadLocalDriver.Value; [SetUp] public void Setup() { _threadLocalDriver.Value = WebDriverFactory.Create(BrowserType.Chrome); Driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10); } [TearDown] public void TearDown() { Driver?.Quit(); _threadLocalDriver.Value = null; } } // Ensure test classes declare parallel scope [TestFixture] [Parallelizable(ParallelScope.Self)] public class PolicyTests : BaseTest { ... }

**Result:** Suite execution time reduced from 4+ hours to approximately 1.5 hours with 4 parallel threads on the CI agent. The ThreadLocal pattern ensured complete WebDriver isolation per thread, with zero interference between parallel tests.

* * *

### Q51. How did you implement test data setup and teardown in C# Selenium tests?

**Situation:** On the Cisco TMS project, tests required specific conference room configurations to exist in the system before test execution, and those configurations needed to be cleaned up afterward.

**Task:** I needed reliable test data lifecycle management that kept test environments clean and tests independent.

**Action:** I implemented API-based test data setup and teardown:
    
    
    public class TestDataManager
    {
        private readonly HttpClient _apiClient;
        private readonly List<string> _createdResourceIds = new();
    
        // Setup: create via API (faster than UI navigation)
        public async Task<string> CreateConferenceRoom(ConferenceRoomRequest request)
        {
            var response = await _apiClient.PostAsJsonAsync("/api/rooms", request);
            var result = await response.Content.ReadFromJsonAsync<RoomResponse>();
            _createdResourceIds.Add(result.RoomId);
            return result.RoomId;
        }
    
        // Teardown: clean up all created resources
        [TearDown]
        public async Task CleanupTestData()
        {
            foreach (var id in _createdResourceIds)
            {
                try
                {
                    await _apiClient.DeleteAsync($"/api/rooms/{id}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"[WARN] Cleanup failed for {id}: {ex.Message}");
                }
            }
            _createdResourceIds.Clear();
        }
    }
    
    

**Result:** Test data was always in a known state at test start, and environments were clean after execution. Using the API for setup (rather than UI navigation) reduced test setup time by 60%, making the overall suite faster without sacrificing data integrity.

* * *

### Q52. How did you implement a custom reporting mechanism in C# Selenium tests?

**Situation:** On the Thomson Reuters project, NUnit's default HTML report was insufficient for the stakeholders who needed failure trends, execution history, and module-level pass rates.

**Task:** I needed richer reporting integrated with our ReportPortal instance and enhanced NUnit output.

**Action:** I implemented a custom NUnit listener with ReportPortal integration:
    
    
    public class CustomTestListener : ITestListener
    {
        private readonly ReportPortalClient _rpClient;
        
        public void TestStarted(ITest test)
        {
            _rpClient.StartTest(new StartTestRequest
            {
                Name = test.Name,
                Description = test.FullName,
                Tags = test.Categories.Select(c => c.ToString()).ToList()
            });
        }
    
        public void TestFinished(ITestResult result)
        {
            _rpClient.FinishTest(new FinishTestRequest
            {
                Status = MapStatus(result.ResultState),
                EndTime = DateTime.UtcNow,
                Description = result.Message
            });
            
            if (result.ResultState == ResultState.Failure)
            {
                _rpClient.LogFailure(result.Message, result.StackTrace);
            }
        }
    
        private Status MapStatus(ResultState state) => state switch
        {
            ResultState.Success => Status.Passed,
            ResultState.Failure => Status.Failed,
            ResultState.Skipped => Status.Skipped,
            _ => Status.Interrupted
        };
    }
    
    

**Result:** ReportPortal received real-time test results with rich metadata. Stakeholders had a live dashboard of execution progress, and the failure trend analysis enabled the auto-rerun algorithm I developed to target the most frequently failing tests for priority investigation.

* * *

### Q53. How did you handle Windows-based authentication in C# Selenium?

**Situation:** On the Cisco TMS project, some admin sections of the portal required Windows Integrated Authentication (NTLM), which Selenium's standard navigation couldn't handle.

**Task:** I needed to automate authentication through the Windows authentication dialog.

**Action:** I embedded credentials in the URL and configured Chrome options for auth bypass:
    
    
    // Approach 1: Credentials in URL (HTTP basic auth)
    public void NavigateWithBasicAuth(string username, string password, string url)
    {
        var uri = new Uri(url);
        var authUrl = $"{uri.Scheme}://{username}:{password}@{uri.Host}{uri.PathAndQuery}";
        Driver.Navigate().GoToUrl(authUrl);
    }
    
    // Approach 2: Chrome option for Windows auth (NTLM/Kerberos)
    private static IWebDriver CreateChromeWithWindowsAuth()
    {
        var options = new ChromeOptions();
        options.AddArgument("--auth-server-whitelist=*.cisco.internal");
        options.AddArgument("--auth-negotiate-delegate-whitelist=*.cisco.internal");
        
        // Auto-login with current user credentials
        return new ChromeDriver(options);
    }
    
    // Approach 3: AutoIT for Windows dialog (legacy fallback)
    // [DllImport("user32.dll")]
    // static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
    // Used AutoIT compiled script for Windows auth popup interaction
    
    

**Result:** Windows authentication was handled automatically without manual intervention. The Chrome option approach worked for NTLM/Kerberos in the Cisco corporate environment, and the URL-embedded credential approach was used for basic HTTP authentication in dev environments.

* * *

### Q54. How did you implement table data extraction and validation in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, large tax calculation result tables needed to be validated for data accuracy — comparing expected vs. actual calculated values for multiple jurisdictions.

**Task:** I needed to extract and validate structured data from complex HTML tables efficiently.

**Action:** I implemented a reusable table extraction utility:
    
    
    public class TableHelper
    {
        private readonly IWebDriver _driver;
    
        public List<Dictionary<string, string>> ExtractTableData(IWebElement table)
        {
            var headers = table.FindElements(By.CssSelector("thead th"))
                               .Select(th => th.Text.Trim())
                               .ToList();
    
            var rows = table.FindElements(By.CssSelector("tbody tr"));
            var result = new List<Dictionary<string, string>>();
    
            foreach (var row in rows)
            {
                var cells = row.FindElements(By.TagName("td"))
                               .Select(td => td.Text.Trim())
                               .ToList();
                
                var rowData = headers.Zip(cells, (h, c) => new { h, c })
                                     .ToDictionary(x => x.h, x => x.c);
                result.Add(rowData);
            }
            return result;
        }
    
        public void AssertTableRow(List<Dictionary<string, string>> tableData, 
            string keyColumn, string keyValue, 
            Dictionary<string, string> expectedValues)
        {
            var row = tableData.FirstOrDefault(r => r[keyColumn] == keyValue)
                      ?? throw new AssertionException($"Row with {keyColumn}='{keyValue}' not found.");
    
            foreach (var (col, expected) in expectedValues)
            {
                Assert.That(row[col], Is.EqualTo(expected), 
                    $"Column '{col}' for {keyColumn}='{keyValue}': Expected '{expected}' but got '{row[col]}'");
            }
        }
    }
    
    

**Result:** Tax calculation table validation became fully automated. The generic table extraction utility was reused across multiple pages in the Thomson Reuters application, and column-specific assertions provided precise failure messages that immediately identified which calculation was incorrect.

* * *

### Q55. How did you automate file download validation in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, the system generated tax reports as downloadable Excel and PDF files. Validating that downloads completed and contained correct data was a manual bottleneck.

**Task:** I needed to automate the file download trigger and validate the downloaded file's existence and basic content.

**Action:** I configured Chrome to download to a known directory and implemented download wait logic:
    
    
    public class DownloadHelper
    {
        private readonly string _downloadDir;
        
        public DownloadHelper()
        {
            _downloadDir = Path.Combine(Path.GetTempPath(), "selenium_downloads");
            Directory.CreateDirectory(_downloadDir);
        }
    
        public ChromeOptions ConfigureDownloadDirectory(ChromeOptions options)
        {
            options.AddUserProfilePreference("download.default_directory", _downloadDir);
            options.AddUserProfilePreference("download.prompt_for_download", false);
            options.AddUserProfilePreference("plugins.always_open_pdf_externally", true);
            return options;
        }
    
        public async Task<string> WaitForDownloadAsync(string expectedFileName, int timeoutSeconds = 30)
        {
            var deadline = DateTime.Now.AddSeconds(timeoutSeconds);
            var expectedPath = Path.Combine(_downloadDir, expectedFileName);
    
            while (DateTime.Now < deadline)
            {
                if (File.Exists(expectedPath) && !IsFileLocked(expectedPath))
                    return expectedPath;
                
                await Task.Delay(500);
            }
            throw new TimeoutException($"Download '{expectedFileName}' not completed within {timeoutSeconds}s.");
        }
    
        private bool IsFileLocked(string path)
        {
            try { using var stream = File.OpenRead(path); return false; }
            catch (IOException) { return true; } // Still downloading
        }
    }
    
    

**Result:** Download validation was fully automated. Tests verified that tax reports were generated on schedule, named correctly, and accessible. Broken export functionality — where files were triggering download but contained only error HTML — was caught automatically through file size validation.

* * *

### Q56. How did you implement a Page Factory pattern in C# Selenium?

**Situation:** On the Cisco TMS project, junior engineers were duplicating element location code across page objects and test classes, creating maintenance overhead when locators changed.

**Task:** I needed to introduce a clean element declaration pattern that centralised locators in page objects.

**Action:** I implemented the Page Factory pattern using `[FindsBy]` attributes:
    
    
    using OpenQA.Selenium.Support.PageObjects;
    
    public class TmsSchedulingPage
    {
        [FindsBy(How = How.Id, Using = "newMeetingBtn")]
        private IWebElement NewMeetingButton;
    
        [FindsBy(How = How.CssSelector, Using = ".room-dropdown")]
        private IWebElement RoomDropdown;
    
        [FindsBy(How = How.XPath, Using = "//input[@placeholder='Meeting Title']")]
        private IWebElement MeetingTitleInput;
    
        [FindsBy(How = How.Id, Using = "scheduleBtn")]
        private IWebElement ScheduleButton;
    
        public TmsSchedulingPage(IWebDriver driver)
        {
            PageFactory.InitElements(driver, this);
        }
    
        public void ScheduleMeeting(string title, string room)
        {
            NewMeetingButton.Click();
            MeetingTitleInput.Clear();
            MeetingTitleInput.SendKeys(title);
            new SelectElement(RoomDropdown).SelectByText(room);
            ScheduleButton.Click();
        }
    }
    
    

**Result:** Element declarations were consolidated in page objects using attributes, making the intent of each element clear. Locator changes required updates in a single location. The Page Factory pattern became the team standard across the Cisco TMS project.

* * *

<a name="section-7"></a>

## Section 7: Framework Design — Page Object Model (Q57–Q64)

* * *

### Q57. How did you design a scalable POM architecture for a large application?

**Situation:** On the AXA Insurance project, the portal had over 50 distinct pages and workflows. A flat POM structure would have created unwieldy page objects and code duplication.

**Task:** I needed a hierarchical POM architecture that scaled to 50+ pages without duplication.

**Action:** I designed a three-layer POM:
    
    
    // Layer 1: Base page (shared utilities)
    public abstract class BasePage
    {
        protected readonly IPage Page;
        protected readonly TestSettings Settings;
        protected NavigationHelper Nav;
    
        protected BasePage(IPage page, TestSettings settings)
        {
            Page = page;
            Settings = settings;
            Nav = new NavigationHelper(page, settings);
        }
    
        protected async Task WaitForPageLoad()
            => await Page.WaitForLoadStateAsync(LoadState.NetworkIdle);
    }
    
    // Layer 2: Feature page (module-specific)
    public class ClaimsBasePage : BasePage
    {
        protected ILocator ClaimsHeader => Page.Locator("h1.claims-title");
        protected ILocator BreadcrumbNav => Page.Locator(".breadcrumb");
        
        protected async Task AssertOnClaimsSection()
            => await Expect(ClaimsHeader).ToBeVisibleAsync();
    }
    
    // Layer 3: Specific page (granular actions)
    public class FnolPage : ClaimsBasePage
    {
        private ILocator ClaimTypeDropdown => Page.Locator("#claimType");
        private ILocator IncidentDateInput  => Page.Locator("#incidentDate");
        private ILocator DescriptionField   => Page.Locator("#incidentDescription");
        private ILocator NextBtn            => Page.GetByRole(AriaRole.Button, new() { Name = "Next" });
    
        public async Task CompleteStep1(string claimType, DateTime incidentDate, string description)
        {
            await ClaimTypeDropdown.SelectOptionAsync(claimType);
            await IncidentDateInput.FillAsync(incidentDate.ToString("dd/MM/yyyy"));
            await DescriptionField.FillAsync(description);
            await NextBtn.ClickAsync();
            await WaitForPageLoad();
        }
    }
    
    

**Result:** The three-layer POM scaled cleanly to 50+ pages. Common functionality was inherited from base classes, reducing code duplication by an estimated 40%. New page objects were consistent in structure, making the suite easier to maintain and extend.

* * *

### Q58. How did you implement the Fluent Page Object pattern in C#?

**Situation:** On the AXA project, test scenarios involving multi-step wizard workflows (policy creation, claims submission) had verbose, procedural test code that was hard to read at a glance.

**Task:** I needed a more readable test code style for multi-step workflows that communicated intent clearly.

**Action:** I implemented a fluent builder pattern for wizard-style pages:
    
    
    public class PolicyCreationFlow
    {
        private readonly IPage _page;
    
        // Fluent method chaining - each method returns `this` for chaining
        public async Task<PolicyCreationFlow> SelectPolicyType(string type)
        {
            await _page.Locator("#policyType").SelectOptionAsync(type);
            return this;
        }
    
        public async Task<PolicyCreationFlow> EnterPersonalDetails(PersonalDetails details)
        {
            await _page.FillAsync("#firstName", details.FirstName);
            await _page.FillAsync("#lastName", details.LastName);
            await _page.FillAsync("#dateOfBirth", details.DOB.ToString("dd/MM/yyyy"));
            return this;
        }
    
        public async Task<PolicyCreationFlow> SelectCoverageLevel(string level)
        {
            await _page.Locator("#coverageLevel").SelectOptionAsync(level);
            return this;
        }
    
        public async Task<string> SubmitAndGetPolicyNumber()
        {
            await _page.GetByRole(AriaRole.Button, new() { Name = "Create Policy" }).ClickAsync();
            return await _page.Locator(".policy-number-confirmation").InnerTextAsync();
        }
    }
    
    // Readable test code
    [Test]
    public async Task Should_Create_Comprehensive_Policy()
    {
        var flow = new PolicyCreationFlow(Page);
        var policyNumber = await flow
            .SelectPolicyType("Comprehensive")
            .EnterPersonalDetails(TestData.StandardCustomer)
            .SelectCoverageLevel("Premium")
            .SubmitAndGetPolicyNumber();
    
        Assert.That(policyNumber, Does.StartWith("POL-"));
    }
    
    

**Result:** Test scenarios for wizard workflows became dramatically more readable. Non-technical stakeholders could review test scenarios and understand the business flow being tested without needing to understand the implementation details.

* * *

### Q59. How did you implement component objects for reusable UI components?

**Situation:** On the AXA portal, several UI components — date pickers, address lookup widgets, and file uploaders — appeared across multiple pages and needed identical interaction logic.

**Task:** I needed component objects that encapsulated reusable component interactions, independent of page context.

**Action:** I designed component objects that accepted a scoped `ILocator` as their root:
    
    
    // Component object - scoped to a container locator
    public class AddressLookupComponent
    {
        private readonly ILocator _root;
    
        public AddressLookupComponent(ILocator root)
        {
            _root = root; // Scoped to this component's container
        }
    
        private ILocator PostcodeInput   => _root.Locator("#postcode");
        private ILocator FindAddressBtn  => _root.Locator(".find-address-btn");
        private ILocator AddressDropdown => _root.Locator(".address-results");
    
        public async Task SelectAddress(string postcode, string addressLine)
        {
            await PostcodeInput.FillAsync(postcode);
            await FindAddressBtn.ClickAsync();
            await AddressDropdown.Locator("option").Filter(new() { HasText = addressLine })
                  .ClickAsync();
        }
    }
    
    // Usage in multiple page objects
    public class PolicyPage : BasePage
    {
        private AddressLookupComponent RiskAddress 
            => new AddressLookupComponent(Page.Locator("#risk-address-section"));
        
        private AddressLookupComponent CorrespondenceAddress 
            => new AddressLookupComponent(Page.Locator("#correspondence-address-section"));
    
        public async Task SetRiskAddress(string postcode, string address)
            => await RiskAddress.SelectAddress(postcode, address);
    }
    
    

**Result:** Component objects were reused across 12+ page objects without code duplication. When the address lookup widget was updated by the front-end team, only the single component object required updating — not each page object separately.

* * *

### Q60. How did you manage test data factories for consistent test data generation?

**Situation:** On the AXA project, tests required realistic but unique test data — policy numbers, customer names, dates — and hardcoded values were causing test interference.

**Task:** I needed a test data factory that generated unique, realistic data for each test run.

**Action:** I implemented a builder-pattern test data factory:
    
    
    public class PolicyDataBuilder
    {
        private string _policyType = "Comprehensive";
        private string _firstName  = "Test";
        private string _lastName   = $"User_{DateTime.Now.Ticks}";
        private DateTime _startDate = DateTime.Today.AddDays(1);
    
        public PolicyDataBuilder WithPolicyType(string type) 
        { _policyType = type; return this; }
        
        public PolicyDataBuilder WithCustomer(string firstName, string lastName)
        { _firstName = firstName; _lastName = lastName; return this; }
        
        public PolicyDataBuilder StartingOn(DateTime date)
        { _startDate = date; return this; }
    
        public PolicyData Build() => new PolicyData
        {
            PolicyType = _policyType,
            FirstName  = _firstName,
            LastName   = _lastName,
            StartDate  = _startDate,
            UniqueRef  = $"AUTO_{Guid.NewGuid().ToString()[..8].ToUpper()}"
        };
    
        // Quick factory methods for common scenarios
        public static PolicyData Standard() => new PolicyDataBuilder().Build();
        public static PolicyData Premium()  => new PolicyDataBuilder().WithPolicyType("Premium").Build();
    }
    
    // Usage in test
    var policy = PolicyDataBuilder.Premium().StartingOn(DateTime.Today.AddDays(7)).Build();
    
    

**Result:** Test data was unique per test run, eliminating interference between parallel tests. The builder pattern made test intentions clear, and the factory methods reduced boilerplate for common scenarios.

* * *

### Q61. How did you implement error page detection in your C# framework?

**Situation:** On the AXA portal, application errors sometimes presented as custom error pages (500, 404) rather than thrown exceptions, causing subsequent test steps to fail with confusing locator errors instead of a clear "application error" failure.

**Task:** I needed automatic error page detection that provided clear, immediate failure messages.

**Action:** I implemented an error page interceptor called after every navigation:
    
    
    public class ErrorPageDetector
    {
        private readonly IPage _page;
        
        private static readonly Dictionary<string, string> ErrorIndicators = new()
        {
            { ".error-500-container", "Internal Server Error (500)" },
            { ".error-404-container", "Page Not Found (404)" },
            { ".session-expired-message", "Session Expired" },
            { ".maintenance-mode-banner", "Application In Maintenance Mode" }
        };
    
        public async Task AssertNoErrorPage()
        {
            foreach (var (selector, description) in ErrorIndicators)
            {
                var element = _page.Locator(selector);
                if (await element.CountAsync() > 0)
                {
                    var screenshot = await _page.ScreenshotAsync(new() { FullPage = true });
                    TestContext.AddTestAttachment("error-page.png", "Error Page Screenshot");
                    Assert.Fail($"Application error detected: {description} at URL: {_page.Url}");
                }
            }
        }
    }
    
    // Called in BasePage after every navigation
    protected async Task NavigateToAsync(string path)
    {
        await Nav.GoToAsync(path);
        await _errorDetector.AssertNoErrorPage();
    }
    
    

**Result:** Application errors were immediately surfaced with clear failure messages and screenshots rather than cascading into confusing downstream locator failures. Time-to-diagnose for environment/application errors dropped significantly.

* * *

### Q62. How did you structure test project folder organisation in C#?

**Situation:** On the AXA project, as the test suite grew to 300+ test cases, the initial flat folder structure became difficult to navigate and test assets were hard to discover.

**Task:** I needed a scalable folder structure that organised tests, page objects, utilities, and test data logically.

**Action:** I established a domain-driven project structure:
    
    
    AXA.Automation/
    ├── Core/
    │   ├── Base/           → BaseTest.cs, BasePage.cs
    │   ├── Config/         → TestSettings.cs, appsettings.json
    │   ├── Helpers/        → WaitHelper, NavigationHelper, etc.
    │   └── Extensions/     → IPageExtensions.cs
    ├── Pages/
    │   ├── Common/         → HeaderPage, FooterPage, ModalHelper
    │   ├── Policy/         → PolicyListPage, PolicyDetailPage, PolicyCreatePage
    │   └── Claims/         → FnolPage, ClaimsListPage, ClaimsDetailPage
    ├── Components/
    │   ├── AddressLookupComponent.cs
    │   ├── DatePickerComponent.cs
    │   └── FileUploaderComponent.cs
    ├── Tests/
    │   ├── Policy/         → PolicyCreationTests, PolicySearchTests
    │   └── Claims/         → FnolTests, ClaimsProcessingTests
    ├── TestData/
    │   ├── Json/           → claim-scenarios.json, policy-data.json
    │   ├── Documents/      → test-receipt.pdf, test-photo.jpg
    │   └── Schemas/        → api-response-schemas/
    └── Reports/
        ├── screenshots/    → auto-populated on failure
        └── traces/         → auto-populated on failure
    
    

**Result:** The structure scaled cleanly to 300+ tests without confusion. New team members could locate any test asset within seconds. The domain-aligned folder structure also matched the business structure of the AXA portal, making navigation intuitive.

* * *

### Q63. How did you implement test categorisation and selective execution in C#?

**Situation:** On the J&J Salesforce project, the CI pipeline needed to run different test subsets at different stages — smoke tests on every commit, full regression nightly, and module-specific tests when individual modules were deployed.

**Task:** I needed a test categorisation system that supported flexible, selective execution from CI parameters.

**Action:** I implemented NUnit `[Category]` attributes with Jenkins parameter integration:
    
    
    // Category constants
    public static class TestCategory
    {
        public const string Smoke      = "Smoke";
        public const string Regression = "Regression";
        public const string CaseManagement = "CaseManagement";
        public const string Payroll    = "Payroll";
        public const string LiveChat   = "LiveChat";
        public const string Api        = "API";
    }
    
    // Test with multiple categories
    [Test]
    [Category(TestCategory.Smoke)]
    [Category(TestCategory.CaseManagement)]
    public async Task Should_Create_HR_Case_Successfully() { ... }
    
    [Test]
    [Category(TestCategory.Regression)]
    [Category(TestCategory.Payroll)]
    public async Task Should_Process_Payroll_Deduction_Correctly() { ... }
    
    
    
    
    <!-- .runsettings for category selection -->
    <RunSettings>
      <NUnit>
        <Where>cat == Smoke</Where>    <!-- Or: cat == Regression, cat == Payroll, etc. -->
      </NUnit>
    </RunSettings>
    
    
    
    
    # Jenkins - run smoke only
    dotnet test --filter "Category=Smoke"
    
    # Jenkins - run full regression
    dotnet test --filter "Category=Regression"
    
    

**Result:** CI pipelines ran precisely the right test subset at each stage. Smoke tests completed in under 5 minutes for per-commit feedback, nightly regression provided full coverage, and module-specific runs supported the module-by-module deployment strategy on J&J.

* * *

### Q64. How did you implement a test execution listener for real-time CI feedback?

**Situation:** On the Google project, the CI pipeline had long-running test suites and stakeholders wanted real-time feedback on test progress, not just a final report.

**Task:** I needed a real-time event listener that reported test status to ReportPortal and the CI console as each test completed.

**Action:** I implemented a custom NUnit `ITestListener`:
    
    
    public class RealTimeTestReporter : ITestListener
    {
        private readonly ReportPortalService _rp;
        private int _passed, _failed, _skipped;
    
        public void TestStarted(ITest test)
        {
            Console.WriteLine($"[START] {test.Name}");
            _rp.StartTest(test.Name, test.FullName);
        }
    
        public void TestFinished(ITestResult result)
        {
            switch (result.ResultState.Status)
            {
                case TestStatus.Passed:
                    _passed++;
                    Console.WriteLine($"[PASS]  {result.Test.Name}");
                    break;
                case TestStatus.Failed:
                    _failed++;
                    Console.WriteLine($"[FAIL]  {result.Test.Name}");
                    Console.WriteLine($"        {result.Message}");
                    _rp.LogFailure(result.Message, result.StackTrace);
                    break;
                case TestStatus.Skipped:
                    _skipped++;
                    Console.WriteLine($"[SKIP]  {result.Test.Name}");
                    break;
            }
            
            // Running totals
            Console.WriteLine($"        Progress: {_passed}P / {_failed}F / {_skipped}S");
            _rp.FinishTest(MapStatus(result));
        }
    }
    
    

**Result:** Jenkins console output provided real-time pass/fail progress throughout execution. Stakeholders monitoring the build could see failures as they occurred rather than waiting for the full suite to complete, enabling faster triage decisions.

* * *

<a name="section-8"></a>

## Section 8: BDD with SpecFlow (Q65–Q72)

* * *

### Q65. How did you implement SpecFlow BDD with C# Playwright on the AXA project?

**Situation:** On AXA, business stakeholders wanted to be able to read and contribute to test scenarios, but the existing xUnit-based tests were too technical for non-developers.

**Task:** I needed to implement a BDD layer using SpecFlow that made test scenarios business-readable while still leveraging the existing Playwright framework.

**Action:** I integrated SpecFlow with the existing C# Playwright framework:
    
    
    # Features/Claims/FnolSubmission.feature
    Feature: First Notification of Loss (FNOL) Submission
      As an AXA policyholder
      I want to submit a claim online
      So that I can receive compensation without calling customer service
    
      Background:
        Given I am logged in as a valid policyholder
    
      @Smoke @Claims
      Scenario: Submit a comprehensive theft claim successfully
        Given I have a Comprehensive policy "POL-12345"
        When I submit a theft claim with the following details:
          | Field         | Value              |
          | Incident Date | 3 days ago         |
          | Description   | Vehicle theft      |
        Then my claim reference should be generated
        And the claim status should be "Submitted"
    
    
    
    
    // Steps/Claims/FnolSteps.cs
    [Binding]
    public class FnolSteps
    {
        private readonly ScenarioContext _context;
        private readonly FnolPage _fnolPage;
        
        public FnolSteps(ScenarioContext context, FnolPage fnolPage)
        {
            _context = context;
            _fnolPage = fnolPage;
        }
    
        [When(@"I submit a theft claim with the following details:")]
        public async Task WhenISubmitATheftClaim(Table table)
        {
            var claimData = table.ToDictionary();
            var claimRef = await _fnolPage.SubmitClaim(claimData);
            _context["ClaimReference"] = claimRef;
        }
    
        [Then(@"my claim reference should be generated")]
        public void ThenClaimReferenceIsGenerated()
        {
            Assert.That(_context["ClaimReference"]?.ToString(), Does.Match(@"CLM-\d{4}-\d+"));
        }
    }
    
    

**Result:** Business analysts started contributing new Gherkin scenarios directly, expanding coverage without requiring QA engineering involvement. The living documentation aspect of SpecFlow also gave stakeholders a current view of exactly what was being tested.

* * *

### Q66. How did you implement SpecFlow hooks for setup and teardown?

**Situation:** On the AXA SpecFlow project, each scenario needed a fresh browser context and authentication state, and certain scenario tags required specific test data setup.

**Task:** I needed centralised setup/teardown hooks that scaled to different scenario types.

**Action:** I implemented SpecFlow hooks with tag-specific conditions:
    
    
    [Binding]
    public class TestHooks
    {
        private readonly ScenarioContext _scenarioContext;
        private IBrowser _browser;
        private IBrowserContext _context;
    
        [BeforeScenario]
        public async Task BeforeScenario()
        {
            var playwright = await Playwright.CreateAsync();
            _browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });
            _context = await _browser.NewContextAsync(new()
            {
                StorageStatePath = "auth-state.json"
            });
            var page = await _context.NewPageAsync();
            _scenarioContext["Page"] = page;
        }
    
        [BeforeScenario("@RequiresTestPolicy")]
        public async Task SetupTestPolicy()
        {
            // API call to create required policy for this scenario
            var policyApi = new PolicyApiClient(Settings.ApiBaseUrl);
            var policyRef = await policyApi.CreateTestPolicy(PolicyDataBuilder.Standard().Build());
            _scenarioContext["TestPolicyRef"] = policyRef;
        }
    
        [AfterScenario]
        public async Task AfterScenario()
        {
            if (_scenarioContext.TestError != null)
            {
                var page = _scenarioContext.Get<IPage>("Page");
                await page.ScreenshotAsync(new() 
                { 
                    Path = $"screenshots/{_scenarioContext.ScenarioInfo.Title}.png",
                    FullPage = true 
                });
            }
            await _context.CloseAsync();
            await _browser.CloseAsync();
        }
    
        [AfterScenario("@RequiresTestPolicy")]
        public async Task CleanupTestPolicy()
        {
            if (_scenarioContext.ContainsKey("TestPolicyRef"))
            {
                var policyApi = new PolicyApiClient(Settings.ApiBaseUrl);
                await policyApi.DeletePolicy(_scenarioContext["TestPolicyRef"].ToString());
            }
        }
    }
    
    

**Result:** Scenario isolation was complete and automated. Tag-based hooks ensured that only scenarios requiring test data performed the API setup/teardown, keeping the suite efficient. Screenshots on failure were generated automatically for every failing scenario.

* * *

### Q67. How did you implement SpecFlow data tables and scenario outlines?

**Situation:** On the AXA project, the same claim submission flow needed to be validated with multiple claim types to ensure all business rules applied correctly across products.

**Task:** I needed to parameterise scenarios to cover multiple data combinations without duplicating feature file content.

**Action:** I used Scenario Outlines with Examples tables:
    
    
    Scenario Outline: Claim processing follows correct business rules per policy type
      Given I have a "<PolicyType>" policy
      When I submit a "<ClaimType>" claim
      Then the expected outcome should be "<ExpectedOutcome>"
      And the claim should be assigned to "<AssignedTeam>"
    
      Examples:
        | PolicyType    | ClaimType   | ExpectedOutcome  | AssignedTeam      |
        | Comprehensive | Theft        | Auto-Approved    | Claims Processing |
        | Comprehensive | Fire Damage  | Manual Review    | Loss Adjusters    |
        | Third Party   | Own Damage   | Rejected         | Customer Services |
        | Third Party   | Third Party  | Auto-Approved    | Claims Processing |
    
    # Data Table within a step
    Scenario: Create policy with full customer details
      When I create a policy with:
        | Field      | Value          |
        | FirstName  | Shajeeb        |
        | LastName   | Shahul Hameed  |
        | DOB        | 01/01/1985     |
        | Email      | test@axa.ie    |
    
    
    
    
    [When(@"I create a policy with:")]
    public async Task WhenICreateAPolicy(Table table)
    {
        var details = table.Rows.ToDictionary(r => r[0], r => r[1]);
        await _policyPage.CreatePolicy(
            details["FirstName"], details["LastName"],
            details["DOB"], details["Email"]);
    }
    
    

**Result:** 12 claim combination scenarios ran from a single Scenario Outline with no duplicated steps. Business analysts could add new combinations by adding rows to the Examples table without requiring any code changes — exactly the non-developer involvement model the project needed.

* * *

### Q68. How did you implement Dependency Injection in SpecFlow with C#?

**Situation:** On the AXA SpecFlow project, page objects and helper classes needed to be shared between step definition classes without being re-instantiated for every step.

**Task:** I needed a dependency injection approach that shared instances across step classes within the same scenario.

**Action:** I used SpecFlow's built-in BoDi DI container:
    
    
    // Register dependencies in Hooks
    [Binding]
    public class DependencyRegistration
    {
        private readonly IObjectContainer _container;
    
        public DependencyRegistration(IObjectContainer container)
        {
            _container = container;
        }
    
        [BeforeScenario]
        public async Task RegisterDependencies()
        {
            var playwright = await Playwright.CreateAsync();
            var browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });
            var context = await browser.NewContextAsync(new() { StorageStatePath = "auth-state.json" });
            var page = await context.NewPageAsync();
    
            // Register shared instances
            _container.RegisterInstanceAs(playwright);
            _container.RegisterInstanceAs(browser);
            _container.RegisterInstanceAs(context);
            _container.RegisterInstanceAs(page);
            _container.RegisterInstanceAs(new FnolPage(page, Settings));
            _container.RegisterInstanceAs(new PolicyPage(page, Settings));
        }
    }
    
    // Step classes receive dependencies via constructor injection
    [Binding]
    public class FnolSteps
    {
        private readonly FnolPage _fnolPage;
        private readonly ScenarioContext _context;
    
        public FnolSteps(FnolPage fnolPage, ScenarioContext context)
        {
            _fnolPage = fnolPage; // Injected - same instance shared across step classes
            _context = context;
        }
    }
    
    

**Result:** Page objects were shared correctly across multiple step definition classes within a scenario. Object creation overhead was minimised, and the DI approach made the step definitions clean and focused purely on business logic.

* * *

### Q69. How did you implement API step definitions alongside UI steps in SpecFlow?

**Situation:** On the AXA project, some BDD scenarios needed to combine API actions (creating test data efficiently) with UI verification steps in the same Gherkin scenario.

**Task:** I needed to mix API and UI steps naturally in SpecFlow without creating separate feature files for each layer.

**Action:** I implemented API step definitions that shared context with UI steps:
    
    
    Scenario: Policy created via API should appear in UI
      Given a Comprehensive policy exists with reference "POL-API-001"    # API step
      When I navigate to the policy management page                        # UI step
      And I search for policy "POL-API-001"                               # UI step
      Then the policy should be displayed with status "Active"            # UI step
    
    
    
    
    [Binding]
    public class ApiSetupSteps
    {
        private readonly IAPIRequestContext _apiContext;
        private readonly ScenarioContext _scenarioContext;
    
        [Given(@"a Comprehensive policy exists with reference ""(.*)""")]
        public async Task GivenPolicyExists(string policyRef)
        {
            var response = await _apiContext.PostAsync("/api/policies", new()
            {
                DataObject = new { Type = "Comprehensive", Reference = policyRef }
            });
            Assert.That(response.Status, Is.EqualTo(201));
            _scenarioContext["PolicyRef"] = policyRef;
        }
    }
    
    

**Result:** Mixed API + UI scenarios ran end-to-end within the same SpecFlow framework. API setup steps were significantly faster than UI-based setup, improving overall suite speed while maintaining business-readable scenario language.

* * *

### Q70. How did you generate living documentation from SpecFlow in C#?

**Situation:** On the AXA project, stakeholders wanted to review current test coverage without accessing the source code repository or running the test suite.

**Task:** I needed to generate a readable, always-current documentation artefact from the SpecFlow feature files.

**Action:** I integrated SpecFlow+ LivingDoc into the CI pipeline:
    
    
    # Install SpecFlow+ LivingDoc CLI
    dotnet tool install --global SpecFlow.Plus.LivingDoc.CLI
    
    # Generate after test run (in Jenkinsfile)
    livingdoc test-assembly AXA.Automation.Tests.dll \
      -t TestExecution.json \
      --output AXA_LivingDoc.html \
      --title "AXA Insurance Automation Coverage"
    
    
    
    
    <!-- .csproj - enable test execution output -->
    <ItemGroup>
      <PackageReference Include="SpecFlow.Plus.LivingDocPlugin" Version="3.9.57" />
    </ItemGroup>
    
    

**Result:** Every CI run produced an updated HTML living documentation file published as a Jenkins artefact. Stakeholders bookmarked the Jenkins artefact URL and used it as their reference for current test coverage without needing repository access. The visual pass/fail indicators per scenario made it immediately useful for release readiness assessments.

* * *

### Q71. How did you handle shared state between SpecFlow scenarios using Context classes?

**Situation:** On the AXA project, related scenarios in a feature file needed to share the claim reference created in one scenario for validation in a subsequent scenario.

**Task:** I needed a clean pattern for scenario-level state sharing without using global static variables.

**Action:** I implemented typed context classes injected via SpecFlow's DI:
    
    
    // Context class - scenario-scoped shared state
    public class ClaimsContext
    {
        public string ClaimReference { get; set; }
        public string PolicyNumber   { get; set; }
        public string ClaimStatus    { get; set; }
        public DateTime SubmittedAt  { get; set; }
    }
    
    // Register in DI
    _container.RegisterTypeAs<ClaimsContext, ClaimsContext>();
    
    // Used across multiple step definition classes
    [Binding]
    public class FnolSteps
    {
        private readonly ClaimsContext _claimsContext;
        
        [When(@"I submit a claim for policy ""(.*)""")]
        public async Task SubmitClaim(string policyNumber)
        {
            _claimsContext.PolicyNumber = policyNumber;
            _claimsContext.ClaimReference = await _fnolPage.SubmitClaim(policyNumber);
            _claimsContext.SubmittedAt = DateTime.UtcNow;
        }
    }
    
    [Binding]
    public class ClaimsVerificationSteps
    {
        private readonly ClaimsContext _claimsContext;
    
        [Then(@"the claim should be retrievable by its reference")]
        public async Task ClaimShouldBeRetrievable()
        {
            // Access state set by FnolSteps
            Assert.That(_claimsContext.ClaimReference, Is.Not.Null);
            await _claimsListPage.SearchForClaim(_claimsContext.ClaimReference);
        }
    }
    
    

**Result:** State was shared cleanly between step classes within a scenario without any static variables or ScenarioContext dictionary string keys. Typed context classes provided compile-time safety and IntelliSense support, significantly reducing context-key typo bugs.

* * *

### Q72. How did you implement SpecFlow tags for test categorisation and conditional execution?

**Situation:** On the AXA project, SpecFlow scenarios needed to be categorised for selective CI execution — smoke vs. regression, module-specific, and environment-specific.

**Task:** I needed a tagging strategy that aligned with the CI pipeline's execution model.

**Action:** I defined a consistent tag taxonomy and implemented tag-based hooks and filtering:
    
    
    @Smoke @Claims @Regression
    Scenario: Submit FNOL claim - happy path
    
    @Regression @Claims @RequiresTestPolicy
    Scenario: Validate claim status progression through workflow
    
    @UAT @Smoke
    Scenario: Verify portal login
    
    @Ignore @KnownIssue-JIRA-1234
    Scenario: Payment authorisation edge case (pending fix)
    
    
    
    
    // Tag-based hook
    [BeforeScenario("@RequiresTestPolicy")]
    public async Task SetupRequiredPolicy() { ... }
    
    // Tag-based skip
    [BeforeScenario("@Ignore")]
    public void SkipIgnoredScenarios()
    {
        Assert.Ignore("Scenario tagged @Ignore - skipped pending resolution.");
    }
    
    
    
    
    # CI selective execution
    dotnet test --filter "Category=Smoke"           # Smoke only
    dotnet test --filter "Category=Regression"      # Full regression
    dotnet test --filter "Category=Claims"          # Claims module only
    dotnet test --filter "Category!=KnownIssue"    # Exclude known issues
    
    

**Result:** Selective execution worked cleanly in CI via tag filtering. The `@KnownIssue-JIRA-123` pattern provided traceability from skipped tests to JIRA tickets, and the `@RequiresTestPolicy` hook ran expensive setup only for the scenarios that needed it.

* * *

<a name="section-9"></a>

## Section 9: CI/CD, Parallel Execution & Reporting (Q73–Q82)

* * *

### Q73. How did you configure parallel test execution in C# NUnit for Playwright?

**Situation:** On the AXA project, the 300+ Playwright test suite took 45 minutes sequentially. The CI pipeline needed faster feedback.

**Task:** I needed to configure parallel execution while maintaining browser context isolation.

**Action:** I configured NUnit parallel execution with independent browser contexts per test:
    
    
    // AssemblyInfo.cs

[assembly: Parallelizable(ParallelScope.Fixtures)] 

[assembly: LevelOfParallelism(5)] // BaseTest - each fixture gets its own Playwright instance [TestFixture] public class BaseTest { private IPlaywright _playwright; private IBrowser _browser; protected IBrowserContext Context; protected IPage Page; [OneTimeSetUp] public async Task LaunchBrowser() { _playwright = await Playwright.CreateAsync(); _browser = await _playwright.Chromium.LaunchAsync(new() { Headless = true }); } [SetUp] public async Task CreateContext() { Context = await _browser.NewContextAsync(new() { StorageStatePath = "auth-state.json" }); Page = await Context.NewPageAsync(); } [TearDown] public async Task CloseContext() => await Context.CloseAsync(); [OneTimeTearDown] public async Task CloseBrowser() { await _browser.CloseAsync(); _playwright.Dispose(); } }

**Result:** Suite execution time reduced from 45 minutes to approximately 12 minutes with 5 parallel fixtures. Each test ran in a completely isolated browser context, and the `OneTimeSetUp`/`OneTimeTearDown` pattern meant browser launch happened once per fixture rather than per test.

* * *

### Q74. How did you integrate C# test results into Jenkins pipeline?

**Situation:** On multiple projects, Jenkins needed to parse NUnit test results for build status decisions and trend reporting.

**Task:** I needed to configure NUnit output in a format Jenkins could consume and act on.

**Action:** I configured the NUnit test runner and Jenkins pipeline integration:
    
    
    # .runsettings
    <RunSettings>
      <NUnit>
        <DefaultTimeout>60000</DefaultTimeout>
        <WorkDirectory>TestResults</WorkDirectory>
        <OutputXmlFolderHint>TestResults</OutputXmlFolderHint>
      </NUnit>
    </RunSettings>
    
    
    
    
    // Jenkinsfile
    pipeline {
        agent any
        
        stages {
            stage('Build') {
                steps { sh 'dotnet build --configuration Release' }
            }
            
            stage('Install Browsers') {
                steps { sh 'pwsh bin/Release/net8.0/playwright.ps1 install --with-deps chromium' }
            }
            
            stage('Run Smoke Tests') {
                steps {
                    sh 'dotnet test --filter "Category=Smoke" --results-directory TestResults --logger "nunit;LogFilePath=TestResults/smoke-results.xml"'
                }
            }
    
            stage('Run Full Regression') {
                when { branch 'main' }
                steps {
                    sh 'dotnet test --results-directory TestResults --logger "nunit;LogFilePath=TestResults/regression-results.xml"'
                }
            }
        }
    
        post {
            always {
                nunit testResultsPattern: 'TestResults/*.xml'
                archiveArtifacts artifacts: 'screenshots/**, traces/**', allowEmptyArchive: true
                publishHTML([
                    reportDir: 'TestResults',
                    reportFiles: 'index.html',
                    reportName: 'Test Report'
                ])
            }
            failure {
                emailext(
                    subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Test failures detected. See: ${env.BUILD_URL}",
                    to: "qa-team@client.com"
                )
            }
        }
    }
    
    

**Result:** Jenkins parsed NUnit XML results to track test trends over time, trigger failure notifications, and gate deployments. Archived screenshots and traces were accessible directly from the Jenkins build page for each failure.

* * *

### Q75. How did you implement test sharding for distributed execution in C#?

**Situation:** On the Google project, the test suite grew to a scale where even parallel execution on a single agent was insufficient to meet the feedback time requirements.

**Task:** I needed to distribute the test suite across multiple CI agents simultaneously.

**Action:** I implemented test sharding using NUnit filtering with shard parameters:
    
    
    # Split tests across 3 agents using modulo on test count
    # Agent 1: runs tests 0, 3, 6, 9...
    dotnet test --filter "TestIndex % 3 == 0"
    # Agent 2: runs tests 1, 4, 7, 10...
    dotnet test --filter "TestIndex % 3 == 1"
    # Agent 3: runs tests 2, 5, 8, 11...
    dotnet test --filter "TestIndex % 3 == 2"
    
    
    
    
    // Jenkinsfile - parallel agents
    stage('Distributed Test Execution') {
        parallel {
            stage('Shard 0') {
                agent { label 'test-agent' }
                steps { sh 'dotnet test --filter "Category=Regression&Shard=0"' }
            }
            stage('Shard 1') {
                agent { label 'test-agent' }
                steps { sh 'dotnet test --filter "Category=Regression&Shard=1"' }
            }
            stage('Shard 2') {
                agent { label 'test-agent' }
                steps { sh 'dotnet test --filter "Category=Regression&Shard=2"' }
            }
        }
    }
    
    

**Result:** Distributing the Google test suite across 3 agents reduced total execution time to approximately one-third. The sharding approach was transparent to the test code itself — categorisation at authoring time determined shard assignment.

* * *

### Q76. How did you implement test result aggregation from multiple parallel runs?

**Situation:** On the J&J Salesforce project, parallel execution across multiple agents produced separate NUnit XML result files that needed to be merged for a unified report.

**Task:** I needed to aggregate distributed test results into a single coherent report for stakeholders.

**Action:** I implemented XML merging with NUnit report generation:
    
    
    # Merge multiple NUnit result files
    dotnet tool install -g NUnit.ResultsMerger
    
    NUnit.ResultsMerger \
      TestResults/agent1-results.xml \
      TestResults/agent2-results.xml \
      TestResults/agent3-results.xml \
      --output TestResults/merged-results.xml
    
    # Generate HTML report from merged results
    dotnet tool install -g trx2junit
    
    
    
    
    // Jenkinsfile post-step
    post {
        always {
            sh 'NUnit.ResultsMerger TestResults/agent*-results.xml --output TestResults/final-results.xml'
            nunit testResultsPattern: 'TestResults/final-results.xml'
        }
    }
    
    

**Result:** A single unified test report was generated regardless of how many agents ran the suite. Stakeholders received a consistent, complete view of test results without needing to understand the distributed execution architecture.

* * *

### Q77. How did you implement .runsettings for environment-specific Playwright configuration?

**Situation:** On the AXA project, different environments (Local, CI, UAT) required different Playwright configurations — headless mode, timeout values, browser types, and base URLs.

**Task:** I needed environment-specific configuration that could be applied via a single parameter change in CI.

**Action:** I created environment-specific `.runsettings` files:
    
    
    <!-- ci.runsettings -->
    <RunSettings>
      <TestRunParameters>
        <Parameter name="BaseUrl" value="https://uat.portal.axa.ie" />
        <Parameter name="Headless" value="true" />
        <Parameter name="DefaultTimeout" value="30000" />
        <Parameter name="BrowserType" value="chromium" />
        <Parameter name="Environment" value="CI" />
      </TestRunParameters>
      <Playwright>
        <BrowserName>chromium</BrowserName>
        <LaunchOptions>
          <Headless>true</Headless>
          <SlowMo>0</SlowMo>
        </LaunchOptions>
      </Playwright>
    </RunSettings>
    
    
    
    
    <!-- local.runsettings -->
    <RunSettings>
      <TestRunParameters>
        <Parameter name="BaseUrl" value="https://localhost:5001" />
        <Parameter name="Headless" value="false" />
        <Parameter name="DefaultTimeout" value="60000" />
      </TestRunParameters>
    </RunSettings>
    
    
    
    
    # CI pipeline
    dotnet test --settings ci.runsettings
    
    # Local development
    dotnet test --settings local.runsettings
    
    

**Result:** Configuration switching between environments required only a single `--settings` parameter change. No code was modified between local and CI execution, ensuring the same tests ran consistently in both contexts.

* * *

### Q78. How did you implement GitHub Actions CI workflow for C# Playwright?

**Situation:** On a project using GitHub for source control, I needed to configure a GitHub Actions pipeline that triggered test execution on pull requests.

**Task:** I needed a GitHub Actions workflow that installed dependencies, ran tests, and published results with zero manual steps.

**Action:** I implemented a complete GitHub Actions YAML workflow:
    
    
    name: Playwright Tests
    
    on:
      pull_request:
        branches: [main, develop]
      push:
        branches: [main]
    
    jobs:
      test:
        runs-on: ubuntu-latest
        
        strategy:
          matrix:
            browser: [chromium, firefox]   # Run on multiple browsers
        
        steps:
          - uses: actions/checkout@v4
          
          - name: Setup .NET
            uses: actions/setup-dotnet@v4
            with:
              dotnet-version: '8.0.x'
          
          - name: Install dependencies
            run: dotnet restore
          
          - name: Build
            run: dotnet build --no-restore
          
          - name: Install Playwright browsers
            run: pwsh bin/Debug/net8.0/playwright.ps1 install --with-deps ${{ matrix.browser }}
          
          - name: Run tests
            run: dotnet test --settings ci.runsettings
            env:
              BROWSER: ${{ matrix.browser }}
              BASE_URL: ${{ secrets.UAT_BASE_URL }}
          
          - name: Upload test results
            uses: actions/upload-artifact@v4
            if: always()
            with:
              name: test-results-${{ matrix.browser }}
              path: |
                TestResults/
                screenshots/
                traces/
    
    

**Result:** Pull requests were automatically validated against both Chromium and Firefox in parallel. Branch protection rules prevented merging if either browser job failed. The matrix strategy doubled browser coverage without doubling pipeline complexity.

* * *

### Q79. How did you implement code coverage measurement for your C# automation tests?

**Situation:** On the AXA project, stakeholders occasionally confused test suite size with code coverage — conflating "300 tests" with "comprehensive coverage." I needed a way to demonstrate actual coverage.

**Task:** I needed to measure and report meaningful test coverage metrics for the automation suite.

**Action:** I integrated Coverlet for code coverage measurement:
    
    
    <!-- .csproj -->
    <ItemGroup>
      <PackageReference Include="coverlet.collector" Version="6.0.0" />
    </ItemGroup>
    
    
    
    
    # Run with coverage
    dotnet test --collect:"XPlat Code Coverage" --results-directory TestResults
    
    # Generate HTML report
    dotnet tool install -g dotnet-reportgenerator-globaltool
    reportgenerator \
      -reports:TestResults/**/coverage.cobertura.xml \
      -targetdir:TestResults/CoverageReport \
      -reporttypes:Html
    
    
    
    
    // Jenkins - publish coverage
    publishCoverage adapters: [coberturaAdapter('TestResults/**/coverage.cobertura.xml')],
                   sourceFileResolver: sourceFiles('STORE_LAST_BUILD')
    
    

**Result:** Code coverage reports were published alongside test results on every CI run. Coverage data revealed that the payment processing module had lower-than-expected automation coverage, leading to targeted test expansion that caught the API data integrity defect described in earlier questions.

* * *

### Q80. How did you implement test result notifications in Slack from C# Playwright CI?

**Situation:** On the J&J project, the distributed team needed immediate notification of CI failures without having to monitor Jenkins dashboards across time zones.

**Task:** I needed automated Slack notifications that included meaningful failure context rather than just a build number.

**Action:** I implemented Slack notifications via Jenkins and a custom failure summary:
    
    
    // Jenkinsfile
    post {
        failure {
            script {
                def failedTests = parseFailedTests('TestResults/results.xml')
                def message = """
    *Build FAILED* :x:
    *Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
    *Failed Tests:* ${failedTests.size()}
    ${failedTests.take(5).collect { "• ${it}" }.join('\n')}
    *Details:* ${env.BUILD_URL}
                """
                slackSend(channel: '#qa-automation', color: 'danger', message: message)
            }
        }
        success {
            slackSend(channel: '#qa-automation', color: 'good',
                message: ":white_check_mark: Build PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
    
    def parseFailedTests(String resultsFile) {
        def results = readFile(resultsFile)
        def xml = new XmlSlurper().parseText(results)
        return xml.'**'.findAll { it.name() == 'test-case' && it.@result == 'Failed' }
                   .collect { it.@name.toString() }
    }
    
    

**Result:** The team received immediate, actionable Slack notifications on failure, including the names of the first 5 failing tests. Distributed team members in different time zones could triage failures without accessing Jenkins, significantly reducing the feedback loop for release-blocking defects.

* * *

### Q81. How did you implement test execution dashboards using ReportPortal with C#?

**Situation:** On the Thomson Reuters project, ReportPortal was selected as the test reporting solution. I needed to integrate the C# NUnit/Selenium suite with ReportPortal's API.

**Task:** I needed real-time test result streaming to ReportPortal from the C# test framework.

**Action:** I integrated the ReportPortal NUnit agent:
    
    
    <!-- .csproj -->
    <ItemGroup>
      <PackageReference Include="ReportPortal.NUnit" Version="5.0.0" />
    </ItemGroup>
    
    <!-- ReportPortal.config.json -->
    {
      "enabled": true,
      "server": {
        "url": "https://reportportal.company.com/api/v1/",
        "project": "TaxAutomation",
        "authentication": { "apiKey": "#{RP_API_KEY}#" }
      },
      "launch": {
        "name": "Thomson Reuters Tax Regression",
        "attributes": [
          { "key": "environment", "value": "#{ENV}#" },
          { "key": "branch",      "value": "#{GIT_BRANCH}#" }
        ]
      }
    }
    
    
    
    
    // Custom log during test execution
    private static readonly ILog Log = LogManager.GetLogger(typeof(TaxCalculationTests));
    
    [Test]
    public void Should_Calculate_UK_VAT_Correctly()
    {
        Log.Info("Starting UK VAT calculation test");
        Log.Debug($"Input values: Amount=£1000, Rate=20%");
        
        // ... test steps ...
        
        Log.Info($"Calculated VAT: £{actualVat}");
        Assert.That(actualVat, Is.EqualTo(200));
    }
    
    

**Result:** ReportPortal received real-time streaming results with step-level logs, screenshots, and metadata. The defect classification ML model learned from our test history, and auto-analysis accuracy reached approximately 80% after 3 months of training — significantly reducing manual triage overhead.

* * *

### Q82. How did you implement scheduled test runs for continuous environment monitoring?

**Situation:** On the J&J Salesforce project, post-deployment smoke tests needed to run automatically after every production release, and a nightly health check was required to confirm the environment was stable for next-day development.

**Task:** I needed scheduled and event-triggered test execution beyond the standard PR/push triggers.

**Action:** I configured multiple Jenkins job types with different triggers:
    
    
    // Nightly regression job
    pipeline {
        triggers {
            cron('H 2 * * 1-5')  // 2 AM Mon-Fri
        }
        stages {
            stage('Nightly Regression') {
                steps {
                    sh 'dotnet test --filter "Category=Regression" --settings ci.runsettings'
                }
            }
        }
    }
    
    // Post-deployment smoke (triggered by deployment pipeline)
    pipeline {
        triggers {
            upstream(upstreamProjects: 'SalesforceDeployment', threshold: hudson.model.Result.SUCCESS)
        }
        stages {
            stage('Smoke Tests') {
                steps {
                    sh 'dotnet test --filter "Category=Smoke" --settings production.runsettings'
                }
            }
        }
    }
    
    

**Result:** Environment health was monitored automatically. Nightly regressions caught environment drift before it impacted the development team. Post-deployment smoke tests confirmed production stability within 5 minutes of each release, giving stakeholders immediate confidence in deployments.

* * *

<a name="section-10"></a>

## Section 10: Debugging, Stability & Best Practices (Q83–Q100)

* * *

### Q83. How did you identify and eliminate the root causes of flaky tests in C# Playwright?

**Situation:** On the AXA project, approximately 8% of tests were intermittently failing. The team had been retrying them without investigating root causes.

**Task:** I needed a systematic approach to identify and permanently fix flaky tests rather than masking them with retries.

**Action:** I implemented a flakiness analysis process:
    
    
    // Track flaky tests with metadata
    public class FlakinessTracker
    {
        private static readonly Dictionary<string, FlakyTestRecord> _records = new();
    
        public static void RecordRetry(string testName, string failureReason, Exception ex)
        {
            if (!_records.ContainsKey(testName))
                _records[testName] = new FlakyTestRecord { TestName = testName };
            
            _records[testName].RetryCount++;
            _records[testName].FailureReasons.Add(failureReason);
            _records[testName].LastException = ex.GetType().Name;
            
            Console.WriteLine($"[FLAKY] {testName} - retry #{_records[testName].RetryCount} | Reason: {failureReason}");
        }
    
        // Report in AfterTestRun hook
        [AfterTestRun]
        public static void ReportFlakiness()
        {
            var flakyTests = _records.Values.Where(r => r.RetryCount > 0).OrderByDescending(r => r.RetryCount);
            foreach (var test in flakyTests)
            {
                Console.WriteLine($"FLAKY REPORT: {test.TestName} | Retries: {test.RetryCount} | Reasons: {string.Join(", ", test.FailureReasons.Distinct())}");
            }
        }
    }
    
    

Root causes I systematically fixed:

  * **Timing issues** : Replaced all `Thread.Sleep` with Playwright `WaitFor*` or `Expect`
  * **State bleed** : Enforced browser context isolation per test
  * **Environment coupling** : Added pre-test health checks
  * **Locator fragility** : Migrated to role-based and data-testid locators



**Result:** Flaky tests reduced from 8% to under 1% over two sprints of systematic root cause analysis and fixing. The tracking data identified the 10 highest-frequency flaky tests — all of which had timing-related root causes that were fixable with Playwright's built-in waiting mechanisms.

* * *

### Q84. How did you implement proper logging strategy in C# Playwright tests?

**Situation:** On the Google project, test failures in CI were difficult to diagnose because the only evidence was the assertion failure message — no context about what led to the failure.

**Task:** I needed structured logging throughout test execution to provide actionable diagnostic context.

**Action:** I implemented structured logging with NLog:
    
    
    // NLog configuration
    var config = new NLog.Config.LoggingConfiguration();
    var fileTarget = new NLog.Targets.FileTarget("logfile")
    {
        FileName = $"logs/test-{DateTime.Now:yyyyMMdd-HHmmss}.log",
        Layout = "${longdate} | ${level:uppercase=true} | ${callsite} | ${message} ${exception:format=tostring}"
    };
    config.AddRule(LogLevel.Debug, LogLevel.Fatal, fileTarget);
    NLog.LogManager.Configuration = config;
    
    // Base page logging
    public abstract class BasePage
    {
        protected readonly ILogger Logger;
    
        protected BasePage(IPage page)
        {
            Logger = LogManager.GetCurrentClassLogger();
            
            // Log all Playwright actions automatically
            page.Console += (_, msg) => Logger.Debug($"[Browser Console] {msg.Type}: {msg.Text}");
            page.RequestFailed += (_, req) => Logger.Warn($"[Request Failed] {req.Url}: {req.Failure}");
            page.PageError += (_, err) => Logger.Error($"[Page Error] {err}");
        }
    
        protected async Task ClickAsync(ILocator locator, string description = "")
        {
            Logger.Info($"Clicking: {description} ({locator})");
            await locator.ClickAsync();
        }
    }
    
    

**Result:** Test logs provided a complete narrative of each test execution — every action, browser console message, and network failure was recorded. Failures in CI were diagnosable from logs alone in the majority of cases, reducing the need for local reproduction.

* * *

### Q85. How did you implement environment-agnostic test design?

**Situation:** On the J&J project, tests were written with hardcoded environment assumptions — specific user IDs, case numbers, and URLs that only existed in UAT.

**Task:** I needed to make tests work correctly across Predev, UAT, Preprod, and Production without code changes.

**Action:** I implemented full environment abstraction:
    
    
    // Environment-specific configuration
    public class EnvironmentConfig
    {
        private static readonly Dictionary<string, EnvironmentSettings> Configs = new()
        {
            ["predev"]   = new() { BaseUrl = "https://predev.jnj-gs.com", IsProduction = false },
            ["uat"]      = new() { BaseUrl = "https://uat.jnj-gs.com",    IsProduction = false },
            ["preprod"]  = new() { BaseUrl = "https://preprod.jnj-gs.com", IsProduction = false },
            ["prod"]     = new() { BaseUrl = "https://jnj-gs.com",         IsProduction = true  }
        };
    
        public static EnvironmentSettings ForEnvironment(string envName)
            => Configs.TryGetValue(envName.ToLower(), out var config)
                ? config
                : throw new ArgumentException($"Unknown environment: {envName}");
    }
    
    // Conditional test execution based on environment
    [Test]
    [Category("Regression")]
    public async Task Should_Process_Payroll_Case()
    {
        if (Settings.Environment.IsProduction)
            Assert.Ignore("Payroll data modification tests skipped in Production.");
        
        // Test logic...
    }
    
    

**Result:** The same test suite ran across all four J&J environments with a single environment variable change. Production-safe guards prevented destructive data tests from running against live systems, a critical requirement for the J&J programme.

* * *

### Q86. How did you implement API response caching in tests to improve speed?

**Situation:** On the Google project, repeated API calls to BigQuery for the same reference data in setup steps were adding significant overhead to the test suite startup time.

**Task:** I needed to cache read-only reference data to avoid redundant API calls without compromising test isolation.

**Action:** I implemented a lazy-loading cache for reference data:
    
    
    public class ReferenceDataCache
    {
        private static readonly SemaphoreSlim _lock = new(1, 1);
        private static Dictionary<string, object> _cache = new();
    
        public static async Task<T> GetOrFetchAsync<T>(string key, Func<Task<T>> fetcher)
        {
            if (_cache.TryGetValue(key, out var cached))
                return (T)cached;
    
            await _lock.WaitAsync();
            try
            {
                // Double-check after acquiring lock
                if (_cache.TryGetValue(key, out cached))
                    return (T)cached;
    
                var value = await fetcher();
                _cache[key] = value;
                return value;
            }
            finally
            {
                _lock.Release();
            }
        }
    
        [AfterTestRun]
        public static void ClearCache() => _cache.Clear();
    }
    
    // Usage
    var regionData = await ReferenceDataCache.GetOrFetchAsync("regions", 
        () => ApiClient.GetAllRegionsAsync());
    
    

**Result:** Reference data API calls were eliminated after the first test that needed them. Suite startup time reduced noticeably, and the thread-safe implementation worked correctly with parallel test execution.

* * *

### Q87. How did you write maintainable XPath expressions in C# Selenium?

**Situation:** On the Thomson Reuters Tax project, the test suite contained complex XPath expressions that were brittle and unreadable — breaking on minor DOM changes and impossible to understand at a glance.

**Task:** I needed XPath patterns that were both robust and readable, while establishing team conventions.

**Action:** I established XPath best practices and documented the patterns:
    
    
    // BAD - fragile positional XPath
    By.XPath("/html/body/div[3]/div[1]/table/tbody/tr[4]/td[2]")
    
    // BAD - too implementation-specific
    By.XPath("//div[@class='x-grid-cell-inner x-unselectable']")
    
    // GOOD - semantic, stable patterns
    // By attribute value
    By.XPath("//input[@data-field='taxRate']")
    // By text content  
    By.XPath("//button[normalize-space(text())='Calculate Tax']")
    // By label association
    By.XPath("//label[text()='Jurisdiction']/following-sibling::select")
    // By partial text (case-insensitive)
    By.XPath("//*[contains(translate(text(),'ABCDEF','abcdef'), 'tax period')]")
    // Parent by child
    By.XPath("//td[contains(@class,'jurisdiction-cell')]/parent::tr")
    // Ancestor traversal
    By.XPath("//span[@class='validation-error']/ancestor::div[@class='form-field']")
    // Axes for complex relations
    By.XPath("//th[text()='Amount']/following-sibling::th[1]") // Next sibling header
    
    

**Result:** XPath breakage rate dropped significantly after adopting semantic patterns. Code reviews enforced the conventions, and new engineers adopted the readable patterns immediately because the documentation examples were directly from the project's real DOM structures.

* * *

### Q88. How did you handle test isolation in a shared test environment?

**Situation:** On the J&J Salesforce project, the UAT environment was shared between the automation team and manual testers, causing tests to interfere with each other through shared data.

**Task:** I needed a test isolation strategy that worked without exclusive environment ownership.

**Action:** I implemented strict data ownership through unique namespacing:
    
    
    public class TestDataNamespace
    {
        private static readonly string RunId = $"AUTO_{DateTime.UtcNow:yyyyMMddHHmm}_{Environment.MachineName}";
    
        public static string ForCase(string baseName) => $"{RunId}_{baseName}";
        public static string ForUser(string role)      => $"auto.{role}.{DateTime.Now.Ticks}@test.jnj.com";
        
        // Filter manual data from automation
        public static bool IsAutomationData(string identifier) 
            => identifier?.StartsWith("AUTO_") == true;
    }
    
    // Usage - every created entity is uniquely namespaced
    var caseTitle = TestDataNamespace.ForCase("HR_Leave_Request");
    // Produces: "AUTO_202401151430_AGENT01_HR_Leave_Request"
    
    // Cleanup: only delete automation-created data
    public async Task CleanupAutomationData()
    {
        var allCases = await ApiClient.GetCasesAsync();
        var automationCases = allCases.Where(c => TestDataNamespace.IsAutomationData(c.Title));
        foreach (var c in automationCases)
            await ApiClient.DeleteCaseAsync(c.Id);
    }
    
    

**Result:** Automation data was completely isolated from manual test data. Automated cleanup ran reliably without risk of deleting manually created test records. The pattern also enabled safe parallel execution in the shared environment.

* * *

### Q89. How did you implement security credential management in C# automation?

**Situation:** On the AXA project, test credentials were initially stored in plaintext in configuration files that were committed to the Git repository.

**Task:** I needed to remove credentials from source control and implement secure credential management.

**Action:** I implemented environment variable-based credential management:
    
    
    public class SecureCredentialManager
    {
        // Never hardcode credentials - always from environment
        public static string GetUsername()
            => Environment.GetEnvironmentVariable("TEST_USERNAME")
               ?? throw new InvalidOperationException("TEST_USERNAME environment variable not set.");
    
        public static string GetPassword()
            => Environment.GetEnvironmentVariable("TEST_PASSWORD")
               ?? throw new InvalidOperationException("TEST_PASSWORD environment variable not set.");
    
        // For CI: credentials stored in Jenkins credentials store
        // For local: stored in .env file (gitignored)
    }
    
    
    
    
    # .gitignore
    .env
    appsettings.local.json
    auth-state.json
    
    # .env (local only, never committed)
    TEST_USERNAME=automation.user@axa.ie
    TEST_PASSWORD=SECURE_PASS_HERE
    
    
    
    
    // Jenkins - inject from credentials store
    withCredentials([
        usernamePassword(credentialsId: 'axa-test-credentials', 
                         usernameVariable: 'TEST_USERNAME', 
                         passwordVariable: 'TEST_PASSWORD')
    ]) {
        sh 'dotnet test'
    }
    
    

**Result:** No credentials existed in the Git repository after the migration. Jenkins credentials store managed secrets securely with rotation capabilities. A security audit subsequently confirmed the repository had no credential exposure.

* * *

### Q90. How did you handle test execution on mobile viewport in C# Playwright?

**Situation:** On the AXA portal, the customer-facing insurance website had a responsive design that behaved differently on mobile viewports — some elements were hidden, menus collapsed, and touch interactions replaced hover events.

**Task:** I needed to test the mobile viewport experience as part of the standard regression suite.

**Action:** I implemented mobile device emulation using Playwright's device descriptors:
    
    
    public class MobileTests : BaseTest
    {
        protected override async Task<IBrowserContext> CreateContextAsync()
        {
            // Use predefined mobile device
            var iPhone = Playwright.Devices["iPhone 14"];
            return await Browser.NewContextAsync(iPhone);
        }
    
        [Test]
        public async Task Mobile_ClaimsJourney_Should_Work_On_iPhone14()
        {
            await Page.GotoAsync($"{Settings.BaseUrl}/claims");
            
            // Mobile-specific interactions
            await Page.Locator(".mobile-menu-hamburger").TapAsync();  // Tap, not click
            await Page.Locator(".nav-link-claims").TapAsync();
            
            await Expect(Page.Locator(".claims-mobile-container")).ToBeVisibleAsync();
            await Expect(Page.Locator(".desktop-sidebar")).ToBeHiddenAsync();  // Hidden on mobile
        }
    }
    
    // Custom viewport
    var context = await Browser.NewContextAsync(new()
    {
        ViewportSize = new ViewportSize { Width = 375, Height = 812 },
        IsMobile = true,
        HasTouch = true,
        UserAgent = "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)..."
    });
    
    

**Result:** Mobile viewport testing was integrated into the standard regression suite. Three responsive design defects — including a form field that was completely inaccessible on small screens — were caught that had been missed by desktop-only testing.

* * *

### Q91. How did you implement Selenium Grid for distributed test execution?

**Situation:** On the Thomson Reuters Tax project, the team needed to run cross-browser tests across multiple machines simultaneously to meet the test execution time budget.

**Task:** I needed to configure Selenium Grid for distributed cross-browser execution.

**Action:** I set up Selenium Grid and integrated it with the C# test framework:
    
    
    # Hub
    java -jar selenium-server.jar hub
    
    # Nodes
    java -jar selenium-server.jar node --hub http://hub:4444 --browser chrome
    java -jar selenium-server.jar node --hub http://hub:4444 --browser firefox
    
    
    
    
    // Remote WebDriver factory for Grid
    public static IWebDriver CreateRemoteDriver(BrowserType browserType, string gridUrl)
    {
        DriverOptions options = browserType switch
        {
            BrowserType.Firefox => new FirefoxOptions(),
            BrowserType.Edge    => new EdgeOptions(),
            _                   => new ChromeOptions()
        };
    
        // Enable video recording capability
        ((ChromeOptions)options).AddAdditionalOption("se:recordVideo", true);
        
        return new RemoteWebDriver(
            new Uri(gridUrl ?? "http://selenium-grid:4444"),
            options.ToCapabilities(),
            TimeSpan.FromSeconds(120) // Remote connection timeout
        );
    }
    
    // Docker Compose Grid (modern approach)
    // docker-compose.yml with selenium/hub + selenium/node-chrome + selenium/node-firefox
    
    

**Result:** Selenium Grid enabled cross-browser parallel execution across dedicated machines. The setup tripled browser coverage without increasing execution time, as Chrome, Firefox, and Edge tests ran simultaneously on separate nodes.

* * *

### Q92. How did you handle timezone-related test failures in C# Playwright?

**Situation:** On the J&J Global Services project, date-based assertions were failing intermittently when tests ran across midnight UTC or when agents were in different timezones.

**Task:** I needed to make date handling timezone-aware throughout the test framework.

**Action:** I implemented timezone-explicit date handling:
    
    
    public class DateTimeHelper
    {
        // Always work in UTC internally; convert for display
        public static string FormatForUI(DateTime utcDate, string timezone = "Europe/Dublin")
        {
            var tzInfo = TimeZoneInfo.FindSystemTimeZoneById(timezone);
            var localDate = TimeZoneInfo.ConvertTimeFromUtc(utcDate, tzInfo);
            return localDate.ToString("dd/MM/yyyy");
        }
    
        // Parse UI dates with explicit timezone
        public static DateTime ParseUIDate(string dateStr, string timezone = "Europe/Dublin")
        {
            var parsed = DateTime.ParseExact(dateStr, "dd/MM/yyyy", null);
            var tzInfo = TimeZoneInfo.FindSystemTimeZoneById(timezone);
            return TimeZoneInfo.ConvertTimeToUtc(parsed, tzInfo);
        }
        
        // Relative dates (avoid hardcoding "today")
        public static string Today(string timezone = "Europe/Dublin")
            => FormatForUI(DateTime.UtcNow, timezone);
        
        public static string DaysAgo(int days, string timezone = "Europe/Dublin")
            => FormatForUI(DateTime.UtcNow.AddDays(-days), timezone);
    }
    
    // Usage in tests
    await Page.FillAsync("#incidentDate", DateTimeHelper.DaysAgo(3));
    
    

**Result:** Timezone-related failures disappeared entirely. The explicit timezone handling worked correctly regardless of the CI agent's system timezone, and the helper was adopted as the project standard for all date operations.

* * *

### Q93. How did you implement test dependency management in C#?

**Situation:** On the AXA project, some tests had implicit ordering dependencies — a policy needed to exist before a claim could be submitted — but this dependency wasn't explicit in the test structure.

**Task:** I needed to make test dependencies explicit and safe without creating brittle ordered execution.

**Action:** I replaced implicit test ordering with explicit API-based setup:
    
    
    // WRONG - implicit dependency on test execution order
    [Test, Order(1)]
    public async Task Create_Policy() { ... } // Must run before...
    
    [Test, Order(2)]
    public async Task Submit_Claim_Against_Policy() { ... } // ...this
    
    // CORRECT - explicit setup via API
    [TestFixture]
    public class ClaimSubmissionTests : BaseTest
    {
        private string _testPolicyRef;
    
        [OneTimeSetUp]
        public async Task CreateTestPolicy()
        {
            // Create pre-condition via API (fast, reliable)
            var response = await ApiContext.PostAsync("/api/policies", new()
            {
                DataObject = PolicyDataBuilder.Standard().Build()
            });
            var policy = await response.JsonAsync<PolicyResponse>();
            _testPolicyRef = policy.Reference;
        }
    
        [OneTimeTearDown]
        public async Task DeleteTestPolicy()
        {
            await ApiContext.DeleteAsync($"/api/policies/{_testPolicyRef}");
        }
    
        [Test]
        public async Task Should_Submit_Claim_For_Valid_Policy()
        {
            // Uses _testPolicyRef - no ordering dependency on other tests
            var claimsPage = new ClaimsPage(Page);
            await claimsPage.NavigateToNewClaim(_testPolicyRef);
            // ...
        }
    }
    
    

**Result:** Tests became truly independent — any test could run in any order or in isolation. The API-based setup was also significantly faster than UI-based setup, and the `OneTimeSetUp` pattern meant the policy was created once per fixture, not per test.

* * *

### Q94. How did you write effective test names and descriptions in C#?

**Situation:** On the Thomson Reuters project, test names like `Test1`, `TestValidation`, and `CheckTax` were meaningless in failure reports — it was impossible to understand what had broken from the name alone.

**Task:** I needed to establish test naming conventions that made failure reports immediately informative.

**Action:** I introduced and enforced a naming convention:
    
    
    // Pattern: Should_[ExpectedBehaviour]_When_[Condition]
    
    // BAD - meaningless names
    [Test] public void Test1() { }
    [Test] public void CheckTax() { }
    [Test] public void TestValidation() { }
    
    // GOOD - self-documenting names
    [Test]
    public async Task Should_Calculate_UK_VAT_At_20_Percent_When_Standard_Rate_Applied() { }
    
    [Test]
    public async Task Should_Reject_Claim_When_Policy_Is_Expired() { }
    
    [Test]
    public async Task Should_Display_Validation_Error_When_Mandatory_Field_Is_Empty() { }
    
    [Test]
    public async Task Should_Return_404_When_Policy_Reference_Does_Not_Exist() { }
    
    // With NUnit Description for complex scenarios
    [Test]
    [Description("Validates that the FNOL submission correctly maps all claim data to the backend, including regulatory compliance checkpoints for UK FCA requirements")]
    public async Task Should_Persist_All_FNOL_Data_Correctly_For_FCA_Compliance() { }
    
    

**Result:** Jenkins failure reports became self-explanatory. Stakeholders could read a list of failing test names and immediately understand what functionality was broken. The naming convention also improved test discoverability — engineers searching for existing coverage could find relevant tests by name.

* * *

### Q95. How did you implement accessibility testing in C# Playwright?

**Situation:** On the AXA project, the client had accessibility compliance requirements (WCAG 2.1 AA) for their insurance portal, and manual accessibility audits were being conducted too infrequently.

**Task:** I needed to integrate automated accessibility checking into the regression suite.

**Action:** I integrated Axe accessibility engine with Playwright C#:
    
    
    // Install: Deque.AxeCore.Playwright
    
    [Test]
    public async Task Policy_Dashboard_Should_Meet_WCAG_AA_Standards()
    {
        await Page.GotoAsync($"{Settings.BaseUrl}/dashboard");
        await Page.WaitForLoadStateAsync(LoadState.NetworkIdle);
    
        var axe = new AxeBuilder(Page)
            .WithTags("wcag2a", "wcag2aa", "wcag21aa")
            .Exclude(".live-chat-widget") // Third-party widget - not in scope
            .Analyze();
    
        var violations = axe.Violations;
        
        if (violations.Any())
        {
            var report = string.Join("\n\n", violations.Select(v =>
                $"[{v.Impact.ToUpper()}] {v.Help}\n" +
                $"Description: {v.Description}\n" +
                $"Affected elements: {string.Join(", ", v.Nodes.Select(n => n.Target))}"));
            
            Assert.Fail($"Accessibility violations found:\n{report}");
        }
    }
    
    // Targeted component accessibility check
    [Test]
    public async Task Claims_Form_Should_Have_Proper_Label_Associations()
    {
        var claimsSection = Page.Locator("#claims-form");
        var axe = new AxeBuilder(Page)
            .Include("#claims-form")
            .WithRules("label", "form-field-multiple-labels");
        
        Assert.That(axe.Analyze().Violations, Is.Empty);
    }
    
    

**Result:** Accessibility violations were caught automatically in CI. Three WCAG violations — missing form labels, insufficient colour contrast, and missing ARIA attributes — were identified and fixed before client review, preventing accessibility compliance findings.

* * *

### Q96. How did you implement test documentation within C# test code?

**Situation:** On long-running projects like J&J (30 months), test scenarios became complex enough that their intent was unclear without external documentation.

**Task:** I needed inline documentation that stayed current with the test code and communicated intent to future maintainers.

**Action:** I established documentation conventions at multiple levels:
    
    
    /// <summary>
    /// Validates the complete FNOL (First Notification of Loss) claim submission workflow
    /// for Comprehensive policy holders. Covers the regulatory FNOL-1 to FNOL-5 journey
    /// as defined in AXA's FCA compliance framework.
    /// </summary>
    /// <remarks>
    /// Pre-conditions: Active Comprehensive policy must exist (created in OneTimeSetUp).
    /// Regulatory reference: FCA ICOBS 8.1.2 - Claims handling requirements.
    /// Related JIRA: AXA-1234, AXA-1235.
    /// </remarks>
    [Test]
    [Description("Full FNOL journey - regulatory compliance validation (FCA ICOBS 8.1.2)")]
    public async Task Should_Complete_Full_FNOL_Journey_For_FCA_Compliance()
    {
        // ARRANGE: Navigate to claims and verify start state
        var claimsPage = new ClaimsPage(Page);
        await claimsPage.NavigateToNewClaim(_testPolicyRef);
    
        // ACT: Complete claim submission (5-step wizard)
        // Step 1: Claim type and incident details
        await claimsPage.Step1_SelectClaimType("Vehicle Theft");
        // Step 2-5 follow...
        
        // ASSERT: Verify compliance checkpoints
        // Checkpoint 1: Acknowledgement within 5 business days (FCA requirement)
        await Expect(Page.Locator(".acknowledgement-sent-indicator")).ToBeVisibleAsync();
    }
    
    

**Result:** Test documentation was always current because it lived with the code. New engineers understood the regulatory context of tests immediately, reducing the risk of "refactoring" tests that appeared redundant but were covering specific compliance requirements.

* * *

### Q97. How did you implement API contract testing between frontend and backend teams?

**Situation:** On the AXA project, the frontend Playwright automation and backend API teams were working in parallel sprints. Frontend tests frequently broke when backend API changes were deployed without coordination.

**Task:** I needed a lightweight contract testing mechanism that flagged breaking API changes before they caused UI test failures.

**Action:** I implemented consumer-driven contract tests as a dedicated fast-running suite:
    
    
    [TestFixture]
    [Category("Contract")]
    public class PolicyApiContractTests : ApiTestBase
    {
        [Test]
        public async Task GET_Policy_Should_Return_Required_Fields()
        {
            var response = await ApiContext.GetAsync("/api/policies/POL-CONTRACT-TEST");
            Assert.That(response.Status, Is.EqualTo(200));
    
            var body = JsonDocument.Parse(await response.TextAsync()).RootElement;
    
            // Assert required consumer fields exist with correct types
            AssertField(body, "policyNumber",  JsonValueKind.String);
            AssertField(body, "status",        JsonValueKind.String);
            AssertField(body, "policyType",    JsonValueKind.String);
            AssertField(body, "effectiveDate", JsonValueKind.String);
            AssertField(body, "premium",       JsonValueKind.Number);
        }
    
        [Test]
        public async Task POST_Claim_Should_Accept_Required_Fields()
        {
            var response = await ApiContext.PostAsync("/api/claims", new()
            {
                DataObject = new { policyNumber = "POL-12345", claimType = "Theft", incidentDate = "2024-01-01" }
            });
            Assert.That(response.Status, Is.AnyOf(200, 201));
        }
    
        private void AssertField(JsonElement root, string field, JsonValueKind expectedKind)
        {
            Assert.That(root.TryGetProperty(field, out var prop), Is.True, $"Missing field: '{field}'");
            Assert.That(prop.ValueKind, Is.EqualTo(expectedKind), $"Field '{field}' wrong type");
        }
    }
    
    

**Result:** Contract tests ran in under 2 minutes and were triggered on every backend deployment. Three breaking API changes — all field removals — were caught before they caused UI automation failures, eliminating what had previously been a source of regular sprint disruption.

* * *

### Q98. How did you implement wait strategies for AJAX-heavy applications in Selenium?

**Situation:** On the Thomson Reuters Tax project, the application made dozens of AJAX calls during page interactions, and standard waits frequently either timed out prematurely or waited longer than necessary.

**Task:** I needed fine-grained, condition-based waiting that accurately tracked AJAX completion.

**Action:** I implemented JavaScript-based AJAX completion detection:
    
    
    public class AjaxWaitHelper
    {
        private readonly IJavaScriptExecutor _js;
        private readonly WebDriverWait _wait;
    
        // Wait for jQuery AJAX calls to complete
        public void WaitForJQueryAjax()
        {
            _wait.Until(driver =>
            {
                var activeAjax = _js.ExecuteScript("return jQuery.active") as long?;
                return activeAjax == 0;
            });
        }
    
        // Wait for Angular HTTP calls to complete
        public void WaitForAngularHttp()
        {
            _wait.Until(driver =>
            {
                try
                {
                    return _js.ExecuteScript(
                        "return window.getAllAngularTestabilities().filter(x=>!x.isStable()).length === 0;") 
                        is true;
                }
                catch { return false; }
            });
        }
    
        // Generic network idle (no active XHR/Fetch)
        public void WaitForNetworkIdle()
        {
            _js.ExecuteScript(@"
                window.__pendingRequests = 0;
                var origOpen = XMLHttpRequest.prototype.open;
                XMLHttpRequest.prototype.open = function() {
                    window.__pendingRequests++;
                    this.addEventListener('loadend', function() { window.__pendingRequests--; });
                    origOpen.apply(this, arguments);
                };");
            
            _wait.Until(driver => 
                ((long?)_js.ExecuteScript("return window.__pendingRequests ?? 0")) == 0);
        }
    }
    
    

**Result:** AJAX wait handling was precise and fast. Tests waited exactly as long as needed — no more fixed sleeps, no premature interactions. The network idle detection approach proved particularly robust on the Thomson Reuters application's complex multi-call page loads.

* * *

### Q99. How did you implement test health metrics and reporting dashboards?

**Situation:** Across long-running projects, understanding the health of the automation suite over time was as important as individual test results. Trends in pass rate, execution time, and flakiness needed to be visible.

**Task:** I needed a metrics strategy that tracked suite health over time and surfaced actionable insights.

**Action:** I implemented metrics collection and integrated with ReportPortal trends:
    
    
    public class TestMetricsCollector
    {
        private readonly List<TestExecutionMetric> _metrics = new();
    
        public void RecordTestExecution(string testName, bool passed, long durationMs, 
            int retryCount, string category)
        {
            _metrics.Add(new TestExecutionMetric
            {
                TestName   = testName,
                Passed     = passed,
                DurationMs = durationMs,
                RetryCount = retryCount,
                Category   = category,
                Timestamp  = DateTime.UtcNow,
                BuildNumber = Environment.GetEnvironmentVariable("BUILD_NUMBER"),
                Environment = Environment.GetEnvironmentVariable("TEST_ENV")
            });
        }
    
        [AfterTestRun]
        public void PublishMetrics()
        {
            var summary = new
            {
                TotalTests    = _metrics.Count,
                PassRate      = _metrics.Count(m => m.Passed) * 100.0 / _metrics.Count,
                FlakyTests    = _metrics.Count(m => m.RetryCount > 0),
                AvgDurationMs = _metrics.Average(m => m.DurationMs),
                Slowest       = _metrics.OrderByDescending(m => m.DurationMs).Take(5).Select(m => m.TestName)
            };
            
            Console.WriteLine($"[METRICS] Pass Rate: {summary.PassRate:F1}% | Flaky: {summary.FlakyTests} | Avg Duration: {summary.AvgDurationMs:F0}ms");
            
            // Write to JSON for external dashboard consumption
            File.WriteAllText("metrics/execution-summary.json", JsonSerializer.Serialize(summary, new JsonSerializerOptions { WriteIndented = true }));
        }
    }
    
    

**Result:** Suite health trends were visible over time. The metrics revealed a gradual increase in average test duration over six months — traced to accumulated sleeps introduced by junior engineers. The flakiness count trend drove the systematic root-cause analysis described in Q83.

* * *

### Q100. What is your approach to keeping a C# Playwright/Selenium framework maintainable over years?

**Situation:** Having worked on the J&J Salesforce project for 30 months and the Google Street View project for 27 months, I've experienced first-hand what it takes to keep a framework healthy over multi-year engagements.

**Task:** I needed a long-term maintenance philosophy, not just technical solutions, to ensure the framework remained an asset rather than a liability over time.

**Action:** I applied seven principles consistently across all long-running projects:
    
    
    1. SIMPLICITY FIRST
       Design for the engineer who joins in month 18, not month 1.
       If a pattern needs a long explanation, simplify it.
    
    2. DOCS AS CODE
       All architectural decisions documented as ADRs in the repo.
       README always runnable end-to-end by a new joiner.
    
    3. AUTOMATION HEALTH BACKLOG
       Dedicate 15-20% of each sprint to framework maintenance.
       Track automation debt as first-class backlog items.
    
    4. STRICT DEPENDENCY MANAGEMENT
       Pin NuGet dependency versions in .csproj.
       Test browser/framework upgrades in a dedicated branch.
       Review breaking changes in Playwright/Selenium changelogs each minor version.
    
    5. ZERO TOLERANCE FOR HARDCODING
       Regular code reviews flagging any hardcoded values.
       Automated linting rules for common anti-patterns.
    
    6. TEST THE TESTS
       Monthly "framework audit" — run smoke tests, check execution time trends,
       review flaky test count, check for stale locators.
    
    7. SHARED OWNERSHIP
       Rotate framework ownership responsibilities.
       Everyone on the QA team owns the framework, not just the lead.
    
    
    
    
    // Architectural Decision Record example (ADR-003)
    // Title: Use Playwright Locator API over ElementHandle
    // Status: Accepted
    // Context: ElementHandle creates stale references on DOM re-renders.
    // Decision: All locators use ILocator. ElementHandle use requires team lead approval.
    // Consequences: Tests are more stable. ElementHandle-specific capabilities require wrappers.
    
    

**Result:** Both the J&J and Google frameworks were production-quality and actively maintained at project end — not deprecated legacy burdens. The automation-first In-Sprint model was sustainable because the framework remained clean, the team shared ownership, and technical debt was addressed systematically rather than accumulated silently.

* * *

_End of C# Playwright & Selenium — 100 Automation Aptitude Questions & Answers_ _Prepared for Shajeeb Shahul Hameed | Senior QA Automation Lead Interviews_
