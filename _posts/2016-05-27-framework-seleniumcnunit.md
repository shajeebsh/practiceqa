---
title: "Framework -Selenium,C#,Nunit"
date: 2016-05-27 16:21:05 
author: shajeebhameed
layout: page
slug: framework-seleniumcnunit
status: publish
---

namespace ClsTest.Pages

{

public class HomePage

{

private IWebDriver _driver = null;

private string _baseUrl = null;

public HomePage(IWebDriver driver, string baseUrl)

{

_driver = driver;

_baseUrl = baseUrl;

}

public HomePage Navigate()

{

_driver.Navigate().GoToUrl(_baseUrl);

return this;

}

}

}

using OpenQA.Selenium;

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

namespace ClsTest.Pages

{

public class PackagesPage

{

private IWebDriver _driver = null;

private string _baseUrl = null;

public PackagesPage(IWebDriver driver, string baseUrl)

{

_driver = driver;

_baseUrl = baseUrl;

}

public PackagesPage Navigate()

{

_driver.Navigate().GoToUrl(_baseUrl);

return this;

}

public PackagesPage UploadPackage()

{

_driver.FindElement(By.LinkText("Upload Package")).Click();

return this;

}

}

}

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using OpenQA.Selenium;

using NUnit.Framework;

using ClsTest.Utils;

namespace ClsTest.Tests

{

[TestFixture]

public class Booking

{

private IWebDriver _driver = null;

private string _baseUrl = null;

[TestFixtureSetUp]

public void TestInitialize()

{

_driver = SingleTonClass.Instance.Driver;

}

[TestFixtureTearDown]

public void TestCleanup()

{

}

[Test]

[TestCase("11111", " Test case now 1111 ")]

public void Homepage(string tcId, string tcDescription)

{

var objNewConf = new PageFactory(_driver)

.HomePage()

.Navigate();

}

[Test]

[TestCase("2222", " Test case now 2222 ")]

public void PackagePage(string tcId, string tcDescription)

{

new PageFactory(_driver).Packages()

.Navigate();

}

[Test]

[TestCase("3333", " Test case now 3333 ")]

public void UploadPackage(string tcId, string tcDescription)

{

new PageFactory(_driver)

.Packages()

.Navigate()

.UploadPackage();

}

}

}

using OpenQA.Selenium;

using OpenQA.Selenium.Firefox;

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

namespace ClsTest.Utils

{

public class SingleTonClass

{

private SingleTonClass()

{

}

private static IWebDriver _driver = null;

private static readonly Lazy<SingleTonClass> _instance = new Lazy<SingleTonClass>(MakeSingleTonInstance);

private static SingleTonClass MakeSingleTonInstance()

{

return new SingleTonClass();

}

public static SingleTonClass Instance

{

get

{

return _instance.Value;

}

}

public IWebDriver Driver

{

get

{

_driver = new FirefoxDriver();

return _driver;

}

}

~SingleTonClass()

{

if (_driver != null)

_driver.Quit();

}

}

}

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using OpenQA.Selenium;

using System.Threading.Tasks;

using ClsTest.Pages;

namespace ClsTest.Utils

{

public class PageFactory

{

private IWebDriver _driver = null;

public PageFactory(IWebDriver driver)

{

_driver = driver;

}

public HomePage HomePage()

{

return new HomePage(_driver, "https://www.nuget.org/");

}

public PackagesPage Packages()

{

return new PackagesPage(_driver, "https://www.nuget.org/packages");

}

}

}

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

namespace ClsTest.Utils

{

public interface ITestTag

{

bool HasCorrectItemTitle { get; }

string Id { get; }

string Link { get; }

string Title { get; }

string GetInfo();

bool UpdateItem(NUnit.Core.TestResult result);

}

}
