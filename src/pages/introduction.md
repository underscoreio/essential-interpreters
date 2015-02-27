# Introduction

This short book explores the construction of interpreters, looking at three techinques:

- basic untyped interpreters over algebraic data types;
- monadic interpreters; and
- interpreters using the free monad.

Why are interpreters interesting? Because we can view the vast majority of programs written in a functional style as transforming data from one form to another, and this is what interpreters do. The patterns for writing interpreters are as useful for [composing graphics](https://github.com/underscoreio/doodle) as they are for [composing web services](https://www.youtube.com/watch?v=VVpmMfT8aYw).
