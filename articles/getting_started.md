---
title: "Getting Started"
layout: article
---

## Getting Started

Each `Reactor` you create needs a `Dispatcher` to execute tasks. By
default, with no configuration, you'll get a synchronous
Dispatcher. This works fine for testing but is probably not what you
want for a real application.

Since it's not desirable to create too many threads in an asynchronous
application and *something* has to keep track of those few Dispatchers
that are divvyed out to the components that need them, you need to
instaniate an `Environment` which will create those Dispatchers based on
either the default configuration (provided in a properties file in the
Reactor JAR file) or by providing your own configuration.

## The Environment

There's no magic to it. You simply "new" up an instance of `Environment`
and, when creating Reactors, Streams, and Promises, pass a reference to
this Environment into the Specs (essentially a "builder" helper
class). The Environment instance is where the `reactor.` system
properties live and it's also the place where the small number of
Dispatchers that are intended to be used by any component in your
application that needs one reside.

You can, of course, create Dispatchers directly in your code. There may
be times--like embedding in other threading models--where that's
desirable. But in general, you should refrain from directly
instantiating your own Dispatchers and instead use those configured to
be created by the `Environment`.

## Events, Selectors and Consumers

Three of the most foundational components in Reactor’s `reactor-core`
module are the `Selector`, the `Consumer`, and the `Event`. A `Consumer`
can be assigned to a `Reactor` by using a `Selector`, which is a simple
abstraction to provide flexibility when finding the `Consumer`s to
invoke for an `Event`. A range of default selectors are available. From
plain `String`s to regular expressions to Spring MVC-style URL
templates.

### Selector Matching

There are different kinds of Selectors for doing different kinds of
matching. The simplest form is just to match one object with
another. For example, a `Selector` created from a `String` "parse" will
match another `Selector` whose wrapped object is also a `String` "parse"
(in this case it's just like a `String.equals(String)`.

But a `Selector` can also match another `Selector` based on
`Class.isAssignableFrom(Class<?>)`, regular expressions, URL templates,
or the like. There are helper methods on the `Selectors` abstract class
to make creating these `Selector`s very easy in user code.

Here's is an example of wiring a `Consumer` to a `Selector` on a
`Reactor`:

```java
// This helper method is like jQuery’s.
// It just creates a Selector instance so you don’t have to "new" one up.
import static reactor.event.selector.Selectors.$;

Environment env = new Environment();

// This factory call creates a Reactor.
Reactor reactor = Reactors.reactor()
  .env(env) // our current Environment
  .dispatcher(Environment.EVENT_LOOP) // use one of the BlockingQueueDispatchers
  .get(); // get the object when finished configuring

// Register a consumer to handle events sent with a key that matches "parse"
reactor.on($("parse"), new Consumer<Event<String>>() {
  @Override
  public void accept(Event<String> ev) {
    System.out.println("Received event with data: " + ev.getData());
  }
});

// Send an event to this Reactor and trigger all actions that match the given key
reactor.notify("parse", Event.wrap("data"));
```

In Java 8, the event wiring would become extremely succinct:

```java
// Use a POJO as an event handler
class Service {
  public <T> void handleEvent(Event<T> ev) {
    // handle the event data
  }
}

@Inject
Service service;

// Use a method reference to create a Consumer<Event<T>>
reactor.on($("parse"), service::handleEvent);

// Notify consumers of the 'parse' topic that data is ready
// by passing a Supplier<Event<T>> in the form of a lambda
reactor.notify("parse", () -> {
  slurpNextEvent()
});
```

## Event Headers

Events have optional associated metadata in the `headers`
property. Events are meant to be stateless helpers that provide a
consumer with an argument value and related metadata. If you need to
communicate information to consumer components, like the IDs of other
Reactors that have an interest in the outcome of the work done on this
`Event`, then set that information in a header value.


```java
// Just use the default selector instead of a specific one
r.on(new Consumer<Event<String>>() {
    public void accept(Event<String> ev) {
      String otherData = ev.getHeaders().get("x-custom-header");
      // do something with this other data
    }
});

Event<String> ev = Event.wrap("Hello World!");
ev.getHeaders().set("x-custom-header", "ID_TO_ANOTHER_REACTOR");
r.notify(ev);
```

## Registrations

When assigning an `Consumer` to a `Reactor`, a `Registration` is
provided to the caller to manage that assignment. `Registration`s can be
cancelled, which removes them from the `Reactor` or, if you don't want
to remove an consumer entirely but just want to pause its execution for
a time, you can accept `pause()` and later `resume()` which will cause
the `Dispatcher` to skip over that `Consumer` when finding `Consumer`s
that match a given `Selector`.


```java
Registration reg = r.on($("test"), new Consumer<Event<?>>() { … });

// pause this consumer so it's not executed for a time
reg.pause();

// later decide to resume it
reg.resume();
```

## Routing

Reactor includes five different kinds of routing for assigned consumers:
broadcast, random, round-robin, first, and last. This means that, of the
given `Consumer`s assigned to the same `Selector`, the routing will
determine whether to execute all the consumers, one of them randomly
selected, one of them selected in a round robin fashion, the first
matching consumer, or the last matching consumer. The default is to use
broadcast routing.
