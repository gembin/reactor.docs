---
title: "The Reactor Environment"
layout: article
---

## The Reactor Environment

Before you can do anything useful with Reactor, you need to create a
long-lived `Environment` instance which holds the configured Dispatchers
and properties that influence how Reactor components behave. Each
instance of an Environment will contain its own set of Dispatchers, so
be careful how many instances of these you create. If possible, only one
instance per JVM is recommended.

Create a new Environment using the default properties file configuration
reader, which reads the configuration from
`META-INF/reactor/default.properties`:

```java
Environment env = new Environment();
```

When you create Reactors, Streams, Promises, and other components that require a Dispatcher, pass this Environment instance to the Spec for that component. For example, to reference this Environment when creating Reactors:

```java
Reactor r = Reactors.reactor()
  .env(env)
  .get();
```

## Dispatchers

There are a default set of Dispatchers created and they should cover the
majority of use cases. But it might be the case that you don't need a
particular type of Dispatcher at all (say you're only using the
`RingBufferDispatcher`) so you want to prevent that type of Dispatcher
from being configured. You'd simply make a copy of the Reactor
[default.properties](https://github.com/reactor/reactor/blob/master/reactor-core/src/main/resources/META-INF/reactor/default.properties)
and set the `.type` of the Dispatcher you want to turn off to a blank.

By default, a ThreadPoolExecutorDispatcher is configured.

```
reactor.dispatchers.threadPoolExecutor.type = threadPoolExecutor
reactor.dispatchers.threadPoolExecutor.size = 0
# Backlog is how many Task objects to warm up internally
reactor.dispatchers.threadPoolExecutor.backlog = 1024
```

If you don't need this Dispatcher, then set the property
`reactor.dispatchers.threadPoolExecutor.type` to a blank:

```
reactor.dispatchers.threadPoolExecutor.type =
```

## Profiles

If you don't want to change the default properties you can activate a
"profile". To activate a profile named "testing", create a properties
file called `META-INF/reactor/testing.properties`. When you run your
Reactor application, activate the profile by referencing it in the
`reactor.profiles.active` System property:

```
$ java -Dreactor.profiles.active=testing -jar reactor-app-1.0.jar
```

If you don't want Reactor's default properties to be read at all, create a .properties file with the configuration you want and set the system property `reactor.profiles.default` to the name of the profile you want to load as the "default" profile. For example, to use the "production" profile as the default, create a properties file called `META-INF/reactor/production.properties`. When running your Reactor application, set the `reactor.profiles.default` System property to "production":

```
$ java -Dreactor.profiles.default=production -jar reactor-app-1.0.jar
```

## The Default Dispatcher

When creating components that need Dispatchers, unless you specify a particular Dispatcher to use, Reactor will give your component the "default" Dispatcher. In its default configuration, that would be the "ringBuffer" Dispatcher. To change that, set the System property `reactor.dispatchers.default` to the name of the Dispatcher to be considered the default:

```
# change to the eventLoop for a default Dispatcher
reactor.dispatchers.default = eventLoop
```
