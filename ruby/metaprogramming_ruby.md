# Metaprogramming Ruby

## Part 1 - Metaprogramming Ruby
> Will write code that writes code that writes code for food.
### The M Word
Metaprogramming is writing code that writes code.

More formal: Metaprogramming is writing code that manipulates language constructs at runtime.

The authors of Active Record applied this concept, they wrote the code that defines accessors at runtime for any class that inherits from ActiveRecord::Base.

#### Metaprogramming and Ruby
If you want to manipulate language constructs, those constructs must exist at runtime.

Code generators and compilers also are code that writes code.
In a broad sense, this XML generation is an example of metaprogramming.
This brand: use a program to generate or manipulate another program and then run the second program.
You usually read and maybe even modify it after it's generated.

Different meaning (this book): focusing on code that manipulates itself at runtime.
Dynamic metaprogramming vs static metaprogramming.

Only a few languages allow you do to metaprogramming seamlessly and elegantly, and Ruby is one of them.

C: 2 different worlds: compile time (you have all variables and functions) and runtime where you have a bunch of machine code
C++: some lang constructs survive compilation, that's why you can ask for the object's class
Java: distiction between compile and runtime is fuzzier, you can list the methods of the class and climb up a chain of superclasses.
Ruby: most constructs are available at runtime.

Metaprogramming is so deeply entrenched in the Ruby language that it’s not even sharply separated from “regular” programming.

### Monday. The Object Model.
You'll refactor some legacy code and learn a couple of tricks.
#### Open Classes
Bill adds a method #to_alphanumeric to a String class.

#### Inside Class Definitions
In Ruby, there is no real distinction between code that defines a class and code of any other kind:
You can put any code you want in a class definition:
```ruby
3.times do
  class C
    puts "Hello"
  end
end
'Hello'
'Hello'
'Hello'
```
If you define the same class several times, the first time Ruby will define class, the second time - reopens the existing class and add methods to it.

The class keyword in Ruby is more like a scope operator than a class declaration.
> Spell: Open Class
You can reopen existing classes - even standard library classes such as String or Array —and modify them on the fly.

Examples:
```ruby
# money gem
standard_price = 100.to_money("USD")
```
Gem 'money' adds #to_money method to the Numeric class.

This spell is common for libraries.

##### The Problem with Open Classes
You may (inadvertently) define methods that are already define in the class and that may cause problems.
Some people may frown up this technique and call it Monkeypatching.

#### Inside the Object Model
##### What’s in an Object
##### Is Monkeypatch Evil?
Monkeypatches can be useful, but you should use them with care, they can be difficult to track, especially in larger systems.
Carefully check the existing methods, adding methods is safer then modifying the existing ones.
Monkeypatches can be safer by using Refinements.

###### Instance variables
Unlike in static languages, in Ruby there's no connection between an object’s class and its instance variables.
Different instances of the same class can contain different instance variables.
You can think of them as keys and values in a hash.

##### Methods
An object’s instance variables live in the object itself, and an object’s methods live in the object’s class
Objects of the same class share methods but don’t share instance variables.

##### The Truth About Classes
Classes themselves are nothing but objects.
Classes, like any object, have their own class, called Class.

```
'hello'.class # => String
String.class # => class
```
Class in Ruby is quite literally the class itself, and you can manipulate it like you would manipulate any other object.

You can call Class.new to create new classes while your program is running.
Other languages allow you to read class-related information, Ruby allows you to write that information at runtime.

Classes also have (instance) methods:
```ruby
# The "false" argument here means: ignore inherited methods
Class.instance_methods(false)
# => [:allocate, :new, :superclass]
```

##### Modules
```ruby
Class.superclass # => Module
```
A class is a module with three additional instance methods ( new , allocate , and superclass ) that allow you to create objects or arrange classes into hierarchies.

Distinction between classes and modules is to make the code more explicit: the module is usually included somewhere, the class is instantiated or inherited.

#### Constants
Class names are nothing but constants.
Any reference that begins with an uppercase letter is a constant.
A constant is similar to variable - you can even change it's value, but you'll get a warning.

Important difference: the scope of constants follows its own special rules.

##### The Paths of Constants
You can have multiple files with the same name, as long as they live in different directories.
You can even refer to a constant by its path, as you’d do with a file.

```ruby
module M
  class C
    X = 'a constant'
  end
  C::X # => "a constant"
end
M::C::X # => "a constant"
```

If you’re sitting deep inside the tree of constants, you can provide the absolute path to an outer constant by using a leading double colon as root:

```ruby
Y = 'a root-level constant'
module M
  Y = 'a constant in M'
  Y
  # => "a constant in M"
  ::Y
  # => "a root-level constant"
end
```
Module#constants returns all constants in the current scope.
Module.nesting => the current path.

##### The Rake Example
The earliest versions of Rake defined classes with obvious names, such as Task and FileTask.
To avoid clashes they were then nested in the Rake namespace.

If someone had an old Rake build file they could use:
```ruby
# gems/rake-0.9.2.2/lib/rake/classic_namespace.rb
Task = Rake::Task
FileTask = Rake::FileTask
FileCreationTask = Rake::FileCreationTask
```

#### Objects and Classes Wrap-Up
What’s an object? It’s a bunch of instance variables, plus a link to a class.
The object’s methods don’t live in the object—they live in the object’s class, where they’re called the instance methods of the class.

What’s a class? It’s an object (an instance of Class ), plus a list of instance methods and a link to a superclass.
Class is a subclass of Module , so a class is also a module.

These are instance methods of the Class class. A class has its own methods, such as new.
Like any object, classes must be accessed through references.
You already have a constant reference to each class: the class’s name.

#### Using Namespaces
#### Loading and Requiring
```
load('motd.rb')
```
Loading and execute code, with side-effects. Constants may pollute your program with the names of its own constants.
```
load('motd.rb', true)
```
Ruby creates an anonymous module, uses that module as a Namespace to contain all the constants from motd.rb , and then destroys the module.

Load is to execute code, require is to import libraries - class names are the reason why you import it.
require tries only once to load each file, while load executes the file again every time you call it.

### What Happens When You Call a Method?
```ruby
MySubclass.ancestors # => [MySubclass, MyClass, Object, Kernel, BasicObject]
```
#### Modules and Lookup
When you include a module in a class (or even in another module), Ruby inserts the module in the ancestors chain, right above the including class itself.
```
module M1
  def my_method
    'M1#my_method()'
  end
end
class C
  include M1
end
class D < C; end
D.ancestors # => [D, C, M1, Object, Kernel, BasicObject]
```

A second way to insert a module in a class’s chain of ancestors: the prepend method:
It's like include but inserts the module below the including class.

```ruby
class C2
  prepend M2
end
class D2 < C2; end
D2.ancestors # => [D2, M2, C2, Object, Kernel, BasicObject]
```

If you try to include a module in the same chain of ancestors multiple times, the later include (higher in hierarchy) will take no effect.

##### The Kernel
Ruby includes some methods, such as print , that you can call from anywhere in your code. It looks like each and every object has the print method. Methods such as print are actually private instance methods of module Kernel :
```ruby
Kernel.private_instance_methods.grep(/^pr/) # => [:printf, :print, :proc]
```
Kernel gets into every object’s ancestors chain.

##### The Awesome Print Example
```ruby
# gems/awesome_print-1.1.0/lib/awesome_print/core_ext/kernel.rb
module Kernel
  def ap(object, options = {})
  # ...
  end
end
```

#### Method Execution
##### The self Keyword
Every line of Ruby code is executed inside an object—the so-called current object (self)
All instance variables are instance variables of self , and all methods called without an explicit receiver are called on self.
As soon as your code explicitly calls a method on some other object, that other object becomes self.

##### What private Really Means
You cannot call a private method with an explicit receiver, event with self.

Can object x call a private method on object y if the two objects share the same class?
No
Can you call a private method that you inherited from a superclass?
The answer is yes, because you don’t need an explicit receiver to call inherited methods on yourself.

##### The Top Level
who’s self if you haven’t called any method yet?
```ruby
self
# => main
self.class
# => Object
```
main is the top-level context.

##### Class Definitions and self
In a class or module definition (and outside of any method), the role of self is
taken by the class or module itself.
```ruby
class MyClass
  self # => MyClass
end
```

#### Refinements
Spell: refinement
```ruby
module StringExtensions
  refine String do
    def to_alphanumeric
      gsub(/[^\w\s]/, '')
    end
  end
end
```
To activate the changes, you have to do so explicitly, with the using method:
using StringExtensions

'Using' will work until the end of the module (if you’re in a module definition) or the end of the file

In some situations refinements may be confusing.

## Tuesday: Methods
In languages like Java and C—feature a compiler presides over objects communication.
For every method call, the compiler checks to see that the receiving object has a matching method.
That's static typing.
Dynamic languages — such as Python and Ruby—don’t have a compiler policing method calls.
If some object doesn't have the called method, everything will work fine until it reaches a specific line of code.

Advantage of static type checking: the compiler can spot some of your mistakes before the code runs.
The price of it: you often have to write lots of boilerplate methods.

In Ruby, boilerplate methods aren’t a problem, you can use some techniques to avoid them.

### Dynamic Methods
#### Calling Methods Dynamically
```ruby
obj.send(:my_method, 3)
```
Spell: Dynamic Dispatch

#### Method Names and Symbols
In most cases, symbols are used as names of things—in particular, names of metaprogramming-related things such as methods.
Symbols are good for such things cause they are immutable.

##### Pry example
```ruby
# ...
defaults.merge!(options).each do |key, value|
  send("#{key}=", value) if respond_to?("#{key}=")
end
```
##### Privacy Matters
Object#send can call private methods.
You can use #public_send instead.

#### Defining Methods Dynamically
Spell: Dynamic Method
```ruby
class MyClass
  define_method :my_method do |my_arg|
    my_arg * 3
  end
end
```

### method_missing
BasicObject#method_missing responds by raising a NoMethodError, but you can override this behavior

#### Overriding method_missing
```ruby
class Lawyer
  def method_missing(method, *args)
    puts "You called: #{method}(#{args.join(', ')})"
    puts "(You also passed it a block)" if block_given?
  end
end
bob = Lawyer.new
bob.talk_simple('a', 'b') do
# a block
end
❮
You called: talk_simple(a, b)
(You also passed it a block)
```
Overriding method_missing allows you to call methods that don’t really exist.

#### Ghost Methods
When you need to define many similar methods, you can just respondo to calls through the method_missing.
It's like saying to the object, “If they ask you something and you don’t understand, do this.”
From the caller’s side, a message that’s processed by method_missing looks like a regular call.

##### The Hashie Example
Hashie::Mash - a more powerful OpenStruct class: if you want a new attribute, just assign a value to the attribute, implemented through method_missing.

#### Dynamic Proxies
Some objects rely almost exclusively on the Ghost Methods, these objects are often wrappers: for another object, a web service or code written in a different language.
##### The Ghee Example
This gem gives you easy access to Github API.
Ta API reveals tens of types of objects but the gem code is concise, because it uses Ghost Methods.
It contains a Ghee::ResourceProxy class.

For each type of GitHub object, such as gists or users, Ghee defines one subclass of Ghee::ResourceProxy.

When you call a method that changes the state of an object (e.g. Ghee::API::Gists#star), Ghee places an HTTP call to the corresponding GitHub URL. But when you call a method that just reads from an attribute, it ends up at method_missing.

method_missing forwards the call to the object returned by Ghee::ResourceProxy#subject
Subject also makes an http call, the call depends on what subclass you are using.
subject receives the GitHub object in JSON format and converts it to a Hashie::Mash.

So a call like my_gist_url will be forwarded to subject#method_missing and then to hashie #method_missing.

##### respond_to_missing?
If you ask an object if it responds to Ghost Message it will answer false.
This behavior can be problematic.
Ruby provides a clean mechanism to make respond_to? aware of Ghost Methods:
You can override respond_to_missing?
```ruby
class Computer
# ...
  def respond_to_missing?(method, include_private = false)
    @data_source.respond_to?("get_#{method}_info") || super
  end
end
```
Override respond_to_missing? every time you override method_missing.

##### const_missing
When you reference a constant that doesn’t exist, Ruby calls Module#const_missing and passes the name of the constant to as a symbol.

Don't introduce too many Ghost Methods - it'll be harder to debug.
Remember to fall back on BasicObject#method_missing when you get a call you don’t know how to handle (by calling super in method_missing)

### Blank Slates
When the name of a Ghost Method clashes with the name of a real, inherited method, the latter wins.
You can avoid some of such problems by creating a Blank Slate, that will have less inherited methods.

#### Basic Object
The easiest way to create a Blank Slate is to inherit from BasicObject.
```ruby
im = BasicObject.instance_methods
im # => [:==, :equal?, :!, :!=, :instance_eval, :instance_exec, :__send__, :__id__]
```
#### Removing Methods
Module#undef_method - remove any method, including the inherited ones
Module#remove_method - remove the method from reciever, keep for others.
E.g. builder undefs all methods except those used internally by ruby and instance_eval (name !~ /^(__|instance_eval$)/) to keep it's syntax like:
```ruby
xml.semester {
  xml.class 'Egyptology'
  xml.class 'Ornithology'
}
```

### Dynamic Methods vs. Ghost Methods
Ghost Methods can be dangerous (always call super, always redefine respond_to_missing? to avoid some problems)
But still they can lead to puzzling bugs.

Ghost Methods are not real methods but more of intercepted calls.
Dynamic Methods are just regular methods that are defined with define_method instead of def.

Ghost methods: when you have a large number of method calls, or when you don’t know what method calls you might need at runtime.
Use Dynamic Methods if you can and Ghost Methods if you have to.

## Wednesday: Blocks
### The Day of the Blocks
### Quiz: Ruby#
#### The using Keyword
C# provides a keyword named using to continue even if the exception was raised at one point.
```с
using (conn)
{
  conn.ReadData();
  DoMoreStuff();
}
```
Ruby's using:
```ruby
module Kernel
  def using(resource)
    begin
      yield
    ensure
      resource.dispose
    end
  end
end
```
#### Blocks Are Closures
Code that runs is actually made up of two things: the code itself and a set of bindings.
Binding: local variables, instance variables, self, ... Bindings are basically names bound to objects.

Blocks contain both the code and a set of bindings - they are ready to run.

When you define the block, it captores the local bindings and carries them along when you pass the block into a method.
```ruby
def my_method
  x = "Goodbye"
  yield("cruel")
end
x = "Hello"
my_method {|y| "#{x}, #{y} world" } # => "Hello, cruel world"
```
You can also define additional bindings (e.g. local variables) inside the block, but they disappear after the block ends.

CS scientist would say a block is a closure.
All the bindings reside in a scope.

#### Scope
Scope: a set of bindings (self, local variables, current object instance methods and variables, tree of constants, global variables)

##### Changing Scope
A program moves through scopes while being executed. It starts within the top-level scope.
Some langs (Java, C#) allow “inner scopes” to see variables from “outer scopes. In Ruby scopes are clearly separated.

##### Global Variables and Top-Level Instance Variables
Global variables can be accessed by any scope: $var
It' difficult to track who is changing what, so use them sparingly, if ever.

Top-level instance variables are available when main plays the role of self.
Top-level instance variables are generally considered safer than global variables—but not by a wide margin.

Whenever the program changes scope, some bindings are replaced by a new set of bindings.

##### Scope Gates
3 places where the program changes the scope:
- Class definitions
- Module definitions
- methods

Keywords class, module and def act like a Scope Gate.
he code in a class or module definition is executed immediately, the code in a method definition is executed later.

##### Flattening the Scope
There may be situations where you want to pass bindings through a Scope Gate.
You can replace 'class' with a method call.
```ruby
my_var = "Success"
MyClass = Class.new do
  # Now we can print my_var here...
  puts "#{my_var} in the class definition!"
  define_method :my_method do
  "#{my_var} in the method"
  end
end
```
If you replace Scope Gates with method calls, you allow one scope to see variables from another scope.
Spell: Flat scope.

##### Sharing the Scope
You want to share a variable among a few methods, and you don’t want anybody else to see that variable.
```ruby
def define_methods
  shared = 0
  Kernel.send :define_method, :counter do
    shared
  end
  Kernel.send :define_method, :inc do |x|
    shared += x
  end
end
```
Spell: Shared Scope.
They are not used much in practice.

### instance_eval()
BasicObject#instance_eval - evaluates a block in the context of an object:
The block is evaluated with the receiver as self, so it can access the receiver’s private methods and instance variables.
```ruby
class MyClass
  def initialize
    @v = 1
  end
end
obj = MyClass.new
obj.instance_eval do
  self # => #<MyClass:0x3340dc @v=1>
  @v # => 1
end
```
The block that you pass to instance_eval can still see the bindings from the place where it’s defined.
Spell: Context Probe.

#### Breaking Encapsulation
Context Probe breaks incapsulation.
But in some situations it'll be helpful, e.g. you want to take a quick peek inside an object.

#### instance_exec()
Accepts arguments.
When instance_eval switches self to the receiver, all the instance variables in the caller fall out of scope.
In some cases you may use instance_exec and pass variable to it.
```ruby
class D
  def twisted_method
    @y = 2
    # @x is the C instance var
    C.new.instance_exec(@y) {|y| "@x: #{@x}, @y: #{y}" }
  end
end```

##### Breaking Encapsulations for tests: Padrino
Another acceptable reason to break encapsulation is arguably testing.
```ruby
# instead of creating and configuring a new logger
Padrino.logger.instance_eval{ @log_static = false }
```
You can criticize these tests for being fragile.
Encapsulation is a powerfult tool, break it at your own risk.

#### Spell: Clean Room
Creating an object just to evaluate blocks inside it.
An environment where you can evaluate your blocks.
The ideal Clean Room doesn’t have many methods or instance variables cause their names can clash with the names in the environment that the block comes from.

Instances of BasicObject usually make for good Clean Rooms,
Example: in "Better DSL".

### Callable Objects
Packaging code with blocks: get the code, call the block (with yield).
Other code packaging: procs,  lambdas, methods.

#### Proc Objects
Blocks are not objects but if you want to store a block and execute it later, you need an object.
```ruby
# way #0 of converting block into a Proc
inc = Proc.new {|x| x + 1 }
# more code...
inc.call(2) # => 3
```
Spell: Deferred Evaluation
Ruby provides two Kernel Methods that convert a block to a Proc : lambda and proc.
```ruby
# way #1
dec = lambda {|x| x - 1 }
dec.class # => Proc
dec.call(2) # => 1
# the same with proc
dec = proc {|x| x - 1 }

# way 2
p = ->(x) { x + 1 }
```
#### The & Operator
A block is like an additional, anonymous argument to a method.
But yield is not enough if:
- you want to pass the block to another method
- you want to convert a block to a Proc

To attach a binding to the block, you can add '&' to the name of the block.
The 'block' argument must be last in the list of arguments.

```ruby
def math(a, b)
  yield(a, b)
end

def do_math(a, b, &operation)
  math(a, b, &operation)
end

do_math(2, 3) {|x, y| x * y}
```

Convert a method to a proc - remove the '&'
Converting proc to a block - add the '&'

```ruby
def my_method(greeting)
  "#{greeting}, #{yield}!"
end
my_proc = proc { "Bill" }
my_method("Hello", &my_proc)
```

##### The HighLine Example
```ruby
hl = HighLine.new
friends = hl.ask("Friends?", lambda {|s| s.split(',') })
puts "You're friends with: #{friends.inspect}"
```
HighLine#ask passes a the Proc to an object of class Question, which stores the Proc as an instance variable.
Later, after collecting the user’s input, the Question passes the input to the stored Proc .

#### Procs vs. Lambdas
Procs created with lambda differ in some respects from Procs created any other way.

##### Procs, Lambdas, and return
In a lambda, return just returns from the lambda.
In a proc it returns from the scope where the proc itself was defined.

```ruby
def another_double
  p = Proc.new { return 10 }
  result = p.call
  return result * 2 # unreachable code!
end
another_double # => 10
```

That may lead to buggy code:
```ruby
def double(callable_object)
  callable_object.call * 2
end
p = Proc.new { return 10 }
double(p)
# => LocalJumpError (the proc tries to return from the top-level scope)
# this code would work fine with lambda
```
You can avoid this kind of mistake if you avoid using explicit returns:
```ruby
p = Proc.new { 10 }
```
##### Procs, Lambdas, and Arity
Call a lambda with the wrong arity, and it fails with an ArgumentError.
A proc fits the argument list to its own expectations.

If there are too many arguments, a proc drops the excess arguments. If there are too few arguments, it assigns nil to the missing arguments.

##### Procs vs. Lambdas: The Verdict
Generally speaking, lambdas are more intuitive than procs because they’re more similar to methods.

#### Method Objects
The last member of the callable objects’ family: methods.

```ruby
class MyClass
  def initialize(value)
    @x = value
  end
  def my_method
    @x
  end
end
object = MyClass.new(1)
m = object.method :my_method
m.call
# => 1
```
By calling Kernel#method, you get the method itself as a Method object, which you can later execute with Method#call .
You also have Kernel#singleton_method.
A Method object is similar to a block or a lambda.
```ruby
# Method => proc
Method#to_proc
# block => method
define_method { ... }
```
A lambda is evaluated in the scope it’s defined in, a Method is evaluated in the scope of its object.

#### Unbound Methods
are like Method s that have been detached from their original class or module.
You can turn a Method into the Unbound Method by Method#unbind or by calling Module#instance_method:
```ruby
unbound = MyModule.instance_method(:my_method)
```
You can’t call an UnboundMethod, but you can use it to generate a normal method by binding to an object with UnboundMethod#bind.
UnboundMethods that come from a class can only be bound to objects of the same class (or a subclass), UnboundMethods that come from a module have no such limitation

You can also bind an UnboundMethod by passing it to Module#define_method:
```ruby
String.class_eval do
  define_method :another_method, unbound
end
"abc".another_method
# => 42
```

##### The Active Support Example
ActiveSupport has it's own autoloading system, that overrides Kernel#load.
But if you want default Kernel#load behavior for one of the classes, you can use Loadable.exclude_from method.
```ruby
module Loadable
  def self.exclude_from(base)
    base.class_eval { define_method(:load, Kernel.instance_method(:load)) }
  end
  ...
end
```
The code gets Kernel#load as an UnboundMethod (with instance_method), and then defines a new load directly on MyClass.
That method overrides Loadable#load.

That's a solution for a very specific case, don't try that at home :)

### Writing a Domain-Specific Language
The app should send a notification when one of many things happen (the list changes every week).
You need to write a DSL so that other programmers can use it like this:

```ruby
event "we're earning wads of money" do
  recent_orders = ...
  # (read from database)
  recent_orders > 1000
end
```
If the block returns true, then you get an alert via mail. If it returns false, then nothing happens.
#### First DSL
```ruby
def event(description)
  puts "ALERT: #{description}" if yield
end
load 'events.rb'
```
##### Sharing Among Events
The methods can share variables but they would clutter the top-level scope.
```ruby
event "monthly sales are suspiciously high" do
  monthly_sales > target_sales
end
event "monthly sales are abysmally low" do
  monthly_sales < target_sales
end
```
#### A Better DSL
Your boss wants you to add a setup instruction to the RedFlag DSL:
```ruby
setup do
  puts "Setting up sky"
  @sky_height = 100
end
setup do
  puts "Setting up mountains"
  @mountains_height = 200
end
.....
```
The DSL still checks events, and it also executes all the setups before each event.

```ruby
def event(description, &block)
  @events << {:description => description, :condition => block}
end
def setup(&block)
  @setups << block
end
@events = []
@setups = []
load 'events.rb'
# iterate events and setups and call blocks when necessary
```
The array is a top-level instance variable.
But these top-level variables are like global ones, so let's get rid of them.

##### Removing the “Global” Variables
To get rid of the global variable you can use a Shared Scope.
But such way the code is not preasantly simple as it used to be.

##### Adding a Clean Room
In current program events can change each other’s shared toplevel instance variables:
You want events to share variables with setups, but you don’t necessarily want events to share variables with each other.
If you decide that events should be as independent from each other, you might want to execute them in Clean Room.

# Thursday: Class Definitions
In Java and C# by class definitions you tell how your objects are supposed to behave. Nothing happens until you create an obj of that class and then call methods.
In ruby defining class actually runs code.

Keep in mind that a class is just a souped-up module, anything about classes applies to modules (class definition = module definition)

## Class Definitions Demystified
### Inside Class Definitions
You can put any code inside a class definition, not only define methods there.
Class definitions return value of the last statement:
```ruby
result = class MyClass
  self
end
result # => MyClass
```
Classes and modules are just objects and thet can be self.

#### The Current Class
When you run ruby code - you always have a current object - self.
There’s no equivalent keyword to get a reference to the current class, but it's easy to determine a class by looking at the code.
The class keyword has a limitation: it needs the name of a class - in some situations you may not know it.

You need some way other than the class keyword to change the current class.
##### class_eval()
Module#class_eval (module_eval) evaluates a block in the context of an existing class:
```ruby
def add_method_to(a_class)
  a_class.class_eval do
    def m; 'Hello!'; end
  end
end
add_method_to String
"abc".m
# => "Hello!"
```
It is very different from BasicObject#instance_eval.
instance_eval only changes self, while class_eval changes both self and the current class.
class_eval reopens the class,

class opens a new scope, losing sight of the current bindings,
class_eval has s Flat scope and can reference vars from the outer scope.

Module_eval/class_eval also have module_exec/class_exec, that passes extra params to a block.

Use instance_eval to modify an object that is not a class, use class eval to add methods to a class with def.

#### Class Instance Variables
All instance variables belong to the current object self, that's the same for classes.
```ruby
class MyClass
  @my_var = 1
end
```

##### Class Variables
```ruby
class C
  @@v = 1
end
```
Class variables can be accessed by subclasses and by regular instance methods, class instance variables can't
But class variables may lead to strange behavior:
```ruby
@@v = 1
class MyClass
  @@v = 2
end
@@v
# => 2
```
That's because class variables belong to the class hierarchy, try to avoid them and use class instance variables instead.

#### Quiz: Class Taboo
```ruby
c = Class.new(Array) do
  def my_method
  'Hello!'
  end
end
MyClass = c # 'MyClass'
```

### Singleton Methods
Spell: Singleton Method
Add methods to a specific object:
```ruby
str = "just a regular string"
def str.title?
  self.upcase == self
end
str.title? # => false
str.methods.grep(/title?/) # # => [:title?]
str.singleton_methods # => [:title?]
```

#### The Truth About Class Methods
Class methods are actually Singleton methods of a class.
The definitions of a Singleton Method and the definition of a class method are the same.
In the definitions, object can be an object reference, a constant class name, or self .
```ruby
def object.a_singleton_method; end
def MyClass.another_class_method; end
```
The syntax might look different in the three cases, but the underlying mechanism is always the same.

#### Class Macros
Ruby objects don’t have attributes. To have smth like them you need to define mimic methods: a reader and a writer.
You can use Module#attr_* family to automate this task. You can use them whenever self is a module or a class.
Spell: Class Macro; A method such as attr_accessor is called a Class Macro.

##### Class Macros Applied
Fixing the method names, that are unconventional without breaking the code that uses them:
```ruby
class Book
  def title # ...
  def subtitle # ...
  def lend_to(user)
    puts "Lending to #{user}"
    # ...
  end
  def self.deprecate(old_method, new_method)
    define_method(old_method) do |*args, &block|
      warn "Warning: #{old_method}() is deprecated. Use #{new_method}()."
      send(new_method, *args, &block)
    end
  end
  deprecate :GetTitle, :title
  deprecate :LEND_TO_USER, :lend_to
  deprecate :title2, :subtitle
end
```
### Singleton Classes
#### The Mystery of Singleton Methods
The Singleton Method can’t live in obj , because obj is not a class.
And it can't live in the class of the object, because if it did, all instances of that class would share it.

When you ask an object for its class, Ruby doesn’t always tell you the whole truth.
Instead of the class that you see, an object can have its own special, hidden class - the Singleton Class of the object (metaclass, eigenclass)

Ruby has a special syntax that place you in the scope of the singleton class:
```ruby
class << an_object
  # your code here
end
# you can return self out of the scope:
obj = Object.new
  singleton_class = class << obj
self
end
singleton_class.class
# => Class

# the handy way to get a Singleton Class
"abc".singleton_class
```
Singleton class is a Class, but very special:
They are invisible (until you use singleton_class or class << syntax)
They have only a single instance.
They can't be inherited.

A singleton class is where an object’s Singleton Methods live:

```ruby
def obj.my_singleton_method; end
singleton_class.instance_methods.grep(/my_/)
# => [:my_singleton_method]
```

#### Method Lookup Revisited
##### Singleton Classes and Method Lookup
Singleton classes names are not meant to be uttered by humans.
```ruby
class C
  def a_method
    'C#a_method()'
  end
end
class D < C; end
obj = D.new
obj.a_method # => "C#a_method()"

# Adding a singleton method:

class << obj
  def a_singleton_method
    'obj#a_singleton_method()'
  end
end

obj.singleton_class.superclass # => D
```
If an object has a singleton class, Ruby starts looking for methods in the singleton class first.
Let's add a class method:
```ruby
class C
  def a_class_method
    'C.a_class_method()'
  end
end
```
Singleton classes became slightly more visible in Ruby 2.1
Now you can ask for singleton class ancestors, the result will include ancestors that are also singleton classes.
```ruby
C.singleton_class # => #<Class:C>
D.singleton_class # => #<Class:D>
D.singleton_class.superclass # => <Class:C>
C.singleton_class.superclass # => <Class:Object>
```
The superclass of the singleton class of the object's class is the singleton class of the superclass of that object's class.
The reason fot it is that thanks to this arrangement, you can call a class method on a subclass:
```ruby
D.a_class_method # => "C.a_class_method()"
```
It’s only possible because method lookup starts in #D and goes up to #D ’s superclass #C, where it finds the method.
#D, #C - singleton classes of the corresponding classes
(Class methods are just singleton class methods of the classes)

##### The Great Unified Theory

##### Meta Squared
Singleton classes are classes, so they also have singleton classes, but there is no practical usage of them.

The superclass of the singleton class of an object is the object’s class
The superclass of the singleton class of a class is the singleton class of the class’s superclass.

##### Class Methods Syntaxes
```ruby
# 3 ways to define class methods:
def MyClass.a_class_method; end
class MyClass
  def self.another_class_method; end
end
# explicitlu opens a singleton class 
class MyClass
  class << self
    def yet_another_class_method; end
  end
end
```

#### Singleton Classes and instance_eval()
`class_eval` changes the current class and self,
`instance_eval` also changes the current class; it changes it to the singleton class of the receiver.

But the standard meaning of instance_eval is this: “I want to change self .”

##### Class Attributes
The practical use of the singleton classes:
What if you want to define an attribute on a class instead?
You can reopen Class and define the attribute there instead, but that adds the attribute to all classes.

You can define the attribute in the singleton class:
```ruby
class MyClass
  class << self
    attr_accessor :c
  end
end
```
That will add reader amd writer methods and an instance variable to the singleton clas of Myclass.

### Quiz
Spell: Class Extension

You can't define class method in a Module, include that module into the class and get a class method, the method will remain in the Module's singleton class.

To solve this you can define my_method as a regular instance method of MyModule and include it in the singleton class of the class.
```ruby
module MyModule
  def my_method; 'hello'; end
end

class MyClass
  class << self
    include MyModule
  end
end
```

#### Class Methods and include()
Spell: Object Extension
You can use this tricks for objects as well:
```ruby
obj = Object.new
class << obj
  include MyModule
end
```
But that's a clumsy way to extend a class or an object.

#### Object#extend
Object#extend is simply a shortcut that includes a module in the receiver’s singleton class.
```ruby
# extending object
obj = Object.new
obj.extend MyModule
obj.my_method
# => "hello"
# extending class
class MyClass
  extend MyModule
end
```
### Method Wrappers
You can give an alternate name to a Ruby method by using Module#alias_method :
```ruby
class MyClass
  def my_method; 'my_method()'; end
  alias_method :m, :my_method
end
obj = MyClass.new
obj.my_method # => "my_method()"
obj.m # => "my_method()"
```
If you alias a method and then redefine it, the alias will still refer to the original method.

Spell: Around Alias
- alias the method
- redefine it
- call the old method from the new method

Downsides:
- polluting your classes with additional method name
- if you accidentally load that code twice it will cause an exception
- they are monkeypatching

Ruby 2.0 introduced 2 additional ways to wrap additional functionality around an existing method.

#### More Method Wrappers
Refinements have one additional feature that enables you to use them in place of Around Aliases: call super in a method
Spell: Refinement Wrapper
```ruby
module StringRefinement
  refine String do
    def length
      super > 5 ? 'long' : 'short'
    end
  end
end
using StringRefinement
"War and Peace".length
# => "long"
```

Another way: using Module#prepend + super
Spell: Prepend Wrapper
```ruby
module ExplicitString
  def length
    super > 5 ? 'long' : 'short'
  end
end
String.class_eval do
  prepend ExplicitString
end
```
It's not local like a Refinement, but is considered cleaner than both a Refinement Wrapper and an Around Alias.

### Quiz
Saving a real method in an alias and using it in the new method implementation:
```ruby
class Integer
  alias_method :old_plus, :+

  def +(int)
    old_plus(1).old_plus(int)
  end
end
```

# Friday: Code That Writes Code
## Kernel#eval
Just evaluates a passed string.

### Binding Objects
- a whole scope packaged as an object.
You can capture a local scope and carry it around. Later, you can execute code in that scope by using the Binding object in conjunction with eval.

Creating a binding:
```ruby
class MyClass
  def my_method
    @x = 1
    binding
  end
end
b = MyClass.new.my_method
```
Evaluate code in a carried scope:
```ruby
eval @x, b
```
TOPLEVEL_BINDING - predefined constant, binding of a top-level scope.

Pry has a good use of bindings. The call to binding.pry opens a Ruby interpreter in the current bindings, right inside the running process.

Irb parses the standart input and passes each line to eval (Code Processor)
```ruby
eval(statements, @binding, file, line)
```
File and line - for the information for the stack trace.

#### Strings of Code vs. Blocks
eval requires string, class_eval and instance_eval can take both strings and blocks.
You should avoid strings of code.

##### The Trouble with eval()
With great power comes great danger.
Strings of code are difficult to read and modify.
Danger of code injection.

##### Defending yourself of code injection
- ban eval
- use alernative techniques like define_method

##### Tainted Objects and Safe Levels
Ruby automatically marks objects from external sources as tainted:
```ruby
user_input = gets()
puts user_input.tainted? # true
```
You can set safe levels ($SAFE global variable)
0 - can format hard disc, 3 - every object is tainted by default

You can't eval tainted strings.
By using safe levels carefully, you can write a controlled environment for eval. (Spell: Sandbox)
E.g. Erb builds a quick Sandbox.

##### Kernel#eval() and Kernel#load()
take the name of a source file and execute code from that file.
load and require are somewhat similar to eval, cause evaluating file is not that different from evaluating a string,
You don’t have as many security concerns cause you easily control the contents of the file, but safe levels have some limitations for these methods.

### Hook Methods
You can execute code when events happen in the object model (e.g. a class gets inherited or methods are defined, undefined or removed)
```ruby
# this code will be executed every time someone inherits from String
class String
  def self.inherited(subclass)
    puts "#{self} was inherited by #{subclass}"
  end
end
```
By default, Class#inherited does nothing, but you can override it with your own code.

```ruby
Module#included
Module#prepended
Module#method_added
Module#method_removed
Module#method_undefined
```
These methods work for regular methods, added to class, not the singleton methods.
For singleton methods you can use: `BasicObject#singleton_method_added , singleton_method_removed , and singleton_method_undefined`

#### Plugging into Standard Methods
Instead of using #included method you can override Module#include itself to plug into from the other side, but don't forget to call #super.
```ruby
module M; end
class C
  def self.include(*modules)
    puts "Called: C.include(#{modules})"
    super
  end
  include M
end
```

#### Adding class methods with #include
```ruby
module CheckedAttributes
  def self.included(klass)
    klass.extend ClassMethods
  end
  module ClassMethods
    # ...
  end
end

class Person
  include CheckedAttributes
  # ...
end
```
That trick is used in VCR gem and in some rails code (though replaced with the new way in most places)

# Part 2. Metaprogramming in Rails
### Active Support’s Concern Module
#### The Include-and-Extend Trick
Rails before concerns - 'include-and-extend trick' (link in the VCR example)
This trick has some problems:

Each module that defines class methods must also define a similar included hook that extend s its includer.
The trick may be replaced with just extending a class:
```ruby
class Base
  include Validations
  extend Validations::ClassMethods
# ...
```

Another problem: The Problem of Chained Inclusions
Imagine that you include a module that includes another module.
When Ruby calls `SecondLevelModule.included`, the base parameter is `FirstLevelModule`.
BaseClass includes `FirstLevelModule`, which in turn includes `SecondLevelModule`.
As a result, the methods in SecondLevelModule::ClassMethods become class methods on FirstLevelModule, not the `BaseClass`

Rails 2 did include an ugly fix to this problem.
It forced Rails to distinguish first-level modules from other modules, and each module needed to know whether it was supposed to be first-level.
So this trick created more problems that it solved. To address these issues, the authors of Rails crafted ActiveSupport::Concern.

### ActiveSupport::Concern
“concern” - a module that extend s ActiveSupport::Concern
```ruby
module TestConcern
  extend ActiveSupport::Concern

  def an_instance_method; "an instance method"; end

  module ClassMethods
    def a_class_method; "a class method"; end
  end

  # or

  class_methods do
    def a_class_method; "a class method"; end
  end
end
```

#### A Look at Concern’s Source Code
When a module extends Concern, Ruby calls the extended Hook Method, and extended defines an `@_dependencies` class instance variable on the includer.

##### `Module#append_features`
is a core Ruby method that will be called whenever you include a module.
`included` is a Hook Method that is normally empty, and it exists only in case you want to override it.
`append_features` is where the real inclusion happens, it checks whether the included module is already in the includer’s chain of
ancestors, and if it’s not, it adds the module to the chain.
You're not supposed to override `append_features` normally, but `Concerns` wants to do that.

##### `Concern#append_features`
- modules that extend Concern get an `@_dependencies` Class Variable
- they get an override of `append_features`

Never include a concern in another concern.
`append features` is executed as a class method of the concern: self is the concern, and base is the module that is including it,

When you enter append_features, you want to check whether your includer is itself a concern.
If it has an `@_dependencies` Class Variable, then you know it is a concern.
In this case you just add yourself to its list of dependencies, and you return false to signal that there was no inclusion.

If your includer is not itself a concern:
- check whether you’re already an ancestor of this includer
- if not: recursively include your dependencies in your includer

After this you must add yourself to the chain of ancestors by calling the standard `Module.append_features` with
`super`. And you have to extend the includer with your own ClassMethods module.

##### Concern Wrap-Up
ActiveSupport::Concern is a minimalistic dependency management system, wrapped into a single module with just a few lines of code.
Using concern is easy:
```ruby
extend ActiveSupport::Concern
# ...
module ClassMethods
# ...
end
```
Some programmers think that Concern hides too much magic behind a seemingly innocuous call to include, and this hidden complexity carries hidden costs.

### A Lesson Learned
that you can take from ActiveSupport::Concern:
> metaprogramming is not about being clever—it’s about being flexible.
Don’t strive for a perfect design at the beginning, don't use metaprogramming before you need it
Try to use most obvious techniques, until the things get complex and advanced techniques will be needed.

## The Rise and Fall of alias_method_chain
### The Reason for alias_method_chain
```ruby
module Greetings
  def greet
    "hello"
  end
end
class MyClass
  include Greetings
end
```
You may want to wrap optional functionality around greet with Around Aliases:
```ruby
class MyClass
  include Greetings
  def greet_with_enthusiasm
  "Hey, #{greet_without_enthusiasm}!"
  end
  alias_method :greet_without_enthusiasm, :greet
  alias_method :greet, :greet_with_enthusiasm
end

MyClass.new.greet # => "Hey, hello!"
MyClass.new.greet_without_enthusiasm # => "hello!"
```

The method wrapping is common in rails: you end up with 3 methods: method, method_with_feature , and method_without_feature. First 2 of them include the new feature.

`Module#alias_method_chain` was a part of `ActiveSupport`, it was used to solve the problem of duplicating those aliases.

`alias_method_chain` is actually still in Active Support. 
It creates those methods and has a few more features, such as yielding to a block so that the caller can override the default naming, and dealing with methods that end with an exclamation or a question mark.

### One Last Look at Validations
Old version of ActiveRecord::Validations (Rails 2)
```ruby
module ActiveRecord
  module Validations
    def self.included(base)
      base.extend ClassMethods
      base.class_eval do
        alias_method_chain :save, :validation
        alias_method_chain :save!, :validation
      ...
      end
...
```
These lines reopen the ActiveRecord::Base class and hack its save and save! methods to add validation.
It created methods `save`, `save_with_validation` with automatic validation and `save_without_validation`.

`Validations` module still needs to define two methods named `save_with_validation` and `save_with_validation!`` :

### The Fall of alias_method_chain
`alias_method_chain` had several problems:
First, problems of the Around Alias itself (polluting the class with methods, monkeypatching)
And it was too clever: it could become hard to track which version of a method you were actually calling.
It was also unnecessary in most cases: Ruby has a built-in way of wrapping functionality around an existing method.
E.g. calling `super`
```ruby
module EnthusiasticGreetings
  include Greetings
  def greet
    "Hey, #{super}!"
  end
end
class MyClass
  include EnthusiasticGreetings
end
```
Using `super` is not as glamorous, but simpler. Recent versions of `ActiveRecord::Validations` acknowledge that simplicity by using a regular override instead of alias_method_chain.
```ruby
def save(options={})
  perform_validations(options) ? super : false
end
```

### The Birth of Module#prepend
Using a Prepend Wrapper:
```ruby
module EnthusiasticGreetings
  def greet
    "Hey, #{super}!"
  end
end
class MyClass
  prepend EnthusiasticGreetings
end
```

### A Lesson Learned
Metaprogramming code can sometimes get complicated, and it can even cause you to overlook more traditional, simpler techniques.

Resist the temptation to be too clever in your code.
Ask yourself whether there is a simpler way to reach your goal than metaprogramming.

However, metaprogramming is still one of the tastier ingredients in the Rails pie ^_^

## The Evolution of Attribute Methods
Active Record generates attribute methods (accessors (write, read, query)) by looking at the columns of the database.
You may assume them to be `Ghost Methods` defined via `method_missing` or `Dynamic methods` defined via `define_method`, but things are more complicated.

### The History of Complexity
The first version of attribute_methods (in rails 1.0) was rather simple:
When you creates an ActiveRecord::Base object, its `@attributes` instance variable was populated with the name of the attributes from the database.

Accessors were defined through `method_missing`: first, it determined if the method was read, write or query accessor by regexps and then passed to read_attribute, write_attribute or query_attribute methods.

#### Rails 2: Focus on Performance
When calling a `Ghost Method` ruby first checks all ancestors and then go to `method_missing`. So in general calling a `Ghost Method` is slower than calling a normal method.
In most cases this difference is negligible.
But for accessor methods performance suffered.
That could be solved by using Dynamic Methods instead of Ghost Methods, but Rails authors went for a mixed solution: they kept both Ghost and Dynamic methods.

##### Ghosts Incarnated
The code for attribute methods moved from ActiveRecord::Base itself to a separate ActiveRecord::AttributeMethods module, which is then included by Base.

When you call a method such as `Task#description=` for the first time, the call is delivered to `method_missing`, which calls `define_attribute_methods`.
`define_attribute_methods` defines accessors for all the database columns dynamically.

The next time you call `description=` or any other accessor that maps to a database column, it's already a non-ghost method.

##### Attributes That Stay Dynamic
There are attributes, that are not defined by db column:
```ruby
my_query = "tasks.*, (description like '%garage%') as heavy_job"
task = Task.find(:first, :select => my_query)
task.heavy_job? # => true
```
They can be different for each object, so there’s no point in
generating Dynamic Methods.

##### Rails 3 and 4: More Special Cases
Attribute methods code became larger and more sophisticated.
Rails 2 improved performance by turning Ghost Methods into Dynamic Methods.
Rails 4 goes one step further: when it defines
an attribute accessor, it also turns it into an UnboundMethod and stores it in a method cache. If a second class has an attribute by the same name, Rails 4 just retrieves the previously defined accessor from the cache and binds it to the second class.

If you use ruby 2.0 or later, `define_method_attribute` retrieves an `UnboundMethod` from a cache of methods, and it binds the method to the current module with `define_method`. The cache of methods is stored in a constant named `ReaderMethodCache`.
`ReaderMethodCache` is an instance of an anonymous class—a subclass of
`AttributeMethodCache`.

### Lesson Learned
> keep your code as simple as possible, and add complexity as you need it

## One Final Lesson
> Metaprogramming Is Just Programming
Metaprogramming is just another powerful set of coding tools that you can wield to write code that’s simple, clean, and well tested.

# Appendixes
### Common Idioms
#### Mimic Methods
function-like methods, e.g. `puts` or `obj.my_attribute=`
The more you look into Ruby, the more you find creative uses for them:
e.g. `private` and `protected` are Mimic Methods

And *Class Macros*, such as `attr_reader`

```ruby
# Camping framework
class Help < R '/help'
    def get
    # rendering for HTTP GET...
```

`R` is actually a mimic method that takes a string and returns an instance of `Class`.

### Nil Guards
```ruby
a ||= []
```
#### Attribute Trouble
```ruby
class MyClass
  attr_accessor :my_attribute
    def set_attribute(n)
    my_attribute = n
  end
end
obj = MyClass.new
obj.set_attribute 10
obj.my_attribute # => nil
```
Ruby has no way of knowing whether this code is an assignment to a local variable called `my_attribute` or a call to a Mimic Method called `my_attribute=`.

When in doubt, Ruby defaults to the first option.
Use `self` explicitly when you assign to an attribute of the current object:

```ruby
class MyClass
  def set_attribute(n)
    self.my_attribute = n
  end
end
```

What if `MyClass#my_attribute=` happens to be private ?
There is one of Ruby’s few ad-hoc exceptions: attribute setters such as `my_attribute=` can be called with self even if they’re private:

```ruby
class MyClass
    private :my_attribute
end
obj.set_attribute 11 # No error!
obj.send :my_attribute # => 11
```

Spell: Lazy Instance Variable

```ruby
class C
  def elements
    @a ||= []
  end
end
```

#### Nil Guards and Boolean Values
Nil Guards don't work well with boolean values.
Nil Guards are unable to distinguish false from nil.
It can cause the occasional hard-to-spot bug, so you shouldn't use Nil Guards
to initialize variables that can have false or nil legitimate values.

### Self Yield
An object can also pass itself to the block.
```ruby
conn = Faraday.new("https://twitter.com/search") do |faraday|
  faraday.response :logger
  faraday.adapter Faraday.default_adapter
  faraday.params["q"] = "ruby"
  faraday.params["src"] = "typd"
end
```
You can get the same results by passing a hash of parameters to Faraday.new — but the block-based style has the advantage of making it clear that all the statements in the block are focusing on the same object.

```ruby
module Faraday
  class << self
    def new(url = nil, options = {})
    # ...
      Faraday::Connection.new(url, options, &block)
    end
# ...
```

```ruby
module Faraday
  class Connection
    def initialize(url = nil, options = {})
      # ...
      yield self if block_given?
      # ...
    end
# ...
```

Even `instance_eval` and `class_eval` optionally yield self to the block

#### The tap() Example
E.g. you want to puts `x` in the middle of the chain:
```ruby
['a', 'b', 'c'].push('d').shift.tap {|x| puts x }.upcase.next

class Object
  def tap
    yield self
    self
  end
end
```

### Symbol#to_proc()
```ruby
class Symbol
  def to_proc
    Proc.new {|x| x.send(self) }
  end
end

names.map(&:capitalize.to_proc)
names.map(&:capitalize)
```
You don’t have to write Symbol#to_proc , because it’s already provided by Ruby.

Ruby implementation supports more then 1 argument:
```ruby
# without Symbol#to_proc:
[1, 2, 5].inject(0) {|memo, obj| memo + obj }
# with Symbol#to_proc:
[1, 2, 5].inject(0, &:+)
# => 8
# => 8
```

## Domain-Specific Languages
GPL (General Programming Language)
vs
DSL (Domain Specific Language)

You can bend a GPL into something resembling a DSL for your specific problem.

### Internal and External DSLs
E.g. Markaby is plain old Ruby, but it looks like a specific language for HTML
generation.
It is an internal DSL, it lives within a larger, general-purpose language.
By contrast, languages that have their own parser, such as make, are often called external DSLs. E.g. Ant build language.
The interpreter is written in Java, but the Ant language is completely different from Java.

The advantage of an internal DSL is that you can fall back to your GPL when you need it, but the syntax will be constrained by the GPL behind it.

With Ruby, you can write an internal DSL that looks more like an ad hoc language for the specific problem, cause of the Ruby's flexible syntax.

That's why Ruby devs often use the internal DSL's
The standard build languages for Java and C are external DSLs, while the standard build language for Ruby (Rake) is just a Ruby library—an internal DSL.

#### DSLs and Metaprogramming
Definition of Metaprogramming with the knowledge about DSL: “designing a domain-specific language and then using that DSL to write your program.”

Metaprogramming provides the bricks that you need to build DSLs.

> At the beginning of this book, we defined metaprogramming as “writing code
that writes code” (or, if you want to be more precise, “writing code that
manipulates the language constructs at runtime”).



