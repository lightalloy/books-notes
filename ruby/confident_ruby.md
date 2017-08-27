This book is about ruby and about joy.

# Chapter 1. Introduction

Ruby is designed to make programmers happy.
– Yukhiro "Matz" Matsumoto

Ruby programming is very clear, short and intent-revealing.

## Ruby in real world
But in real world code changes and looses its beauty. Code becomes convoluted and the joy of coding goes away.

## Confident code
Idealistic beginnings => less satisfying daily reality, lose of fun.

Approaches to write Ruby code which can help reverse this downward spiral.
It is a collection of time-tested techniques and patterns, tied together by a common theme: self confidence.

This book focuses on individual methods. It can lead to make improvements to overall project design, but main task is writing clear, uncluttered methods.
Our goal is to write methods which tell a story.

## Code as narrative
A single method is like a page in the story.
## The four parts of a method
Any given line of a method is serving one of the following roles:

1) Collecting input
2) performing work
3) Delivering output
4) handling failures
("diagnostics" and "cleanup" also appear, but are less common)

"un-confident" code: the haphazard mixing of the parts of a method.
Responsibilities are disorganized, this kind of code is often difficult to refactor and rearrange without breaking it.

Methods that tell a good story lump these four parts of a method together in distinct "stripes", rather than mixing them.
First, collect input. Then perform work. Then deliver output. Finally, handle failure, if necessary.

## How this book is structured

This book is a catalogue of patterns, not architectural but small, more like idioms or style guidelines (implementation patterns, code construction).

6 parts of the book:
1) writing methods in terms of messages and roles
2) "Performing Work", design of your methods, set the stage for the patterns to come.

Meat of the book - patterns:
3) patterns for collecting input
4) patterns for delivering results
5) techniques for dealing with failures

6) examples of refactoring for real-life open-source projects

Patterns description:
1) indication
2) synopsis
3) rationale
4) worked example
5) conclusion

# Chapter 2. Performing work

Think about intent of a method instead of the method's environment.

## 2.1. Sending a strong message
More than classes, inheritance, or encapsulation, the fundamental feature of an object oriented program is the sending of messages.   
We need to be able to trust that the objects we send messages to will respond to, and understand, those messages.
Elements to achieve it:
1) identifying the messages
2) identifying the roles
   Role is a set of related responsibilities, makes sense to be handled by an object (not the same as class)
   More than one object might play a given role, a single object might play more than one role.
   Identify the corresponding receiver role for each message.
   With the list of roles you can write the steps.
3) Bridge the new roles with the existing objects

### Avoiding the MacGyver method
Think about the mission, not the tools (classes, methods existing in the system, orm, etc)
### Letting language be constrained by the system
If we don't think about it, the roles we identify will be synonymous with the classes we already know about; and the messaging language we come up with will be dictated by the methods already defined on those classes.
Maybe an ideal set of classes already exist. But more likely, if we use just add methods to existing classes, clarity will suffer or the code will be cluttered by with checks to see if collaborator exists.

### Talk like a duck
Roles are names for duck types.
Simple interfaces that are not tied to any specific class and which are implemented by any object which responds to the right set of messages.
2 pitfalls of duck types:
- fail to determine a duck we really need
- giving up too early - instead of confident sending messages, checking is_a?(Duck), respond_to?(:quack) and constant nil checking
```ruby
duck && duck.quack?, duck.try(:fly) and duck.nil? # is also type checking
```
Switch Statement Smell (switching on type) - clutters logic, makes testing harder, embeds knowledge in numerous places.

Confident: figuring out what kind of messages and roles we need, and then ensuring we only allow ducks which are fully prepared to quack into our main method logic.

# Chapter 3. Collecting input
How to assemble, validate, and adapt method inputs such that we have a flock of reliable, obedient ducks to work with.
## 3.1 Introduction to collecting input
Some methods don't receive any input but they usually don't accomplish much, might be replaced by a constant.
Input can come in a usual way, in a form of a constant, or other method in the same class, or an instance variable.
Referred classes are also a type of input.
### Indirect inputs
```ruby
Time.now # indirect input
```
Any time we send a message to an object other than self in order to use its return value, we're using indirect inputs.
More levels of indirection => more tied code; more likely to break when it changes (Law of Demeter).   
One indirect input value combined with another to produce a needed value - one of the richest sources of bugs:
```ruby
user= ENV['USER']
prefs = YAML.load_file("/home/#{user}/time-prefs.yml")
```
Sometimes it's useful to distinguish 'input collection' phase from 'work' phase
Take a little time to think about what inputs a method relies on, it can take a large impact on the style, clarity, and robustness of the code.

### From roles to objects
We think how to map from the inputs that are available to the roles that the method would ideally interact with.
Bridge the gap from the objects we have to the roles we need.   
We've determined the inputs; we need to decide how to acquire those inputs.
Strategies for ensuring a method has collaborators which can be relied upon to play their assigned roles:

1) Coerce objects into the roles we need them to play (заставить имеющиеся объекты играть нужные роли)
2) Reject unexpected values which cannot play the needed roles (отказаться обрабатывать некорректные данные)
3) Substitute known-good objects for unacceptable inputs (заменить неприемлемые данные хорошими объектами ^_^)

### Guard the borders, not the hinterlands
Programming defensively in every method is redundant. Most of the techniques found in this section are best suited to the borders of your code.   
Ensure the objects passing into code have required interfaces.
Once objects pass through this border guard, they can be implicitly trusted to be good neighbors.

## 3.2 Use built-in conversion protocols
### Indications, Synopsis, Rationale
You want to ensure that inputs are of a specific core type, e.g. Integers.
Use Ruby's defined conversion protocols, like #to_str, #to_i, #to_path, or #to_ary.
We can provide a greater flexibility in the types of inputs our method can accept by only typing few charachters.
### Examples
#### Struct#index
Ruby arrays use #to_int to convert the array index argument to an integer, so the following works:
```ruby
Place = Struct.new(:index, :name, :prize) do
  def to_int
    index
  end
end
winners = ['Homestar', 'King of Town']
first = Place.new(0, "first", "Peasant's Quest game")
winners[first] # => first.to_int => 0; winners[0] => 'Homestar' 
```

#### Config file
You can define custom config class with a #to_path method.
File.open calls to #to_path on its filename argument to get the filename string.
Standard library has another non-String class, useful for representing filenames.
```ruby
require 'pathname'
config_path = Pathname("~/.emacs").expand_path
File.open(config_path).lines.count # => 5
```
File#open doesn't care cause Pathname can be converted to a String via its #to_path method.
### A list of standard conversion methods
The Ruby core and standard libraries often use standard conversion methods like #to_str, #to_int, and #to_path.
They are able to interact with any objects which respond to those methods.
List of the standard conversion methods used in Ruby core.   
Some of them are conditionally called by Ruby core code but never implemented by Ruby core classes. E.g. #to_path

| Method        | Target class  | Type  |
| ------------- |:-------------:| -----:|
|#to_a | Array | Explicit |
|#to_ary | Array | Implicit |
|#to_c | Complex | Explicit |
|#to_enum | Enumerator | Explicit |
|#to_h | Hash | Explicit |
|#to_hash | Hash | Implicit |
|#to_i | Integer | Explicit |
|#to_int | Integer | Implicit |
|#to_io | IO | Implicit |
|#to_open | IO | Implicit |
|#to_path | String | Implicit |
|#to_proc | Proc | Implicit |
|#to_r | Rational | Explicit |
|#to_regexp | Regexp | Implicit |
|#to_s | String | Explicit |
|#to_str | String | Implicit |
|#to_sym | Symbol | Implicit |

### Explicit and implicit conversions
 #to_s - explicit conversions: from classes which are mostly or entirely unrelated to the target class.
 #to_str - implicit conversions: from a class that is closely related to the target class.

E.g. Time object needs to be explicitly converted to String #to_s
We can't do 
```ruby
"the time is now: " + Time.now # => can't convert Time into String (TypeError)
```

Ruby checks if the secong arg of String#+ is string-ish by sending #to_str. If the object complains it doesn't respond to #to_str, String class raises TypeError.   
String is the only core class that supports #to_str method. But you can define your own string-ish objects. E.g. ArcicleTitle with #to_str method.

Ruby core classes almost never call explicit conversions, except for the string interpolation:
```ruby
"the time is now: #{Time.now}"
```
So explicit conversions (#to_s) are for programmers, implicit conversions (#to_str) are used in Ruby core classes.
### If you know what you want, ask for it
If we assume that an input to a method is one of the core classes (String, Integer, Array, Hash), you can make it explicit by calling one of the conversion protocol methods. (#to_i, #to_int!, #to_s, #to_str!, #to_h!)
Want to be forgiving: use explicit conversions (#to_i, #to_s) (nil.to_i => 0)
If the wrong type may indicate defect => implicit (#to_int) (nil.to_i => undefined method `to_int' for nil:NilClass), conv will serve as an assertion.  
Implicit conversions are more strict.
Both conversions: The client doesn't have to pass in objects of a specific class; just objects convertable to that class.

## 3.3 Conditionally call conversion methods
### Indications, Synopsys, Rationale
Broaden the range of inputs our method can accept:
use conversion protocols to transform inputs without forcing all inputs to understand those protocols.
Make the call to a conversion method conditional on whether the object can respond to that method.   
### Examples
#### Opening files
Why does this work?
```ruby
"/home/avdi/.gitconfig".respond_to?(:to_path) # => false
File.open("/home/avdi/.gitconfig")
# => #<File:/home/avdi/.gitconfig>
```
file.c does following:
See if the passed object responds to #to_path, use the result of calling that method if it does.
Ensure the result is String by calling #to_str.  
Сode can either supply the expected type, or pass something which responds to a documented conversion method.
### Violating duck typing, just this once
 #respond_to? violates duck-typing principles
But that's justifiable, if you want a method that can take input either as a core type (such as a String), or as a user-defined class which is convertible to that type. 

## 3.4 Define your own conversion protocols
### Indications, Synopsis, Rationale
Create a way for third-party objects to convert themselves to the type and "shape" our method needs.
Define new implicit conversion protocols mimicking Ruby's native protocols such as #to_path.
E.g. you expect to work with two-element arrays of integers representing X/Y coordinates.
### Example:
#### Accepting either a Point or a pair
We can define the #to_coords conversion protocol.
```ruby
def draw_line(start, endpoint)
  start = start.to_coords if start.respond_to?(:to_coords)
  start = start.to_ary
# ...
end

class Point
  attr_reader :x, :y, :name
  def initialize(x, y, name=nil)
    @x, @y, @name = x, y, name
  end
  def to_coords
    [x,y]
  end
end
```
We can use raw x/y pairs or Point interchangeably.
Client code can also define classes with #to_coords method.
So the method can accept both "plain old data" and sophisticated data (our classes and client-defined classes)

## 3.5 Define conversions to user-defined types
### Indications, Synopsis, Rationale
Your method logic requires input in the form of a class that you defined.
You want to convert instances of other classes to needed type implicitly, if it's possible.
Define your own conversion protocol for converting arbitrary objects to instances of the target class.
So it'll be possible to accept third-party objects as if they were "native".
### Examples:
#### Converting feet to meters
```ruby
class Meters
extend Forwardable
  def_delegators :@value, :to_s, :to_int, :to_i

  def initialize(value)
    @value = value
  end
  def to_meters
    self
  end
  def -(other)
    self.class.new(value - other.to_meters.value)
  end

  protected
  attr_reader :value
end

class Feet
  # ...
  def to_meters
    Meters.new((value * 0.3048).round)
  end
end
```
Any object which doesn't support the #to_meters protocol will trigger a NoMethodError.
But you can make calculations with any objects that have #to_meters method.

## 3.6 Use built-in conversion functions
### Indications, Synopsis, Rationale
You really want to convert an input object to a core type, no matter what.
Use Ruby's capitalized conversion functions, such as Integer and Array.
It allows maximum flexibility for inputs while preventing nonsensual conversions.
### Examples
#### Pretty printing arguments
The Kernel module also provides a set of unusually-named conversion functions: Array(), Float(), String(), Integer(), Rational(),and Complex().
They each share a name with a Ruby core class. Technically they are all methods, but we can call them functions
They are available everywhere and they interact only with their arguments, not with self.
Some standard libraries also provide capitalized conversion methods.
E.g. the uri library provides the URI() method, pathname library provides Pathname() method.
Capitalized conversion functions have 2 traits:
- idempotent
- can convert a wider array of input types than the equivalent #to_* methods
When using Integer() we are saying "convert this to an Integer, if there is any sensible way of doing so".
Exception: String() just calls #to_s
#### Hash.[]
```ruby
inventory = ['apples', 17, 'oranges', 11, 'pears', 22]
Hash[*inventory] # => { apples: 17, oranges: 11, pears: 22 }
```
This is useful for converting flat arrays of keys and values into hashes.

## 3.7 Use the Array() conversion function to array-ify inputs
### Indications, Synopsis, Rationale
Your method logic requires data in Array form, but the type of the input may take many forms.
Use the Array() conversion function to coerce the input into an Array.
No matter what form the input data is, our method will work with array.
### Accepting one or many arguments
Kernel#Array takes an argument and tries very hard to convert it to Array.
```ruby
def log_reading(reading_or_readings)
  readings = Array(reading_or_readings)
  readings.each do |reading|
  ...
  end
end
```
## 3.8 Define conversion functions
### Indications, Synopsis, Rationale
You want a public API to accept inputs in multiple forms, but internally you want to normalize objects to a single type of your own devising.
Define an idempotent conversion function and apply it to incoming objects.
E.g. Point(), which yields Points from integer, arrays, etc
Inputs are immediately converted to the needed type or rejected, so we don't have to worry.
### Examples
#### A conversion function for 2D points
Our conversion method should be concise and idempotent.
We'll define this method in a module, so that it can be included into any object which needs to perform conversions.
```ruby
module Conversions
  module_function
  
  def Point(*args)
    case args.first
    when Point then args.first
    when Array then Point.new(*args.first)
    when Integer then Point.new(*args)
    when String then
      Point.new(*args.first.split(':').map(&:to_i))
    else
      raise TypeError, "Cannot convert #{args.inspect} to Point"
    end
  end

  Point = Struct.new(:x, :y) do
    def inspect
      "#{x}:#{y}"
    end
  end
end
```
If we're uncertain about the type of input variables, we just surround them with calls to Point().
### [About module_function](https://github.com/lightalloy/what-i-have-learned/blob/master/ruby/module_function.md)

### Combining conversion protocols and conversion functions
It would be nice if we could also make it open to extension, so that client code can define new conversions to Point objects.
- #to_ary for both arrays and objects interchangeable with them
- #to_point for objects which know about the Point class and define a custom Point conversion of their own

```ruby
def Point(*args)
  case args.first
  when Integer then Point.new(*args)
  when String then Point.new(*args.first.split(':').map(&:to_i))
  when ->(arg){ arg.respond_to?(:to_point) } 
    args.first.to_point
  when ->(arg){ arg.respond_to?(:to_ary) }
    Point.new(*args.first.to_ary)
  else
    raise TypeError, "Cannot convert #{args.inspect} to Point"
  end
end
```

So we can now accepts client objects that respond to #to_ary ot #to_point methods.

### [Lambdas as case conditions](https://github.com/lightalloy/what-i-have-learned/blob/master/ruby/case_lambda_conditions.md)
Ruby's Proc objects have the threequals defined as an alias to #call
Case statements use the "threequals" (#===) operator to determine if a condition matches.

```ruby
even = ->(x) { (x % 2) == 0 }

case number
when 42
  puts "the ultimate answer"
when even
  puts "even"
else
  puts "odd"
end
```

## 3.9 Replace "string typing" with classes
The string is a stark data structure and everywhere it is passed there is much duplication of process. It is a perfect vehicle for hiding information. (Alan Perlis)
### Indications, Synopsis, Rationale
Input data is represented as a specially-formatted String. There are numerous case statements switching on the content of the String.
Replace strings that have special meanings with user-defined types.
Less redundancy, clarify object design, reduce mistakes.
### Examples
#### Traffic light states
Represent the states of the traffic light as a special kind of object.
```ruby
State = Struct.new(:name, :next_state) do
# ...
end
```
Instead of making them instances, we can make them subclasses:
```ruby
class Stop < State
  def color;
    'red';
  end
  def next_state; Proceed.new; end
end
# class Proceed, class Caution
```

We can create a conversion function so callers can supply a String or Symbol, which will be converted into the approprate State, if it exists.
```ruby
class TrafficLight
  def change_to(state)
    @state = State(state)
  end
# ...
  private
  def State(state)
    case state
    when State then state
    else self.class.const_get(state.to_s.capitalize).new
    end
  end
end

light = TrafficLight.new
light.change_to(:caution)
light.signal
```
### Conclusion
Repetitive case statements switching with the same variable (string)
Too easy to introduce an invalid value for the @state variable.
Solution: make the type system and polymorphic method dispatch.

Could be used for strings and symbols. Less common, but also may be used with Integers (set of "magic numbers")

## 3.10 Wrap collaborators in Adapters
### Indications, Synopsys, Rationale
A method may receive collaborators of various types, with no common interface between them.
Wrap input objects with adapters to give them a consistent interface.
An adapter encapsulates distracting special-case handling and ensures the special case is dealt with once and only once.
### Examples
#### Logging to IRC (contrived)
Class should be compatible with:
An in-memory Array, file, TCP or UDP socket, IRC Bot object
```ruby
class BenchmarkedLogger
  def initialize(sink=$stdout)
    @sink = sink
  end
  def info(message)
  ...
    @sink << ("[%1.3f] %s\n" % [duration, message])
  end
end
```
All sinks support the #<< operator except the IRC bot.
We could switch on the type of the input, but that's not confident.
So we can create the adapter:
```ruby
class BenchmarkedLogger
  class IrcBotSink
    def initialize(bot)
      @bot = bot
    end
    def <<(message)
      @bot.handlers.dispatch(:log_info, nil, message)
    end
  end
  def initialize(sink)
    @sink = case sink
    when Cinch::Bot then IrcBotSink.new(sink)
    else sink
    end
  end
end
```
If bot is received, it's adapted with IrcBotSink adapter, other objects remain as is.

## 3.11 Use transparent adapters to gradually introduce abstraction
### Indications, Synopsys, Rationale
A class has many preexisting dependencies on specific types of collaborators, making the introduction of adapters difficult
Make adapter objects transparent delegators to the objects they adapt.
Separation of concerns by tiny steps.
### Examples
#### Irc bot logging
We are not writing BenchmarkedLogger from scratch, it already has numerous switches on the type of the "sink", coupled with calls to Cinch::Bot-specific method calls.
We also don't have an infinite amount of time to rewrite it from scratch.
Wrapping Cinch::Bot instances with an adapter which only supports the #<< operator will break every method!
We can introduce a transparent adapter object: any other messages sent to the object will be passed through to the underlying Cinch::Bot object.
```ruby
require 'cinch'
require 'delegate'
class BenchmarkedLogger
  class IrcBotSink < DelegateClass(Cinch::Bot)
    def <<(message)
      handlers.dispatch(:log_info, nil, message)
    end
  end
  def initialize(sink)
    @sink = case sink
    when Cinch::Bot then IrcBotSink.new(sink)
    else sink
    end
  end
end
```
DelegateClass class generator - generates a base class in which all the methods of the passed class are implemented to delegate to an underlying object.
We just need to provide methods that we want to add to the adapter's interface.
Replace references of Cinch::Bot class with IrcBotSink after introducing adapter. It's a straightforward operation so it's fairly safe.

## 3.12 Reject unworkable values with preconditions
### Indications, Synopsis, Rationale
Some input values to a method cannot be converted or adapted to a usable form, accepting them will be harmful to the app.
Reject them early with precondition clauses, it's better to fail early, than partially succeed and raise a confusing exception.
### Examples
#### Employee hire date
Constructor should guard against initial values which are not compatible with the class invariant.
If not, every other method will be burdened with the additional responsibility of checking whether the value is present.
Class needs to set boundaries.
```ruby
class Employee
  attr_accessor :name
  attr_reader :hire_date
  def initialize(name, hire_date)
    @name = name
    self.hire_date = hire_date
  end
  def hire_date=(new_hire_date)
    # guard dates
    raise TypeError, "Invalid hire date" unless new_hire_date.is_a?(Date)
    @hire_date = new_hire_date
  end
  ...
end
```
Preconditions can also be used to check for invalid inputs to individual methods:
```ruby
def issue_service_award(employee_address, hire_date, award_date)
  unless (FOUNDING_DATE..Date.today).include?(hire_date)
    raise RangeError, "Fishy hire_date: #{hire_date}"
  ...
end
```
Preconditions were originally supposed to be callers responsibility:
Design by Contract(DbC) - caller should never call a method with values which violate that method's preconditions.
In languages that support DbC the runtime ensures that this contract is observed, and raises an exception automatically when a caller tries to supply bad arguments to a method.
Ruby doesn't support DbC, so preconditions were moved into the beginning of protected_method.
### Executable documentation
Preconditions serve double duty: guard method from invalid inputs, serve as executable documentation of the kind of inputs the method expects.

## 3.13 Use #fetch to assert the presence of Hash keys
### Indication, Synopsys, Rationale
A method takes a Hash, certain hash keys are required for correct operation.
Assert the presence of the required key with Hash#fetch.
### Example: A wrapper for useradd(8)
fetch can discern falsey values (nil or false) from non-existant ones.
```ruby
def add_user(attributes)
  login = attributes.fetch(:login)
  password = attributes.fetch(:password)
  ...
end
```
### Customizing #fetch
Fetch can take a block, it'll be executed if key is not found:
```ruby
password = attributes.fetch(:password) do
  raise KeyError, "Password (or false) must be supplied"
end
```
## 3.14 Use #fetch for defaults
### Indication, Synopsys, Rationale
Some hash keys are optional, you want to provide default values for missing keys
### Example: Optionally receiving a logger
```ruby
# distinguish false logger from non-existant one
logger = options.fetch(:logger) { Logger.new($stdout) }
if logger == false
  logger = Logger.new('/dev/null')
end
```
### Reusable #fetch blocks
```ruby
DEFAULT_LOGGER = -> { Logger.new($stderr) }
def emergency_kittens(options={})
  logger = options.fetch(:logger, &DEFAULT_LOGGER)
  # ...
end
def emergency_puppies(options={})
  logger = options.fetch(:logger, &DEFAULT_LOGGER)
# ..
end
```
### Two-argument #fetch
You can pass a second argument instead of a block.
```ruby
logger = options.fetch(:logger, Logger.new($stdout))
```
Difference: When the default is passed as an argument to #fetch, it is always evaluated whether it is needed or not. The block is triggered only if needed.

## 3.15 Document assumptions with assertions
### Indication, Synopsys, Rationale
A method receives input data from an external system, a format is undocumented and volatile.
Assertions on input data will improve your understanding of the data, and warn you when the format changes.
When we communicate with systems we have no control over and lacking documentation, we need more type-checking and coercions at the borders.

```ruby
class Account
  def refresh_transactions
    transactions = bank.read_transactions(account_number)
    # not a bad smell here!
    transactions.is_a?(Array) or raise TypeError, "transactions is not an Array"
    transactions.each do |transaction|
      amount = transaction.fetch("amount")
      amount_cents = (Float(amount) * 100).to_i
      cache_transaction(:amount => amount_cents)
    end
  end
end
```

State assumptions at the borders.
Fail early instead of allowing misunderstood inputs to contaminate the system.
It reduces the need for type-checking and coercion in other methods.

## 3.16 Handle special cases with a Guard Clause
If you are using an if-then-else, if and else legs are seen as equally important.
Guard clause says, "This is rare, and if it happens, do something and get out."
### Indication, Synopsys, Rationale
In certain unusual cases, the entire body of a method should be skipped. 
Use guard clause for it.
### Example: Adding a "quiet mode" flag
```ruby
def log_reading(reading_or_readings)
  unless @quiet
    readings = Array(reading_or_readings))
    ....
  end
end

# refactored:
def log_reading(reading_or_readings)
  return if @quiet
  readings = Array(reading_or_readings))
  ....
end
```
Use guard clause to deal with special cases quickly.

## 3.17 Represent special cases as objects
### Indications, Synopsys, Rationale
There is a special case which must be taken into account in many different parts of the program. E.g. behaves differentely if user is not logged in.
Use polymorphism, it'll eliminate dozens of repetitive conditionals clauses (nil checks)
### Example: A guest user
### Representing current user as a special case object
```ruby
# GuestUser represents a case when user is not logged in
class GuestUser
  def initialize(session)
    @session = session
  end

  def name
    'Anonymous'
  end

  def authenticated?
    false
  end

  def cart
    SessionCart.new(@session)
  end

  def has_role?(role)
    false
  end
end

def current_user
  if session[:user_id]
    User.find(session[:user_id])
  else
    GuestUser.new(session)
  end
end
```
No need to check like this:
```ruby
current_user ? current_user.name : 'Anonymous' and so on.
```
### Making the change incrementally
Sometimes you can't refactor globally and remove all checks at once.
Sprouting a class, "pilot" program for redesign:
```ruby
def greeting
  user = current_user || GuestUser.new(session)
  "Hello, #{user.name}, how are you today?"
end
```
### Keeping the special case synchronized
Special Case and Normal Case objects  need to be synced:
For more complex interfaces create a shared test suite that is run against both the normal-case and special-case classes to verify that they both respond to the same set of methods.
```ruby
shared_examples_for 'a user' do
  it { should respond_to(:name) }
  it { should respond_to(:authenticated?) }
  ...
end

describe GuestUser do
  subject { GuestUser.new(stub('session')) }
  it_should_behave_like 'a user'
end
describe User do
  subject { User.new }
  it_should_behave_like 'a user'
end
```

## 3.18 Represent do-nothing cases as null objects
### Indications, Synopsis, Rationale
One of the inputs to a method may be nil. It flags a special case. The handling of the special case is to do nothing.
E.g. if the logger is nil don't perform logging.
Replace the nil value with a special Null Object collaborator object. The NullObject has the same interface as the usual collaborator, but it responds to messages by taking no action.
It eliminates checking for nil and provides a way to "nullify" interactions with external services for testing, "quiet mode" or disconnected operations.
### Examples
#### Logging shell commands
```ruby
if @logger
  @logger.info "Executing: ffmpeg #{ffmpeg_flags.join(' ')}"
end
```
Different loggers can log to a file, STDERR, log server, etc. All loggers share the same interface.
A nil logger is a special case.
```ruby
class NullLogger
  def debug(*) end
  def info(*) end
  def warn(*) end
  def error(*) end
  def fatal(*) end
end

class FFMPEG
  def initialize(logger=NullLogger.new); end
end
```
So we can get rid of the if statement.
### Generic Null Object
We may want to avoid writing empty method definitions for every message NullObject should respond to.
So we can define #method_missing:
```ruby
class NullObject < BasicObject
  def method_missing(*)
  end
  def respond_to?(name)
    true
  end
end
```
Inherits from BasicObject, which has only 8 methods.

### Crossing the event horizon
Sometimes we need to send messages to the result of method.
```ruby
metrics ||= NullObject.new
metrics.requests.attempted += 1
```
We can create a slightly modified object:
```ruby
class NullObject
  def method_missing(*)
    self
  end
  # ...
end
```
Instead of returning an implicit nil, our #method_missing will return self.self (NullObject instance).
By returning self, we give our NullObject "infinite depth". This is so-called "Black Hole Null Object". Black Holes can be useful, but also can be dangerous.
The black hole spreads one method call after another, into areas of the code we hadn't intended for them to reach.
Black holes are "truthy".
```ruby
def create_widget(attributes={}, data_store=nil)
  data_store ||= NullObject.new
  data_store.store(Widget.new(attributes))
end

widget = factory.create_widget(widget_attributes, data_store)
manifest = widget.manifest
manifest.draft? # => true
manifest.approved? # => true
```
Confusing!
Use black holes with care. We should ensure that a black hole never "leaks" out of an object's library or object neighborhood.
If a method that forms part of our API might return a null object, we should convert the null object back to nil before returning it.
```ruby
def Actual(object)
  case object
  when NullObject then nil
  else object
  end
end

Actual(User.new) # => #<User:0x00000002218d18>
Actual(nil) # => nil
Actual(NullObject.new) # => nil
```
We can use the Actual() function as a filter to prevent null objects from leaking.
```ruby
def create_widget(attributes={}, data_store=nil)
  data_store ||= NullObject.new
  Actual(data_store.store(Widget.new(attributes)))
end
```
### Making Null Objects falsey.
NullObject looks like a variation of Ruby NilClass. 
We may want to make it act falsey. But Ruby doesn't let us define our own falsey objects, our class behavior will be not consistant.
The whole point of a Null Object is to avoid conditionals, so don't try to make it "falsey". If we can't avoid them we can use "if Actual()"

## 3.19 Substitute a benign value for nil
### Indications, Synopsis, Rationale
A non-essential input may not always be supplied. E.g. person's geolocation data. Substitute benign value for missing params.
You'll eliminate checks for the presence of optional information.
### Example: Displaying member location data
```ruby
def render_member(member, group)
  location = Geolocatron.locate(member.address) || group.city_location
  html = ""
  html << " <div class='fn'>#{member.fname} #{member.lname}</div>"
  html << " <img class='photo' src='#{member.avatar_url}'/>"
  html << " <img class='map' src='#{location.map_url}'/>"
end
```
Similar to the Null Object. A benign value can have semantically meaningful data or behavior associated with it.

## 3.20 Use symbols as placeholder objects
### Indications, Synopsis, Rationale
An optional collaborator may or may not be used depending on how a method is called. E.g. api accepts user credentials, but only when making requests to require.
Use a meaningful symbol, rather than nil, as a placeholder value. So unexpected usage will raise meaningful errors.
### Example: Optionally authenticating with a web service
Need authentication to get more than 20 results
Without symbolic placeholders:
```ruby
def list_widgets(options={})
  credentials = options[:credentials]
  ..
  if page_size > 20
    user = credentials.fetch(:user)
    password = credentials.fetch(:password)
  end
  ...
end
# credentials can be nil for some cases so if they are needed but are nil 
# it'll cause confusing error
# `list_widgets': undefined method `fetch' for nil:NilClass (NoMethodError)
```
### Symbolic placeholders
```ruby
credentials = options.fetch(:credentials) { :credentials_not_set } # => undefined method `fetch' for :credentials_not_set:Symbol (NoMethodError)
```
So the error will be more meaningful.
If it's common problems or library has many users, we may want to raise more specific exception or even use a Special Case object.
But symbolic placeholder is the fastest way to provide meaningful error message. 

## 3.21 Bundle arguments into parameter objects
### Indications, Synopsis, Rationale
Multiple methods take the same list of parameters. E.g. 2D points, each taking a pair of X/Y coordinates as arguments.
Combine parameters that commonly appear together into a new class.
### Parameter Object review
```ruby
Point = Struct.new(:x, :y)
```
Parameter Objects become "method magnets" — they become a natural home for behavior which was previously scattered around in other methods.
e.g. converting points to hashes.
Double Dispatch pattern to enable the Point to "draw itself":
```ruby
Point = Struct.new(:x, :y) do
  # ...
  def draw_on(map)
  # ...
  end
end
```
Reduced list of parameters, better readability.
Easier coordinates validation.
Point-specific behavior is in Point and is not duplicated in other methods.
### Adding optional parameters
A need to couple of variations for simple points:
1) Starred Point
```ruby
class StarredPoint < Point
  def draw_on(map)
    # draw a star instead of a dot...
  end
  def to_hash
    super.merge(starred: true)
  end
end
```
2) Fuzzy Point (has a vicinity radius)
```ruby
class FuzzyPoint < SimpleDelegator
  def initialize(point, fuzzy_radius)
    super(point)
    @fuzzy_radius = fuzzy_radius
  end
  def draw_on(map)
    super # draw the point
    # draw a circle around the point...
  end
  def to_hash
    super.merge(fuzzy_radius: @fuzzy_radius)
  end
end
```
Point can be both starred and fuzzy. So draw fuzzy, starred point:
```ruby
map = Map.new
p1 = FuzzyPoint.new(StarredPoint.new(23, 32), 100)
map.draw_point(p1)
```
The Map code remains unchanged.

## 3.22 Yield a parameter builder object
Bundling parameters into objects => clients must know about many different classes to use in API.
Solution: hide parameter object construction and yield the parameter object or a parameter builder object.
Approachable frontend for complex object creation. It separates interface from implementation, allowing major changes behind the scenes while maintaining a stable API.

### Example: a convenience API for drawing points
```ruby
map = Map.new
p1 = FuzzyPoint.new(StarredPoint.new(23, 32), 100)
map.draw_point(p1)
```
Client coders must know about FuzzyPoint and StarredPoint.
Breaking in 2 methods: draw_point and draw_starred_point.
```ruby
map.draw_point(7, 9) do |point|
  point.magnitude = 3
end
map.draw_starred_point(18, 27) do |point|
  point.name = "home base"
end

def draw_point(point_or_x, y=:y_not_set_in_draw_point)
  point = point_or_x.is_a?(Integer) ? Point.new(point_or_x, y) :
point_or_x
  yield(point) if block_given?
  point.draw_on(self)
end
```
### Net/HTTP vs. Faraday
```ruby
conn = Faraday.new(url: 'https://example.com')
response = conn.get '/' do |req|
  req.headers['Authorization'] = 'Bearer ABC123'
end
```
### Yielding a builder
```ruby
class PointBuilder < SimpleDelegator
  def initialize(point)
    super(point)
  end

  def fuzzy_radius=(fuzzy_radius)
    __setobj__(FuzzyPoint.new(point, fuzzy_radius))
  end
 
  def point
    __getobj__
  end
end
```
So in Map#draw_point you yield builder:
```ruby
class Map
  def draw_point(point_or_x, y=:y_not_set_in_draw_point)
     ...
     builder = PointBuilder.new(point)
     yield(builder) if block_given?
     ...
  end
end

map.draw_starred_point(7, 9) do |point|
  point.name = "gold buried here"
  point.magnitude = 15
  point.fuzzy_radius = 50
end
```

## 3.23 Receive policies instead of data
Edge case: some callers of a method may want to be notified about file missing, some not.
Receive a block or Proc which will determine the policy for that edge case.
### Example: Deleting files
```ruby
def delete_files
  begin
    # deleting
  rescue => e
    if block_given? then yield(file, error) else raise end
  end
end
delete_files(['does_not_exist', 'does_exist']) do
# ignore errors
end
delete_files(['does_not_exist', 'does_exist']) do |file, error|
  puts error.message
end
```
Multiple policies.
A method receives options which can contain policies.
```ruby
delete_files(
['file1', 'file2'],
on_error: ->(file, error) { warn error.message },
on_symlink: ->(file) { File.delete(File.realpath(file)) })
```
But that's probably a smell, in Ruby first try to find a more OO approach.

# 4 Delivering Results
## 4.1 Write total functions
A method may have zero, one, or many meaningful results.
Return the collection of results in all cases.
E.g.
0 elements - []
1 element - [element]
more - [el1, el2, ...]

Calling code will be able to handle results without special clauses.
Possibly using Array(result).

## 4.2 Call back instead of returning
Client code may need to take action depending on whether a command method made a change to the system.
Conditionally yield to a block on completion rather than returning a value.
A callback on success is more meaningful than a true or false return value.

CQS command-query separation:
Write methods which either have side effects (commands), or return values (queries), but never both.
```ruby
if import_purchase(date, title, user_email)
  send_book_invitation_email(user_email, title)
end

# refactor to

import_purchase(date, title, user_email) do |user, purchase|
  send_book_invitation_email(user.email, purchase.title)
end

# import_purchase calls/yields block on success and doesn't have to return true of false
def import_purchase(date, title, user_email, &import_callback)
  # ...
  unless user.purchased_titles.include?(title)
    # ...
    import_callback.call(user, purchase)
  end
end

```
Yielding to a block on success respects the command/query separation principle.
More intention-revealing than a true/false return value.
Suits better for conversion to a batch model of operation

## 4.3 Represent failure with a benign value
A method returns a nonessential value. Sometimes the value is nil.
Return a default value, e.g. as an empty string
Unlike nil, a benign value does not require special checking code to prevent NoMethodError.
Nil breaks things.
```ruby
def latest_tweets(number)
  # ...fetch tweets...
rescue Net::HTTPError
  ''
end
```

## 4.4 Represent failure with a special case object
A query method may not always find what it is looking for.
Return a Special Case object instead of nil. E.g. GuestUser.
It's one more way to avoid forcing client code to test return values for nil all the time. This topic is covered in "Represent special cases as objects".

## 4.5 Return a status object
A command method may have more possible outcomes than success/failure.
Represent the outcome of the method with a status object.
A status object can represent more nuanced outcomes and provide extra info.

3 outcomes: success, redundant, errors. One way is to return symbol (:success, :redundant, :errored)
Another possibility is to represent the method's outcome as a status object.
```ruby
class ImportStatus
  self.success() new(:success) end
  self.redundant() new(:redundant) end
  self.failed(error) new(:failed, error) end
  
  attr_reader :error
  
  def initialize(status, error=nil)
    @status = status
    @error = error
  end

  def success?
    @status == :success
  end

  def redundant?
    @status == :redundant
  end
  
  def failed?
    @status == :failed
  end
end
```

## 4.6 Yield a status object
A command method may have more possible outcomes than success/failure, and we don't want it to return a value.
Represent the outcome of the method with a status object with callback-style methods, and yield that object to callers.
```ruby
class ImportStatus
  def self.success() new(:success) end
  #... same for other statuses
  
  attr_reader :error
  def initialize(status, error=nil)
    @status = status
    @error = error
  end  

  def on_success
    yield if @status == :success
  end
  #... same for other statuses
end
```
Instead of returning an ImportStatus object, we yield one when the #import_purchase method has reached one of its possible conclusions.
In #import_purchases you yield the needed status.
```ruby
def import_purchase(date, title, user_email)
  # ...
  if smth
    yield ImportStatus.redundant
  else
    yield ImportStatus.success
  end
rescue => error
  yield ImportStatus.failed(error)
# ...
end
```
On client code:
```ruby
import_purchase(date, title, user_email) do |result|
  result.on_success do
    send_book_invitation_email(user_email, title)
  end
  ...
end
```
#import_purchase is a Command, not a Query

Yielding a status object helps partition methods into pure commands and queries.

## 4.7 Signal early termination with throw
A method needs to signal a distant caller that a process is finished and there is no need to continue.
Use throw to signal the early termination, and catch to handle it.

```ruby
throw :done

catch(:done)
  parser.parse(html) unless html.nil?
end
```
Don't use frequently.
throw and catch are more often used in framework code than in application code.
It's the best possible solution for a thorny problem.

# 5.Handling Failure
## 5.1. Prefer top-level resque clause
A method contains a begin/rescue/end block.
Switch to using Ruby's top-level rescue clause syntax.
Clear visual separation of 2 scenarios: "happy" and failure.
```ruby
def bar
# happy path goes up here
rescue #---------------------------
# failure scenarios go down here
end
```

## 5.2 Use checked methods for risky operations
A method contains an inline begin/rescue/end block to deal with a possible
failure resulting from a system or library call.
Wrap the system or library call in a method that handles the possible exception.
E.g. wrap a call to IO.popen in a checked method.
```ruby
def checked_popen(command, mode, error_policy=->{raise})
  IO.popen(command, mode) do |process|
    return yield(process)
  end
rescue Errno::EPIPE
  error_policy.call
end
```
Call #check_popen with a block that should be executed when everything is good.
```ruby
checked_popen(command, "w+", ->{message}) do |process|
... # write-read, etc
end
```
### To Adapters
Next step is wrapping calls to external library in adaper classes.
(3.10 Wrap collaborators in Adapters)
Wrapping system and library calls in adapter classes has many benefits: decouple design of your own code and the others' code that you don't control.
(Alistar Cockburn - Hexagonal Architecture)

begin/rescue/end blocks distract from the primary goal of the method, so extract them.

## 5.3 Use bouncer methods
An error is indicated by program state rather than by an exception. 
E.g. a failed shell command sets the $? variable to an error status.

Write the method to check the error state and raise an exception.
DRY up logic, no lower-error-checking in high level logic.

```ruby
def check_child_exit_status
  unless $?.success?
    raise ArgumentError,
      "Command exited with status "\
       "#{$?.exitstatus}"
  end
end
```

And just call this method when needed.
Alternative version - check_child_exit_status accepts block and yields it.


