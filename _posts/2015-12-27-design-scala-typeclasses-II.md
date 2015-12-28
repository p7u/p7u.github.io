---
layout: post
title:  "On software design : Scala : Type classes (pt II)"
date:   2015-12-27 00:00:00
excerpt: "Type classes in Scala and Haskell are amazing"
tags: [scala, type classes, haskell]
categories: [programming, design]
comments: true
---

In the [first part]({% post_url 2015-12-13-design-scala-typeclasses-I %})
of this article I presented some alternatives to the type classes pattern while
emphasizing the disadvantages coming with them.

#### Type classes in Scala

**_Is there a better way to solve the problems presented in the first part and
have 'the best of the two worlds', meaning, to code against the `Document`
interface while still being able to use the `Aggregate[Document]` view and, at
the same time, keep the `Document` interface decoupled and free of orthogonal
concerns?_**

It seems like, functional programming languages offer an alternative, superior
in terms of modularity and extensibility. This solution is called the 'type
class pattern'.

What follows is the way to implement the pattern in Scala.

The definition of the type class itself:

{% highlight scala %}

trait Aggregate[T] {

  def insert(o: T): Aggregate[T]

  def size: Int

}

{% endhighlight %}

An aggregate view provider is marked as being `implicit`.

{% highlight scala %}

implicit object DocumentAggregateConverter
  extends AggregateAdapter[Document] {

  override def toAggregate(target: Document): Aggregate[Document] = {

    new Aggregate[Document] {

      override def insert(o: Document): Aggregate[Document] = ???

      override def size: Int = ???
    }
  }
}

{% endhighlight %}

Another implementation artifact is a parameterized method that will take care of
the conversion.

{% highlight scala %}

def asAggregate[A](a: A)(implicit c: AggregateAdapter[A]) = c.toAggregate(a)

// alternative syntax
def asAggregate[T: AggregateAdapter](t: T) = implicitly[AggregateAdapter[T]].toAggregate(t)

// usage

val d = new Document {}

asAggregate(d).size

{% endhighlight %}

A couple of remarks:

* **_Using the parameter `implicit c: AggregateAdapter[A]` or the more
abbreviated form `implicitly[AggregateAdapter[T]]` results in the Scala compiler
supplying an instance of the required `AggregateAdapter` from the lexical scope.
Therefore, you could easily define, in different scopes, two or more different
implementations of the same type class for the same type._**

* **_By using the `asAggregate()` method, the client code will not have to deal
with adapters, but instead will use the type it's interested in directly._**

#### How about Haskell?

Why Haskell? Type classes were originally developed in Haskell as an alternative
to ad-hoc polymorphism.

Like in the case of Scala, this is not a Haskell tutorial, therefore I won't
insist on the Haskell features.

Since adopting the previous use case for Haskell requires language features that
would obscure the example, I will use a more convenient scenario.

Assume that a hypothetical feature discovered during the design phase is the
necessity for some entities in the domain model to be `Printable` (possibly
because of the need to send a representation of them to a printer). What follows
is the same reasoning as in the case of `Aggregate`s.

{% highlight haskell %}

class Printable t where
   prepare :: t -> String

type Paragraph = String

type File = String

data Document = Document {
	title    :: String,
	pars     :: [Paragraph]
}

data FileSystem = FileSystem {
	path    :: String,
	content :: [File]
}

instance Printable Document where
	prepare (Document t p) = show(t) ++ foldr (\acc x -> acc ++ " " ++ x ) "" p

instance Printable FileSystem where
	prepare (FileSystem p c) = p

printDocument :: (Printable a) => a -> String
printDocument a = prepare(a)

printDocument (Document "t" ["p", "u"])

{% endhighlight %}

A couple of remarks resulting from examining this code:

* **_Haskell has built-in support for type classes._** The keyword `class` is
used to define a type class. The keyword `instance` is for providing the
definition of what, in a certain scope, means to apply the type class to a
certain type.
Another notable observation is the fact that the definition
`instance Printable FileSystem where` defines an instance of the type class,
but than instance is not named.

* **_Haskell is much more terse than Scala._** However, Scala shines in a
different way, and I will explain what this means in the next section.

#### Conclusion

There are a few concepts worthy of jolting down :

* **_It is said that a type is added to a type class (and becomes a member
of it) if the required definitions can be provided._**

* **_Members of a type class are at any time dependent on the current scope._**
Therefore you don't care if the creator of a type anticipated the type class you
want it to belong to. If required, you can simply create your own definition and
then use it accordingly.

* **_A type does not have to know it is a member of a type class (does not
`extend` that type class), and it can be added to new type classes without the
need to modify it._** Unlike using normal interfaces, where you would have to make
that type implement the interface.

In Scala's case the search for type classes is performed locally in the scope of
the method call that triggered it. And because it's an explicitly named instance
in Scala, you can inject another instance into the scope and it will be picked
up for use for the implicit. In contrast with Haskell, whose instances are
anonymous, therefore the Haskell compiler will look into a global name space to
resolve the instance in the current scope, which is very limiting.

To sum up, a review of what type classes are and how they serve you:

* Type classes will enforce a contractual relationship between an implementer
and an implementee (same as interfaces).
* The type classes can serve as a bridge pattern – gaining
separation of orthogonal concerns (same as adapters).
* An interesting application is called `retroactive extension`. This means you
can 'attach' new behavior, concerns or, in other words, retrofit existing
types whose source code is inaccessible.
