---
title: "Cucumber - Selenium"
date: 2015-06-16 11:21:30 
author: shajeebhameed
layout: page
slug: cucumber-selenium
status: publish
---

1. https://thomassundberg.wordpress.com/2014/05/29/cucumber-jvm-hello-world/
  2. https://c0deattack.wordpress.com/2012/03/28/cucumber-jvm-with-cucumber-java-cucumber-junit-example/
  3. http://www.hascode.com/2014/12/bdd-testing-with-cucumber-java-and-junit/

https://www.youtube.com/watch?v=pD4B839qfos&list=PL_noPv5wmuO_t6yYbPfjwhJFOOcio89tI \- Runner file - CucumberRunner.java \- features \- myfeature.feature \- StepDefenition.java Parameterization to pass variable sorround it wil "" eg "About" DataTable Scenario outline datatable Scenario Hooks - equalent to @Before and @after concept of junit @Before Tagged hooks - Run only in a particular scenario. @Before("@web") Maven - Used to build, manage dependencies like cucumber plugin etc also have Test Runner class -Test Runner class cucumber.Options{features={"src/test/resouces"} } Reporting - HTML and JSON format HTML - cucumber.Options{format={"pretty", "html:target/html/"} } JSON - cucumber.Options{format={"pretty", "json:target/json/output.json"} } Test Suites - Run multiple feature file at the same time. add attribute to Scenarios = like @Appilcation Screnario1, @webdvr screnario2, @srvr screnario3, etc... cucumber.Options{tags={"@Application, @webdvr"} } My EXAMPLE============================== Feature: poc for framework Scenario Outline: first test Given navigate to "http://www.rememberthemilk.com/" When login with username as "<name>" And password as "<pwd>" Then login should be successfull And logout the application Examples: |name|pwd| |shaji.sss|1234| |shaji.ss|1234| \-------------------- public class testStepDefinition { WebDriver driver = null; @Given("^navigate to \"([^\"]*)\"$") public void navigate_to(String arg1) throws Throwable { driver.get(arg1); driver.navigate().to(arg1); //When click login link driver.findElement(By.className("login-link")).click(); } @When("^click login link$") public void click_login_link() throws Throwable { driver.findElement(By.className("login-link")).click(); } @When("^login with username as \"([^\"]*)\"$") public void login_with_username_as(String arg1) throws Throwable { driver.findElement(By.id("username")).sendKeys(arg1); } @When("^password as \"([^\"]*)\"$") public void password_as(String arg1) throws Throwable { driver.findElement(By.id("password")).sendKeys(arg1); driver.findElement(By.id("login-button")).click(); } @Then("^login should be successfull$") public void login_should_be_successfull() throws Throwable { Assert.assertTrue("Not found on landing page", driver.getTitle().equals("Remember The Milk - Shaji's Tasks")); } @Then("^logout the application$") public void logout_the_application() throws Throwable { WebDriverWait wait = new WebDriverWait(driver, 30); WebElement logoutLink = driver.findElement(By.linkText("Logout")); wait.until(ExpectedConditions.elementToBeClickable(logoutLink)); logoutLink.click(); driver.close(); } @Before public void testSetup(){ driver = new FirefoxDriver(); } @After public void close(){ driver.quit(); } } ====================== Drop Downs, Checkboxes and Radios Buttons Select dropdown = new Select(driver.findElement(By.id("country"))); dropdown.selecyByVisibleText("India") driver.findElement(By.id("sex")).click(); driver.findElement(By.name("anem")).sendKeys("ammasd"); Background Script for Feature file \- Taking out of some common steps \- Add Background: in the feature file and add those repeated common steps under it. This will execute first before running each scenarios \- equalent to @before but in Feature file Multiple Step Definition Classes \- Seperate SD class for each functionality or whatever our convenient. Can also use abstract class with inteheritance. Page Objects \- Refactoring of Step defenition classes using abstract class, inheritance etc. ========================== import org.junit.runner.RunWith; @RunWith(Cucumber.class) @Cucumber.Options( format = {"pretty", "json:targetfolder/json.output.json", "html:targetfolder/html/"} features = "src/test/resource" tags = {"@Application","@someother"} ) =========================== 
  * **SpecFlow was inspired by[Cucumber](<http://cukes.info/>) and is using [Gherkin](<https://github.com/cucumber/gherkin/wiki>)**

https://visualstudiogallery.msdn.microsoft.com/9915524d-7fb0-43c3-bb3c-a8a14fbd40ee http://www.specflow.org/whatsnew-19/ 

### What are Features and Scenarios?

A feature can be conceptualized as an indivisible unit of functionality embedded in the project to which it belongs. For example, an authentication challenge and response user interface is usually considered a feature while an entire authentication system necessarily comprises many features. A single **_Feature_** is typically contained in its own file (ending in `.feature`). Each Feature is usually elaborated though multiple **_Scenarios_**. A Scenario is a block of statements inside a feature file that describe some behaviour desired or deprecated in the feature to which it belongs. A scenario might check that the login interface provides input fields for the requisite responses, that failures are logged or otherwise reported, that user ids are locked out after a given number of failed attempts, and so forth. Each scenario exercises the implementation code to prove that for each anticipated condition the expected behaviour is indeed produced. Recall that scenarios specify **What** and should avoid answering the question: **How**? Each Scenario consists of three classes of statements, **Given** , **When** and **Then** which effectively divide each scenario into three stages. Each stage of a scenario consists of one or more statements that are used to match to test step definitions. The conventional arrangement is: 
    
    
    Feature: Some terse yet descriptive text of what is desired
    In order that some business value is realized
    An actor with some explicit system role
    Should obtain some beneficial outcome which furthers the goal
    To Increase Revenue | Reduce Costs | Protect Revenue  (pick one)
    
      Scenario:   Some determinable business situation
          Given some condition to meet
             And some other condition to meet
           When some action by the actor
             And some other action
             And yet another action
           Then some testable outcome is achieved
             And something else we can check happens too
    
      Scenario:  A different situation
          ...
    

For Cucumber features, the keywords used here are **Feature** , **Scenario** , **Given** , **When** ,**Then** , and **And**. Feature is used to provide identification of the test group when results are reported. To date ( 2015 Jun 05 ) the `Feature` statement and its descriptive text block are not parsed by Cucumber other than as an identifier and documentation. Nonetheless, the `Feature`statement arguably contains the most important piece of information contained in a feature file. It is here that you answer the question of just why this work is being done. And if you do not have a very good, defensible, reason that can be clearly elucidated in a few sentences then you probably should not be expending any effort on this feature at all. First and foremost,**BDD** absolutely **must** have some concrete business value whose realization can be measured before you write a single line of code. _[(see popping the why? stack)](<http://www.mattblodgett.com/2009/01/pop-stack.html>)_ As with `Feature`, `Scenario` is used only for identification when reporting failures and to document a piece of the work. The clauses ( _steps_ ) that make up a Scenario each begin with one of: Given, When, Then, And and But ( _and sometimes*****_ ). These are all [Gherkin](<https://github.com/cucumber/cucumber/wiki/Gherkin>)keywords / Cucumber methods that take as their argument the string that follows. They are the steps that Cucumber will report as passing, failing or pending based on the results of the corresponding step matchers in the step_definitions.rb files. The five keywords ( _and*****_ ) are all equivalent to one another and completely interchangeable. 

### [](<https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder#what-are-step-definitions>)What are Step Definitions?

The string following the keyword in the feature file is compared against all the matchers contained in all of the loaded step_definitions.rb files. A step definition looks much like this: 
    
    
    Given /there are (\d+) froobles/ do |n|
      Frooble.transaction do
        Frooble.destroy_all
        n.to_i.times do |n|
          Frooble.create! :name => "Frooble #{n}"
        end
      end
    end
    

The significant things here are that the method (Given) takes as its argument a [regexp](<http://en.wikipedia.org/wiki/Regexp>)bounded by `/` ( _although a quoted “simple string” can be used instead_ ) and that the matcher method is followed by a block. In other words, written differently in Ruby this matcher method could look like this: 
    
    
    Given( /there are (\d+) froobles/ ) { |n|
    .  .  .
    }

Among other things, this means that the step definition method blocks receive all of the matcher arguments as string values. Thus `n.to_i.times` and not simply `n.times` ( _but also look into Cucumber transforms_ ). It also means that step matchers themselves can be followed by the special regexp modifiers, like **i** if you want to avoid issues involving capitalization. In the feature example provided above we had the scenario statement: `And some other action`. This could be matched by any of the following step definition matchers if present in any `step_definitions.rb` file found under the features root directory. 
    
    
    Given /some other action/ do
    Then "some other action" do
    When /some other Action/i do 
    When /some other (Action)/i do |action|
    Then /(\w+) other action/i do |prefix_phrase|
    Given /(\w+) other (\w+)/i do |first_word,second_word|
    But /(\w+) Other (.*)/i do |first_word,second_phrase|
    And /(.*) other (.*)/i do |first_phrase,second_phrase|
    

The step definition match depends only upon the pattern given as the argument passed to the Given/When/Then method and not upon the step method name itself. I have therefore adopted the practice of only using `When /I have a match/ do` in my step definitions files as **When** has a more natural appearance, to me, for a matcher. Others find that the word “Given” has a more natural language feel in this context. If Cucumber finds more than one matcher in all of the step definitions files matches a scenario statement then it complains that it has found multiple step definition matches for that step and forces you to distinguish them. You can instruct Cucumber to just choose one of the candidates instead by passing it the `--guess` option on the command line. It is considered better form by some to surround with double quotation marks, **`" "`** , all of the elements in the feature step clauses that are meant to be taken as values for variables passed to the step definition. This is just a convention. However, if you choose to follow this road then you must adjust your step definition matchers accordingly if the `"` characters are now considered part of the literal matcher string. For example: Feature statement: 
    
    
    Given some determinable "business" situation
    

step definition: 
    
    
    When /determinable "(.*)" situation/ do |s|
    

Finally, you can have step definitions call other step definitions, including those contained in other step definitions files. This is one ( _but not the recommended_ ) means by which you can specify the procedural details by combining other steps. For example: 
    
    
    When /some "(.*)" action/  do |act|
       .  .  .
    end
    
    When /in an invoiced non-shipped situation/ do
      step( "some \"invoiced\" action" )
      step( "some \"non-shipped\" action" )
      .  .  .
    end
    

### [](<https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder#steps-within-steps-an-anti-pattern>)Steps within Steps: An anti-pattern

If one step definition calls another step definition then the matcher argument to the called**Given/When/Then** method must be enclosed with string delimiters. Because of this, if you have adopted the practice of demarcating parameter values present in feature steps with double quotation marks, you must escape these quotation marks when calling another definition_steps.rb matcher from inside a step_definitions.rb file. You must take care not to include the quote marks in the step_definitions parameter matchers, for `"(.*)"` is not the same as `(.*)` or `(".*")`. If you use quote delimited values in the `.feature` file steps and do not account for them in the corresponding `step_definition.rb` matcher regexp then you will obtain variables that contain leading and trailing quotes as part of their value. 
    
    
    Scenario: Quotes surround value elements
      Given some "required" action
    
    # step_definitions
    When /some (.*) action/ do |a|
      a => "required"
    
    When /some "(.*)" action/ do |a|
      a => required
    

Once upon a time, one could simply nest **Given** , **When** , and **Then** matchers within step definitions and thereby directly call steps from within other steps. This practice led to increase coupling between `step_definition` files, then it was frowned upon, and finally the ability was removed (well, _strongly_ deprecated). If you use nested steps then you must now call them using the **`step`** method. You may still encounter the following forms in step definition files created before this change: 
    
    
    When /my matcher named (.*)/ do |match|
      Then "my other matcher named \"#{match}\""
    end
    
    When /my matcher named (.*)/ do |match|
      When %Q(my other matcher named "#{match}")
    end
    

When encountered these nested **Then**and**When** matcher statements should be replaced with the `step` method:****
    
    
    When /my matcher named (.*)/ do |match|
      step( "my other matcher named \"#{match}\"" )
    end
    
    When /my matcher named (.*)/ do |match|
      step %Q(my other matcher named "#{match}")
    end
    

Using the %Q method (usually shortened to just %) within the step method removes the necessity to escape (`\`) any embedded quotation characters (`"`). Multiple steps may be called _en bloc_ using the **`steps`** (_note the plural form_) method which itself takes a string argument. However, with the steps method, the Gherkin keywords deprecated in simple nested steps are still required: 
    
    
    When /my matcher named (.*)/ do |match|
      steps %Q{
        Then step my other matcher named "#{match}" 
         And the next matcher with value "{match}"
      }
    end
    

Always keep in mind that Cucumber is simply a DSL wrapper around the Ruby language whose full expressiveness remains available to you in the step definition files (_but not in the feature files_). On the other hand, do not lose sight that every step called as such in a step definition file is first parsed by [Gherkin](<https://github.com/cucumber/cucumber/wiki/Gherkin>) and therefore must conform to the same syntax as used in feature files. Returning to our example of “Bob” the user, one could define things in the`step_definitions` file like this: 
    
    
    When /"Bob" logs in/ do |user|
      steps( %Q(
        Then I visit "/login"
          And I enter "#{user}" in the "user name" field
          And I enter "#{user}-test-passwd" in the "password" field
          And I press the "login" button
      ) )
    

That is acceptable ( _barely_ ) usage in your step_definitions because your users are never going to see how ugly it looks. Instead, given that the necessary classes and methods exist, “Bob” could, and should, be authenticated without recourse to the user interface thus: 
    
    
    When /"Bob" logs in/ do |user|
      @current_user = User.find_by_username!(user) # ! method raises exception on failure
      @current_session = UserSession.create!(@current_user)  # ! method raises exception on failure
      .  .  .
    end
    

In fact, this step should be further refactored into a method. And said method can reside in the same `.rb` file as the step definition. This both simplifies the step and encourages the reuse of the resulting method. It also adds immeasurably to easy comprehension for people unfamiliar with the project history who may later have cause to review your tests. Can you spell **M-A-I-N-T-E-N-A-N-C-E**? 
    
    
    When /"Bob" logs in/ do |user|
      create_new_user_session_for( user )
    end
    
    
    
    def create_new_user_session_for( user_in )
      # TODO create proper setter and getter methods for instance variables
      @current_user = User.find_by_username!( user_in ) # ! method raises exception on failure
      @current_session = UserSession.create!(@current_user)  # ! method raises exception on failure
      .  .  .
    end
    

Of course, when you are testing the login user interface the ugly approach seems unavoidable, but in fact it is not. Providing for the purposes of testing that certain conventions are followed respecting user names and passwords the following works just as well and is much cleaner. Plus you have removed all inter-step dependencies. feature statement: 
    
    
    When "Bob" logs on through the logon page
    

step definition: 
    
    
    When /"([\w[\d\w]+)" logs on through the logon page/ do |user_name|
      visit(logon_path)
      fill_in( "User Name", :with => user_name )
      fill_in( "Password", :with => user_name + "-test-passwd" )
      click_button( "Logon" )
    end
    

Having just shown you how to do it now take heed that you do not write **any** step definitions that call other steps (_you will do this too, but try hard not to_). At times this will seem like the quickest solution to a troublesome bit of environment building. However, for anything beyond trivial use it is always better to implement a custom method using the **api** provided by Cucumber ( _or by any other libraries you have installed_ ) and then call that method directly from your step. You can either keep these custom methods in the same file as the regular steps or stick them in any convenient file ending in `.rb` that is located in the support directory ( _well, anywhere that cucumber can find it really_ ) in which case you must enclose your methods within the following block: 
    
    
    Cucumber::Rails::World.class_eval do
      def your_method(parm)
        .  .  .
      end
    end
    

A cleaner way of doing this that avoids using the evil **_eval_** method is to create a class that contains only your add-on methods and then instantiate it inside the Cucumber World. Something like this: 
    
    
    # features/support/local_env.rb
    . . .
    class LocalHelpers
    . . .
      def execute( command )    
        stderr_file = Tempfile.new( 'script_stdout_stderr' )
        stderr_file.close
        @last_stdout = `#{command} 2> #{stderr_file.path}`
        @last_exit_status = $?.exitstatus
        @last_stderr = IO.read( stderr_file.path )
      end
    . . .
    end
    . . .
    World do
      LocalHelpers.new
    end
    

My rule of thumb is that if a step definition is called from another step definition then its contents probably should be extracted out into a custom method. For example a logon step definition is likely to be used repeatedly throughout many features. Turning it into a method is probably called for. 
    
    
    def logon_for_user( uname )
      visit(logon_path)
      fill_in( "User Name", :with => uname )
      fill_in( "Password", :with => uname + "-test-passwd" )
      click_button( "Logon" )
    end

When /“([\w[\d\w]+)” logs on through the logon page/ do |user_name| logon_for_user( user_name ) end Before you ask: Yes, these _helper_ methods can go into the same step definitions file that uses them. I put them at the top above all the matchers. And they can be called from elsewhere in your step definitions so you need not repeat yourself. 

### [](<https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder#before-after-and-background>)Before, After and Background

If all your feature’s scenarios share the same ‘set-up’ feature steps, then Cucumber provides the _Background_ section. Steps contained within a Background section are run before each of the scenarios. 
    
    
    Feature: .  .  .
      Background: .  .  .
      Scenario: .  .  .
    

Step definition files have a corresponding method available in the `before(condition) do . . .` method, which has however a matching `after(condition) do . . .` method as well. Recall that we are working in Ruby, and therefore the condition which enables the before/after block is anything that besides `false` or `nil` (like a **tag** , for instance). Also be aware that **all** eligible `before` methods are run before any scenario statements are processed, and that they are run in the order encountered. Likewise, every eligible `after` block will run at the completion of every scenario, again in the order that it is encountered. These two methods are powerful tools, but be aware that if you use them excessively then you will hang yourself eventually. 

### [](<https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder#what-is-a-good-step-definition>)What is a good Step Definition?

Opinions vary of course, but for me a **_good_** step definition has the following attributes: 

  * The matcher is short.
  * The matcher handles both positive and negative (true and false) conditions.
  * The matcher has at most two value parameters
  * The parameter variables are clearly named
  * The body is less than ten lines of code
  * The body does not call other steps

My template for a step definition presently looks like this: 
  1. statement identifier expectation “value”
  2. statement identifier not expectation “value” When /statement identifier( not)? expectation “([^\”]+)"/i do |boolean, value| actual = expectation( value ) expected = !boolean # this works because in ruby nil == false and !nil == true message = “expectation failed for #{value}” assert( actual == expected, message ) end

For example ( admittedly contrived ): 
    
    
    When /product ([^\"]+) should( not)? belong to category ([^\"]+)/i do |product, boolean, category|
      actual = ( Product.find_by_stock_number!( product )category ) == category 
      expected = !boolean
      message = "Product '#{product}' should#{boolean} belong to category '#{category}'"
      assert( actual == expected, message )
    end
    

Hunting through multiple files and directories for a multi-purpose step-definition matcher is considerably more tedious than looking for an invariant string of text. Therefore I recommend that you preface such steps with comments that cover the expected usage. For example: 
    
    
    # product should belong to category
    # product should not belong to category
    When /product ([^\"]+) should( not)? belong to category ([^\"]+)/i do |product, boolean, category|
    . . .
    

Which can be conveniently `grep` _ed_ for ( _on *nix systems at least_ ) using: `find features -name \*rb | xargs grep "product .* should belong to category"`

### [](<https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder#what-are-tags>)What are “tags”?

Cucumber provides a simple method to organize features and scenarios by user determined classifications. This is implemented using the convention that any space delimited string found in a feature file that is prefaced with the commercial at (**`@`**) symbol is considered a tag. As distributed, Cucumber-Rails builds a Rake task that recognizes the _`@wip`_ tag. However, any string may be used as a tag and any scenario or entire feature can have multiple tags associated with it. For example: 
    
    
      @init
      Feature:  .  .  .
      .  .  .
      @wip @authent
      Scenario:  A user should authenticate before accessing any resource.
         Given I do have a user named "testuser"
          When the user visits the login page
             And the user named "testuser" authenticates successfully
         Then I should see .  .  .
             .   .  .
    

Given that the forgoing is contained in a file called `features/login/login.feature` and that the `cucumber-rails` gem is installed and configured then you can exercise this scenario, along with any others that are similarly tagged, in any of the following ways: 

> _Note: You will probably need to preface command line instructions with_ : `bundle exec`
    
    
    $ rake cucumber:wip
    $ cucumber --profile=my_profile --tags=@wip features
    $ cucumber --profile=my_profile --tags=@authent features/login
    $ cucumber --profile=my_profile --tags=@init 
    

However, the following will not work, unless you [build a custom rake task](<https://github.com/cucumber/cucumber/wiki/Using-Rake>) for it: 
    
    
    $ rake cucumber:authent
    

There is an obscure _gotcha_ with this particular combination of tags. The default profile contained in the distributed `config/cucumber.yml` contains these lines: 
    
    
    <%
    .  .  .
    std_opts = "--format #{ENV['CUCUMBER_FORMAT'] || 'progress'} --strict --tags ~@wip"
    %>
    default: <%= std_opts %> features
    .  .  .
    

Note the trailing option `--tags ~@wip`. Cucumber provides for negating tags by prefacing the `--tags` argument with a tilde character(**`~`**). This tells Cucumber to not process features and scenarios so tagged. If you do not specify a different profile ( `cucumber -p profilename` )then the default profile will be used. If the default profile is used then the `--tags ~@wip` will cause Cucumber to skip any scenario that is so tagged. This will override the `--tags=@authen` option passed in the command line and so you will see this: 
    
    
    $ cucumber --tags=@authent
    Using the default profile...
    
    0 scenarios
    0 steps
    0m0.000s
    

Since version 0.6.0, one can no longer overcome this default setting by adding the `--tags=@wip` to the Cucumber argument list on the command line because now all `--tags`options are ANDed together. Thus the combination of `--tags @wip` **AND** `--tags ~@wip`fails everywhere. You either must create a special profile in `config/cucumber.yml` to deal with this or alter the default profile to suit your needs. Note as well that `@wip` tags are a special case. If any scenario tagged as `@wip` passes all of its steps without error and the `--wip` option is also passed then Cucumber reports the run as failing since scenarios that are marked as a work in progress are not supposed to pass. Note as well that the `--strict` and `--wip` options are mutually exclusive. The number of occurrences of a particular tag in your feature set may be controlled by appending a colon followed by a number to the end of the tag name passed to the tags option, as in `$ cucumber --tags=@wip:3 features/log*`. The existence of more than the specified number of occurrences of that tag in all the features that are exercised during a particular cucumber run will produce a warning message. If the `--strict` option is passed as well, as is the case with the default profile, then instead of a warning the run will fail. Limiting the number of occurrences is commonly used in conjunction with the `@wip` tag to restrict the number of unspecified scenarios to manageable levels. Those following [Kanban](<http://en.wikipedia.org/wiki/kanban>)or [Lean Software Development](<http://en.wikipedia.org/wiki/Lean_software_development>) based methodologies will find this facility invaluable. As outlined above, tags may be negated by prefacing the tag with the tilde (**`~`**) symbol. In other words, you can exclude all scenarios that have a particular tag (providing that tag is not elsewhere passed to cucumber as a parameter). For example, the following will only exercise all scenarios found in the directory tree rooted at `features/wip` that do not have the tag _`@ignore`_ : 
    
    
    $ cucumber --require=features --tag=~@ignore features/wip
    

A convention that I have adopted is tagging all scenarios created to track down a specific defect with tags of the form `@issue_### where ###` is the issue number assigned to the defect. This both handles multiple related scenarios and provides a convenient and self-documenting way to verify with cucumber that a specific defect either has been completely resolved or that a regression has occurred. Be aware that tags are heritable within Feature files. Scenarios inherit tags from the Feature statement.
