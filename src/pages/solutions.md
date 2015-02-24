# Solutions

## Free Monoids

### Basics
\label{sol:free_monoid_basics}

```scala
List(1, 2, 3, 4).foldLeft(0){_ + _}
List(1, 2, 3, 4).foldLeft(1){_ * _}
```

### Monoid Interpreter
\label{sol:free_monoid_interpreter}

If you have taken *Essential Scalaz* you should recognise this is a limited form of `foldMap`. Here's one implementation of it

```scala
import scalaz.Monoid

object FoldMonoid {
  def apply[A](in: List[A])(implicit monoid: Monoid[A]) =
    in.foldLeft(monoid.zero)(monoid.append)
}
```

## Free Monads

### Unnatural Transformations
\label{sol:free_monoid_unnatural}

```scala
val listExe: Option ~> List = new (Option ~> List) {
  def apply[A](in: Option[A]): List[A] =
    in map (List(_)) getOrElse Nil
}
```

### Timer Monad Instance
\label{sol:free_monoid_timer_monad}

```scala
implicit val logMonad = new Monad[Timer] {
  def bind[A, B](fa: Timer[A])(f: (A) ⇒ Timer[B]): Timer[B] = {
    val now = System.nanoTime()
    val result = fa.value.map(f).getOrElse(Timer(DList(), None))
    Timer((fa.times :+ now) ++ result.times, result.value)
  }
  def point[A](a: ⇒ A): Timer[A] = {
    Timer(DList(System.nanoTime()), some(a))
  }
}
```

### Timer Natural Transformation
\label{sol:free_monoid_timer_natural}

```scala
val timerExe: Option ~> Timer = new (Option ~> Timer) {
  def apply[A](in: Option[A]): Timer[A] =
    Timer(DList(), in)
}
```
