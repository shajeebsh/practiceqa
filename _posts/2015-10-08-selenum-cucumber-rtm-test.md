---
title: "Selenum cucumber - RTM Test"
date: 2015-10-08 04:13:23 
author: shajeebhameed
layout: page
slug: selenum-cucumber-rtm-test
status: publish
---

**Feature** : poc for framework **Background** : **Given** navigate to "http://www.rememberthemilk.com/" @firsttest **Scenario Outline** : first test #**Given** navigate to "http://www.rememberthemilk.com/" **When** login with username as "<name>" **And** password as "<pwd>" **Then** login should be successfull **And** logout the application Examples: |name|pwd| |shaji.sss|1234| |shaji.ss|1234| public class AbstractBasePage { protected WebDriver driver; public AbstractBasePage(WebDriver drvr){ this.driver = drvr; } protected WebDriver getDriver(){ if(driver ==null){ driver = new FirefoxDriver(); } return driver; } public LandingPage navigateToRTMPage(String link){ this.driver.navigate().to(link); return new LandingPage(this.driver); } public void closeDriver(){ this.driver.close(); } public void quitDriver(){ this.driver.quit(); } } public class LandingPage extends AbstractBasePage { public LandingPage(WebDriver driver) { super(driver); } public LoginPage navigateToLoginPage(){ this.driver.findElement(By.className("login-link")).click(); return new LoginPage(this.driver); } } 

public class LoginPage extends AbstractBasePage {

public LoginPage(WebDriver drvr) {

super(drvr);

}

public LoginPage setUsername(String username){

driver.findElement(By.id("username")).sendKeys(username);

return new LoginPage(this.driver);

}

public LoginPage setPassword(String password){

driver.findElement(By.id("password")).sendKeys(password);

return new LoginPage(this.driver);

}

public HomePage clickLogin(){

driver.findElement(By.id("login-button")).click();

return new HomePage(this.driver);

}

}public class HomePage extends AbstractBasePage {

public HomePage(WebDriver drvr) {

super(drvr);

}

public HomePage verifyHomePage(){

Assert.assertTrue("Not found on landing page", this.driver.getTitle().equals("Remember The Milk - Shaji's Tasks"));

return new HomePage(this.driver);

}

public LoginPage clickLogout(){

WebDriverWait wait = new WebDriverWait(driver, 30);

WebElement logoutLink = driver.findElement(By.linkText("Logout"));

wait.until(ExpectedConditions.elementToBeClickable(logoutLink));

logoutLink.click();

return new LoginPage(this.driver);

}

}

public class testStepDefinition extends BaseStepDefinition {

WebDriver driver = getDriver();

LandingPage landingPage;

LoginPage loginPage;

HomePage homePage;

@Given("^navigate to \"([^\"]*)\"$")

public void navigate_to(String arg1) throws Throwable {

//driver = new FirefoxDriver();

//driver.get(arg1);

//driver.navigate().to(arg1); //When click login link

// driver.findElement(By.className("login-link")).click();

landingPage = new LandingPage(driver);

landingPage.navigateToRTMPage(arg1);

}

@When("^click login link$")

public void click_login_link() throws Throwable {

//driver.findElement(By.className("login-link")).click();

loginPage = landingPage.navigateToLoginPage();

}

@When("^login with username as \"([^\"]*)\"$")

public void login_with_username_as(String arg1) throws Throwable {

//driver.findElement(By.id("username")).sendKeys(arg1);

loginPage.setUsername(arg1);

}

@When("^password as \"([^\"]*)\"$")

public void password_as(String arg1) throws Throwable {

//driver.findElement(By.id("password")).sendKeys(arg1);

//driver.findElement(By.id("login-button")).click();

loginPage.setPassword(arg1);

homePage = loginPage.clickLogin();

}

@Then("^login should be successfull$")

public void login_should_be_successfull() throws Throwable {

//Assert.assertTrue("Not found on landing page", driver.getTitle().equals("Remember The Milk - Shaji's Tasks"));

homePage.verifyHomePage();

}

@Then("^logout the application$")

public void logout_the_application() throws Throwable {

//WebDriverWait wait = new WebDriverWait(driver, 30);

//WebElement logoutLink = driver.findElement(By.linkText("Logout"));

//wait.until(ExpectedConditions.elementToBeClickable(logoutLink));

//logoutLink.click();

//driver.close();

loginPage = homePage.clickLogout();

loginPage.closeDriver();

}

@After

public void close(){

loginPage.quitDriver();

}

}

@RunWith(Cucumber.class)

@Cucumber.Options(

format={"pretty", "html:target/html/"},

features={"src/cucumber/"},

tags={"@firsttest", "@secondtest"}

)

public class CucuRunner {

}

//https://www.youtube.com/watch?v=wNhQSFpAg_o&index=4&list=PL_noPv5wmuO_t6yYbPfjwhJFOOcio89tI
