`Reactor` is a foundation for asynchronous applications on the JVM. It
provides abstractions for Java, Groovy and other JVM languages to make
building event and data-driven applications easier. It’s also really
fast. On modest hardware, it's possible to process around 15,000,000
events per second with the fastest non-blocking `Dispatcher`. Other
dispatchers are available to provide the developer with a range of
choices from thread-pool style, long-running task execution to
non-blocking, high-volume task dispatching.


## Build Instructions

`Reactor` uses a Gradle-based build system. Building the code yourself
should be a straightforward case of:


```
git clone git@github.com:reactor/reactor.git
cd reactor
./gradlew test
```

This should cause the submodules to be compiled and the tests to be
run. To install these artifacts to your local Maven repo, use the handly
Gradle Maven plugin:

```
./gradlew install
```

## Installation

Maven artifacts are available in the SpringSource
[snapshot](http://repo.springsource.org/libs-snapshot/org/projectreactor)
and
[milestone](http://repo.springsource.org/libs-milestone/org/projectreactor)
repositories. To use Reactor in your own project, add these repositories
to your build file's repository definitions. For a Gradle project, this
would be:

```groovy
ext {
  reactorVersion = '1.0.0.RC1'
}

repositories {
  maven { url 'http://repo.springsource.org/libs-milestone' }
}

dependencies {
  // Reactor Core
  compile "org.projectreactor:reactor-core:$reactorVersion"
  // Reactor Groovy
  compile "org.projectreactor:reactor-groovy:$reactorVersion"
  // Reactor Spring
  compile "org.projectreactor:reactor-spring:$reactorVersion"
}

## Overview

`Reactor` is fundamentally event-driven and
[reactive](http://www.reactivemanifesto.org/). It provides abstractions
to facilitate publishing and consuming events on Reactors. A `Reactor`
can be backed by a number of different `Dispatcher` implementations so
that tasks assigned to respond to certain events can be configured to be
run on a single thread as in an Actor or purely Reactor pattern, on a
thread chosen from a thread pool, or queued in an
(https://github.com/LMAX-Exchange/disruptor)[LMAX Disruptor RingBuffer]. Which
`Dispatcher` implementation needed is dependent on the kind of work
being done: blocking IO operations should be performed in a pooled
thread and fast, non-blocking computations should probably be executed
on a `RingBuffer`.
