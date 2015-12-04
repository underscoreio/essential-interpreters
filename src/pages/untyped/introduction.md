# Untyped Interpreters

In this section we discuss classic interpreters for untyped languages. These interpreters are essentially a fold over an algebraic data types. If you have worked through [Essential Scala][link-essential-scala] these patterns should be familiar. 

## A Quick Aside on Types

What do we mean by untyped languages? The kind of languages were are going to implement here are sometimes called "dynamically typed" or even "unityped". It will help us to have a bit more precision in our use of these terms, so let's briefly discuss the concepts.

When programming languages researchers talk about types, they talk about properties of programs that can be worked out *before the program is run*. Types exist at compile-time, while values exist at run-time.

Why make this distinction? Because static and dynamic types lead to very different places. As a simple example, in a statically typed language values with different types can have the same representation. For instance, on the JVM an `Int` is represented by 32-bits of memory, as is a `Float`. Without knowing the type of the values we can't work out how to interpret an arbitrarily chosen 32-bit chunk of memory. Likewise, it's the knowledge of the types that allows us to reuse representations for different types, a procedure known as *type erasure*. In a dynamically typed language, the representation of `Ints` and `Floats` would have to differ, so at runtime the language implementation could decide how to treat each value. We'll quickly get into advanced uses of types that have no runtime representation (such as the `Id` monad). Given these differences it is best to be precise in our terminology and only talk about types as a static property of programs.

For more on types and related properties of soundness and safety, I recommend the [very accessible section in PAPL][link-safety-soundness-papl].
