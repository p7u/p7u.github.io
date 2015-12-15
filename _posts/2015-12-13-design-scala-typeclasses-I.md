---
layout: post
title:  "On architecture and design : Scala : Type classes (pt I)"
date:   2015-12-13 00:00:00
excerpt: "Why type classes are needed"
tags: [scala, type classes]
categories: [programming, design]
comments: true
---

This is the first article in a series focused on finding and examining ideas,
patterns, idioms that will increase the quality of software.

Naturally, I will dive into concepts like modularity, extensibility, design but
also architectural patterns will be scrutinized.

As a first task, I chose to look into type classes and chose Scala as an
instrument to facilitate my endeavor.

According to the [official website](http://www.scala-lang.org/), Scala is a
"general purpose programming language designed to express common programming
patterns in a concise, elegant, and type-safe way".

For getting the most out of this article, basic familiarity with Scala's syntax
is required. Also object-oriented concepts like inheritance, subtyping or
polymorphism should not be foreign.

#### Basic reasoning

One of the major breakthroughs in the history of computer programming has been
the introduction of languages that offer the programmer new tools to
conceptualize, to move away from the nitty-gritty details of the machine and
build abstractions. These abstractions help us reason in a better way about our
designs, and will also lead to more modular and decoupled programs with the
benefits of easier extensibility and maintenance in the long term.

Therefore, when faced with the task of designing a new system, the programmer
will attempt to 'translate' the concepts of the problem domain into abstractions
in the solution domain. For example, when he has to design a document management
system, the programmer will recognize the concept of a `Document` as a key
abstraction in his problem/solution domain. In object-oriented jargon, he will
create a `Document` interface. When using Scala, the `Document` becomes a type
(in scala an interface is called a `trait`). For an introduction to types in
Scala, read [this article]({% post_url 2015-11-01-scala-types-invariance %}).

He will then discover that his abstractions carry several properties, related to
the problem or application domain. In the case of the `Document`, it could be
persisted, or you can add paragraphs to it, or it can be printed, and so on. The
object-oriented world will recognize these properties as methods of an interface.

Fundamentally, the programmer will look for a way to create a contract and to
enforce it across several entities that embody the essence of that contract. For
example, the programmer will discover a contract called `Eq` for things that can
be equated. The contract could define the properties `==` (equal) and `/=` (not
equal) applicable for that contract. If then we have another abstraction (say,
`Document`) and it makes sense to apply the equality functions to a document,
the programmer can enforce the contract of `Eq` on `Document`.

#### Example

To make things clearer, I'll give an example.

**_Let's consider the concept of aggregating. This is an abstraction since you can
'aggregate' many 'things', like, datastore columns/rows, in-memory collection
elements, files, `Document`s, and so on._**

I have defined a trait, `Aggregate`, which models this abstract concept.
In this hypothetical example, only a couple of operations are supported:
inserting into an aggregate and querying the size of the aggregate (the number
of parts that comprise the aggregate).

{% highlight scala %}

trait Aggregate[T] {

  def insert(o: T): Aggregate[T]

  def size: Int

}

{% endhighlight %}

Since this aggregate behavior can be shared by many abstractions, the `Aggregate`
type is parameterized, or, more formally, the operations it supports can be
applied identically to different types, a characteristic which is sometimes
defined as parametric polymoprphism. You can
[find here]({% post_url 2015-11-01-scala-types-invariance %}) an introduction
to parameterized types.

**_The usual way to model the fact that an abstraction 'supports' a contract
is through inheritance._**

{% highlight scala %}

trait Document extends Aggregate[Document] {

  def addParagraph(p: String): Document = ???

  def numberOfWords: Int = ???

  override def insert(o: Document): Aggregate[Document] = ???

  override def size: Int = ???
}

trait FileSystem extends Aggregate[FileSystem] {

  def numberOfFiles: Int = ???

  override def insert(o: FileSystem): Aggregate[FileSystem] = ???

  override def size: Int = ???
}

{% endhighlight %}

The implementations have been purposely omitted.

#### Remarks

There are a few interesting comments to be made by looking at this example.

**_A first observation is the fact that `Document`s or `FileSystem`s themselves
represent concepts in the domain they are modeling. They have properties relevant to the
problem to be solved but it can be argued that representing a `Document` as an
aggregate of `Document`s is an orthogonal matter, that can 'cut across' many
different abstractions and that does not necessarily concern the `Document`
itself._**

**_Furthermore, using this 'traditional' approach based on inheritance, for a class
or a trait to be 'member' of an interface, it must declare that at the site of
its own definition (using the `extends` or `with` keywords in our case). In
other words, the `Document` type 'is aware' that it is also an aggregate.
Therefore, all `Document` implementations will be strongly coupled to the
`Aggregate` contract._**

This raises another interesting question. What if, at some point in time,
someone decides that `Document`s need to implement another contract? Not only
that every consumer of the `Document` API is forced to deal with characteristics
that might not be interesting to him (the `Document` interface becomes 'fatter'),
but also the source code of the `Document` needs to be changed. Also, all types
implementing the `Document` contract (all implementations) need to be adapted to
the new change (they need to implement the new contract and their source code
needs to be changed too).

While it is possible, via polymorphism, or because of the elasticity of the
type system, to have different clients talk only to the interface they are
interested in (in our case, a method expecting a parameter of type
`Aggregate[Document]` can be passed a `Document` instance because a `Document`
*can also be seen as an* `Aggregate[Document]`), this solution will only
partially mitigate some of these problems.

#### Adapters to the rescue

I'll assume that the creator of the `Document` abstraction is aware of the
aforementioned disadvantages. As an alternative, he designs an adapter that is
responsible for encapsulating the `Aggregate` behavior of the target type.

{% highlight scala %}

trait AggregateAdapter[T] {

  def toAggregate(target: T): Aggregate[T]
}

object DocumentAggregateAdapter
  extends AggregateAdapter[Document] {

  override def toAggregate(target: Document): Aggregate[Document] = {

    new Aggregate[Document] {

      override def insert(o: Document): Aggregate[Document] = ???

      override def size: Int = ???
    }
  }
}

{% endhighlight %}

**_However, this approach comes with disadvantages, too. Leaving aside the ones that
are commonly associated with the 'Adapter' pattern, the most important is that,
ideally, the client code would want to code against the `Document` adaptee
interface, but also retain the possibility to view the `Document` as an
`Aggregate` without the forced shift that the adapter option represents._**

So at this point, it seems like the designers have reached a dead end.

#### Closing remarks

In the second part of this article, I will show that it's possible to have 'the
best of the two worlds', meaning, to code against the `Document` interface while
still being able to use the `Aggregate[Document]` view and at the same time,
keep the `Document` interface decoupled and free of other orthogonal concerns.
