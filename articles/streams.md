---
title: "Streams"
layout: article
---

## Streams

A
[Stream](http://reactor.github.io/docs/api/reactor/core/composable/Stream.html)
is a stateless event processor that offers a reactive alternative to
callback spaghetti programming. Values passing through a Stream can be
transformed, filtered, and consumed by calling the appropriate
method. Entire graphs of operations can be constructed from a Stream
since many methods transparently return new Streams which start new
flows of reactions.

## Deferred and Composables

Streams aren't created directly since a `Stream` is a consumer-side
concern. The producer side is called a
[Deferred](http://reactor.github.io/docs/api/reactor/core/composable/Deferred.html). The
`Deferred` is the object that ties the producer to the consumer and it's
this object that data is passed into. A `Deferred` is also a
[Consumer](http://reactor.github.io/docs/api/reactor/function/Consumer.html),
so to pass data into a `Stream`, call the
[accept(T)](http://reactor.github.io/docs/api/reactor/function/Consumer.html#accept(T))
method.

```java
Environment env = new Environment();

// Create a deferred for accepting stream values
Deferred<String, Stream<String>> deferred = Streams.<String>defer()
  .env(env)
  .dispatcher(RING_BUFFER)
  .get();

Stream<String> stream = deferred.compose();

// consume values
stream.consume(new Consumer<String>() {
  public void accept(String s) {
    // handle string when available
  }
});

// producer calls accept
deferred.accept("Hello World!");
```

Generally the two halves of the Deferred/Stream combination are not in
the same scope. If you're creating an API for your application, you
would likely create a `Deferred` inside a method call and retain a
reference to that Deferred so you can publish values to it
asynchronously. You would then return a `Stream<T>` to the caller so it
could interact with the values.

## Composition Methods

The real power of the Stream API lies in the composition functions for transforming and filtering data.

```java
Stream<String> filtered = stream
  .map(new Function<String, String>() {
    public String apply(String s) {
      // turn input String into output String
      return s.toLowerCase();
    }
  })
  .filter(new Predicate<String>() {
    public boolean test(String s) {
      // test String
      return s.startsWith("nsprefix:");
    }
  });
```

The `filtered` Stream now contains only lowercased values that start with "nsprefix:".

## Unbounded or Batched

Streams are by default unbounded, which means they have no beginning and
no end. Streams also have a method called `batch(int size)` which
creates a new `Stream` that works with values in batches. In batch mode,
`Stream` methods like `first()` and `last()` make it easy to work with
the beginning and ends of batches.

The `first()` and `last()` methods provide Streams whose values are only
the first of the Stream, in the case of an unbounded Stream, or the
first value of each batch, if the Stream is in batch mode. The Stream
returned from `last()` contains values from the end of a Stream (in
unbounded mode, that means whenenver `flush()` is called) or the last
values in each batch if the Stream is in batch mode.

Values can be collected into a `List<T>` by calling
`Stream.collect()`. This new Stream is populated by accumulated values
whenever `flush()` is called or the configured batch size is reached.

## Reduce

Streams can also be reduced. Provided with a `Supplier<T>` which returns
a new instance of the accumulator at the beginning of the Stream or the
beginning of each batch. A `Tuple2` is passed into the provided
`Function` containing the current accumulator value and the next data
element passing through the Stream. The value returned from the provided
`Function` will be sent along again as the accumulator value the next
time a data value passes through the Stream.
