## Interpreting Arithmetic Expressions

Let's start our exploration by writing an interpreter for arithmetic expressions. We start with an algebraic data type to represent our expressions.

```scala
sealed trait Expression
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Literal(get: Double) extends Expression
```

A `Literal` represents a literal value---that is, an expression that evaluates to "itself." In our case this is a number. The other case classes, `Plus`, `Minus`, and so on, represent arithmetic operations.

We'll implement our interpreter as a method `eval`. `Expression` is an algebraic data type so `eval` is a structural recursion.

```scala
sealed trait Expression {
  def eval: Double =
    this match {
      case Plus(l, r)     => l.eval + r.eval
      case Minus(l, r)    => l.eval - r.eval
      case Multiply(l, r) => l.eval * r.eval
      case Divide(l, r)   => l.eval / r.eval
      case Literal(v)     => v
    }
}
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Literal(get: Double) extends Expression
```

We can write some simple examples to show expressions evaluate to the expected result.

```scala
Plus(Literal(1), Minus(Literal(2), Literal(3))).eval
// res: Double = 0.0

Plus(Literal(1), Minus(Literal(3), Literal(2))).eval
// res: Double = 2.0
```

We can make expressions easier to read by adding some methods to `Expression`.

```scala
sealed trait Expression {
  def eval: Double =
    this match {
      case Plus(l, r)     => l.eval + r.eval
      case Minus(l, r)    => l.eval - r.eval
      case Multiply(l, r) => l.eval * r.eval
      case Divide(l, r)   => l.eval / r.eval
      case Literal(v)     => v
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
  // Smart constructor that creates Literal with type Expression
  def literal(in: Double): Expression =
    Literal(in)
}
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Literal(get: Double) extends Expression
```

With this we can write more readable expressions, but this will not be a focus of the book.

```scala
import Expression._
(literal(1) + literal(3) - literal(2)).eval
// res: Double = 2.0
```

We've now seen a very simple first interpreter. Our interpreters will always have the same structure:

- we'll have a data structure, sometimes called an *abstract syntax tree* or *intermediate representation*, describing the program; and
- our interpreter will consume this data structure and produce a result.

This structure lets us make firm the distinction between *expressions* and *values*, terms we have been using informally so far. An expression is program text. It is something we can store in a text file, print out and stick on our wall, or collect in a book. A value is something that exists only in the computer's memory. Interpretation or evaluation is the process of transforming an expression to a value.

Concretely, our interpreters will be some variation on `Expression => Value`. In this case the type is `Expression => Double` as values are exactly Scala's `Double`. 
