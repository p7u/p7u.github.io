---
layout: post
title:  "A deep dive into Scala's type system: Path-dependent types and type projections"
date:   2016-05-03 00:00:00
excerpt: "The dot (.) and hash (#) operators explained"
tags: [scala, type system, functional, path-dependent, structural]
categories: [programming, types]
comments: true
---

I will continue the series on Scala's type system with a discussion about
path-dependent types, type projections and structural types.

#### Path-dependent types and type projections

Types within Scala are referred to via two mechanisms: the dot (`.`) and hash
(`#`) operators.

The `.` operator is doing the same for types as it does for members of an object,
namely, it refers to a type found on a specific object instance.

This is called a path-dependent type.

Consider this example:

{% highlight scala %}

trait MultiCastEventSink[T <: Event] {

  trait Sink[-In] {
    def notify(o: In)
  }

  // notify all internal sinks that an event happened
  def notify(o: T)

  // notify only the internal sinks present in the sequence
  def notifyOnly(o: T, s: Seq[this.Sink[T]])

  // notify the internal sinks present in the sequence, if found
  def notifyAny(o: T, s: Seq[MultiCastEventSink[T]#Sink[T]])
}

{% endhighlight %}

This trait is an abstraction of an event sink, similar to the one in
[this article]({% post_url 2015-11-29-scala-types-contravariance %}). This is a
`delegating` or `multi casting` event sink that sends all notifications to
an internal pool of sinks. In addition, this trait contains the definition of a
`regular` `Sink`, for reasons that I will describe later. Obviously, most of the
implementation details have been omitted.

A couple of `MultiCastEventSink` implementations follow as example:

{% highlight scala %}

val firstSink = new MultiCastEventSink[UserEvent] {

  override def notify(o: UserEvent): Unit = ???

  override def notifyOnly(o: UserEvent, s: Seq[Sink[UserEvent]]): Unit = ???

  override def notifyAny(o: UserEvent, s: Seq[MultiCastEventSink[UserEvent]#Sink[UserEvent]]): Unit = ???
}

val secondSink = new MultiCastEventSink[UserEvent] {

  override def notify(o: UserEvent): Unit = ???

  override def notifyOnly(o: UserEvent, s: Seq[Sink[UserEvent]]): Unit = ???

  override def notifyAny(o: UserEvent, s: Seq[MultiCastEventSink[UserEvent]#Sink[UserEvent]]): Unit = ???
}

{% endhighlight %}

Then a couple of `regular ``Sink`s are created:

{% highlight scala %}

val sink1 = new firstSink.Sink[UserEvent] {
  override def notify(o: UserEvent): Unit = ???
}

val sink2 = new secondSink.Sink[UserEvent] {
  override def notify(o: UserEvent): Unit = ???
}

{% endhighlight %}

Notice the syntax: `new firstSink.Sink[UserEvent]`. The `.` operator is used to
access a type on an already existing instance.

Consider the method `notifyOnly(o: T, s: Seq[this.Sink[T]])`. The method will
notify only the sinks present in the sequence that an event has happened. For
reasons you will understand shortly, there's a strict guarantee that all the
`Sink[T]` implementations in the sequence will be found in
`MultiCastEventSink`'s internal pool.

The second parameter of this method is a `Seq` of `Sink`s, but not of any `Sink`s.
These are `Sink`s that must have been constructed using a reference of the
current object: `this.Sink[T]]`. This means that you can’t use a type from a
different object, of the same class, to satisfy any type constraints made using
the dot operator.

You can think of this as if there’s a path of specific object instances
connected by the dot operator. For a variable to match your type, it must follow
the same object instance path.

Therefore, the statement `firstSink.notifyOnly(new UserEvent {}, Seq(sink1))` will
compile, as `sink1` was constructed using the `firstSink` instance reference.

The statement `secondSink.notifyOnly(new UserEvent {}, Seq(sink1))` `will not`
compile as `sink1` `was not` constructed using the `secondSink` instance
reference.

Also, consider the method: `notifyAny(o: T, s: Seq[MultiCastEventSink[T]#Sink[T]])`.
The second parameter is a `Seq` of `Sink`s, but in this case, they could be
`Sink`s that have been constructed using the `.` operator using `any`
`MultiCastEventSink` reference.

The `#` operator is a looser restriction than the `.` operator. It’s known as a
type projection, which is a means of referring to a nested type without
requiring a path of object instances. This means that you can reference a nested
type as if it wasn't nested.

Consequently, the statement `secondSink.notifyAny(new UserEvent {}, Seq(sink1))`
`will compile`.

#### Structural types

Imagine the following scenario: you have a trait and you want to make sure that
all types which are `mixing in` that trait must conform to the following
requirement: they must have a method with a given signature.

As a more concrete example, I want to embellish an `EventSource` with the
additional capability of self auditing. That could mean that the `EventSource`
will update possible interested entities with details about its internal state,
the number of published events, etc.

One way to accomplish this result is to use inheritance. The problem with this
approach is that you might not always be able or want to add a trait to the
classes you are using.

I'll use the `EventSource` from
[this article]({% post_url 2015-11-15-scala-types-covariance %}) as a base for
constructing the example.

The definition of the trait follows:

{% highlight scala %}

trait AuditableEventSource[T <: Event] extends Source[T] {

  // the event source must be audit-able
  this: { def audit() } =>
}

{% endhighlight %}

If you are wondering what `T <: Event` means,
[this article]({% post_url 2016-04-04-scala-types-bounds %}) will shed some
light.

A type that `mixes in` the trait has to conform to the structural specification:

{% highlight scala %}

class AuditableSystemEventSource extends AuditableEventSource[SystemEvent] {

  // structurally imposed
  def audit() = ???

  // imposed by inheritance
  override def get(): SystemEvent = ???
}

{% endhighlight %}
