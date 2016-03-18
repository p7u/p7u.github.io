---
layout: post
title:  "A deep dive into Scala's type system : Variance (part I) : Types & Invariance"
date:   2015-11-01 00:00:00
excerpt: "Types, type systems and invariance with Scala"
tags: [type system, scala, functional, variance]
categories: [programming, types]
comments: true
---

This is the first article in a series that will examine, as the title suggests,
Scala's type system, and, as a secondary concern, concepts and ideas that are
related to types and type systems.

The best way to emphasize theoretical notions is to give examples.
Accordingly, I chose the Scala as the instrument for this endeavor.

According to the [official website](http://www.scala-lang.org/), Scala is a
"general purpose programming language designed to express common programming
patterns in a concise, elegant, and type-safe way".

For getting the most out of the article, basic familiarity with Scala's syntax
is required. Also object-oriented concepts like inheritance, subtyping or
polymorphism should not be foreign.

#### Types

Computer programming theory defines a type as a means of classification identifying
an abstract concept, along with additional characteristics like the operations that
can be done on values of that type, the way values of that type can be stored in
memory, and so on.

For example, an `Int` type can be defined as a bounded set of integers, along
with the operations that can be performed on an integer, like addition,
subtraction, division, and so on. Additionally, `Int`s can be described by
characteristics like memory representation (for example, 32-bit integers in some
programming languages).

An `Int`, alongside other useful types, is a kind of type called a 'data type'
and it is present by default in most of the programming languages.

Many programming languages allow the programmer to create types. Sometimes these
types denote abstractions in the problem domain. And most of the times these
types are aggregates of other types.

#### The type system

A type becomes an essential property that is used by the compiler to determine
the correctness of your code.
The compiler incorporates a component called 'type checker' that will ensure
that, given a set of constraints, no violations of these constraints exist.

The set of constraints that are verified by the type checker is also called a
'type system'.

Unsurprisingly, a type is also a constraint. A type checker essentially
associates types with computed values. By examining the flow of these values, the
type checker attempts to prove that no type errors can occur.

This way, instead of having errors that manifest at runtime, the
compiler will not accept any programs that violate type restrictions. In Scala,
the way to define a type is by either defining a class, an object or by using
the `type` keyword directly.

#### Type parametrization

Type parameters allow you to define methods or types in terms of another type(s).
A type parameter is a type definition that’s taken in as a parameter when
calling a method, constructing a type, or extending a type. It can also be said
that type parameters ‘capture’ types.

It shouldn't be too surprising to find out that type parameters are also
constraints and that they can also be validated by the type checker. In Scala,
type parameters are defined by using angle brackets.

{% highlight scala %}

trait Box[T] {

  def put(t: T)

  def get: T
}

{% endhighlight %}

In this example, a type, `Box` is defined. As the name suggests, a
`Box` is a sort of container, therefore things can be put into and retrieved from
it. Also, this is a box that can hold many different things. We can call it
generic Box. To model this generic box, type parameters are used.

Now someone can create and use boxes that contain different things :

{% highlight scala %}

val integerBox = new Box[Int] {
  override def put(t: Int): Unit = ???

  override def get: Int = ???
}

val stringBox = new Box[String] {
  override def put(t: String): Unit = ???

  override def get: String = ???
}

{% endhighlight %}

#### Variance

So what makes type parameters useful and interesting?

Type parameters become important in the context of variance.

Once two types are in a subtyping relationship, the question of variance - how
they relate to each other in view of their type parameters - becomes essential.
Concretely, considering a `Box[T]`, if `Int` is a subtype of `Number`, can
`Box[Int]` be used where `Box[Number]` is expected or the other way around?

Variance also describes how type parameters can be altered to create conformant
(assignable) types. In type theory, given a parameterized type `T`, `T[A]`
conforms to `T[B]` if you can assign `T[B]` to `T[A]` without errors.

Variance is yet another type of constraint that can be enforced with the help
of the type checker. Type variance is complementary to type parameterization and
specifies constraints like invariant, covariant and contravariant to the type
system.

For this part I will focus only on invariance, leaving the other two forms for
the next articles.

#### Invariance

Invariance refers to the unchanging nature of the type parameter.
Formally, if a type is invariant, then, assuming existing types `T`, `A`, `B`,
if `T[A]` conforms to `T[B]` then `A` must be the equivalent type of `B`.

The type from the previous example, `Box`, is invariant.

For a more concrete illustration, imagine a programmer developing a warehousing
solution. He will create a few types and a method for packaging things in boxes.

{% highlight scala %}

trait CanBePacked

trait Furniture extends CanBePacked

trait Books extends CanBePacked

def packFurnitureIntoBox(b: Box[CanBePacked]) = {
  // work hard to pack things
}

val bBox: Box[Books] = new Box[Books] {
  override def put(t: Books): Unit = ???

  override def get: Books = ???
}

// disallowed
// packFurnitureIntoBox(bBox)

{% endhighlight %}

As noted, the method call `packFurnitureIntoBox(bBox)` will not be allowed by
the compiler.

The reason is straightforward. Imagine the `Box` type was not invariant. Then an
unsuspecting programmer could try to pack furniture in book boxes, as the
example underlines.

#### Closing remarks

In the next article I will demonstrate that it's actually possible to relax the
invariance restriction placed upon the type `Box`, I will discuss the forces
that could lead to such a venture, and of course I will also clarify things with
an example.
