---
layout: post
category : "Blog"
tagline: ""
tags: [Reactive]

---
{% include JB/setup %}

Software development practices have not always kept up with the environments they are developing software for.  Modern software systems are no longer single monolithic processes that connect to a persistent data store.  Instead the modern system often retrieves and stores data across multiple stores, typically located somewhere over the internet instead of on the local network.  Off location data services like these necessarily come with greater latency than local services.  Somewhat paradoxically, users of modern applications have much less tolerance for slow response times than they did just a few years ago.

<!--excerpt-->

A new set of architectural principles has grown out of this need to manage larger and more disparate data while at the same time making the user experience as snappy and responsive as possible.  These principles are bundled under the heading of reactive programming.  These principles are described in the [Reactive Manifesto](http://www.reactivemanifesto.org)  I will try to distill the reactive programming principles at a high level in this post and then provide more concrete examples in future posts.

## Reactive Traits

The reactive traits are a set of architectural properties that combine to create reactive applications.  A reactive application is one which can react to events, load, failures and users  on its own without any human interaction by network admins or developers.  The four reactive principles are event driven, scalable, resilient, and responsive.

![Reactive Principles](/images/reactivePrinciples.png)

### Event-Driven

An event driven application is one that is based on asynchronous communication of events or messages between system components.  Systems designed this way are naturally loosely coupled because the sender of the event has no dependency on the receiver, and vice versa.  Event driven applications are also more make more efficient usage of system resources because asynchronous code is non-blocking, thereby eliminating the overhead of traditional synchronous locking and synchronization.  The result of executing asynchronous code on a multi-threaded system is that threads (which are a limited resource) are more likely to be available to process incoming requests because there are no idle but unavailable threads due to blocking.

Event handlers in reactive applications must be thread safe.  This means that the handler cannot store any internal state which could affect the behavior of the handler from one message to the next.  All application state should be passed into and out of event handlers as part of the messages themselves.  This becomes very important when we consider scalability shortly.

### Scalable

A scalable application is one that can be expanded (or contracted) based on its usage.  The obvious example is scaling out, where you add more nodes in a server farm or spin up another server in the cloud.  But being reactive also means supporting scaling up, meaning that the application should be able to deployed on a single or multiple core machine and take full advantage of all cores, no matter how many, without having to be re-written.

A system that is event driven is also written to scale up or out with relative ease due to the loose coupling and location independence of the system components.  Because reactive components are stateless, it is easy to add or remove nodes at runtime.  It makes no difference if the nodes are running in the same process or not, same physical machine or not, or even if they are in a completely different data center.  An event handler simply responds to messages and does work, regardless of where it is located. 

### Resilient

An application is resilient if it is able to withstand errors and failures automatically, with minimal assistance from operations personnel and with minimal impact to end users.  A key to resilience is isolation, and the same loosely coupled nature of event driven applications that makes them scalable also isolates individual components and makes them resilient.  But resilience in reactive programming is not simply a beneficial side effect, it is something that is planned for from the onset.  Failures themselves can be encapsulated as messages and sent off to another part of the system that has the ability to deal with it properly.  Just as it is easy to add and remove nodes for scalability, failed nodes can be taken down and replaced automatically with little to no impact on the overall performance of the application.  

### Responsive

A responsive application is the net result of applying the previous three reactive traits:  Event driven, scalable, and resilience.  And all of these traits in their own right need to be responsive.  

If an asynchronous event or message handler is slow to respond then how can it be considered resilient?  For an application to be reactive, excessive latency should be treated as a failure, and the resilience trait states that the application should handle this failure immediately and automatically.  Furthermore, if the application is not responsive in its ability to handle excessive load, then it is not scalable in any sense that is of any use to the end user.

## A simple example

At the beginning of this article I oversimplified when I made the paradoxical statement that users have less tolerance for slow response times than in the past despite the fact that modern systems typically have to deal with more latency.  What users truly have zero tolerance for is a blocked application.  With a reactive system, latency is less of a problem because latency does not necessitate blocking.  Blocking is what users truly will not tolerate.

We are all familiar with auto complete functionality on web sites.  For example, a user, Bob, types in the first few letters of a stock ticker symbol into a financial application and almost immediately is presented with all tickers that start with the letters he typed.  Bob would find the site almost unusable if once he typed the first letter the application blocked and prevented him from typing more while it goes out and retrieves all symbols that start with the single letter he typed.  In fact, regardless of whatever optimizations the developer comes up with, if the application **ever** prevents Bob from typing another character into that text box, Bob is going to be one unhappy customer and will probably start going to a different site to get his stock quotes.

The type ahead stock symbol example above is a simple but excellent example of reactive programming.  Typically this is implemented using an AJAX call to some web service to perform the lookup of ticker symbols and then registering some client side java-script code to execute when the result comes back display the results in the correct location.  This pattern of execution is the same whether this is done with custom JavaScript, JQuery, or a library like Knockout or Angular.  This same pattern can be applied throughout the software stack, not simply at the UI layer.

In future posts I will be digging deeper and showing code examples for reactive programming across the entire software stack in both .Net and on the JVM.