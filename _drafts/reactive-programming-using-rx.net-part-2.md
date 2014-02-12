---
layout: post
category : "Centare Blog"
tags: [Reactive, Rx, .Net]
overrideTitle: "Reactive Programming using Rx .Net"
tagline: Part 2

---
{% include JB/setup %}

in this third post in my [reactive programming]({% post_url 2014-01-20-intro-to-reactive-programming %}) blog series I want to continue where we left off in the [second post]({% post_url 2014-02-03-reactive-programming-using-rx.net-part-1 %}) in discussing the Rx .Net framework.  Last time I talked about how to construct Observable sequences in accordance with the *reactive principles*.  Know that this is not the only way to construct an Observable, but it is the most general way.  The Rx library ships with many convenience factory methods that allow you to easily construct Observable sequences from delegates, async tasks, and from events.

For the remainder of this post I want to talk about consuming Observables.

<!--excerpt-->

