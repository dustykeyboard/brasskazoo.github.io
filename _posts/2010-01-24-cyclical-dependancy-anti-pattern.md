---
layout: post
title: 'A cyclical dependancy anti-pattern: Identity Theft'
date: '2010-01-24T19:00:00.000+11:00'
categories: software-development code-quality

author: brasskazoo
tags:
- Java
- Software Development
- Code Quality
- Anti-pattern
modified_time: '2012-01-02T19:29:44.164+11:00'
thumbnail: http://1.bp.blogspot.com/-VVa7NthaqSQ/TsCmcCsBWAI/AAAAAAAAAAM/jdwoIv7pCHc/s72-c/class-diagram1.png
blogger_id: tag:blogger.com,1999:blog-4222683507658340156.post-8876093533940678582
blogger_orig_url: http://blog.brasskazoo.com/2010/01/cyclical-dependancy-anti-pattern.html
---

## Anti-patterns:
In software engineering, an anti-pattern (or
antipattern) is a design pattern that appears obvious but is ineffective or 
far from optimal in practice.<sup>1</sup> Or as I like to think of
it, the type of code that makes you want to rip your eyes out, like in that 
movie _Event Horizon_...

##  Identity Theft
Here's a pattern that I had witnessed several times in a particular codebase, which I believe suffers from some design flaws:

<img border="0" height="212" src="http://1.bp.blogspot.com/-VVa7NthaqSQ/TsCmcCsBWAI/AAAAAAAAAAM/jdwoIv7pCHc/s320/class-diagram1.png" width="320"/>

FooListing is a class that retrieves and maintains a list of Foo objects

## Description
`FooListing `is a repository-type object that holds a list of `Foo` objects, which it populates
from the database using its `populate()` function. The `Foo` object also has a `populate()`
method, but this method instantiates a `FooListing`, and uses the `FooListing` populate function
with its key - so now the `FooListing `contains the desired `Foo `object. Our original `Foo`
object now contains a reference to the `Foo `object that it `wants to be` - so it does what
anyone would do, steals its identity and _hides the body_.

`Foo.populate()` looks something like this:

{% highlight java %}
public void populate() {
  FooListing listing = new FooListing() 
  // Set up FooListing 
  FooListing.setSearchKey(id); 
  if (listing.size > 0) {
      Foo foo = listing.get(0); 
      this.setBar(foo.getBar); 
      // continue to copy over properties 
      // ... 
  } else { 
      LOG.warn("Foo not found for key: " + id); 
  } 
}
{% endhighlight %}

## Why this is bad
Firstly, apart from the questional morality, it
is duplicated code. `FooListing` already has the capability to create and
populate a `Foo` object from the database. Two locations for this code means
twice the maintenance if something changes, more possibility
of bugs, etc.

`Foo` and `FooListing` have become tightly coupled and
dependant on each other under this design - there is a cyclic dependancy which 
is a code smell, and may cause headaches when writing unit tests.

There is also a waste of resoures creating the uneccesary `FooListing `and `Foo`
objects inside of `Foo.populate()`, at least some of that could be avoided by
client code accessing instances of `Foo` via `FooListing.populate`.

It also doesn't make sense that a data access object like `Foo `should be concerned
with information about how it is created. `Foo `should act more like a bean, 
or an immutable class.

Another dangerous aspect of this design is the
implication that client code accessing `Foo` cannot be certain that `Foo` has
been instantiated correctly. If the client code has a faulty key for `Foo`
that does not exist in the database, when it creates new `Foo(key)` there is
no way to know that `Foo.populate()` has failed to find the correct value, and 
instead they are left with a faulty `Foo` instance which was not what they
requested. 

## Solution
The best solution for this isolated pattern is to
completely remove (or deprecate) the `Foo.populate()` method, and replace 
calls to it with `FooListing` instances.

If `FooListing` fails to find a
matching `Foo`, the client code should realise this when `FooListing` returns
them a null object. The client code can handle and recover from this case in 
context.

Implementing a `getFirstResult()` function in `FooListing` could
be beneficial if there are many cases where the code with otherwise be calling 
`get(0)`.

We could also simplify the calling code so that retrieving a
result is a one-line operation - i.e. `get()` calls `populate()` if the list 
has not already been populated. 

{% highlight java %}
public final class FooListing {
  private List<Foo$gt; _listing;
  private int _id; 

  public FooListing { 
  } 

  public FooListing(int id) { 
    _searchId = id; 
  } 

  public void populate() { 
    _listing = new ArrayList<foo>(); 
    // Query the database and add to _listing 
  } 

  public Foo get(int index) { 
    if (_listing == null) { 
      populate(); 
    } 
    if (_listing.isEmpty() || index >= _listing.size()) {
      return null; 
    } else { 
      return _listing.get(index); 
    } 
  } 

  public Foo getFirstResult() { 
    return this.get(0); 
  } 

  public List<foo> getList() { 
    return _listing; 
  } 

  public void setSearchId(int id) { 
    _id = id; 
  } 
}
{% endhighlight %}

And the client code would only need to call: 

{% highlight java %}
Foo foo = new FooListing(id).getFirstResult();
{% endhighlight %}

<img border="0" height="89" src="http://4.bp.blogspot.com/-uKiL-2jQ8Ak/TsCmc0E2KHI/AAAAAAAAAAQ/UNKieTcwLxU/s400/class-diagram2.png" width="400" />

## References
1. [http://en.wikipedia.org/wiki/Anti-patterns](http://en.wikipedia.org/wiki/Anti-patterns)