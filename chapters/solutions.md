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
