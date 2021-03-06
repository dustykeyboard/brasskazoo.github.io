---
layout: post
title: Using WebDriver, jBehave to test dynamic web forms
date: '2011-10-23T22:29:00.000+11:00'
categories: software-development web-development code-quality

author: brasskazoo
tags:
- Java
- jBehave
- WebDriver
- html forms
- Code Quality
- junit
modified_time: '2012-01-02T19:27:10.757+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-3914292823695159126
blogger_orig_url: http://blog.brasskazoo.com/2011/10/using-webdriver-jbehave-to-test-dynamic.html
---

I've been updating an ancient yet integral Perl form handler script (based on 
[bnbform](http://bignosebird.com/carchive/bnbform.shtml)), and I will be 
introducing some fairly significant changes. Thus I'd like to make sure I 
don't introduce any errors, or changes that would affect existing forms on our 
site.

The form handler allows arbitrary fields in a form, supported by hidden fields 
that define its behaviour (e.g. a field with the name 'required' contains the 
mandatory field names). 

Basically I'd like to be able to verify the server response, including
rejecting bad/missing data and success or 'thank you' screens. 
To do this, a combination of [JBehave](http://jbehave.org/), 
[WebDriver](http://code.google.com/p/selenium/w/list?q=label:WebDriver) and 
[jQuery](http://jquery.com/) could give us a solution.

## Form manipulation (WebDriver + JQuery)

I'll need to construct a mock form,
populate it and submit it to the server. Then I'll grab the response and check 
it. 
To make the tests a bit more dynamic, I'll create a 'clean slate' html page 
with just the form element and a submit button: 

{% highlight xml %}
<form id="requestForm" action="http://localhost/cgi-bin/requestForm.cgi"
method="post" name="RequestForm">
  <fieldset id="formFields" class="submit"></fieldset>
  <fieldset id="submitField" class="submit">
    <input class="submit" type="submit" value="Submit" />
  </fieldset>
</form>
{% endhighlight %}

Then I'll programatically add form fields using a javascript function (in 
theory could also do this in Java code). For convenience sake I'm using 
jQuery: 

{% highlight java %}
function addField(elLabel, elId, elName, elType, elValue) {
    // TODO - parameter validation/defaults
    var div = $("<div/>");

    if (elLabel) {
        $("<label/>", {
            id: elId + "_lbl",
            "for": elId,
            "text": elLabel
        }).appendTo(div);
    }

    $("<input />", {
        id: elId, 
        name: elName, 
        type: elType, 
        value: elValue 
    }).appendTo(div); 

    div.appendTo("#formFields");
} 
{% endhighlight %}

This will create a div element containing a label and the input element.  
Then, in our test case, its simply a matter of getting an instance of 
[`JavascriptExecutor`](http://selenium.googlecode.com/svn/trunk/docs/api/java/org/openqa/selenium/JavascriptExecutor.html) 
and executing the javascript function for each field: 

{% highlight java %}
WebDriver driver = ... // Initialise
JavascriptExecutor js = (JavascriptExecutor) driver; 

js.executeScript("addField('Name', 'nameField', 'nameField', 'text', 
    'John Citizen');"); 
{% endhighlight %}

With this setup, I can easily build the form to contain specific elements; 
specific form configurations that I want to test! 
##      Server response (WebDriver + JUnit)

When we're ready, WebDriver can
submit the form and check the contents of the page returned: 

{% highlight java %}
// Submit form
driver.findElement(By.id("requestForm")).submit(); 

// Example result 
Assert.assertEquals("Thanks!", 
driver.findElement(By.tagName("h1")).getText()); 
Assert.assertEquals("Thank you for your submission", 
    driver.findElement(By.tagName("p")).getText()); 
{% endhighlight %}

##      Story time! (JBehave)
Implementing some BDD for the existing
functionality, and my impending changes, will help ensure that the form 
continues to behave in the expected manner.  So lets write some stories that 
should cover some existing functionality! 
(File: `test/resources/basic-form.story`)

````
Scenario: A basic 2-field form is submitted

Given a text field with name 'Name' and value 'John Citizen' 
 And a text field with name 'Submit By' and value 'john@localhost' 
 When form is submitted 
 Then the confirmation page is displayed 

Given a text field with name 'Name' and value 'John Citizen' 
 And a text field with name 'Submit By' and value '' 
 And 'Submit By' is required 
 When form is submitted 
 Then the error page is displayed
````

I've included the type and just enough details in the '`Given`' lines to 
create a unique form element using our javascript function.

The mention
of '`required`' will include the hidden input field that bnbform uses to track 
mandatory fields

The '`When`' line obviously relates to WebDriver
submitting the form, and '`Then`' is the assertions relating to the response. 
We're simply expecting either a success page or validation error page. Any 
other unexpected outcome (e.g. an error in the script) will result in a failed 
test. 
## StepsUsing the code above as a guide, this step class will cover the 
stories:

{% highlight java %}
public class FormSteps {
    private WebDriver _driver; 

    @BeforeStory 
    public void beforeEachStory() { 
        _driver = new HtmlUnitDriver(true); 
        // Using a bundled HTML file instead of modifying the server 
        final String htmlFile = getClass() 
            .getResource("/base-form.html").toString(); 
        _driver.get(htmlFile); 
    } 

    @Given("a $fieldType field with name '$fieldName' and value '$fieldValue'")
    public void includeField(String fieldType, String fieldName, 
            String fieldValue) throws Exception { 
        JavascriptExecutor js = (JavascriptExecutor) _driver; 
        js.executeScript("addField('" + fieldName + "', '" 
            + fieldName.toLowerCase() + "', '" + fieldName.toLowerCase() 
            + "', '" + fieldType + "', '" + fieldValue + "')"); 
    } 

    @Given("$fieldName is fieldAttribute") 
    public void addFieldAttribute(String fieldName, String fieldAttribute) 
        throws Exception { 
        // TODO 
    } 

    @When("form is submitted") 
    public void submitForm() throws Exception { 
        _driver.findElement(By.id("requestForm")).submit(); 
    } 

    @Then("the $result page is displayed") 
    public void verifyResult(String result) { 
        final String heading = _driver.findElement(By.tagName("h1")).getText();
        final String body = _driver.findElement(By.tagName("p")).getText();

        if ("confirmation".equals(result)) { 
            Assert.assertEquals("Thanks!", heading); 
            Assert.assertEquals("Thank you for your submission", body); 
        } else { 
            Assert.assertEquals("Error!", heading); 
            Assert.assertEquals("Bad data in form", body); 
        }
    } 
} 
{% endhighlight %}

After implementing the standard class structure outlined at 
[http://jbehave.org/reference/stable](http://jbehave.org/reference/stable), I 
have two additional classes: FormSteps and FormStories. FormStories, which 
extends JUnitStories, is the test case runner. 
The functionality of the form can be further defined by additional scenarios 
and stories, so existing functionality and the new enhancements can be 
verified. 

*NOTE*: You may need to make sure your IDE copies the `*.story` files to the
test class directory (maven should automatically copy them if they are under 
test/resources). 