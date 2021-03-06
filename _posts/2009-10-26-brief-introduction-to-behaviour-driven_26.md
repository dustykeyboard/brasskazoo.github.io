---
layout: post
title: A brief introduction to Behaviour-Driven Development
date: '2009-10-26T11:32:00.000+11:00'
categories: software-development code-quality

author: brasskazoo
tags:
- BDD
- integration testing
- Software Development
- Code Quality
modified_time: '2011-12-12T17:25:28.912+11:00'
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-5743298781314521682
blogger_orig_url: http://blog.brasskazoo.com/2009/10/brief-introduction-to-behaviour-driven_26.html
---

[Behaviour-Driven Development ](http://dannorth.net/introducing-bdd)(BDD) is a 
methodology developed by agile developer Dan North in 2006.

It was created on top of an existing methodology named [</a><a 
href="http://en.wikipedia.org/wiki/Test_Driven_Development">Test-Driven 
Development](http://en.wikipedia.org/wiki/Test_Driven_Development) (TDD), a 
fairly widely known and discussed technique. Put simply, TDD specifies that 
simple test cases should be written first, and the developer then writes the 
smallest amount of code possible to make the unit tests pass.

Whilst TDD is firmly in the realm of the developer, BDD attempts to bridge the 
gap between developers, testers, business analysts, and other stakeholders; it 
is closer to technical specifications rather than code: 

>By developing a consistent vocabulary across groups, we work
towards eliminating miscommunication and ambiguity. And by putting the 
emphasis on behaviour, we are more closely working with requirement 
specifications and the business value of the product's 
function.
>-- Dan North

##    Stories

Like many agile processes, BDD employs the concept of 'user stories' in the form of
a narrative, in a specific format:

````
Story: [Title]

As a [role]
I want [feature]
So that [benefit]
````

This format clearly identifies the actor, the system feature and the business
value or benefit of the story.  Each story also has acceptance criteria, in a 
form of scenarios: 

````
Scenario 1: [Title]

Given [context] 
  And [some more context] 
  ... 
 When [event] 
 Then ensure [outcome] 
 And ensure [another outcome] 
 ... 
````

The use of the word _ensure_ identifies outcomes that are the
responsibility of the scenario. Also using the word _should_ in the
scenario indicates the desired outcome could be affected by or reliant on 
another part of the system, and implicitly suggests a challenge to the 
assertion - "Should it? Should it _really_?"

##    Test Cases
Now we get to the core test-driven aspect - writing tests to
cover each scenario. North encourages starting thinking of tests in a sentence 
starting with 'should' - as in "test (that the system) should [do something]" 
which converts to a test case titled 'testShouldDoSomething', making intention 
of each test clear, and we should be able to relate it to the acceptance 
criteria.

##    Now for an example...
In this simple example we have a ticket machine on
a bus, where a trip duration is purchased by a passenger. The machine has a 
coin slot, an LCD to display the purchased time, a 'print' button, and a 
ticket printer (it does not have a coin return!). Each scenario begins with 
the machine in an initialised state (i.e. no current purchase balance).

````
Story: Purchasing a bus ticket

As a bus passenger 
I want to purchase a bus ticket 
So that I can board the bus 

Acceptance Criteria 

Scenario 1: Inserting initial coins 
Given that the ticket machine is initialised 
 When coins are inserted 
 Then ensure the purchased travel time is displayed on the LCD screen 

Scenario 2: Inserting additional coins 
Given that the ticket machine is initialised 
 And coins have been inserted 
 When coins are inserted 
 Then ensure the incremented travel time is displayed on the LCD screen 

Scenario 3: Printing the ticket 
Given that the ticket machine is initialised 
 And coins have been inserted 
 When the print button is pressed 
 Then ensure ticket is printed with the purchased travel time 
 And ensure that the LCD screen is cleared
````

We can see from the narrative the full extent of the story: The user, the 
system feature, and the value of the feature.  The acceptance criteria can 
then be transformed into unit tests:

{% highlight java %}
// Scenario 1
public void testShouldSetTimeOnFirstCoinInsert() {...} 

// Scenario 2 
public void testShouldIncrementTimeOnCoinInsert() {...} 

// Scenario 3 
public void testShouldPrintTicketWithPurchasedTime() {...} 
public void testShouldClearScreenAfterPrint() {...} 
public void testShouldFailWhenPrintWithNoPurchase() {...}
{% endhighlight %}

Each test case verifies an outcome of the scenario, and for scenario 3 also 
verifies an error condition (i.e. when the print button is pressed and there 
has been no coins inserted).  With the test cases supporting the acceptance 
criteria, we're affectingly applying test-driven development practices to 
business value, visible by stakeholders outside of the development team. 
Considering the difficulty many of us may have convincing non-technical people 
(and management) of the benefits of code quality strategies, to be able to 
actively engage them at higher levels with story narratives and plain English 
acceptance criteria is quite beneficial.

##    Consider these points
_Summarised from Dan North's [blog](http://dannorth.net/whats-in-a-story)_

1. The story title should describe an activity 
1. The story narrative should include a role, a feature and a benefit. - "_As a [role] I want [feature] so that [benefit]_"
1. The scenario title should describeribe what’s different 
1. The scenario should be expressed in terms of Givens, Events and Outcomes 
1. The givens should define all of, and no more than, the required context 
1. The event should describe the feature 
1. The story should be small enough to fit in an iteration 

##    References

1. [http://dannorth.net/introducing-bdd](http://dannorth.net/introducing-bdd)
1. [http://en.wikipedia.org/wiki/Test_Driven_Development](http://en.wikipedia.org/wiki/Test_Driven_Development)
1. [http://dannorth.net/whats-in-a-story](http://dannorth.net/whats-in-a-story)
1. [http://behaviour-driven.org/Introduction](http://behaviour-driven.org/Introduction)
1. [http://stackoverflow.com/questions/2509/what-are-the-primary-differences-between-...](http://stackoverflow.com/questions/2509/what-are-the-primary-differences-between-tdd-and-bdd#2548)
1. [http://en.wikipedia.org/wiki/Behavior_Driven_Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development)