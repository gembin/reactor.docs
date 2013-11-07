---
title: "Tuples"
layout: article
---

## Tuples

A `Consumer` only has one argument in it's `accept(T ob)` method. But
it's often convenient to pass multiple arguments to an Consumer. Tuples
are a convenience class provided by Reactor to help. Tuples are
type-safe and can be used to replace custom beans with strongly-typed
properties. They are similar to Scala's
[Product and Tuple](http://www.scala-lang.org/api/current/index.html#scala.Product)
classes.

For example, pass a `String` and a `Float` to an event Consumer, create
a `Tuple2<String, Float>`.

```java
reactor.on($("topic"), new Consumer<Tuple2<String, Float>>() {
  public void accept(Tuple2<String, Float> tuple) {
    String s = tuple.getT1();
    Float f = tuple.getT2();
  }
});

Tuple2<String, Float> tuple = Tuple.of("Hello World!", Float.valueOf(1.0));
reactor.notify("topic", Event.wrap(tuple));
```
