---
layout: post
category : "Centare Blog"
tags: [Reactive, Rx, .Net]
overrideTitle: "Reactive Programming using Rx .Net"
tagline: Part 2

---
{% include JB/setup %}

in this third post in my [reactive programming]({% post_url 2014-01-20-intro-to-reactive-programming %}) blog series I want to continue where we left off in the [second post]({% post_url 2014-02-03-reactive-programming-using-rx.net-part-1 %}) in discussing the Rx .Net framework.  Last time I talked about how to construct Observable sequences in accordance with the [reactive principles](http://reactivemanifesto.org).  Know that this is not the only way to construct an Observable, but it is the most general way.  The Rx library ships with many convenience factory methods that allow you to easily construct Observable sequences from delegates, async tasks, and from events.

<!--excerpt-->

For the remainder of this post I want to talk about composing Observable sequences into new sequences and I also want to cover consuming Observables.  Just as last time, I want to make sure that we are adhering to the *reactive principles* throughout.  Recall our requirements from the first post:

>Display a text box for a user to type in a product name.  Once they've typed in a few characters, populate a side bar displaying the full name of every product that starts with the typed in letters.  Then, for each product returned, look up the top 10 customers for each. The product names must be displayed as soon as they are available, and the customer data must be filled in as it arrives.  The application must remain usable while the data is being populated.

Also recall that we actually have to search for products across 3 separate catalogs, but the user should be unaware of this fact.  At the end of the last post we had 2 methods to create Observable streams, one to lookup all products from a specific catalog beginning with the letters the user typed, and the second method to lookup the list of all customers who purchased a given item.

I think the first thing we should do next is create a method that combines the 3 catalog lookups and returns a single Observable sequence.  There are a couple different ways to combine Observable sequences, the two most basic ways being the `Concat` and `Merge` methods.  The difference between these two demonstrates one important difference between push-based Observable sequences and pull-based Enumerables: time.  `Concat` and `Merge` differ in how they handle time.  **MORE HERE**

<div class="row">
<div class="col-md-10 col-sm-12" style="margin-bottom: 50px;">
  <img class="img-responsive" src="/images/marble-diagram-concat.png" alt="Concat Marble Diagram">
  <p style="text-align: center;">adsf</p>
</div>
<div class="col-md-10 col-sm-12" style="margin-bottom: 50px;">
  <img class="img-responsive" src="/images/marble-diagram-merge.png" alt="Merge Marble Diagram">
  <p style="text-align: center;">adsf</p>
</div>
</div>

