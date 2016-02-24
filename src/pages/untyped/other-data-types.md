## Other Data Types

We'll now extend our interpreter to include some simple string operations: appending strings and converting them to upper and lower case. This will require two changes to our interpreter

- we have to use a more complex representation of values, which can now be numbers or strings; and
- we have to allow the possibility of evaluation resulting in an error.

We'll first create an algebraic data type to represent the two types of values our programs can evaluate to. The `name` instance variable is something we'll use later.

```scala
sealed trait Value {
  val name: String =
    this match {
      case Number(_) => "Number"
      case Chars(_) => "Chars"
    }
}
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

We don't currently have any way of working out the type of value an expression evaluates to without actually evaluating it. This means our expression case classes representing operations have to allow any expression as arguments. We can't statically rule out, for example, expressions that evaluate to a `Chars` being used as arguments to `Plus`. Doing so requires a type system, which we'll see how to introduce in a later section.

These additions to our language make our interpreter more complex in two ways: we have check that values have the correct tag for the operation they are applied to, and we have to change the result type of the interpreter to allow for the possibility of error. We'll represent the possibility of error during evaluation using the `Xor` type, encoding errors as `Strings`.

```scala
type Result[A] = Xor[String,A]
```

Our interpreter now has type `Expression => Result[Value]`. Checking tags requires a bit more infrastructure, which we'll now build.

We'll use two main abstractions, refinements and injections. A `Refinement` is a function `Value => Result[A]` that converts, if possible, a `Value` to a specific Scala type. For example, we can refine `Chars` to `String`. If the conversion cannot be performed we return an error message instead. We go the other way with an `Injection`, which is just a function `A => Value`. We'll add in some implicit machinery to make this all easier to use.

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

Injections are even simpler as we don't have to worry about errors. We can consider then to be a more controlled use of an implicit conversion.

```scala
object Injection {
  type Injection[I,O] = I => O

  implicit class InjectionOps[I](in: I) {
    def inject[O](implicit i: Injection[I,O]): O =
      i(in)
  }
}
```

With this infrastructure in place we can now define some refinements and injections. 

```scala
object Errors {
  def wrongTag[A](received: Value, expected: String): Result[A] =
    s"""|Expected value with tag $expected
        |but received value $received with tag ${received.name}""".stripMargin.left
}

object Refinements {
  import Refinement._
  type ValueRefinement[A] = Refinement[Value,A]

  def make[A](name: String)(f: PartialFunction[Value,A]): ValueRefinement[A] = {
    val lifted = f.lift
    (v: Value) => lifted(v).fold(Errors.wrongTag[A](v, name))(a => a.right)
  }

  implicit val doubleRefine: ValueRefinement[Double] =
    make[Double]("Number"){ case Number(d) => d }
  implicit val stringRefine: ValueRefinement[String] =
    make[String]("Chars"){ case Chars(s) => s }
}

object Injections {
  import Injection._
  type ValueInjection[A] = Injection[A,Value]

  implicit val injectDouble: ValueInjection[Double] = Number.apply _
  implicit val injectString: ValueInjection[String] = Chars.apply _
}
```

Finally we add some utilities to lift functions `A => B` and `(A,B) => C` to functions that operate on `Values`.

```scala
object Lift {
  import Expression._
  import Refinement._
  import Refinements._
  import Injection._
  import Injections._

  def apply[A: ValueRefinement, B: ValueInjection](f: A => B)(a: Value): Result[Value] =
    (a.refine[A] map (a => f(a).inject[B]))

  def apply[A: ValueRefinement, B: ValueRefinement, C: ValueInjection](f: (A, B) => C)(a: Value, b: Value): Result[Value] =
    (a.refine[A] |@| b.refine[B]) map ((a,b) => f(a,b).inject[C])
}
```
With this machinery in place we can now write our interpreter quite directly.

```scala
sealed trait Expression {
  import Expression._
  import Injections._
  import Refinements._

  def eval: Result[Value] =
    this match {
      case Plus(l, r)     =>
        (l.eval |@| r.eval).tupled flatMap {
          (Lift.apply[Double,Double,Double]{ _ + _ } _).tupled
        }
      case Minus(l, r)    =>
        (l.eval |@| r.eval).tupled flatMap {
          (Lift.apply[Double,Double,Double]{ _ - _ } _).tupled
        }
      case Multiply(l, r) =>
        (l.eval |@| r.eval).tupled flatMap {
          (Lift.apply[Double,Double,Double]{ _ * _ } _).tupled
        }
      case Divide(l, r)   =>
        (l.eval |@| r.eval).tupled flatMap {
          (Lift.apply[Double,Double,Double]{ _ / _ } _).tupled
        }

      case Append(l, r)   =>
        (l.eval |@| r.eval).tupled flatMap {
          (Lift.apply[String,String,String]{ _ ++ _ } _).tupled
        }

      case UpperCase(s)   =>
        s.eval flatMap { Lift.apply[String,String]{ _.toUpperCase } _ }
      case LowerCase(s)   =>
        s.eval flatMap { Lift.apply[String,String]{ _.toLowerCase } _ }

      case Literal(v)   =>
        v.right
    }
}
```

The complete code for this interpreter, including examples, is in the file [`ArithmeticAndStrings.scala`][link-arithmetic-and-strings].
