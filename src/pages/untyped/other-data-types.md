## Other Data Types

We'll now extend our interpreter to include some simple string operations. This will require two changes to our interpreter

- we have to use a more complex representation of values, which can now be numbers or strings; and
- we have to allow the possibility of evaluation resulting in an error.

We'll first create an algebraic data type to represent the two types of values our programs can evaluate to.

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

We don't currently have any way of working out the type of value an expression evaluates to without actually evaluating it. This means our expression case classes representing operations have to accept all expressions as arguments. We can't statically rule out, for example, expressions that evaluate to a `Chars` as arguments to `Plus`. Doing so requires a type system, which we'll see how to introduce in a later section.

These additions to our language makes our interpreter more complex in two ways: we have check that values have the correct tag for the operation they are applied to, and we have to change the result type of the interpreter to allow for the possibility of error. We'll represent the possibility of error during evaluation using the `Xor` type, encoding errors as `Strings`.

```scala
type Result[A] = Xor[String,A]
```

Our interpreter now has type `Expression => Result[Value]`. Checking tags requires a bit more infrastructure, which we'll now build.

We'll use two main abstractions. A `Refinement` is a function `Value => Result[A]` that refines, if possible, a `Value` to a specific Scala type. For example, we can refine `Chars` to `String`. If the refinement fails we return an error message. We go the other way with an `Injection`, which is just a function `A => Value`. We'll add in some implicit machinery to make this all easier to use.

We'll start with refinements. As we'll want to reuse this code across interpreters we'll make it generic in the type of values (the type `I` in `Refinement` below). As a refinement is just a function all we add is an implicit class to make applying refinements easier.

```scala
import cats.data.Xor

object Refinement {
  type Result[A] = Xor[String,A]
  type Refinement[I,O] = I => Result[O]

  implicit class RefinementOps[I](in: I) {
    def refine[O](implicit r: Refinement[I,O]): Result[O] =
      r(in)
  }
}
```

Injections are even simpler as we don't have to worry about errors.

```scala
object Injection {
  type Injection[I,O] = I => O

  implicit class InjectionOps[I](in: I) {
    def inject[O](implicit i: Injection[I,O]): O =
      i(in)
  }
}
```

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
