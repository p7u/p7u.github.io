---
layout: post
title:  "A deep dive into Scala's type system: Type Constraints"
date:   2016-04-04 00:00:00
excerpt: "Can you apply restrictions on type parameters?"
tags: [scala, type system, functional, bound, context, manifest, restrict]
categories: [programming, types]
comments: true
---

The first parts ([one]({% post_url 2015-11-01-scala-types-invariance %})
[two]({% post_url 2015-11-15-scala-types-covariance %})
[three]({% post_url 2015-11-29-scala-types-contravariance %})
) of the series describe the concept of variance.

[Part one]({% post_url 2015-11-01-scala-types-invariance %}) is an useful
read as it gives a good introduction to types and prepares the field.

All mechanisms presented here offer insights on how to apply constraints to the
type parameters.

#### Type bounds

Looking back to [part three]({% post_url 2015-11-29-scala-types-contravariance %})
of the variance series of articles, the two methods, `errorEventFired` and
`appEventFired` are certainly in need of some improvements.

Assuming that both of these methods handle `ApplicationEvent`s and `ErrorEvent`s
in the same way, the duplication can be removed considering that the two events
are sub-types of `SystemEvent`:

{% highlight scala %}

def systemEventFired[E <: SystemEvent](e: E, sink: Sink[SystemEvent]): Unit = {
  // do some processing related to the event
  // ......
  // notify the event sink
  sink.notify(e)
}

// ApplicationEvent is a subtype of SystemEvent
systemEventFired(new ApplicationEvent {}, ses)

// ErrorEvent is a subtype of SystemEvent
systemEventFired(new ErrorEvent {}, ges)

{% endhighlight %}

An useful heuristic for understanding the `<:` symbol is:

{% highlight Text only %}

The selected type `E` must be equal to or a subtype of the `upper bound type`

{% endhighlight %}

The mechanism that allows this kind of constraint setting, made available by
Scala's type system is called `type bound`.

This kind of bound is called an `upper bound` and the syntax is
`T <: UpperBoundType`.

Scala also supports `lower bounds`, and, unsurprisingly, the syntax is
`T >: LowerBoundType`.

Consider the following contrived example, a method that receives an event and an
event source and filters out the events that are errors:

{% highlight scala %}

def filterOutErrorEvents[E >: ApplicationEvent](e: E, src: Source[SystemEvent]): Unit = {
  // the source might also emit an error event, but it is not certain
  src.get()
}

{% endhighlight %}

A useful heuristic for understanding the `>:` symbol is:

{% highlight Text only %}

The selected type `E` must be equal to or a supertype of the `lower bound type`

{% endhighlight %}

In this scenario, the `e` param is constrained to be only of type
`ApplicationEvent` and `SystemEvent` but not of type `ErrorEvent`!

The `src` param is constrained to be only of type `Source[SystemEvent]`,
`Source[ApplicationEvent]` and `Source[ErrorEvent]`. This is a direct
consequence of [covariance]({% post_url 2015-11-15-scala-types-covariance %}).

The following call compiles just fine:

{% highlight scala %}

filterOutErrorEvents(new SystemEvent {}, syes)

{% endhighlight %}

As a reminder, the type of `syes` is

{% highlight scala %}

trait SystemEventSource extends Source[SystemEvent]

{% endhighlight %}

Obviously, the method compiles even if you pass it a `Source[ErrorEvent]`:

{% highlight scala %}

trait ErrorEventSource extends Source[ErrorEvent]

private val ees = new ErrorEventSource {
  override def get(): ErrorEvent = ???
}

filterOutErrorEvents(new SystemEvent {}, ees)

{% endhighlight %}

#### Context bounds

A context bound has the syntax `T : M`.

An useful heuristic for understanding the syntax is:

{% highlight Text only %}

`M` is another parameterized type, and an implicit value of type `M[T]` must
exist in the scope

{% endhighlight %}

Now there's a good time to go back to
[this article]({% post_url 2015-12-27-design-scala-typeclasses-II %}).

The example:

{% highlight scala %}

def asAggregate[T: AggregateAdapter](t: T) = implicitly[AggregateAdapter[T]].toAggregate(t)

{% endhighlight %}

is presented as 'alternative syntax' since I did not want to introduce another
concept (bounds) while dealing with type classes.

Applied to this example, the heuristic is:

{% highlight Text only %}

`AggregateAdapter` is another parameterized type, and an implicit value of type
`AggregateAdapter[T]` should exist in the scope.

{% endhighlight %}

The following lines create a `Document` and pass it to the `asAggregate` method,
which will take care of the conversion.

{% highlight scala %}

val d = new Document {}

asAggregate(d).size

{% endhighlight %}


The heuristic in this case:

{% highlight Text only %}

When calling the method with a `Document` parameter, an implicit value of type
`AggregateAdapter[Document]` should exist in the scope

{% endhighlight %}

#### The Manifest Context Bound

This presentation would not be complete without mentioning the manifest context
bound.

Syntax: `T : Manifest`.

`Manifest`s were added to Scala especially for arrays and were generalized to be
useful in other situations where type information must be available at runtime.
In other words, `Manifest`s are a way to achieve `reification` on the JVM.

A Scala `Array` is a parameterized class. For example, an array of integers has
the type `Array[Int]`. So code can be written for generic `Array[T]` types, but
because on the JVM there are different types of arrays for every primitive type
and for objects (for example `int[]`, `short[]` or `String[]`), a way to retain
this specialized type information was required so that generic array
implementations would know what underlying type is required.

To instantiate a generic `Array[T]`, a `Manifest[T]` object is needed.

{% highlight scala %}

def threeTimesPackedIntoArray[T <: Event : Manifest](e: T): Array[T] = {
  val a = new Array[T](3)
  a(0) = e; a(1) = e; a(2) = e
  a
}

threeTimesPackedIntoArray(new SystemEvent {})

threeTimesPackedIntoArray(new ErrorEvent {})

{% endhighlight %}

Using the aforementioned heuristic(s), we have:

{% highlight Text only %}

The selected type `T` must be equal to or a subtype of `Event`, and an implicit
value of type `Manifest[T]` must exist in the scope

{% endhighlight %}

That `Manifest[T]` is generated by the compiler, when needed, with all the
known information for that type at that time.
