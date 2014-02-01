---
layout: post
category : "Centare Blog"
tags: [Reactive, Rx, .Net]
overrideTitle: "Reactive Programming using Rx .Net"
tagline: Part 1

---
{% include JB/setup %}

In my [previous reactive programming post]({% post_url 2014-01-20-intro-to-reactive-programming %}) I talked about the high level principles of reactive programming.  In this and future posts I want to delve deeper into the topic and explore how to actually write reactive applications on different platforms.  The platform of choice for this week is .Net using the [Rx extensions](https://rx.codeplex.com/) from Microsoft Open Technologies.

<!--excerpt-->

For the sake of an admittedly contrived example, suppose you are working on some kind of application and you are given the following requirements:

>Display a text box for a user to type in a product name.  Once they've typed in a few characters, populate a side bar displaying the full name of every product that starts with the typed in letters.  Then, for each product returned, look up the top 10 customers for each. The product names must be displayed as soon as they are available, and the customer data must be filled in as it arrives.  The application must remain usable while the data is being populated.

To make things interesting, we're going to presume that distinct synchronous services already exist to lookup the list of products and customer list .  To add an additional wrinkle, there are actually 3 different product catalogs that we must check, and there is no existing service to provide an aggregate search across all 3.

Most developers will read that and immediately start thinking of asynchronous callbacks.  They are probably thinking this is do-able, but it is going to be ugly.  The power of the Rx extensions is that it allows us to turn this into a very elegant solution.

Rx is based on two very simple interfaces, Observable and Observer:

{%highlight csharp%}
public interface IObservable<out T>
{
  IDisposable Subscribe(IObserver<T> observer);
}

public interface IObserver<in T>
{
  void OnNext(T value);
  void OnError(Exception error);
  void OnCompleted();
}
{%endhighlight%}


Seasoned C# developers may notice that these two interfaces should look remarkably similar to the familiar IEnumerable and IEnumerator interfaces:

{%highlight csharp%}
public interface IEnumerable<out T>
{
  IEnumerator<T> GetEnumerator();
}

public interface IEnumerator<out T> : IDisposable
{
  bool MoveNext();
  new T Current { get; }
}
{% endhighlight %}

As it turns out, IObservable is the <a href="http://en.wikipedia.org/wiki/Duality_(mathematics)">mathematical dual</a> of IEnumerable, and IObserver is the dual of IEnumerator.  The concept of mathematical duality is relatively complicated, but the simplest way to look at it in our scenario is that these interfaces are the exact opposite of their duals.  There is a [great video](http://channel9.msdn.com/Shows/Going+Deep/Expert-to-Expert-Brian-Beckman-and-Erik-Meijer-Inside-the-NET-Reactive-Framework-Rx) on Microsoft's Channel 9 that explains the duality of these two interfaces in depth at about the 7:00 minute mark.  I highly recommend watching the entire video for a truly in depth explanation of these interfaces and the theory behind them.

Why should we care if these interfaces are duals?  Well in essences it means that everything you can do with IEnumerable you can also do with IObservable, except inverted.  Let's think about this a little bit:

<!-- Annoyed that you can't use markdown inside of block level tags -->
<div class="row">
<div class="col-md-8">
<ul>
<li>With an IEnumerable you get an IEnumerator from it via the GetEnumerator() method.</li>
<li>You ask the IEnumerator whether or not it has any more elements in its collection via the MoveNext() method, which will return true or false.</li>
<li>If it returns true, you ask for the element and it gives it to you via the Current() method.</li>
</ul>
</div>

<div class="well well-sm col-md-4">Note that often these steps are obscured by using a <code>foreach</code> loop to iterate over the collection, but they are all still executed under the covers.
</div>
</div>

All of these methods are synchronous and blocking.  Often times the collection is (wisely) implemented as a lazy collection using *yield returns* and so the caller must wait for the collection to compute its next value, if any.  If the collection is not lazy, then the entire contents of the collection are already sitting in memory, which could cause a problem for very large collections.  Lets look at doing the same thing with the IEnumberable's dual, an IObservable collection.  (Note that since everything is inverted, the top bullet point from the list above corresponds to the bottom bullet point in the list below):

* On the IObserver you define 2 methods OnNext() and OnError() that will be called when the next element of the collection is available, or when an error occurs.
	* The reason the single Current() method of IEnumerator dualized into two methods on IObserver is because Current() could also thrown an exception.
* On the IObserver you define a method, OnComplete(), that will be called when the end of the collection has been met.
	* This is an optimization of the true dual method.  The true dual would have been to define a method that is called with a true or false parameter indicating whether or not the collection has been completed.  It makes more sense to only call it in the true case and ditch the parameter.
* With an IObservable you pass an IObserver to it via the Subscribe() method.
    * The IDisposable returned corresponds to the IDisposable that IEnumerator extends.  It can be used as a way to unsubscribe from the observable stream.

<!-- Assuming you mean IObservable and IObserver in the second sentence? -->
The behavior of IObservable and IObserver is also the inverse of the behavior of IEnumerable and IEnumerator.  Where as IEnumerable and IEnumerator have only blocking operations, all of the methods on IObservable and IObserver are asynchronous and non-blocking.  Another way to look at the duality between these interfaces is that IEnumerables are *pull* based collections, where as IObservables are *push* based collections.

Right about now you may be rather unimpressed, after all the .Net runtime already comes with a whole host of techniques and libraries to create asynchronous code.  We already have Task&lt;T&gt; and the TPL and even async/await to make dealing with asynchrnous code simpler.  But the real power of Rx is that because IObservable can be viewed as a *collection* of events, all of the LINQ operators you are familiar with using over IEnumerable collections can also be used over IObservable collections.  Furthermore, these operators are themselves implemented in an asynchronous fashion as much as possible.  So not only does Rx give us pushed based collections, but it also makes these collections composeable.  For the remainder of this post I'm going to talk about how you can build reactive Observable sequences, and I'm also going to demonstrate that just because you've built an Observable doesn't necessarily mean that your code is reactive.  Lets look at our example requirements from above and see what I mean:

Here are two perfectly valid ways to build an Observable collection given a synchronous service that retrieves the collection of data: 

{% highlight c# %}
public IObservable<Product> ProductsStartingWith(string startsWith, int catalogId)
{
  IEnumerable<Product> productList = // Look up products synchronously.
  return productList.AsObservable();
}

public IObservable<Customer> CustomersForProduct(Product p) {
  return Observable.Create(IObserver observer =>
    try
    {
      IEnumerable<Customer> customers = // Look up customers synchronously.
      foreach(var c in customers)
      {
        observer.OnNext(c);
      }
      return Disposable.Empty;
    }
    catch(Exception e)
    {
      observer.OnError(e);
    }
    finally
    {
      observer.OnComplete();
    }
  );
}
{% endhighlight %}

OK, so lets look at what we have so far.  Both of these methods return an IObservable representing a collection of data.  But `ProductsStartingWith` uses the `ToObservable` extension method while `CustomersForProduct` one uses the `Observable.Create` factory method.  Lets compare these:

The `ProductsStartingWith` blocks while it looks up the product list, and the caller does not get a handle to the resulting IObservable until it has been fully populated.  It is also worth noting that an exception will be thrown out of this method if the catalog lookup failed for some reason.  This clearly violates the resilience principle.  So maybe we should have written this one differently.  Lets look at the other method.

The `CustomersForProduct` method makes use of the `Observable.Create` factory method for creating observable streams.  This method takes in an Action or Func that could execute on a separate thread.  The caller will be given the IObservable immediately, and the catalog lookup code is not execute until that IObservable has it's `Subscribe` method called.  The IObserver that is passed into the `Subscribe` method becomes the parameter to the Func in the `Create` method.

So we can see that this method is asynchronous, so far so good for adhering to the reactive principles.  But what happens if the lookup fails?  Well it is pretty easy to see that the OnError method of the IObservable will be called, and given the exception that caused the error.  One really cool side-effect of this is what happens if the IEnumerable that contains the Customers was created using `yield return` continuations.  In that case, the IObserver could still have its `OnNext` method called for *some* of the Customers prior to having its `OnError` method called when it hits a snag.  This is an excellent example the *resiliency principle*.  After looking at these two methods, the second one is clearly the better choice for a reactive application.  We'll presume that we rewrote the first method use the `Observable.Create` factory method as well.

In my next post I will talk about how to consume these IObservable collections in a reactive and elegant fashion so that we can realize the above requirements.

