# Free Monads

In the previous section we saw free monoids. **Free monads** follow the same general idea, representing expressions (in this case monadic expressions) using data structures, and then separately defining an interpreter for these structures.

Free monoids are simple enough we could go directly to the free monoid structure without much in the way of examples. Free monads are a bit more complex (but not much more!) so we're going to first build an example and then look into the implementation.

## Free Options

We are all familiar with `Option`. We know we can use it to model optional values and combine results using the monad for `Option`.

```scala
for {
  x <- Some(1)
  y <- Some(2)
  z <- Some(3)
} yield x + y + z
```

We can do the same thing using the free monad. We first have to lift `Option` into the free monad, which we do using `Free.liftF`. This requires a `Functor` instance for our type (not a monad!)

```scala
import scalaz.Free
import scalaz.std.option._

val free =
  for {
    x <- Free.liftF(some(1))
    y <- Free.liftF(some(2))
    z <- Free.liftF(some(3))
  } yield x + y + z
```

This doesn't actually run our computation. It returns a `Free[Option, Int]` which represents the suspended computation. We can run our computation by passing an interpreter to the `foldMap` method. The interpreter takes a function (which we'll say more about in a bit) that says how to convert the functor in our free monad (currently `Option`) to a monad that will do the actual computation.

The simplest interpreter will just be the identity, converting our `Option` into `Option`. Let's implement this:

```scala
import scalaz.~>

val idExe: Option ~> Option = new (Option ~> Option) {
  def apply[A](in: Option[A]): Option[A] =
    in
}
```

Don't worry about the `\textasciitilde>` symbol. We'll get to that in a bit.

With this definition we can run our computation.

```scala
scala> import scalaz.std.option._

scala> free.foldMap(idExe)
res0: Option[Int] = Some(6)
```

## Natural Transformations

The `\textasciitilde>` syntax, actually a type used with infix notation, represents a [natural transformation](http://docs.typelevel.org/api/scalaz/nightly/#scalaz.NaturalTransformation). A natural transformation is just the functor-level equivalent of a function. A function of type `A => B` transforms `A`s into `B`s. A natural transform with type `A \textasciitilde> B` transforms `A[_]`s into `B[_]`s. As we can see from the example above, they are fairly straightforward to define.

## Exercises

### Unnatural Transformations

Let's explore natural transformations by implementing a transformation `Option \textasciitilde> List` and using it to run our free monad. Finish the implementation of

```scala
val listExe: Option ~> List = ???
```

and then you should be able to run

```scala
scala> import scalaz.std.list._

scala> free.foldMap(listExe)
res2: List[Int] = List(6)
```

\textsc{Solution~\ref{sol:free_monoid_unnatural}}


### Timing. It's All About Timing

Now we've seen how to write natural transformations, let's write a slightly more interesting monad and then create a natural transformation for it.

Start with this definition

```scala
import scalaz.DList
final case class Timer[A](times: DList[Long], value: Option[A])
```

A `DList` is a data structure with an efficient append operation. Now we're going to implement a `Monad` for this data structure. The important point is that when we create a `Timer` using `point` or perform a step in our computation using `bind` (what Scalaz called `flatMap`) we want to record time information using `System.nanoTime()`. When we do a `bind` we also want to keep any times recorded in other steps. Complete the implementation below

```scala
object Timer {
  import scalaz.Monad

  implicit val logMonad: Monad[Timer] = new Monad[Timer] {
      def bind[A, B](fa: Timer[A])(f: (A) ⇒ Timer[B]): Timer[B] = ???
      def point[A](a: ⇒ A): Timer[A] = ???
  }
}
```

\textsc{Solution~\ref{sol:free_monoid_timer_monad}}

Now implement a natural transformation from `Option` to `Timer`.

```scala
val timerExe: Option ~> Timer = ???
```

With this you should be able to run

```scala
scala> val timer = free.foldMap(timerExe)
timer: option.Examples.Timer[Int] = Timer(scalaz.DList@dcd1d9c,Some(6))

scala> timer.times.toList
res4: List[Long] = List(1415887258135741000, 1415887258135758000, 1415887258135774000, 1415887258135783000)

scala> timer.value
res6: Option[Int] = Some(6)
```

\textsc{Solution~\ref{sol:free_monoid_timer_natural}}

\input{chapters/interpreters}
