# Introduction

This short book explores the construction of interpreters, looking at three techniques:

- basic untyped interpreters over algebraic data types;
- monadic interpreters; and
- interpreters using the free monad.

The increase of sophistication of our interpreters increases their flexibility. The final pattern, the free monad, allows us to freely (!) compose together languages and their interpreters.

Before we get into the code, lets stop to ask why are interpreters interesting? Here's [Don Stewart][link-don-stewart], Haskell hacker extraordinaire at Standard Chartered Bank, [on the design of large systems in functional programming][link-don-stewart-so]:

> In my experience, almost all designs fall into the 'compiler' or
> 'interpreter' pattern, using a model of the data and functions on
> that data. That is, problem domains are represented as algebraic
> structures (objects as ADTs with functions over them), and software
> architectures are about mapping from one algebra to another. This is
> the "category theory" design pattern(!)

Transforming data from one representation to another is what interpreters do. The patterns for writing interpreters are as useful for [composing graphics](https://github.com/underscoreio/doodle) as they are for [composing web services](https://www.youtube.com/watch?v=VVpmMfT8aYw). As Don says, they are in some sense the primordial functional patterns, so learn them, and learn them well.

## Interpreters vs Compilers

What is the difference between a compiler and an interpreter? For our purposes, an interpreter will perform effects, while a compiler will merely transform one type to another without performing effects. This is a very artificial distinction. Techniques like JIT compilers, abstract interpretation, and dependently typed languages blur the distinction between the two to the point that ultimately it is meaningless. We can all agree, however, that "transpiler" should be eradicated from our vocabulary.
