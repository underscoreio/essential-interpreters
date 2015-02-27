# Free Monoids

Let's start our exploration of free objects by looking at the **free monoid**. Consider an expression like

```scala
1 |+| 2 |+| 3 |+| 4
```

We know this a monoid expression. Using Scalaz's default monoid instance for `Int` it will evaluate to `10`. Now imagine we want to represent this expression using a data structure, so we can later choose how to interpret it. For example, we might choose to use the multiplication monoid (with `*` as the operation and `1` as the identity) instead of the addition monoid.

We know we can apply brackets arbitrarily, because monoids are required to be associative. Here's an example bracketing:

```scala
(1 |+| (2 |+| (3 |+| 4)))
```

Now we can construct a data structure to representing this expression. We clearly need some kind of `Pair` datastructure to represent the binary operation.

```scala
final case class Pair[A](left: A, right: A)
```

but notice that sometimes the `right` element in a `Pair` is another `Pair`. We can use this definition instead

```scala
sealed trait FreeMonoid[A]
final case class Pair[A](left: A, right: FreeMonoid[A]) extends FreeMonoid[A]
final case class End[A]() extends FreeMonoid[A]
```

If you've taken *Essential Scala* you should recognise this an *algebraic data type*, and also as being equivalent to the `List` data structure we implemented in that course.

We can just use Scala's built-in `List` instead, giving us

```scala
List(1, 2, 3, 4)
```

or equivalently

```scala
1 :: 2 :: 3 :: 4 :: Nil
```

as a representation of the monoid expression `1 |+| 2 |+| 3 |+| 4`. The list data type is in fact the free monoid.

## A Quick Review

Let's quickly recap what we have done. Using `List` we can represent the structure of an expression containing monoid operations without choosing any particular monoid implementation. We have in effect an *abstract syntax tree* representing monoid operations. Given an abstract syntax tree we can implement an interpreter for that tree that gives meaning to the expression it represents.

A free object means one that contains only the structure needs to represent the object in question (in our case, monoids) and nothing else. `List` meets these criteria.

## Exercises

### Basics

Given `List(1, 2, 3, 4)`

- add up the elements of the list; and
- multiply together the elements of the list.

<div class="solution">
```scala
List(1, 2, 3, 4).foldLeft(0){_ + _}
List(1, 2, 3, 4).foldLeft(1){_ * _}
```
</div>


### Monoid Interpreter

Given a `List` and a `Monoid` instance, implement a function that applies the monoid instance to the list. For example:

```scala
FoldMonoid(List(1, 2, 3, 4))(Monoid[Int])
```

<div class="solution">
If you have taken *Essential Scalaz* you should recognise this is a limited form of `foldMap`. Here's one implementation of it

```scala
import scalaz.Monoid

object FoldMonoid {
  def apply[A](in: List[A])(implicit monoid: Monoid[A]) =
    in.foldLeft(monoid.zero)(monoid.append)
}
```</div>
