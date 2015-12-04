## Other Data Types

We'll now extend our interpreter to include some simple string operations. In doing so we'll see why we call this an untyped interpreter.

We'll first create an algebraic data type to represent values in our programs.

```scala
sealed trait Value
final case class Number(get: Double) extends Value
final case class Chars(get: String) extends Value
```

Now we'll extend the `Expression` algebraic data type to include some operations on `Chars`.

```scala
sealed trait Expression
final case class Plus(left: Expression, right: Expression) extends Expression
final case class Minus(left: Expression, right: Expression) extends Expression
final case class Multiply(left: Expression, right: Expression) extends Expression
final case class Divide(left: Expression, right: Expression) extends Expression
final case class Append(left: Expression, right: Expression) extends Expression
final case class UpperCase(string: Expression) extends Expression
final case class LowerCase(string: Expression) extends Expression
final case class Literal(get: Value) extends Expression
```

Our interpreter uses structural recursion as before, but now we have to check that values have the correct tag. This also changes the result of our interpreter---we have the possibility of error. We rerpesent this using the `Xor` type, encoding errors as `Strings`.

```scala
type Result[A] = Xor[String,A]
```

We start by building a bit of infrastructure to check tags.

```scala
def checkNumber(in: Value): Result[Double] =
  in match {
    case Number(v) => v.right
    case Chars(s)  => errors.wrongTag("Number", "Chars", s)
  }
def checkChars(in: Value): Result[String] =
  in match {
    case Chars(s)  => s.right
    case Number(v) => errors.wrongTag("Chars", "Number", v.toString)
  }

def binaryNumericOp(l: Expression, r: Expression)(op: (Double, Double) => Double): Result[Value] =
  ((l.eval flatMap checkNumber) |@| (r.eval flatMap checkNumber)) map { (x,y) => Number(op(x, y)) }
def binaryCharsOp(l: Expression, r: Expression)(op: (String, String) => String): Result[Value] =
  ((l.eval flatMap checkChars) |@| (r.eval flatMap checkChars)) map { (x,y) => Chars(op(x, y)) }

def unaryStringOp(v: Expression)(op: String => String): Result[Value] =
  v.eval flatMap checkChars map { x => Chars(op(x)) }

object errors {
  def wrongTag[A](expected: String, received: String, value: String): Result[A] =
    s"""|Expected value with tag $expected
        |but received value $value with tag $received""".stripMargin.left
}
```

Now our interpreter is just a structural recursion as before.

```scala
def eval: Result[Value] =
  this match {
    case Plus(l, r)     =>
      binaryNumericOp(l, r){ _ + _ }
    case Minus(l, r)    =>
      binaryNumericOp(l, r){ _ - _ }
    case Multiply(l, r) =>
      binaryNumericOp(l, r){ _ * _ }
    case Divide(l, r)   =>
      binaryNumericOp(l, r){ _ / _ }

    case Append(l, r)   =>
      binaryCharsOp(l, r){ _ ++ _ }

    case UpperCase(s)   =>
      unaryStringOp(s){ _.toUpperCase }
    case LowerCase(s)   =>
      unaryStringOp(s){ _.toLowerCase }

    case Literal(v)   =>
      v.right
  }
```

The complete code for this interpreter, including examples, is in the file [`ArithmeticAndStrings.scala`][link-arithmetic-and-strings].

We can see the structure of the interpreter is much the same as before, but now we need to check the tags on values. This has two implications:

- we have to build infrastructure to handle checks; and
- we can now create programs that result in a runtime error.

We know that we can elminate these errors via a type system. In the next chapter we'll see how we can use Scala's type system to check programs written in our embedded language. Before that, however, we going to see how to build more pieces of a "proper" programming language.
