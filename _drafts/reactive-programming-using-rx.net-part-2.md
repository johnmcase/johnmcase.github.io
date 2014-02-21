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

*Marble diagrams* are useful tools often used in Rx to describe complicated interactions between Observables when an operator is applied to them.  Typically they show one or more input sequences on the top, then a box describing the operation being performed, and the output is at the bottom.  Time advances horizontally from left to right and elements that appear vertically aligned happen at the same time.  The marble diagrams for `Concat` and `Merge` are below:

<div class="row">
<div class="col-md-10 col-sm-12" style="margin-bottom: 50px;">
  <img class="img-responsive" src="/images/marble-diagram-concat.png" alt="Concat Marble Diagram">
  <p style="text-align: center;">The <code>Concat</code> operation will delay output of any elements on the second sequence until the first one has completed.</p>
</div>
<div class="col-md-10 col-xs-12" style="margin-bottom: 50px;">
  <img class="img-responsive" src="/images/marble-diagram-merge.png" alt="Merge Marble Diagram">
  <p style="text-align: center;">The <code>Merge</code> operation will output elements from both input sequences at the moment they arrive.  Note that in this diagram the first sequence throws an error which propogates to the output and causes the last element from the other input to be lost.</p>
</div>
</div>

Since for our requirements we don't care about maintaining the order of the original input sequences and since we want to be as real-time as possible, `Merge` looks like the correct choice.  Here is how we can combine our 3 input sequences in code:

{% highlight csharp %}
public IObservable<Product> ProductsStartingWith(string startsWith)
{
  var catalog1 = ProductsStartingWith(startsWith, 1);
  var catalog2 = ProductsStartingWith(startsWith, 2);
  var catalog3 = ProductsStartingWith(startsWith, 3);
  return catalog1.Merge(catalog2).Merge(catalog3);
}
{% endhighlight %}

Great, so now we have built an Observable sequence of all products across all catalogs in a non-blocking asynchronous way.  Now we have to tackle the requirement of looking up the top 10 customers for each product.  We're going to tackle this requirement with a familiar tool: LINQ.  Rx has re-implemented all of the standard LINQ operatiors against IObservable, and has done so in an asynchronous non-blocking fashion whenever possible.  