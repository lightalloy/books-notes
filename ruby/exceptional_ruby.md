# What is a failure?
An *exception* is the occurrence of an abnormal condition during the execution of a software element.
A *failure* is the inability of a software element to satisfy its purpose.
An *error* is the presence in the software of some element not satisfying its specification.

## Failure as a breach of contract
Bertrand Meyer "Design by Contract", Eiffel language.
A method’s contract states “given the following inputs, I promise to return
certain outputs and/or cause certain side-effects”.

The callers responsibility is to ensure that the method’s preconditions—the inputs it depends on—are met.
The method’s responsibility to ensure that its postconditions — those outputs and side-effects—are fulfilled.

It is the method’s responsibility to maintain the invariant of the object it is a member of. The invariant is the set of conditions that must be met for
the object to be in a consistent state. E.g. a TrafficLight class might have an invariant that only one lamp can be turned on at a time.

## Reasons for failure
- a case that was unanticipated by the programmer at the
time of writing.
- the program is running on may have run out of resources, e.g. memory
- some needed external element (e.g. webservice) fail

Whatever the reason for the failure, a robust program needs to have
a plan for handling exceptional conditions.

# The life-cycle of an exception
## It all starts with a raise (or a fail)
`raise` and `fail` are synonyms.
Opinion: using `fail` as a default, useing `raise` only when catching and re-raising an exception.

## Calling raise
With no arguments, it creates a RuntimeError with no message.
With just a string, it creates a RuntimeError with the given string as its message.

Given an explicit class (which must be a subclass of Exception 1 ), it raises
an object of the specified class.

Third argument: to specify a custom backtrace for the exception (useful for assertion methods).
```ruby
def assert(value)
  raise(RuntimeError, "Doh!", caller) unless value
end
assert(4 == 5)
```
## Overriding raise for fun and profit
`raise` and `fail` are ruby methods define in `Kernel`

Overriding to `exit` when smth is raised. It won't affect exceptions raised in
C extensions, e.g. with rb_raise().

### Hammertime
- an error console for Ruby, presents a menu of options when an exception is raised.

### raise internals
What raise does:
- Call #exception to get the exception object.
- Set the backtrace.
- Set the global exception variable
- Throw the exception object up the call stack.

### Step 1: Call #exception to get the exception.
```
def raise(error_class_or_obj, message, backtrace)
  error = error_class_or_obj.exception(message)
  # ...
end
```
`Exception` class defines #exception methods at both the class and
instance level.
Class level -- `Exception.new`
Instance level:
w/o args - return `self`
with args - returns a duplicate of itself. The new obj has a new message and maybe a new backtrace.
MEthod `exception` is similar to `to_proc` or `to_ary` -- it works like an implicit “exception coercion”.

### Implementing your own #exception methods
#### Step 2: #set_backtrace
Call #caller passing 0 for the “start” parameter.
`caller(0)` => get a stack trace which includes the current line

#### Step 3: Set the global exception variable
The $! global variable always contains the currently “active” exception (if
any). raise is responsible for setting this variable.

Separate threads get their own $! variable.

#### Step 4: Raise the exception up the call stack
raise begins to unwind the call stack.
Ruby works its way up the current call stack one method call or block evaluation at a time, looking for `ensure` or `rescue` statements.

If the top level of execution is reached and no `rescue` or `ensure` are met, Ruby prints out the exception message and a stack trace, and terminates the program with a failure status.

### ensure
ensure clauses will always be executed, whether an exception is raised or
not. They are useful for cleaning up resources.

Keep ensure clauses clean, the secondary exception raised there can leave the cleanup unperformed and may lead to difficult debugging.

#### ensure with explicit return
If you explicitly return from a method inside an ensure block, the method will
return as if no exception had been raised at all. The exception will be thrown away.
You should avoid using explicit returns inside ensure blocks.

### Coming to the rescue
> the rescue clause should be a short sequence of sim-
ple instructions designed to bring the object back to a stable
state and to either retry the operation or terminate with failure.

By default rescue only captures StandardError and derived classes.

Such exceptions as NoMemoryError, LoadError, SignalException, etc won't be catched.

The exceptions outside of the `StandardError` hierarchy typically represent conditions that can’t reasonably be handled by a generic catch-all rescue clause, so they are not catched by default.

You may supplu a list of classes:
```
rescue SomeError, SomeOtherError => error
# ...
end
```
#### Dynamic rescue clauses
```ruby
def ignore_exceptions(*exceptions)
  yield
rescue *exceptions => e
  puts "IGNORED: ’#{e}’"
end
puts "Doing risky operation"
ignore_exceptions(IOError, SystemCallError) do
  open("NONEXISTENT_FILE")
end
puts "Carrying on..."
```
rescue can use `===` for matching, but with some limitations.
```
starts_with_a = Object.new
def starts_with_a.===(e)
  /^A/ =~ e.name
end

rescue starts_with_a => error
```
The arguments to rescue must all be classes or modules. But so long as we satisfy that requirement, we can define the match conditions to be anything we want.

#### rescue as a statement modifier
```ruby
# return value of a statement or bil
f = open("nonesuch.txt") rescue nil
```

### If at first you don’t succeed, retry, retry again
```ruby
tries = 0
begin
  tries += 1
  puts "Trying #{tries}..."
  raise "Didn’t work"
rescue
  retry if tries < 3
  puts "I give up"
end
```
Be very careful that your “giving up” criterion will be met eventually.

### raise during exception handling
In C++ this will be concidered a bug.
In ruby we can always raise a new Exception.

One possibility is that we raise an entirely new Exception (the old will be thrown away).

Never allow the original exception to be thrown away. Instead, use Nested Exceptions.

#### Nested Exceptions
- exceptions which have a slot for a reference to the originating exception.
```ruby
class MyError < StandardError
  attr_reader :original
  def initialize(msg, original=$!)
    super(msg)
    @original = original;
  end
end
begin
  begin
    raise "Error A"
  rescue => error
    raise MyError, "Error B"
  end
  rescue => error
    puts "Current failure: #{error.inspect}"
    puts "Original failure: #{error.original.inspect}"
end
```
The default argument to original be the global error variable ($!), the object will “magically” discover the originating exception on initialization.

#### More ways to re-raise
Re-raising with custom message, custom backtrace.

#### Disallowing double-raise
Overriding `raise` to disallow.

#### else
else after a rescue clause is the opposite of rescue;
It's only hit when no exception is raised.

Why would we ever use else instead of simply putting the else-code after
the code which might raise an exception?

Exceptions raised in an else block are not captured by the preceding rescue clauses. Instead they are propagated up to the next higher scope of execution.

The order of processing.
```ruby
begin
  puts "in body"
rescue
  puts "in rescue"
else
  puts "in else"
ensure
  puts "in ensure"
end
puts "after end"
# in body
# in else
# in ensure
# after end
```

#### Uncaught exceptions
Ruby will handle the un-rescued exception by printing a stack trace and terminating the program.

Before the program ends, it'll execute git hooks:
```ruby
trap("EXIT") { puts "in trap" }
at_exit { puts "in at_exit" }
END { puts "in END" }

raise "Not handled"
```

They all have access to global variables like $!.

Useful for logging all fatal exception-induced crashes.

#### Exceptions in threads
work a little differently. By default, the thread terminates and the exception is held until another thread joins it, then the exception is re-raised in the joining thread.

#### Are Ruby exceptions slow?
Raising an exception is an expensive operation in any language, but there is no penalty to have excetions-using code.

We should reserve exceptions for genuinely exceptional circumstances.

## Responding to failures
### Failure flags and benign values
If the failure isn’t a major one, it may be sufficient to return a failure flag.
When the system’s success doesn’t depend on the outcome of the method in question, using a benign value may be the right choice.
Also writing tests will be easier using this values.
```ruby
begin
  response = HTTP.get_response(url)
  JSON.parse(response.body)
rescue Net::HTTPError
  {"stock_quote" => "<Unavailable>"}
end
```

### Reporting failure to the console
```
warn "Uh oh, something bad happened"
$stderr.puts 'Smth!'
```
`warn` also uses `$stderr`

#### Warnings as errors
```ruby
if $DEBUG
    module Kernel
      def warn(message)
        raise message
      end
    end
    warn "Uh oh"
end
```
This way warn will only raise an exception if you run Ruby with the -d flag.

### Remote failure reporting
- A central log server
- An email to the developers
- A post to a third-party exception-reporting service

#### A true story of cascading failures
A simple failure reporter, which used our team’s Gmail account to email the developers with the details of the failure.

Update which caused lots of failures => Google throttling the account => SMTP errors => the workers started crashing hard => unrelated systems which used Gmail acc started failing.

This is a classic example of a failure cascade.

#### Bulkheads
The bulkhead enforces a principle of damage containment in a sheep.

```
begin
  SomeExternalService.some_request
  rescue Exception => error
  logger.error "Exception intercepted calling SomeExternalService"
  logger.error error.message
  logger.error error.backtrace.join("\n")
end
```
• External services
• External processes

#### The Circuit Breaker pattern
- a way to mitigate cascade failures.
When the breaker is Closed, the subsystem is allowed to operate normally.
A counter tracks the number of failures that have occurred.
When the number exceeds a threshold, the breaker trips, and enters the open
state. In this state, the subsystem is not permitted to operate.
After either a timeout elapses or (depending on the implementation) a hu-
man intervenes and resets the breaker, it enters the half-open state.
In this state the system can run, but a single failure will send it back into the “open” state.

The Circuit Breaker pattern eliminates a common cause of failure cascades.

#### Ending the program
When you call `exit`, you're actually raising an exception (`SystemExit`).
`exit 1` == `raise SystemExit.new(1)`

If you want to print a failure message and then raise SystemExit with
a failure status code: `abort "Uh oh"`

*really* want to end the program quickly (no cleanup, exit hooks):
```
exit! false
```

## Alternatives to exceptions
Fail fast is often the best policy, but not always.
Sometimes you need a way to proceed through the steps of the provisioning
process and then get a report at the end telling you what parts succeeded
and what parts failed.

### Sideband data
### Multiple return values
in the form of array splatting.

The downside of this approach is that anyone calling the method must know
that it returns multiple values.

A variation on multiple return values is to return a simple structure with
attributes for the result value and for the status:
```
def foo
  # ...
  OpenStruct.new(:result => 42, :status => :success)
end
```
Fine for simple cases, but it quickly breaks down when we want return the status information from several levels deep of method execution.

### Output parameters
When writing code that executes shell commands, you can provide a transcript parameter for capturing the output of the entire session:
```ruby
require ’stringio’
def make_user_accounts(host, transcript=StringIO.new)
  transcript.puts "* Making user accounts..."
  # ... code that captures STDOUT and STDERR to the transcript
end
...
transcript = StringIO.new
make_user_accounts('192.168.1.123', transcript)
```

### Caller-supplied fallback strategy
Inject a failure policy into the process.
```ruby
def make_user_accounts(host, failure_policy=method(:raise))
# ...
rescue => error
  failure_policy.call(error)
end

policy = lambda {|e| puts e.message}
make_user_accounts("192.168.1.123", policy)
```
### Global variables
We’ll use a thread-local variable instead of global variable.
```ruby
class Provisioner
  def provision
    # ...
    (Thread.current[:provisioner_errors] ||= []) << "Error getting key file..."
  end
end
p = Provisioner.new
p.provision
if Array(Thread.current[:provisioner_errors]).size > 0
  # handle failures...
end
```

Disadvantage: unless we are diligent about resetting the variable, there is no
way of knowing for sure if its value is from the preceding call to #provision,
or from some other prior and unrelated call.
You can mitigate that issue by using a helper, which cleans the error variables when it's done.

### Process reification
Represent the process itself as an object, and give the object an attribute for collecting status data:

```ruby
class Provisionment
  attr_reader :problems
  def initialize
    @problems = []
  end
  def perform
  # ...
    @problems << "Failure downloading key file..."
  end
end
p = Provisionment.new
p.perform
if p.problems.size > 0
# ... handle errors ...
end
```

### Beyond exceptions
Lisp conditions.

## Your failure handling strategy
### Exceptions shouldn’t be expected
Consider carefully whether a situation is truly unusual before
raising an Exception.
E.g. `#save` doesn’t raise an exception when the record is invalid.

When writing an application you expect invalid input from users.
Don't hand invalid input with exceptions.

### A rule of thumb for raising
> ask yourself, ‘Will this code still run if I remove all the excep-
tion handlers?” If the answer is “no”, then maybe exceptions are
being used in non-exceptional circumstances.

### Use throw for expected cases
Sometimes you really want the kind of non-local return that raising an ex-
ception can provide, but for a non-exceptional circumstance.

Use `throw` and `catch`.

In Ruby, throw and catch are all about providing a way to quickly break out of multiple levels of method calls in non-exceptional circumstances.

```ruby
def last_modified(time)
  response[’Last-Modified’] = time
  if request.env[’HTTP_IF_MODIFIED_SINCE’] > time
    throw :halt, response
  end
end
```
### What constitutes an exceptional case?
The answer to all of these is: it depends!

### Caller-supplied fallback strategy
> In most cases, the caller should determine how to handle an error, not the callee.

Great example: `fetch`.
```ruby
h.fetch(:optional_key){ DEFAULT_VALUE }
h.fetch(:required_key) {
raise "Required key not found!"
}
```

Another example:
Enumerable:
```ruby
arr.detect(lambda{"None found"}) {|x| ... }
```

This is a terrifically useful idiom to use in your own methods.
Not sure => consider whether you can delegate the decision to the caller in some way.

```ruby
# render_user yields the passed block
# Fall back to a benign placeholder value:
render_user(u){ "UNNAMED USER" }
# Fall back to an exception:
render_user(u){ raise "User missing a name" }
```

### Some questions before raising
1: Is the situation truly unexpected?
2: Am I prepared to end the program?
3: Can I punt the decision up the call chain?
4: Am I throwing away valuable diagnostics?
When you have just received the results of a long, expensive operation,
that’s probably not a good time to raise an exception because of a trivial formatting error. Use some kind of sideband.
5: Would continuing result in a less informative exception?
In cases like this, it’s better to raise earlier than later.
```ruby
response_code = might_return_nil() or raise "No response code"
```

### Isolate exception handling code
Exception-handling code is messy. When it interrupts the flow of business logic, it makes program harder to understand.
I consider the begin keyword to be a code smell in Ruby.
The “better way” is to use Ruby’s implicit begin blocks:
```ruby
def foo
  # mainline logic goes here
rescue # -------------------
  # failure handling goes here
end
```
#### Isolating exception policy with a Contingency Method
```ruby
def with_io_error_handling
  yield
  rescue
  # handle IOError
end
with_io_error_handling { something_that_might_fail }
# ...
with_io_error_handling { something_else_that_might_fail }
```
### Exception Safety
Some methods are critical, we can't afford them to fail. E.g. error-reporting methods in one of the prev examples.

Methods that could potentially destroy user data or violate security
protocols if they fail are critical.

#### The three guarantees
The weak guarantee: If an exception is raised, the object will be left in a
consistent state.
The strong guarantee: If an exception is raised, the object will be rolled
back to its beginning state.
The nothrow guarantee: No exceptions will be raised from the method. If
an exception is raised during the execution of the method it will be
handled internally.

An implicit fourth(or zero-s) level: no guarantees.

No guarantees => cascading failure.

### When can exceptions be raised?
There are some exceptions, like NoMemoryError and SignalException, that can be raised at any point in execution.

#### Exception safety testing
For some methods, it is a good idea to have well-defined exception semantics, in the form of one of the Three Guarantees.
Exception safety testing.

In the playback phase, the test script is executed once for each call
point found in the record phase. Each time, a different call point is
forced to raise an exception. After each execution, the assertions are
checked to verify that they have not been violated as a result of the
exception that was raised.

#### Validity vs. consistency
Validity Are the business rules for the data met?
Consistency Can the object operate without crashing, or exhibiting unde-
fined behavior?

We can preserve consistency even while permitting the object to be invalid
from a business perspective.

Typically an object’s invariant includes rules to maintain consistency, but it does not necessarily make assertions about validity

### Be specific when eating exceptions
`rescue Exception` is prone to bugs.
If you can’t match on the class of an Exception, try to at least match on the message.
```ruby
begin
# ...
rescue => error
  raise unless error.message =~ /foo bar/
end
```
### Namespace your own exceptions
That gives client code something to match on when calling into the library.
Your code can also raise such exceptions as IOError, SystemCallError.

A way to be considerate to your library clients is to put code in the top-level
API calls which catches non-namespaced Exceptions and wraps them in a namespaced exception before re-raising it (Namespaced exceptions).

```ruby
rescue MyLibrary::Error
  raise
rescue => error
  # This assumes MyLibrary::Error supports nesting:
  raise MyLibrary::Error.new(
  "#{error.class}: #{error.message}",
error)
end
```

### Tagging exceptions with modules
```ruby
error.extend(AcmeHttp::Error)
```

We can extract out the tagging code into a general-purpose helper method:
```ruby
def tag_errors
  yield
rescue Exception => error
  error.extend(AcmeHttp::Error)
  raise
end
```

### The no-raise library API
The Typhoeus 15 library furnishes an excellent example of a no-raise library
API. All of the information about a Typhoeus HTTP request—no matter
what its final disposition—is encapsulated in the Typhoeus::Response class.

### Three essential exception classes
- divide our Exceptions up by which module or subsystem
they come from.
- by software layer—e.g. separate exceptions for UI-level failures and for model-level failures.
- by severity levels: fatal, non-fatal

#### Three classes of exception
- user error
- logic error
- transient error => retry


























































































































































































