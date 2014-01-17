---
layout: post
category : "Centare Blog"
tags: [Reactive, Rx, .Net]
overrideTitle: "Reactive Programming using Rx .Net"
tagline: 

---
{% include JB/setup %}

In my previous post (make link) I talked about the high level principles of reactive programming.  In this and future posts I want to delve deeper into the topic and explore how to actually write reactive applications on different platforms.  The platform of choice for this week is .Net using the [Rx extensions](https://rx.codeplex.com/) from Microsoft Open Technologies.

<!--excerpt-->

Rx is based on two very simple interfaces, Observable and Observer:

{% highlight c# linenos %}
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
{% endhighlight %}


Seasoned C# developers may notice that these two interfaces should look remarkably similar to the familiar IEnumerable and IEnumerator interfaces:

{% highlight c# linenos %}
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

These interfaces are the exact opposite of each other.  In fact, IObservable is the <a href="http://en.wikipedia.org/wiki/Duality_(mathematics)">mathematical dual</a> of IEnumerable, and IObserver is the dual of IEnumerator.  There is a [great video](http://channel9.msdn.com/Shows/Going+Deep/Expert-to-Expert-Brian-Beckman-and-Erik-Meijer-Inside-the-NET-Reactive-Framework-Rx) on Microsoft's Channel 9 that explains the duality of these two interfaces at about the 7:00 minute mark.  I highly recommend watching the video for a truly in depth explanation of these interfaces and the mathematical theory behind them.

Why should we care if these interfaces are duals?  Well in essences it means that everything you can do with IEnumerable you can also do with IObservable, except inverted.  Let's think about this a little bit:

* With an IEnumerable you get an IEnumerator from it via the GetEnumerator() method.  
* You ask the IEnumerator whether or not it has any more elements in its collection via the MoveNext() method.  
* If it returns true, you ask for the element and it gives it to you via the Current() method.  

All of these methods are synchronous and blocking.  Often times the collection is (wisely) implemented as a lazy collection using *yield returns* and so the caller must wait for the collection to compute its next value, if any.  If the collection is not lazy, then the entire contents of the collection are already sitting in memory, which could cause a problem for very large collections.  Lets look at doing the same thing with the dual IObservable collection:

* With an IObservable you pass an IObserver to it via the Subscribe() methods.
    * The IDisposable returned corresponds to the IDisposable that IEnumerator extends.  It can be used as a way to unsubscribe from the observable stream.
* On the IObserver you define a method, OnComplete(), that will be called when the end of the collection has been met.
	* This is an optimization of the true dual method.  The true dual would have been to define a method that is called with a true or false parameter indicating whether or not the collection has been completed.  It makes more sense to only call it in the true case and ditch the parameter.
* On the IObserver you define 2 methods OnNext() and OnError() that will be called when the next element of the collection is available, or when an error occurs.
	* The reason the single Current() method of IEnumerator dualized into two methods on IObserver is because Current() could also thrown an exception.

The behavior of IObservable and IObserver is also the inverse of the behavior of IEnumerable and IEnumerator.  Where as IEnumerable and IEnumerator have only blocking operations, all of the methods on IObservable and IObservable are asynchronous and non-blocking.  Another way to look at the duality between these interfaces is that IEnumerables are *pull* based collections, where as IObservables are *push* based collections.

Right about now you may be rather unimpressed, after all the .Net runtime already comes with a whole host of techniques and libraries to create asynchronous code.  We already have Task&lt;T&gt; and the TPL and even async/await to make dealing with asynchrnous code simple.  But the real power of Rx is that all of the LINQ operators you are familiar with using over IEnumerable collections can also be used over IObservable collections, and furthermore these operators are themselves implemented in a completely asynchronous fashion.  So not only does Rx give us pushed based collections, but it also makes these collections composeable.  Lets dive into some examples and see exactly what this means.

### Examples

1. Creating Observables
2. Subscribing to Observables
3. Composing Observables
    1. Using standard LINQ operators
    2. Using Rx additions to LINQ
4. Working with Subjects (Observable and Observer in one)
5. Working with Schedulers

#### Creating Observables

The Rx library is loaded with many different ways to create Observable instances.  Several of them are not that useful for production code, but are great for testing or for examples.  Some of these are:
* <code>Observable.Return(T)</code>: Creates a sequence with a single value and then completes.
* <code>Observable.Empty()</code>: Creates a sequence that completes without any values.
* <code>Observable.Never()</code>: Creates a sequence that never completes and never produces any values or errors.
* <code>Observable.Throw(Exception)</code>: Creates a sequence that produces one error.
* <code>Observable.Range(min, max)</code>: Creates an sequence with all values from min to max in sequence.
* <code>Observable.Timer(value, timeSpan)</code>: 
* <code>Observable.Interval(timeSpan)</code>: Creates an Observable that 'ticks' an incrementing value at the specified interval.

There are also several factory methods that will convert familiar .Net constructs into Rx Observables:
* <code>Observable.Start(func or action)</code>: This is kind of a lazy version of <code>Observable.Return(value)</code> whereas this version executes the action or function lazily, when the Observable is first subscribed to.
* <code>Observable.FromEventPattern(...)</code>: This creates an Observable that is a .Net event handler.  This can be used to create an Observable stream of UI events, for example.  There are several overloaded versions of this to handle different styles of events.
* <code>Observable.FromTask(task)</code>
* <code>Task&lt;T&gt;.ToObservable()</code>: This extension method creates an Observable that will contain a single value of the result of the task and then terminate.

#### Subscribing to Observables

Talk about the implicit contract of how a stream is terminated (onComplete or onError, but not both)

#### Composing Observables using standard LINQ operators

One of the simplest things you can do with LINQ is to filter out the elements you don't want from a collection.  This is done with the Where function:

{% highlight c# %}
IObservable<long> xs = Observable.Interval(TimeSpan.FromSeconds(1));
IObservable<long> odds = xs.Where(x => x%2 == 1);
IObservable<long> evens = xs.Where(x => x%2 == 0);
{% endhighlight %}

Note how creating both the odds and evens collections demonstrates that it is possible to compose multiple collections from a single collection.  In this example the evens and odds Observables will each produce a value every 2 seconds, alternating on even and odd seconds from when the original collection was created.

Sometimes it is useful to transform the values in a collection from one value to another, including changing the type.  With LINQ this is done with the (poorly named in my opinion) Select() function.  The following example produces an Observable sequence that produces an output every second of type string instead of long:

{% highlight c# %}
IObservable<long> xs = Observable.Interval(TimeSpan.FromSeconds(1));
IObservable<string> transformed = xs.Select(x => String.Format("It has been {0} seconds", x));
{% endhighlight %}

#### Composing Observables using Rx additions to LINQ

Time is not something that is generally a concern when iterating over a synchronous collection, however it can become a significant concern when dealing with asynchronous Observables that make produce outputs at any rate.  To handle these scenarios, Rx added several new LINQ operators that deal specifically with time.

The <code>Buffer</code> operator collects all of the values received for awhile and then publishes all of those values as a (synchronous) IList of those values.  Buffer comes in two flavors, one that buffers for a specific amount of time, and one that buffers until a specific number of values are in the buffer.  There is also a Buffer that works on both time and count that will produce a value when either one of those limits have been reached.  In the following example the bufferedByTime and bufferedByCount sequences are equivalent.

{% highlight c# %}
IObservable<long> xs = Observable.Interval(TimeSpan.FromMilliseconds(250));
IObservable<IList<long>> bufferedByTime = xs.BufferWithTime(TimeSpan.FromSeconds(1));
IObservable<IList<long>> bufferedByCount = xs.BufferWithCount(4);
{% endhighlight %}  

Sometimes when the Observable is pushing values too quickly you don't actually need every value that it produces.  That is where the <code>Sample</code> and <code>Throttle</code> operators come in among others.

The <code>Sample</code> operator simply returns the latest value of the sequence for every specified *TimeSpan*.  For example, the samples sequence in the code below will produce a value every second with the values 9, 19, 29, etc.

{% highlight c# %}
IObservable<long> xs = Observable.Interval(TimeSpan.FromMilliseconds(100));
IObservable<long> samples = xs.Sample(TimeSpan.FromSeconds(1));
{% endhighlight %}

The <code>Throttle</code> operator is similar but it waits until the source stream has not produced a value for a specific amount of time, and then it emits the latest value from the source.  This can be especially useful if you know that the source stream produces values in bursts and you don't care about the intermediate values.  A concrete example of this is listening to keyboard events.  You don't want to do anything while the user is in the middle of typing, but rather you should wait until they have been idle for a few hundred milliseconds, and then do your work.

Sometimes you could have multiple possible sources for the same information, but you only want to use the one that is currently performing the best.  Suppose you want get a real-time stream of stock prices for a particular stock.  There are many different web services for that sort of data, but one one day source A might perform better but on another day source B might perform better.  The <code>Amb</code> (short for ambiguous) operation can help here.  It takes in two  or more observable sequences and only reports values for the one that produced a value first.  Using this operator, we can create observables against BOTH sources, combine them with <code>Amb</code> and the first one to return a value is used to produce the entire stream.  Here is what that would look like in code:

{% highlight c# %}
IObservable<StockTicker> tickerA = // (Create an observable from source A);
IObservable<StockTicker> tickerB = // (Create an observable from source B);

IObservable<StockTicker> ticker = tickerA.Amb(tickerB);

// Alternatively (useful syntax when combining more than 2)
IObservable<StockTicker> ticker = Observable.Amb(tickerA, tickerB, tickerC);
{% endhighlight %}

#### Working with Subjects

blah

#### Working with Schedulers

blah

  All we've done is create some push collections that will generate values in perpetuity.  In order to work with the values pushed out of the collections, we need to <code>Subscribe</code> to them:

{% highlight c# %}
val xs = Observable.Interval(TimeSpan.FromSeconds(1));
using(val subscription = cs.Subscribe(Console.WriteLine))
{
    // While this code is running, values will be printed to the console every second
}

/*
Code executed here will NOT have values printed every second, because the subscription has been disposed of.  The observable collection, however, is still generating values until it falls out of scope.
*/
{% endhighlight %}

Creating the subscription as part of a using block is a convenient way to ensure proper disposal of a subscription, but of course subscriptions can also be passed around to other parts of your application.

