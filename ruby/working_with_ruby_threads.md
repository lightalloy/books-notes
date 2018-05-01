### The promise of multi-threading
Processes copy memory, while threads share memory.
Process spawning is slower than thread spawning.

Threads can you give more 'units' of concurrency for the available resources.

## Chapter 1. You're Always in a Thread
There's always at least one: the main thread.

```ruby
Thread.main
Thread.current == Thread.main # true
```
When the main thread exits, all other threads are immediately terminated and the Ruby process exits.

```ruby
Thread.new { Thread.current == Thread.main } .value # false
```

## Chapter 2. Threads of Execution
Threads are powerfull and let you shoot yourself in the foot.
### Shared address space
Share all of the same references in memory (ie. variables), as well as the AST (ie. the compiled source code).

### Many threads of execution
Multiple threads of execution operating on the same code at the same time.
It's no longer possible to step through these simultaneous threads of execution in any kind of predictable manner, a certain amount of randomness introduced in the way that threads are scheduled.

### Native threads
All of the Ruby implementations studied in this book ultimately map one Ruby thread to one native, operating system thread.

### Non-deterministic context switching
It refers to the work that's done by your operating system thread scheduler, you have no control on them.

The thread scheduler can 'pause' or 'unpause' a thread at any time.
But there are primitives you can use to say, "Hey, thread scheduler, I'm doing something important here, don't let anybody else cut in until I'm done."

#### Context switching in practice
The `Queue` class is a thread-safe data structure that ships with Ruby
A *race condition* involves two threads racing to perform an operation on some
shared state.

E.g. `||=` is not safe.
It's good practice to avoid lazy instantiation in the presence of multiple threads. The race may result in the underlying data becoming incorrect.

Some race conditions may only be exposed under heavy load.

An operation `@results ||= Queue.new` is not atomic. An atomic operation is one which cannot be interrupted before it's complete.

With proper protection in place a multi-step operation can be treated as a single-step.

### Why is this so hard?
Any time that you have two or more threads trying to modify the same thing at
the same time, you're going to have issues.
1) don't allow concurrent modification, or 2) protect concurrent modificatio

## Lifecycle of a Thread












p 17

### Thread#join
- wait to finish
```ruby
thread = Thread.new { sleep 3 }
thread.join
```