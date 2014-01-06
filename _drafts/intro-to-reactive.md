---
layout: post
category : "Centare Blog"
tagline: "Is this visible"
tags: [Reactive]

---
{% include JB/setup %}

Outline:

1. Introduce myself (maybe not)
2. Introduce the blog series
3. Talk about the need for reactive
	1. Multicore
	2. Cloud
	3. Ahmdals law
4. Introduce the reactive principles
	1. Event-Driven
	2. Scalable
	3. Resilient
	4. Responsive
5. Talk about upcoming posts:
	1. .Net Rx
	2. Testing Rx
	3. Rx for JS
	4. Rx on the JVM with Java and Scala  (or is this two posts)
	5. Reactive programming with the Actor model

---

# Reactive Programming #

## Making the case for reactive ##

Software development practices have not always kept up with the environments they are developing software for.  Modern software systems are no longer single monolithic processes that connect to a persistent data store.  Instead the modern system often retrieves and stores data across multiple stores, typically located somewhere over the internet instead of on the local network.  Off location data services like these necessarily come with greater latency than local services.  Somewhat paradoxically, users of modern applications have much less tolerance for slow response times than they did just a few years ago.

(Talk some about multi-core, limited thread pool size, and blocking)

## Reactive Principles ##

The reactive principles are a set of architectural principles that combine to create reactive applications.

### Event-Driven ###

### Scalable ###

### Resilient ###

### Responsive ###

## A simple example ##

At the beginning of this article I oversimplified when I made the paradoxical statement that users have less tolerance for slow response times than in the past despite the fact that modern systems typically have to deal with more latency.  What they truly have zero tolerance for is a blocked application.  With a reactive system, latency is less of a problem because latency does not necessitate blocking.  Blocking is what users truly will not tolerate.

We are all familiar with auto complete functionality on web sites.  For example, a user, Bob, types in the first few letters of a stock ticker symbol into a financial application and almost immediately is presented with all tickers that start with the letters he typed.  Bob would find the site almost unusable if once he typed the first letter the application blocked and prevented him from typing more while it goes out and retrieves all symbols that start with the single letter he typed.  In fact, regardless of whatever optimizations the developer comes up with, if the application **ever** prevents Bob from typing another character into that text box, Bob is going to be one unhappy customer and will probably start going to a different site to get his stock quotes.

The type ahead stock symbol example above is a simple but excellent example of reactive programming.  Typically this is implemented using an AJAX call to some web service to perform the lookup of ticker symbols and then registering some client side java-script code to execute when the result comes back display the results in the correct location.  This pattern of execution is the same whether this is done with custom JavaScript, JQuery, or a library like Knockout or Angular.  This same pattern can be applied throughout the software stack, not simply at the UI layer.  I intend to go into depth on some techniques for doing this in both .Net and on the JVM in future posts.