---
title: "CodeChallenge15-Playwright"
date: 2026-04-21 21:48:14 
author: shajeebhameed
layout: page
slug: codechallenge15-playwright
status: publish
---

## Playwright C# Automation Testing — Common Coding Challenges & Answers

* * *

## 1\. Launch a Browser and Navigate to a URL

**Challenge:** Write a test that launches a Chromium browser, opens a new page, navigates to `https://example.com`, and asserts the page title contains "Example".
    
    
    using Microsoft.Playwright;
    using Microsoft.Playwright.NUnit;
    using NUnit.Framework;
    
    [TestFixture]
    public class NavigationTest : PageTest
    {
        [Test]
        public async Task ShouldHaveCorrectTitle()
        {
            await Page.GotoAsync("https://example.com");
            await Expect(Page).ToHaveTitleAsync(new Regex("Example"));
        }
    }
    
    

**Key Points:**

  * `PageTest` base class from `Microsoft.Playwright.NUnit` provides a ready-to-use `Page` instance.
  * `Expect(Page).ToHaveTitleAsync()` uses Playwright's built-in auto-waiting assertions.



* * *

## 2\. Locating Elements and Clicking

**Challenge:** Navigate to a login page, fill in the username and password fields, and click the submit button.
    
    
    [Test]
    public async Task LoginTest()
    {
        await Page.GotoAsync("https://practicetestautomation.com/practice-test-login/");
    
        await Page.Locator("#username").FillAsync("student");
        await Page.Locator("#password").FillAsync("Password123");
        await Page.Locator("#submit").ClickAsync();
    
        await Expect(Page.Locator(".post-title")).ToHaveTextAsync("Logged In Successfully");
    }
    
    

**Key Points:**

  * `FillAsync()` clears the field and types the value.
  * `Locator()` supports CSS selectors, text, roles, and more.
  * Assertions auto-wait for elements to meet conditions.



* * *

## 3\. Handling Dropdowns

**Challenge:** Select an option from a `<select>` dropdown by its visible label.
    
    
    [Test]
    public async Task SelectDropdownOption()
    {
        await Page.GotoAsync("https://the-internet.herokuapp.com/dropdown");
    
        var dropdown = Page.Locator("#dropdown");
        await dropdown.SelectOptionAsync(new SelectOptionValue { Label = "Option 1" });
    
        await Expect(dropdown).ToHaveValueAsync("1");
    }
    
    

**Key Points:**

  * `SelectOptionAsync()` accepts value, label, or index.
  * `SelectOptionValue` allows specifying by `Value`, `Label`, or `Index`.



* * *

## 4\. Handling Checkboxes and Radio Buttons

**Challenge:** Check a checkbox and verify it is checked.
    
    
    [Test]
    public async Task CheckboxTest()
    {
        await Page.GotoAsync("https://the-internet.herokuapp.com/checkboxes");
    
        var checkbox = Page.Locator("input[type='checkbox']").First;
    
        if (!await checkbox.IsCheckedAsync())
            await checkbox.CheckAsync();
    
        await Expect(checkbox).ToBeCheckedAsync();
    }
    
    

**Key Points:**

  * `CheckAsync()` / `UncheckAsync()` handle checkbox state safely.
  * `IsCheckedAsync()` returns the current state for conditional logic.



* * *

## 5\. Handling Alerts and Dialogs

**Challenge:** Accept a JavaScript `alert` dialog and verify the message.
    
    
    [Test]
    public async Task HandleAlertDialog()
    {
        string? alertMessage = null;
    
        Page.Dialog += async (_, dialog) =>
        {
            alertMessage = dialog.Message;
            await dialog.AcceptAsync();
        };
    
        await Page.GotoAsync("https://the-internet.herokuapp.com/javascript_alerts");
        await Page.Locator("button", new() { HasText = "Click for JS Alert" }).ClickAsync();
    
        Assert.That(alertMessage, Is.EqualTo("I am a JS Alert"));
    }
    
    

**Key Points:**

  * Subscribe to `Page.Dialog` event **before** triggering the dialog.
  * `dialog.AcceptAsync()` accepts; `dialog.DismissAsync()` cancels.



* * *

## 6\. Waiting for Elements (Explicit Waits)

**Challenge:** Wait for a dynamically loaded element to appear before asserting.
    
    
    [Test]
    public async Task WaitForDynamicElement()
    {
        await Page.GotoAsync("https://the-internet.herokuapp.com/dynamic_loading/1");
        await Page.Locator("button", new() { HasText = "Start" }).ClickAsync();
    
        var loadingText = Page.Locator("#finish h4");
    
        // Playwright auto-waits, but you can set explicit timeout
        await Expect(loadingText)
            .ToBeVisibleAsync(new LocatorAssertionsToBeVisibleOptions { Timeout = 10_000 });
    
        await Expect(loadingText).ToHaveTextAsync("Hello World!");
    }
    
    

**Key Points:**

  * Playwright auto-waits by default; explicit timeouts override the default.
  * Avoid `Thread.Sleep()` — always use Playwright's async waiting mechanisms.



* * *

## 7\. Taking Screenshots

**Challenge:** Take a full-page screenshot and save it to disk.
    
    
    [Test]
    public async Task TakeFullPageScreenshot()
    {
        await Page.GotoAsync("https://example.com");
    
        await Page.ScreenshotAsync(new PageScreenshotOptions
        {
            Path = "screenshot.png",
            FullPage = true
        });
    
        Assert.That(File.Exists("screenshot.png"), Is.True);
    }
    
    

**Key Points:**

  * `FullPage = true` captures content beyond the viewport.
  * Element-level screenshots: `locator.ScreenshotAsync()`.



* * *

## 8\. Handling Multiple Windows / Tabs

**Challenge:** Click a link that opens a new tab and assert the URL in the new tab.
    
    
    [Test]
    public async Task HandleNewTab()
    {
        await Page.GotoAsync("https://the-internet.herokuapp.com/windows");
    
        var newPageTask = Page.Context.WaitForPageAsync();
        await Page.Locator("a", new() { HasText = "Click Here" }).ClickAsync();
        var newPage = await newPageTask;
    
        await newPage.WaitForLoadStateAsync();
        Assert.That(newPage.Url, Does.Contain("new"));
    }
    
    

**Key Points:**

  * `Context.WaitForPageAsync()` must be called **before** the click that opens the tab.
  * The new tab is a separate `IPage` instance.



* * *

## 9\. Handling iFrames

**Challenge:** Switch into an iframe and interact with an element inside it.
    
    
    [Test]
    public async Task HandleIFrame()
    {
        await Page.GotoAsync("https://the-internet.herokuapp.com/iframe");
    
        var frame = Page.FrameLocator("#mce_0_ifr");
        var editor = frame.Locator("#tinymce");
    
        await editor.ClearAsync();
        await editor.FillAsync("Playwright iFrame Test");
    
        await Expect(editor).ToHaveTextAsync("Playwright iFrame Test");
    }
    
    

**Key Points:**

  * `FrameLocator()` scopes subsequent locators inside the iframe.
  * No explicit frame switching required — locators are composable.



* * *

## 10\. API Testing with Playwright

**Challenge:** Make a GET request to a REST API and assert the response status and body.
    
    
    [Test]
    public async Task ApiGetRequest()
    {
        var requestContext = await Playwright.APIRequest.NewContextAsync(new()
        {
            BaseURL = "https://jsonplaceholder.typicode.com"
        });
    
        var response = await requestContext.GetAsync("/posts/1");
    
        Assert.That(response.Status, Is.EqualTo(200));
    
        var body = await response.JsonAsync();
        Assert.That(body?.GetProperty("id").GetInt32(), Is.EqualTo(1));
    }
    
    

**Key Points:**

  * `Playwright.APIRequest` provides a lightweight HTTP client.
  * Useful for API setup/teardown steps in UI tests.



* * *

## 11\. Page Object Model (POM)

**Challenge:** Implement a Page Object Model for a login page.
    
    
    // LoginPage.cs
    public class LoginPage
    {
        private readonly IPage _page;
    
        public LoginPage(IPage page) => _page = page;
    
        public async Task GotoAsync() =>
            await _page.GotoAsync("https://practicetestautomation.com/practice-test-login/");
    
        public async Task LoginAsync(string username, string password)
        {
            await _page.Locator("#username").FillAsync(username);
            await _page.Locator("#password").FillAsync(password);
            await _page.Locator("#submit").ClickAsync();
        }
    
        public ILocator SuccessMessage => _page.Locator(".post-title");
    }
    
    // LoginTest.cs
    [TestFixture]
    public class LoginPageTest : PageTest
    {
        [Test]
        public async Task SuccessfulLogin()
        {
            var loginPage = new LoginPage(Page);
            await loginPage.GotoAsync();
            await loginPage.LoginAsync("student", "Password123");
    
            await Expect(loginPage.SuccessMessage).ToHaveTextAsync("Logged In Successfully");
        }
    }
    
    

**Key Points:**

  * POM separates page interactions from test logic.
  * Expose locators as properties for reuse and readability.



* * *

## 12\. Data-Driven Testing with NUnit

**Challenge:** Run the same login test with multiple sets of credentials using `[TestCase]`.
    
    
    [TestCase("student", "Password123", true)]
    [TestCase("wronguser", "wrongpass", false)]
    public async Task DataDrivenLoginTest(string username, string password, bool shouldSucceed)
    {
        await Page.GotoAsync("https://practicetestautomation.com/practice-test-login/");
        await Page.Locator("#username").FillAsync(username);
        await Page.Locator("#password").FillAsync(password);
        await Page.Locator("#submit").ClickAsync();
    
        if (shouldSucceed)
            await Expect(Page.Locator(".post-title")).ToHaveTextAsync("Logged In Successfully");
        else
            await Expect(Page.Locator("#error")).ToBeVisibleAsync();
    }
    
    

**Key Points:**

  * NUnit's `[TestCase]` drives parameterized tests cleanly.
  * Can also use `[TestCaseSource]` for larger datasets from a file or list.



* * *

## 13\. Intercepting Network Requests (Mocking)

**Challenge:** Intercept an API call and return a mocked response.
    
    
    [Test]
    public async Task MockApiResponse()
    {
        await Page.RouteAsync("**/api/users", async route =>
        {
            await route.FulfillAsync(new RouteFulfillOptions
            {
                Status = 200,
                ContentType = "application/json",
                Body = "[{\"id\":1,\"name\":\"Mock User\"}]"
            });
        });
    
        await Page.GotoAsync("https://your-app.com/users");
        await Expect(Page.Locator(".user-name")).ToHaveTextAsync("Mock User");
    }
    
    

**Key Points:**

  * `Page.RouteAsync()` intercepts matching requests.
  * Use `route.AbortAsync()` to simulate network failures.



* * *

## 14\. Keyboard and Mouse Actions

**Challenge:** Hover over an element and then use keyboard shortcuts.
    
    
    [Test]
    public async Task KeyboardAndMouseActions()
    {
        await Page.GotoAsync("https://example.com");
    
        // Hover
        await Page.Locator("h1").HoverAsync();
    
        // Type with keyboard
        await Page.Keyboard.PressAsync("Control+A");
        await Page.Keyboard.TypeAsync("Replaced text");
    
        // Mouse click at coordinates
        await Page.Mouse.ClickAsync(100, 200);
    }
    
    

**Key Points:**

  * `Page.Keyboard` handles key presses and combinations.
  * `Page.Mouse` allows precise coordinate-based interactions.



* * *

## 15\. Parallel Test Execution

**Challenge:** Configure tests to run in parallel across multiple browsers.
    
    
    // In .runsettings or NUnit config:
    // Set parallel workers to match browser count
    
    [TestFixture]
    [Parallelizable(ParallelScope.Self)]
    public class ParallelTests : PageTest
    {
        public override BrowserNewContextOptions ContextOptions() =>
            new() { ViewportSize = new ViewportSize { Width = 1280, Height = 720 } };
    
        [Test]
        public async Task TestOnChromium()
        {
            await Page.GotoAsync("https://example.com");
            await Expect(Page).ToHaveTitleAsync(new Regex("Example"));
        }
    }
    
    
    
    
    <!-- .runsettings -->
    <RunSettings>
      <Playwright>
        <BrowserName>chromium</BrowserName>
      </Playwright>
      <RunConfiguration>
        <MaxCpuCount>4</MaxCpuCount>
      </RunConfiguration>
    </RunSettings>
    
    

**Key Points:**

  * Use `[Parallelizable]` at the fixture or assembly level.
  * Set `BROWSER` env var or `.runsettings` to target different browsers (chromium, firefox, webkit).



* * *

## Quick Reference: Most Used Playwright C# APIs

Action| Method  
---|---  
Navigate| `Page.GotoAsync(url)`  
Find element| `Page.Locator(selector)`  
Click| `locator.ClickAsync()`  
Fill input| `locator.FillAsync(text)`  
Get text| `locator.InnerTextAsync()`  
Assert visible| `Expect(locator).ToBeVisibleAsync()`  
Assert text| `Expect(locator).ToHaveTextAsync(text)`  
Assert title| `Expect(page).ToHaveTitleAsync(text)`  
Wait for selector| `Page.WaitForSelectorAsync(selector)`  
Screenshot| `Page.ScreenshotAsync(options)`  
Intercept request| `Page.RouteAsync(pattern, handler)`  
New tab| `Context.WaitForPageAsync()`  
iFrame| `Page.FrameLocator(selector)`  
Select dropdown| `locator.SelectOptionAsync(value)`  
API request| `Playwright.APIRequest.NewContextAsync()`
