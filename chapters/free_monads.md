# Free Monads

In the previous section we saw free monoids. **Free monads** follow the same general idea, representing expressions (in this case monadic expressions) using data structures, and then separately defining an interpreter for these structures.

Free monoids are simple enough we could go directly to the free monoid structure without much in the way of examples. Free monads are a bit more complex (but not much more!) so we're going to first build an example and then look into the implementation.

## Free Options

We are all familiar with `Option`. We know we can use it to model
