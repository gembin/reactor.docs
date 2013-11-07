---
title: "Usage"
layout: article
---

## Creating an Environment

An `Environment` is aware of active "profiles" (the default profile is
named "default"). The default environment properties reader will look
for properties files in the classpath at location
`META-INF/reactor/default.properties` and, if the system property
`reactor.profiles.active` is set to a comma-separated list of profile
names, the properties reader will look for a
`META-INF/reactor/$PROFILE.properties` file for each profile referenced.

The `Environment` maintains references to the Dispatchers created from
the profile's properties file. In the default configuration, only one
`Environment` instance is needed per JVM to keep the number of threads
used by the Dispatchers relatively small.

```java
final Environment env = new Environment();
```

Pass a reference to this Enivronment when creating components that
require a Dispatcher.

```java
// Reactors are created using a ReactorSpec obtained via factory method
Reactor r = Reactors.reactor().env(env).get();
```

## Handling Events

To assign a handler for a specific `Event`, a `Selector` is required to
do the matching.

```java
reactor.on($("topic"), new Consumer<Event<Message>>() { ... });
// if you don't like the $, use the `object()` method
reactor.on(Selectors.object("topic"), new Consumer<Event<Message>>() { ... });
```

To notify the Reactor of an available Event, call the `notify()` method:

```java
Message msg = msgService.nextMessage();
reactor.notify("topic", Event.wrap(msg));
```

Several types of built-in Selectors are available to match by Object,
Class, RegEx, or URI template.

It's common in Reactor application to publish errors by the exception class. To handle errors this way, use the `ClassSelector`.

```java
// T() is a static helper function on the Selectors object to create a ClassSelector
reactor.on(T(IllegalArgumentException.class), new Consumer<Event<IllegalArgumentException>>() { ... });
```

Errors will be published using the exception class type as the key:

```java
try {
  // do something that might generate an exception
} catch (Throwable t) {
  reactor.notify(t.getClass(), Event.wrap(t));
}
```

Topic selection can be done with regular expressions.

```java
// R() is a static helper function on the Selectors object to create a RegexSelector
reactor.on(R("topic.([a-z0-9]+)"), new Consumer<Event<Message>>() { ... });
// ...or...
reactor.on(Selectors.regex("topic.([a-z0-9]+)"), new Consumer<Event<Message>>() { ... });
```

URI template selection can be done with URI templates.

```java
// U() is a static helper method on the Selectors object to create a UriTemplateSelector
reactor.on(U("/topic/{name}"), new Consumer<Event<Message>>() {
  public void accept(Event<Message> ev) {
    String name = ev.getHeaders().get("name");
  }
})
// ...or...
reactor.on(Selectors.uri("/topic/{name}"), new Consumer<Event<Message>>() { ... });
```

## Event Routing

Reactors send events to Consumers based on the `EventRouter` in use. By
default, no Consumers are filtered out of the selection, which means all
Consumers subscribed to a given Selector will be invoked. There are a
couple built-in filters provide round-robin and random filtering of
Consumers so that if multiple Consumers are subscribed to the same
Selector, they will not all be invoked (as is the case with the default
PassThroughFilter) but a single Consumer will be selected in a
round-robin Selector. Similarly, using a `RandomFilter` will randomly
select a Consumer.
