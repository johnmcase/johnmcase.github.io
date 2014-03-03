---
layout: post
category : "Centare Blog"
tags: [Reactive, Rx, .Net]
overrideTitle: "Reactive Programming using Rx .Net"
tagline: Part 2

---
{% include JB/setup %}

In this third post in my [reactive programming]({% post_url 2014-01-20-intro-to-reactive-programming %}) blog series I want to continue where we left off in the [second post]({% post_url 2014-02-03-reactive-programming-using-rx.net-part-1 %}) in discussing the Rx .Net framework.  Last time I talked about how to use the `Observable.Create` factory method to construct Observable sequences in accordance with the [reactive principles](http://reactivemanifesto.org).  Know that this is not the only way to construct an Observable, but it is the most general way.  The Rx library ships with many convenience factory methods that allow you to easily construct Observable sequences from delegates, async tasks, and from events.

<!--excerpt-->

For the remainder of this post I want to talk about composing Observable sequences into new sequences and I also want to cover consuming Observables.  Just as last time, I want to make sure that we are adhering to the *reactive principles* throughout.  Recall our requirements from the first post:

>Display a text box for a user to type in a product name.  Once they've typed in a few characters, populate a side bar displaying the full name of every product that starts with the typed in letters.  Then, for each product returned, look up the top 10 customers for each. The product names must be displayed as soon as they are available, and the customer data must be filled in as it arrives.  The application must remain usable while the data is being populated.

Also recall that we actually have to search for products across 3 separate catalogs, but the user should be unaware of this fact.  At the end of the last post we had 2 methods to create Observable streams, one to lookup all products from a specific catalog beginning with the letters the user typed, and the second method to lookup the list of all customers who purchased a given item.

I think the first thing we should do next is create a method that combines the 3 catalog lookups and returns a single Observable sequence.  There are a couple different ways to combine Observable sequences, the two most basic ways being the `Concat` and `Merge` methods.  The difference between these two demonstrates one important difference between push-based Observable sequences and pull-based Enumerables: time.  Both `Concat` and `Merge`  take two input sequences and combine them into a single output, but they have different behaviors based on the timing of when the input streams produce values.  `Concat` will cache values from the second stream until the first stream has completed, whereas `Merge` will publish values from both input streams into the output as they happen.  The difference between these two operations is easily illustrated using *Marble Diagrams*.

Marble diagrams are often used in Rx to describe complicated interactions between Observables when applied to a function or method.  Typically they show one or more input sequences on the top, then a box describing the operation being performed, and the output is at the bottom.  Time advances horizontally from left to right and elements that appear vertically aligned happen at the same time.  The marble diagrams for `Concat` and `Merge` are below:

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

Great, so now we have built an Observable sequence of all products across all catalogs in a non-blocking asynchronous way.  Now we have to tackle the requirement of looking up the top 10 customers for each product.  We're going to approach this requirement with a familiar tool: LINQ.  Rx has re-implemented all of the standard LINQ operatiors against IObservable, and has done so in an asynchronous non-blocking fashion whenever possible.  What we need to do is take the Observable sequence of all Products that we just built, and project each element into a new Sequence of its top 10 customers.  The way to do this with LINQ is with the `Select` method.

Remember from the last post that we already have the `CustomersForProduct` method to look up all of the customers in sorted order.  We also need a way to match the customer list back to the product it corresponds to, so we'll actually create an IObservable of a new annonymous object type that wraps the product and it's customer list together.  Even with that wrinkle, this somewhat complicated requirement actually boils down to a single statement:

{% highlight csharp %}
public IObservable<object> ProductsWithTopTenCustomers(IObservable products)
{
  return products.Select(p => 
    new {
      Product = p,
      Customers = CustomersForProduct(p).Take(10)
    });
} 
{% endhighlight %}

Even this is just standard LINQ syntax, this is still non-blocking and asynchronous.  The lambda in the `Select` statement is executed only when data becomes available.  So the caller is given a handle to the resulting IObservable immediately and it will start to produce values as soon as values are available on the input IObservable.

All we have to do at this point is to consume this data and display it on the UI.  The UI code itself is out of scope for this post, but lets just assume we have the following methods available to us:

{% highlight csharp %}
public void DisplayProduct(Product p)
{
  // code to add the product to the list of products on the UI
}
public void DisplayTopTenUsers(Product p, IEnumerable<Customers>)
{
  // code to display the list of customers with the appropriate product
}
public void DisplayErrorMessage(Exception ex)
{
  // code to display a friendly error message (and potentially log the error too)
}
{% endhighlight %}

We're so close to putting everything together here.  Any rich UI developer will tell you that the UI helper methods above cannot be executed on just any old thread, they *must* be executed on the UI Dispatcher thread.  Up until now we've let Rx execute its asynchronous code on any thread, but now we have to constrain it and force it to use the UI Dispatch thread for certain operations.  Fortunately, this is easily done with the `ObserveOn` method and the concept of Schedulers.

A scheduler is simply an Object that tells Rx how it can execute asynchronously.  Rx comes with some pre-built schedulers out of the box, and sure enough there is one that will force execution onto the UI Dispatcher thread.  

After taking the above into consideration, the final piece of code to stitch everything together is as follows: 

{% highlight csharp %}
var products = ProductsStartingWith(<<user input>>);
var productsWithCustomers = ProductsWithtopTenCustomers(products));

products.ObserveOn(SchedulerDispatcher).Subscribe(
  p => DisplayProduct(p),
  ex => DisplayErrorMessage(ex)
);

productsWithcustomers.ObserveOn(SchedulerDispatcher).Subscribe(
  pwc => DisplayTopTenUsers(pwc.Product, pwc.Customers.ToEnumerable()),
  ex => DisplayErrorMessage(ex)
)
{% endhighlight %}

That's it.  Our UI will start to populate product information as soon as it becomes available, and it will also fill in the top ten customer list at a later time, but as soon as it is available as well.  Also the UI will still be responsive to user input while all of the data fetching is happening in the background.

This was obviously a rather contrived and not very realistic example, but I hope that served as a good example of Rx and demonstrated its ability to handle somewhat complicated asynchronous scenarios with code that is easy to write and easy to understand.  For a much deeper dive on Rx, I highly recommend Lee Campbell's [Intro to Rx](http://www.introtorx.com/) online book, the [Channel 9 Rx tag](http://channel9.msdn.com/tags/Rx/), and also the Rx home page on [CodePlex](http://rx.codeplex.com/).

In my next blog post, I intend to leave the world of .Net and look at reactive programming on the JVM.