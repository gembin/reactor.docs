---
title: "Processor"
layout: article
---

## Processor (LMAX Disruptor) Support

The `Processor` is a thin abstraction around the
[LMAX Disruptor RingBuffer](https://github.com/LMAX-Exchange/disruptor)
designed for high performance. Unlike a `Reactor`, a `Processor` has no
awareness of dispatching or Selectors and doesn't directly support
dynamic Consumer assignment (that can be achieved by using the
`DelegatingConsumer`). The primary goal of the `Processor` is to expose
the power of the Disruptor RingBuffer as closely to the core as
possible, without introducing unnecessary overhead.

Creation of a `Processor` happens through a `ProcessorSpec`:

```java
/**
 * Frame object for use inside a RingBuffer.
 */
public class Frame {
  int type;
  Buffer message;
}

// Create a Processor for handling Frames
Processor<Frame> processor = new ProcessorSpec<Frame>()
  .dataSupplier(new Supplier<Frame>() {
    public Frame get() {
      return new Frame();
    }
  })
  .consume(new Consumer<Frame>() {
    public void accept(Frame frame) {

      // handle each updated Frame of data
      switch(frame.type) {
        case 0:
          // handle error frame
          break;
        case 1:
          // handle response frame
          break;
        default:
          break;
      }

    }
  })
  .get();

// Producer prepares operations and updates Frame data
for(Event<Message> evt : events) {
  Message msg = evt.getData();

  Operation<Frame> op = processor.prepare();
  Frame f = op.get();

  f.type = msg.getType();
  f.message = msg.getBuffer();

  // Consumer is invoked on Operation.commit(), which is RingBuffer.publish()
  op.commit();
}
```

## Batching

The `Processor` can also handle operations in batches of any size
(though we've found that smaller batches generally offer higher
throughput) using the `batch(int size, Consumer<Frame> mutator)`
method. Here's an example of creating a batch the same size as the
`List<Message> events` of incoming messages:

```java
processor.batch(events.size(), new Consumer<Frame>() {
  ListIterator<Message> msgs = events.listIterator();

  public void accept(Frame frame) {
    Message msg = msgs.next();

    f.type = msg.getType();
    f.message = msg.getBuffer();
  }
});
```

There's no need to call `commit()` for each operation in batch mode. The
`Consumer<T>` passed into the batch method is a mutator whose purpose is
to set the values of the data object before it is implicitly committed
(published).
