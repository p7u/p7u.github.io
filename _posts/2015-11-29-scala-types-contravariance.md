---
layout: post
title:  "A deep dive into Scala's type system: Variance (part III) > Contravariance"
date:   2015-11-29 00:00:00
excerpt: "Contravariance is the opposite of Covariance"
tags: [type system, scala, functional, contravariance]
categories: [programming, types]
comments: true
---

This this is the last part out of a series of articles that examine the concept
of variance. Following the
[second part]({% post_url 2015-11-15-scala-types-covariance %}) that dealt with
covariance, the focus now will be on contravariance. A thorough reading of the
[first part]({% post_url 2015-11-01-scala-types-invariance %}) is recommended
for getting familiar with the terminology and the basic concepts.

#### Example

In the same vein as in the previous articles, I will continue with a concrete
demonstration. For the sake of convenience, I will adopt the
[same scenario]({% post_url 2015-11-15-scala-types-covariance %}), only this time
using a different perspective.

So now the programmers working on the event-driven system need to find a way to
register/process the events generated in the system. They will create a trait,
`Sink`, that is used to mark components in need to be notified when an event has
been fired.

As a consequence of marking the type parameter with the `-` symbol, the `Sink`
type became contravariant.

{% highlight scala %}

trait Sink[-In] {
  def notify(o: In)
}

{% endhighlight %}

A possible way to notify interested parties that an event happened is to write
a method and to pass it the corresponding event. This method will hypothetically
do some processing and then it will take care of notifying the event sink:

{% highlight scala %}

def appEventFired(e: ApplicationEvent, s: Sink[ApplicationEvent]): Unit = {
  // do some processing related to the event
  // notify the event sink
  s.notify(e)
}

def errorEventFired(e: ErrorEvent, s: Sink[ErrorEvent]): Unit = {
  // do some processing related to the event
  // notify the event sink
  s.notify(e)
}

{% endhighlight %}

A couple of hypothetical `Sink` implementations.

{% highlight scala %}

trait SystemEventSink extends Sink[SystemEvent]

val ses = new SystemEventSink {
  override def notify(o: SystemEvent): Unit = ???
}

trait GenericEventSink extends Sink[Event]

val ges = new GenericEventSink {
  override def notify(o: Event): Unit = ???
}

{% endhighlight %}

The following method calls are accepted by the compiler:

{% highlight scala %}

appEventFired(new ApplicationEvent {}, ses)

errorEventFired(new ErrorEvent {}, ges)

appEventFired(new ApplicationEvent {}, ges)

{% endhighlight %}

#### Contravariance explained

Looking at the series of calls you notice that it is possible to call a method
expecting a `Sink[ApplicationEvent]` with a `Sink[SystemEvent]` and even with
a `Sink[Event]`.
Also, you can call the method expecting a `Sink[ErrorEvent]` with a
`Sink[Event]`.

How is this possible?

By replacing [invariance]({% post_url 2015-11-01-scala-types-invariance %})
with a contravariance constraint, a `Sink[SystemEvent]` becomes a subtype of
`Sink[ApplicationEvent]`. Therefore, contravariance can also be thought of as a
'widening' relationship, since types are 'widened' from more specific to more
generic.

Like in the case of [covariance]({% post_url 2015-11-15-scala-types-covariance %}),
the contravariance annotation makes it possible to create a type hierarchy
between parameterized types that is parallel to the type hierarchy of the types
used as parameters.

Only in this case, the direction of inheritance between parameterized
types like, `Sink[UserEvent]` and `Sink[Event]` is the inverse of the
direction of inheritance between `UserEvent` and `Event`, as depicted in the
following diagram.

![](/images/scala-contravariance-pic.png)

Hence the name, contravariance.

Formally, if a type is covariant, then, assuming existing types `T`, `A`, `B`,
if `T[B]` conforms to (is assignable to) `T[A]` then `B` must be the super type
of `A`.

Conformance follows the direction of inheritance between the parameterized
types, therefore, the following statement would also compile:
`val v: Sink[UserEvent] = ges`

#### Closing remarks

This article explained and demonstrated covariance with the help of a practical
example. Next articles will focus on other aspects related to the type system.
