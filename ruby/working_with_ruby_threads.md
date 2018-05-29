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
`require 'thread'` doesn't load the `Thread` constant. Thread is loaded
by default, but requiring 'thread' brings in some utility classes like `Queue`.

### Thread.new
```ruby
Thread.new { ... }
Thread.fork { ... }
Thread.start(11, 2 ) { |x, y| x + y }
```
- Executes the block: the thread will yield to that block. The end of the block will be reached or an exception will be raised.
- Returns an instance of Thread.
`Thread.new` returns a Thread instance representing the subthread that was just spawned.

### Thread#join
You can use `#join` to wait for it to finish.
```ruby
thread = Thread.new { puts 'I am in thread'; sleep 3 }
thread.join
puts "You'll have to wait 3 seconds to see this"
```
Calling #join on the spawned thread will join the current thread of execution
with the spawned one. With join, the current thread will sleep until the spawned thread exits.

#### Thread#join and exceptions
When one thread raises an unhandled exception, it terminates the thread where the exception was raised, but doesn't affect other threads.

A thread that crashes from an unhandled exception won't be noticed until another thread attempts to join it.
`join` => re-raise exception in the current thread

### Thread#value
first joins with the thread, and then returns the last expression from the block of code the thread executed.

### Thread#status
Possible values:
- `run`
- `sleep` - blocked waiting for a mutex, or waiting on IO
- `false` - finished or killed
- `nil` - raised an unhandled exception
- `aborting` - running, yet dying

### Thread.stop
puts the current thread to sleep and tells the thread scheduler to schedule some other thread. It will remain in this sleeping state until its alternate, `Thread#wakeup` is invoked.

### Thread.pass
Just asks the thread scheduler to schedule some other thread, without putting the thread to sleep. Can't guarantee that the scheduler will take the hint.

### Avoid Thread#raise
This method should not be used -- doesn't respect `ensure` blocks. Allows a caller external to the thread to raise an exception inside the thread.

### Avoid Thread#kill
Same reason as above.

## Chapter 4. Concurrent != Parallel
1) Do multiple threads run your code concurrently? Yes.
2) Do multiple threads run your code in parallel? Maybe.

Concurrent -- working on 2 things, possibly switching between them.
Parralel -- working on 2 things at the same time.

More resources are required in order to work in parallel.

### You can't guarantee anything will be parallel
All you can do is to organize your code to be concurrent, making it parallel is not in your hands.

You should assume that your concurrent code will be running in parallel because it typically will, but you can't guarantee it.

## Chapter 5. The GIL and MRI
### The global lock
GIL - Global Interpreter Lock

There is one, and only one, GIL per instance of MRI (or per MRI process).
If one of those MRI processes spawns multiple threads, that group of threads will share the GIL for that process.

Ruby code will never run in parallel on MRI.

### An inside look at MRI
Remember that each MRI thread is backed by a native thread, and from the kernel's point of view, they're all executing in parallel.
The GIL is implemented as a mutex. The operating system will guarantee that one, and only one, thread can hold the mutex at any time.

### The special case: blocking IO
What happens when a thread executes some Ruby code that blocks on IO?
```ruby
require 'open-uri'
3 .times.map do
  Thread.new do
    open('http://zombo.com')
  end
end.each(&:value)
```
`open` make take a long time depending on the status of the resource.
When a thread is blocked waiting for IO, it won't be executing any Ruby code.
So when it's blocked on IO, the GIL is released.

Under the hood, each thread is using a ppoll(2) system call to be notified when their connection attempt succeeds or fails. After recieving the answer, the ruby code will need to be executed, so the process starts over again.

### Why?
1) To protect MRI internals from race conditions
2) To facilitate the C extension API; MRI can function safely, even in the presence of C extensions that may not be thread-safe
3) To reduce the likelihood of race conditions in your Ruby code

### Misconceptions
#### Myth: the GIL guarantees your code will be thread-safe.
The GIL reduces the likelihood of a race condition, but can't prevent it.

#### Myth: the GIL prevents concurrency.
The GIL prevents parallel execution of Ruby code, but it doesn't prevent concurrency.
You can use multiple threads to parallelize code that is IO-bound.

## Chapter 6. Real Parallel Threading with JRuby and Rubinius
JRuby and Rubinius don't have a GIL - they allow real parallel execution of the code.

JRuby and Rubinius don't have a GIL.
1) JRuby and Rubinius do protect their internals from race conditions with many fine-grained locks.
2) JRruby doesn't support C extension api. For gems that need to run natively, they support Java-based extensions.
Rubinius does support the MRI C extension API and they help gem authors fix thread-safety issues.
3) They don't reduce the likelihood of race conditions in your Ruby code

## Chapter 7. How Many Threads Are Too Many?
It depends. The only way to be sure is to measure and compare.
But there are some heuristics you can use to get started.

### ALL the threads
OSX has a hard limit on the number of threads that one process can spawn.
On a Linux machine it's possible to spawn a lot of threads, but you probably don't want to.

#### Context Switching
Spawning a thread is cheap, but there is overhead for the thread scheduler to manage those threads.
4 cores => 4 threads can be executing instruction at any given time.
Sometimes it makes sense to spawn more threads than the number of CPU cores.

### IO-bound
IO-bound code -- bound by the IO capabilities of your machine (speed of external requests or read/write to disc)
It does make sense to spawn more threads than CPU cores if your code spends time waiting for a response from some IO device.

There will be a sweet spot between utilizing available resources and context switching overhead. Finding the sweet spot is really important.

### CPU-bound
CPU-bound code is inherently bound by the rate at which the CPU can execute
instructions.
E.g. math calculations.
4 cores => code may run in parallel across 4 threads, utilize all the power, no overhead.
more threads => stiall utilizes all the power, but with the overhead.
MRI - performance isn't impacted with the introduction
of more threads because of GIL.

### So... how many should you use?
The situation won't be so clear.
The only way to a surefire answer is to measure.

## Chapter 8. Thread safety
### What's really at stake?
Thread-safe code => it can run in a multi-threaded context and your underlying data will be safe.
Not thread-safe => the worst that can happen is incorrect data.
'check-then-set' race condition:
Multi-step operation.
E.g. checking the order status, then collecting payment and setting the status.
It's likely that the operation will be interrupted before it's finished.
That leads to collecting payments twice.

### The computer is oblivious
The computer is unaware of thread-safety issues.
So there is no alarm for such errors. It's hard to find those them, sometimes they only appear in heavy load.

### Is anything thread-safe by default?
`+=`, `||=` are single operations, but they are not atomic
`Array`, `Hash` are not thread-safe by default.

Any concurrent modifications to the same object are not thread-safe.

## Chapter 9. Protecting Data with Mutexes
### Mutual exclusion
Mutex -- mutual exclusion.
If you wrap some section of your code with a mutex, you guarantee that no two threads can enter that section at the same time.
```ruby
mutex = Mutex.new
counter = 0
10_000.times do
  Thread.new do
    mutex.lock
    counter += 1
    mutex.unlock
  end
end
```
The first thread that hits this bit of code will lock the mutex and become it's owner.
Until the owning thread unlocks the mutex, no other thread can lock it.
More commonly used:
```ruby
mutex.synchronize do
  shared_array << nil
end
```
### The contract
The mutex is shared among all the threads that share the same Mutex instance.
The block of code inside of a Mutex#synchronize call is often called a critical section.

### Making key operations atomic
check-then-set race condition => mutex.synchronize block should include both the 'check' AND the 'set'

### Mutexes and memory visibility
Mutexes carry an implicit memory barrier. So, if one thread holds a mutex to write a value, other threads can lock the same mutex to read it and they will see the correct, most recent value.

### Mutex performance
Mutexes inhibit parallelism.
Mutexes provides safety where it's needed, but at the cost of performance.
Restrict the critical section to be as small as possible, while still preserving the safety of your data.

### The dreaded deadlockrf
One thread is blocked waiting for a resource from another thread (like blocking on a mutex), while this other thread is itself blocked waiting for a resource. There's a deadlock if a situation arises where neither thread can move forward.
Example: 2 threads, 2 mutexes. Each thread acquires one mutex, then attempts to acquire the other.

Solutions:
`Mutex#try_lock`
`try_lock` will not wait if the mutex isn't available.
It will return false if the mutex is not available and true if it does.
In our example when `try_lock` returns `false`, both threads should release their mutexes and return to the starting state.

With this approach, it's fine to use a blocking lock to get the first mutex, but try_lock should be used when acquiring the second mutex.

`livelock` -- threads are in some loop with each other with none progressing.

It's possible that Thread A acquires Mutex A, Thread B acquires Mutex B, then both try_lock the other mutex, both fail, both unlock their first mutex, then both reacquire their first mutex, etc., etc.

A better solution is to define a mutex hierarchy.
Any time that two threads both need to acquire multiple mutexes, make sure they do it in the same order.

## Chapter 10 Signaling Threads with Condition Variables
```ruby
condvar = ConditionVariable.new
mutex = Mutex.new
Thread.new do
  10.times do
    ...
    mutex.synchronize do
      results << smth
      condvar.signal
    end
  end
end
until comics_received >= 10
  mutex.synchronize do
    while results.empty?
      condvar.wait(mutex)
    end
  end
  ...
  comics_received += 1
end
```
Condition variables are used for putting threads to sleep and waking them only once a certain condition is met.

A `ConditionVariable` provides more efficient solution to the consumer-producer problem -- instead of checking many times for the `results.count` inside a mutex, we just wait for the signal from `condvar`

`ConditionVariable#wait` - wait for the specified mutex.
`ConditionVariable#signal` will wake up exactly one thread that's waiting on this ConditionVariable.
`ConditionVariable#broadcast` will wake up all threads currently waiting on
this ConditionVariable.

## Chapter 11. Thread-safe Data Structures
### Implementing a thread-safe, blocking queue
```ruby
q = BlockingQueue
BlockingQueue.new
q.push 'thing'
q.pop #=> 'thing'
```
Requirements: internally thread-safe + the `pop` operation, when called on an empty queue, should block until something is pushed onto the queue.

Basic example with mutex and `ConditionalVariable`:
Condvar rectifies returning nil from empty queue.
```ruby
class BlockingQueue
  def initialize
    @storage = Array.new
    @mutex = Mutex.new
    @condvar = ConditionVariable.new
  end

  def push(item)
    @mutex.synchronize do
      @storage.push(item)
      @condvar.signal
    end
  end

  def pop
    @mutex.synchronize do
      while @storage.empty?
        @condvar.wait(@mutex)
      end

      @storage.shift
    end
  end
end
```
### Queue, from the standard lib
Ruby's standard library ships `Queue` class. This is the only thread-safe
data structure that ships with Ruby. It loads when you do `require 'thread'`
`Queue` has a few more methods than our `BlockingQueue` , but its behaviour regarding push and pop is exactly the same.

Typicall usage: distributing workloads to multiple threads, where one thread is pushing to the queue, and multiple threads are popping. The popping threads are put to sleep until there's some work for them to do.

### Array and Hash
The core `Array` and `Hash` classes are not thread-safe by default, nor should they be.
The JRuby Array and Hash are also not thread-safe.
`thread_safe` gem implements `ThreadSafe::Array` and `ThreadSafe::Hash` which are thread-safe.

### Immutable data structures
Immutable data structures are inherently thread-safe.

## Chapter 12. Writing Thread-safe Code
Idiomatic Ruby code is most often thread-safe Ruby code.

### Avoid mutating globals
#### Even globals can provide thread-safe guarantees
If you need global variables, ensure that data consistency is
preserved (e.g. with mutexes)

#### Anything where there is only one shared instance is a global
- Constants
- the AST
- Class variables/methods

E.g. modifying class variable (`@@clouds`) is not safe.

Modifying the AST at runtime is almost always a bad idea:
changes to the AST shouldn't happen at runtime.

E.g. `kaminari` issue with defining a method dynamically, then calling `alias_method` with that method, then removing it.

### Create more objects, rather than sharing one
Sometimes you just need that global object.
The most common example of this is a network connection.
Solution: thread-locals, connection pools.

### Thread locals
A thread-local lets you define a variable that is global to the scope of the current thread.
```ruby
Thread.current[:redis] = Redis.new
```
So, if your program is running N threads, it will open N connections to Redis.
This N:N connection mapping is fine for small numbers of threads, but gets out of hand when the number of threads starts to increase. For connections, a pool is often a better abstraction.

### Resource pools
A pool object will open a number of connections, or in the more general sense, may
allocate a number of any kind of resource that needs to be shared among threads.
The pool is responsible for keeping track of which connections are
checked out and which are available, preserving thread safety.
When a thread wants to make use of a connection, it asks the pool to check out a connection.
When the thread is done, it checks the connection back in to the pool.
gem `connection_pool`

### Avoid lazy loading
### Prefer data structures over mutexes
Mutexes are harder to understand.
Rather than letting threads access shared objects and implementing the necessary synchronization, you pass shared objects through data structures.
This ensures that only one thread could mutate an object at any given time.

## Chapter 13. Thread-safety on Rails
It's rare that you would spawn threads inside your application logic, but if you're using a multi-threaded web server (like Puma 1 ) or a multi-threaded background job processor (like Sidekiq 2 ), then your application is running in a multi-threaded context.

### Gem dependencies
You'll have to do your research. Most of the gems are thread-safe buty you may need to check a bug-tracker.

### The request is the boundary
A multi-threaded web server will process each request with a separate thread.
Don't share objects between requests => no objects shared between threads.

## Chapter 14. Wrap Your Threads in an Abstraction
### Single level of abstraction
> "one level of abstraction per function." (high- and low-level code don't mix well)
threads, mutexes -- low-level code
Simple threads wrapper that will spawn a thread for each element of an array:
```ruby
module Enumerable
  def concurrent_each
    threads = []
    each do |element|
      threads << Thread.new {
        yield element
      }
    end
    threads.each(&:join)
  end
end
```
### Actor model
In some cases, it will make sense for you to write your own simple abstractions on top of your multi-threaded code, but often you'll get more benefit from a more mature abstraction.

An Actor is a long-lived 'entity' that communicates by sending messages.
long-running entity - long-running thread.

Each Actor has an 'address'. If you know the address of an Actor, you can send it a message.
These messages go to the Actor's mailbox, where they're processed asynchronously when the Actor gets around to it.

`Celluloid` - Actor Model + Ruby Object Model

Including the `Celluloid` module into any Ruby class will turn instances of that class into full-fledged `Celluloid` actors.
Each Celluloid actor is housed by a separate thread, one thread per actor.

'address' is a reference to the object.
Sending messages to an actor is calling methods on an object.

Regular method calls are sent to the actor's mailbox but, behind the scenes, Celluloid will block the caller until it receives the result, just like a regular method call.

Without waiting for the result -- `async`

Async + waiting for the result:
`future` -- Celluloid kicks off that method asynchronously and returns you a `Celluloid::Future` object. Calling `value` will block until the value has been computed.

Celluloid is a great solution to concurrency that puts the abstraction at the right level and wraps up a lot of best practices.

```ruby
fetcher = XKCDFetcher.new
fetcher.next
fetcher.async.next
fetcher.future.next
```
## Chapter 15. How Sidekiq Uses Celluloid
Sidekiq multi-threaded processing is implemented on top of Celluloid.

> As soon as you introduce the Thread constant, you've probably just
> introduced 5 new bugs into your code.

Sidekq is a system composed of Celluloid Actors.
Manager actor - holds the state of the whole system, mediates between collaborator  actors.
Collaborator actors:
the Fetcher - fetches the jobs from Redis
the Processors - perform the actual work of the jobs

### Into the source
In actor-based system the building blocks are not classes or methods, but messages.

### fetch (Manager)
Manager starts:
```ruby
def start
  @ready.each { dispatch }
end
```
`@ready` holds references to the `Processor` actors that are ready to process jobs.
`dispatch` does some housekeeping and does `@fetcher.async.fetch`

The Fetcher actor will process each fetch in turn, as new jobs get pushed into Redis.

The Fetcher first tries to retrieve a unit of work. If it doesn't retrieve any work, it calls itself again (waiting)

When it's finally able to retrieve work, it asynchronously sends the assign message to the Manager: `@mgr.async.assign(work)`

### `Manager#assign`
receives the unit of work.

First, the Manager grabs the next available Processor, then keeps tracks of its status appropriately in its internal data structures.
```ruby
processor = @ready.pop
@in_progress[processor_object_id] = work
@busy << processor
processor.async.process(work) # fire and forget, not waiting for a response.
```
The Manager actor lives in its own thread, it own these instance variables, and doesn't share them with other actors. So it can use regular ruby arrays.

The Processor will perform the work, then send a message back to the Manager when it's finished.

the `process` method is most entirely focused on actually performing the job.

### Wrap-up
The most obvious difference I see between the Sidekiq codebase and a more traditional Ruby codebase is the lack of dependence upon return values.
Return values are seldom used. Instead, actors expect a message to be sent back in return.

Sidekiq is a great example of how simple it can be to integrate multi-threaded
concurrency, via actors, with your business logic.

## Chapter 16. Puma's Thread Pool Implementation
Puma 1 is a concurrent web server for Rack apps that uses multiple threads for concurrency.

### A what now?
Puma's thread pool is responsible for spawning the worker threads and feeding them work.
The thread pool wraps up all of the multi-threaded concerns, and the rest of the server only cares about the app logic.
`Puma::ThreadPool` is actually a generic class.
Once initialized, the pool is responsible for receiving work and feeding it to an available worker thread.
auto-trimming feature - the number of active threads is kept minimum, but more threads can be spawned

### Code
`todo` is a shared array that holds work to be done.
Inside a mutex hold:
Endless loop.
`todo` is empty => no work to be done
If there's no need to shutdown => wait
Waiting: Increments a global counter saying that it's going to wait.
Next, wait on the shared condition variable, this releases the mutex and puts the current thread to sleep.
It won't be woken up again until there's some work to do. Since it released the shared mutex, another thread can go through the same routine.

Once enough work arrives, this thread will get woken up.
It will re-acquire the shared mutex and decrement `@waiting` counter.
Then pops the unit of work from `todo`. (`work = todo.pop if continue`)

Outside the mutex.synchronize block:
```ruby
break unless continue # time to shut down => break
block.call(work, *extra)
```
Not time to shut down => call `block` with the `work` object.
The block is passed in to the constructor and is the block of code that each worker thread will perform.

## Chapter 17. Closing
The safest path to concurrency:
1. Don't do it.
2. If you must do it, don't share data across threads.
3. If you must share data across threads, don't share mutable data.
4. If you must share mutable data across threads, synchronize access to that
data.

### Ruby concurrency doesn't suck

## Chapter 18 Appendix: Atomic Compare-and-set Operations

Compare-and-set (CAS)

Same as `@counter += 1`
```ruby
@counter = 0
# Get the current value of `@counter`.
current_value = @counter
# Increment the retrieved value by 1.
new_value = current_value + 1
# Assign the new value back to `@counter`.
@counter = new_value
```
This multi-step operation is that it's not atomic and may lead to a race condition.

Solutions: using mutex or using `Atomic`
```ruby
require 'atomic'
@counter = Atomic.new(0)
@counter.update { |current_value|
  current_value + 1
}
```
The whole operation is retried if the compare_and_set fails.

For some kinds of operations, locking (with mutexes) is very expensive, so `Atomic` is preferrable.

#### Benchmarks
The results differ wildly between implementations.
With JRuby, lockless and locking variants work on par.
With Rubinius the lockelss variant is much faster.

### CAS as a primitive
Some new technologies, such as Clojure, take full advantage of this and are showing what's really possible with multi-threaded concurrency
Ruby -- gem concurrent-ruby.

## Chapter 19. Appendix: Thread-safety and Immutability
The main problem of thread-safety is concurrent modification.
So, by definition, immutable objects are thread-safe.
Having only immutable objects sounds like a simple solution, but pure functional programming languages are usually hard to master.

### Immutable Ruby objects
Immutability is actually supported in core Ruby using the `Object#freeze` method.
The typical method signature for immutable objects is: methods that would typically mutate the object instead return a new version of the object with the mutation applied.
`gem 'hamster'` -- Efficient, Immutable, Thread-Safe Collection classes for Ruby

### Integrating immutability
The simplest use case is this: when you need to share objects between
threads, share immutable objects.

It's very easy to pass out immutable objects to share, but if you need to have multiple threads modifying an immutable object you still need some form of synchronization.

In this case, immutable data structures work great with CAS operations.
```ruby
@queue_wrapper = Atomic.new(Hamster::Queue.new)
...
@queue_wrapper.update { |queue|
  queue.enqueue(rand(100))
}
....
consumers << Thread.new do
  @queue_wrapper.update { |queue|
    number = queue.head
    queue.dequeue
  }
end
```

#### Questions

1-6)
- What is gil?
- How does it work (in system terms)?
- How does GIL deal with waiting for IO-operations?
7)
- When does it make sense to spawn more threads then the number of processor cores.
- How to decide how many threads you need?
- How to decide how many threads you need if your program involves heavy math calculations?
- What is check-then-set race condition?
8)
- What objects are thread-safe by default?
9)
- what is mutex?
- How to use mutex?
- Provide a classic deadlock example?
- What is a livelock?
- How do you avoid deadlocks/livelocks?
10)
- how do you use conditional variables?
- what if you just check the results instead of signaling?
11)
- What are the caveats of the multithreaded code?
- What global objects are there in your ruby program?
- How do you avoid problems with global objects?
16)
- How does sidekiq work?
- What do manager, fetched and processors do?
17.
- What are the rules for the thread-safe programming?
