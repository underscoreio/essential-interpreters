## Bindings

Our next step is to introduce *bindings* into our language. The term "binding" is one used in programming language theory to describe an association between a name and a value. The basic form of binding in Scala is a `val`.

```scala
val name = "Noel"
```

This Scala statement binds the name `name` to the value `"Noel"`. Whereever we use refer to `name` where this binding is in scope we can substitute in the value `"Noel"`. Other constructs also introduce bindings---method parameters, for example.

By the time you are reading this book I expect you have a strong mental model of the semantics of binding in Scala, but perhaps are not aware of the formal terms for describing its behaviour. Scala follows most functional languages in having lexical scoping rules and allowing functions to close over their environment.

**Describe lexical scoping, free and bound variables, and closures here.**

Implementing these correctly proves to be surprisingly subtle.

**Implementation here.**

Understanding the machinery for implementing binding is useful, but in many cases we just want the same behaviour as Scala. If this is the case we can use a trick, called *higher-order abstract syntax*, to reuse Scala's binding machinery for our own language. We turn to this now.
