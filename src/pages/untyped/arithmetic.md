## Interpreting Arithmetic Expressions

Let's start our exploration by writing an interpreter for arithmetic expressions. 

We start with a very simple algebraic data type to represent our expressions.

```scala
sealed trait Expression
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Value(get: Double) extends Expression
```

We'll implement our interpreter as a method `eval`. It's a straightforward structural recursion.

```scala
sealed trait Expression {
  def eval: Double =
    this match {
      case Plus(l, r)     => l.eval + r.eval
      case Minus(l, r)    => l.eval - r.eval
      case Multiply(l, r) => l.eval * r.eval
      case Divide(l, r)   => l.eval / r.eval
      case Value(v)       => v
    }
}
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Value(get: Double) extends Expression
```

We can write some simple examples to show expressions evaluate to the expected result.

```scala
Plus(Value(1), Minus(Value(2), Value(3))).eval
// res: Double = 0.0

Plus(Value(1), Minus(Value(3), Value(2))).eval
// res: Double = 2.0
```

Adding some method makes expressions easier to read.

```scala
sealed trait Expression {
  def eval: Double =
    this match {
      case Plus(l, r)     => l.eval + r.eval
      case Minus(l, r)    => l.eval - r.eval
      case Multiply(l, r) => l.eval * r.eval
      case Divide(l, r)   => l.eval / r.eval
      case Value(v)       => v
    }

  def +(that: Expression): Expression =
    Plus(this, that)
  def -(that: Expression): Expression =
    Minus(this, that)
  def *(that: Expression): Expression =
    Multiply(this, that)
  def /(that: Expression): Expression =
    Divide(this, that)
}
object Expression {
  // Smart constructor that creates Value with type Expression
  def value(in: Double): Expression =
    Value(in)
}
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Value(get: Double) extends Expression
```

With this we can write readable expressions, but this will not be a focus of the book.

```scala
import Expression._
(value(1) + value(3) - value(2)).eval
// res: Double = 2.0
```

We've now seen a very simple first interpreter. Our interpreters will always have the same structure:

- we'll have a data structure, sometimes called an *abstract syntax tree* or *intermediate representation* describing the program; and
- our interpreter will consume this data structure and produce a result.

The complexity, and hence expressiveness, of our interpreters will increase from here on out.


## Bindings

Add an environment. Discuss scoping.

## Project

A language for interactive fiction? We might have enough material already that we don't need a big project.

