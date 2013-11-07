---
title: "Promises"
layout: article
---

## Promises

A
[Promise](http://reactor.github.io/docs/api/reactor/core/composable/Promise.html)
is a stateful event processor that accepts either a single value or an
error. Once either of these values has been accepted, the `Promise` is
considered complete.

When a `Promise` is populated with a value, the `onSuccess` Consumers
are invoked. When populated with an error, the `onError` Consumers are
invoked. When either a success or an error is set, the `onComplete`
Consumers are invoked.


## Usage

Promises aren't created directly. They're meant to handle future values
when they are available. Promises only provide methods for setting
Consumers--you can't publish a value to a Promise. Instead you use a
`Deferred`, which produces a `Promise` from its `compose()` method. The
Deferred has an `accept()` method on it which is used to populate the
Promise.

Promises also need to be aware of a `Dispatcher` to use when publishing
events. There are static helper methods on the
[Promises](http://reactor.github.io/docs/api/reactor/core/composable/spec/Promises.html)
class for creating them.

```java
Environment env = new Environment();

// Create a deferred for accepting future values
Deferred<String, Promise<String>> deferred = Promises.<String>defer()
  .env(env)
  .dispatcher(RING_BUFFER)
  .get();

// Set a Consumer for the value
deferred.compose().onSuccess(new Consumer<String>() {
  public void accept(String s) {
    // handle string when available
  }
});

// When a value is available, set it on the Deferred
deferred.accept("Hello World!");
```

Promises can also be created in a success or error state right away if the value is already known. This is useful when a Promise should be returned from a method but the value (or error) is immediately available.

```java
// Create a completed Promise
Promise<String> p = Promises.success("Hello World!").get();

// Consumers assigned on a complete Promise will be submitted immediately
p.onSuccess(new Consumer<String>() {
  public void accept(String s) {
    // handle string when available
  }
});

// Create a Promise in an error state
Promise<String> p = Promises.<String>error(new IllegalStateException()).get();
```

## Composition

A key feature of Promises is that they are composable. The
[map(Function<T,V> fn)](http://reactor.github.io/docs/api/reactor/core/composable/Promise.html#map(reactor.function.Function))
function can be used to assign a
[Function<T,V>](http://reactor.github.io/docs/api/reactor/function/Function.html)
that will transform the result value `T` into a `V` and populate a new
`Promise<V>` with the return value.

```java
Deferred<String, Promise<String>> deferred = Promises.<String>defer().get();

// Transform the String into a Float using map()
Promise<Float> p2 = p1.map(new Function<String, Float>() {
  public Float apply(String s) {
    return Float.parseFloat(s);
  }
});

p2.onSuccess(new Consumer<Float>() {
  public void accept(Float f) {
    // handle transformed value
  }
});

deferred.accept("12.2");
```

A Promise can also be filtered. A call to the
[filter(Predicate<T> p)](http://reactor.github.io/docs/api/reactor/core/composable/Promise.html#filter(reactor.function.Predicate))
method will return a new `Promise<T>` that will, depending on the
Predicate check, either be poplated with the result value or be in error
if the Predicate test fails.

```java
Deferred<Float, Promise<Float>> deferred = Promises.<Float>defer().get();

// Filter based on a Predicate
Promise<Float> p2 = p1.filter(new Predicate<Float>() {
  public boolean test(Float f) {
    return f > 100f;
  }
});

p2.then(
  // onSuccess
  new Consumer<Float>() {
    public void accept(Float f) {
      // handle value
    }
  },
  // onError
  new Consumer<Throwable>() {
    public void accept(Throwable t) {
      // predicate test failed
    }
  }
);

deferred.accept("12.2");
```

## Getting the value

Since Promises are designed to be stateful, they also provide methods
for getting the value of the Promise. Promises are
[Suppliers](http://reactor.github.io/docs/api/reactor/function/Supplier.html),
so a call to the
[get()](http://reactor.github.io/docs/api/reactor/core/composable/Promise.html#get())
method will return the current value of the Promise, whether it is
complete or not (Promises also provide methods for checking whether the
Promise is complete, a success, or in error).

Promises can block the calling thread waiting for a result. Call one of
the
[await()](http://reactor.github.io/docs/api/reactor/core/composable/Promise.html#await(long,java.util.concurrent.TimeUnit))
methods to block until a value is available or the Promise is notified
of an error.

Note that the zero-arg `await()` method does *not* block
indefinitely. It will only block for a period of milliseconds as defined
in the System property `reactor.await.defaultTimeout`. This value is 30
seconds by default. It can be changed by setting this property in the
`META-INF/reactor/default.properties` (or any other active
`profileName.properties`).
