# Chapter 1
# Object-Oriented Design

Design Principles:

SOLID:

Single Responsibility
Open-Closed
Liskov Substitution
Interface Segregation
Dependency Inversion

Others:
DRY (Don’t Repeat Yourself)
Law of Demeter (LoD)

Arranging code to efficiently accommodate change is a matter of design.
Design is good if it is easy to change (less costly)

# Chapter 2
# Designing Classes with a Single Responsibility

Design is more the art of preserving changeability than it is the act of achieving
perfection.

Code should be TRUE (Transparent, Reasonable, Usable, Exemplary)

Classes with Single Responsibility are easy to reuse

Determine if class has SR:
- interrogate it (like class is sentient)
- describe what class does in 1 sentence (if there are 'and' or 'or' - class has multiple responsibilities)

Do not redesign unless the cost of doing nothing is higher than the cost of redisign

### Depend on Behavior, Not Data:
- Hide Instance Variables (even from the class that defines them, by wrapping them in methods, e.g. use attr_reader)
  Send messages to access variables, even if you think of them as data.
- Hide Data Structures
  use ruby Struct to hide a structure
  (link to 'revealing references')
  Struct is “a convenient way to bundle a number of attributes together, using accessor methods, without having to write an explicit class.”

### Extract Extra Responsibilities from Methods
(methods should have single responsibility)

#### Methods with SR:
- Expose previously hidden qualities
- Avoid the need for comments
- Encourage reuse
- Are easy to move to another class

### Isolate Extra Responsibilities in Classes
Remove the responsibility without committing to a new class, a way to defer the decision about creating new class.

```ruby
class Gear
...
  Wheel = Struct.new(:rim, :tire) do
    def diameter
      rim + (tire * 2)
    end
  end
end
```
# Chapter 3
# Managing Dependencies

An object depends on another object if when one object changes, the other might be forced to change in turn.

Recognize dependencies:
- The name of another class
- The name of a message that it intends to send to someone other than self
- The arguments that a message requires
- The order of those arguments

A class should have as few dependencies as possible

## Techniques to reduce coupling:

### Dependency Injection:

Don't depend explicitely on class name (in other class methods), rely on message, that object sends

### Isolate Dependencies

If you cannot remove unnecessary dependencies (in a real app), isolate them within your class, reduce when possible.

#### Isolate Instance Creation

If you cannot remove dependency on the class name, isolates creation of a new instance in a method

```ruby
def wheel
  @wheel ||= Wheel.new(rim, tire)
end
```

If you are mindful of dependencies and develop a habit of routinely injecting them, your classes will naturally be loosely coupled

#### Isolate Vulnerable External Messages

Isolate a call to a method to an external object in a separate method.
Isolating the reference provides some insurance against being affected by the change in the external method.
Wrap the most vulnerable dependencies, not every external method.

### Remove Argument-Order Dependencies

Use hash of options instead of a fixed list of parameters.
- removes every dependency on argument order
- dependency on the names of the keys is healthier than of the arguments order
- hash names provide explicit documentation about the arguments

You can accept the dependency on order in some situations (e.g. 2 arguments).

Sometimes using few fixed-order arguments, followed by an options hash is more cost-effective.

#### Explicitly Define Defaults

```ruby
def initialize(args)
  @chainring = args.fetch(:chainring, 40)
  @wheel = args[:wheel]
end
```

When the defaults are more complicated specify defaults by merging a defaults hash.

#### Isolate Multiparameter Initialization

If you use a class which is a part of external framework and it requires fixed-order arguments, create a wrapper module.
It will isolate all knowledge of the external interface in one place and provides an improved interface.
Wrapper is responsible for creating new instances of external class.

```ruby
module GearWrapper
  def self.gear(args)
    SomeFramework::Gear.new(args[:chainring],
                            args[:cog],
                            args[:wheel])
  end
end
```

The sole purpose of this module is to create instances of some other class.
It's factory :)

Do not allow these kinds of external dependencies to permeate your code; protect yourself by wrapping each in a method that	 is owned by your own application.

## Managing Dependency Direction

### Choosing Dependency Direction

Advice for classes:
Depend on things that change less often than you do.

Concrete classes are more likely to change than abstract classes.
Changing a class that has many dependents will result in widespread consequences.

#### Understanding Likelihood of Change

Ruby base classes always change less often than your classes, so it's rather safe to depend on them.

Choose the direction of dependencies based on the ranking along a scale of how likely it is to undergo a change

#### Recognizing Concretions and Abstractions

The abstraction is more stable than concretion by its nature, so depending on abstraction is safer.

Interfaces must be taken into account during design (though ruby doesn't make you explicitely declare the abstraction), for
design purposes you can behave as if your virtual interface is as real as a class.

#### Avoid dependent-laden classes (загруженных зависимостями)

This class (and the whole app) will be very hard to change then.

#### Finding the Dependencies That Matter

# Chapter 4
# Creating Flexible Interfaces

Object-oriented applications are made up of classes but defined by messages.
Conversation between objects takes place using their interfaces.

## Understanding Interfaces

Concepts of interface:
1) Public interface of a class (methods implemented in a class, that are intended to be used by others)
2) A set of messages where the messages themselves define the interface. (Aslmost if an) Interface defines a virtual class, any class that implements the
required methods can act like the interface.

## Defining Interfaces

### Public Interfaces (a face to the world)
- reveal class primary responsibility
- are expected to be invoked by others
- will not change on a whim
- are safe to depend on
- are well documented in the tests

### Private interfaces
- handle implementation details
- are not expected to be sent by other objects
- can change for any reason
- are unsafe to depend on
- may not be referenced in tests

## Responsibilities, Dependencies, and Interfaces

### Finding the Public Interface

Nouns in the application that have both data and behavior are domain objects.
Don't concentrate on domain objects, but notice them.

### Using Sequence Diagrams

Sequence diagrams (UML or UML-like) represent object and messages passing between them.
Sequence diagrams are a vehicle for exposing, experimenting with, and ultimately defining public interfaces.

Turn from class-based design to message-based design.

### Ask for “What” Instead of Telling “How“

### Seeking Context Independence

The things that class knows about other objects makes up its context.
Class has a single responsibility but it expects a context.
It expect other classes to respond to specific messages.

Objects with simple context are easy to reuse and vice versa
To make class more independent concentrate on what class wants.

### Trusting Other Objects

If objects were humans, they would say to each other: “I know what I want and I trust you to do your part.”
It allows objects to collaborate without binding themselves to context and is necessary in any application that expects to grow and change.

### Using Messages to Discover Objects

You can discover the need of an yet undefined object via sequence diagrams, cost of being wrong is very low.
Your need to send a message => discovery of the object.

### Creating a Message-Based Application
Switching your attention from objects to messages allows you to concentrate on designing an application built upon public interfaces.
Sequence diagrams help you to keep the focus on messages.


## Writing Code That Puts Its Best (Inter)Face Forward
Think about interfaces. Create them intentionally.

### Create Explicit Interfaces

Methods in the public interface should:
- be explicitly identified as such
- be more about what than how
- have names that, insofar as you can anticipate, will not change
- take a hash as an options parameter

Methods in the private interface:
- obvious
- do not test private methods or, if you must, segregate those tests from the tests of public methods

Do not allow test to depend on private interface

#### Keywords:

private: least stable, must be called with an implicit receiver
protected: unstable method, allow explicit receivers as long as the receiver is self or an instance of the same class or subclass of self
public: stable, visible everywhere

Ruby supplies mechanisms for circumventing the visibility restrictions of private and protected, so they are more like flexible barriers.
By using them you prevent future programmers from accidentally using a method that you currently consider unstable

Many programmers omit them and instead use comments or a special method naming convention (e.g. a leading ‘_’ to private methods in RoR)

### Honor the Public Interfaces of Others

Try your beat to interact with other classes using only their public interfaces.
If your design forces the use of a private method in another class, first rethink your design.
A dependency on a private method of an external framework is a form of technical debt, avoid it.

### Exercise Caution When Depending on Private Interfaces

If you must depend on private method, isolate this dependency.

### Minimize Context

Minimize the context public interfaces require from others.
Keep the what versus how distinction in mind.
There should not be classes with absent public interface, create it (new method, wrapper class, wrapper method).

## The Law of Demeter

LoD is a set of rules resulting in a loosely coupled objects.
Loose coupling must be balanced against needs.

### Defining Demeter
"only talk to your immediate neighbors"; "use only one dot".

### Consequences of Violations
Certain “violations” of Demeter reduce your application’s flexibility and maintainability, while others make perfect sense.
1) message chains that return attributes - the risk of Demeter violations is low for stable attributes, this may be the most cost-efficient strategy
  tradeoff is permitted as long as you are not changing the value of the attribute you retrieve
2) a chain that reaches across many objects to get to distant behavior -
3) hash.keys.sort.join is not a Demeter violation cause all of the intermediate objects have the same type

### Avoiding Violations
Delegating - passing a message is to pass it on to another object, often via a wrapper method.
It may be useful but only hides tight coupling.

### Listening to Demeter
The train wrecks of Demeter violations are clues that there are objects whose public interfaces are lacking.
If you shift to a message-based perspective, the messages you find will become public interfaces in the objects they lead you to discover.

# Chapter 5
# Reducing Costs with Duck Typing

Duck types are public interfaces that are not tied to any specific class. 
Duck typed objects are defined more by their behavior than by their class.

## Understanding Duck Typing

If one object knows another’s type, it knows to which messages that object can respond.
The objects, that implements Duck public interface can be treated as Duck.

A ruby object can implement many different interfaces (can expose a different face to every viewer).
An an object’s type is in the eye of the beholder.

Interfaces may be not related to one specific class.
It’s not what an object is that matters, it’s what it does.

Across-class types, duck types, have public interfaces that represent a contract that must be explicit and well-documented.

### Overlooking the Duck

The method can possibly have no explicit dependency on certain class but it does depend on receiving an object that can respond to specific method, this is a dependency, that is easy to miss or to discount.

### Compounding the Problem
### Finding the Duck

Don't be blinded by existing classes.

```ruby
# Before refactoring
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(preparers)
    preparers.each {|preparer|
      case preparer
      when Mechanic
        preparer.prepare_bicycles(bicycles)
      when TripCoordinator
        preparer.buy_food(customers)
      when Driver
        preparer.gas_up(vehicle)
        preparer.fill_water_tank(vehicle)
      end
    }
  end
end
```

```ruby
# After refactoring
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(preparers)
    preparers.each {|preparer|
      preparer.prepare_trip(self)}
  end
end
```
(each class of preparers implements prepare_trip method)


### Consequences of Duck Typing

Concrete code is easy to understand but costly to extend.
Abstract code may initially seem more obscure but, once understood, is far easier to change.
Use of a duck type make code more abstract.
The ability to tolerate ambiguity about the class of an object is the hallmark of a confident designer.

#### Polymorphism
Polymorphism is the state of having many forms.
in OOP: the ability of many different objects to respond to the same message (senders need not care about the class of the receiver; receivers supply their own specific version of the
behavior)
A single message thus has many (poly) forms (morphs).
Ways to achieve polymorphism: duck typing, inheritance, sharing code via ruby modules.

## Writing Code That Relies on Ducks

Вesign challenge is to notice that you need a duck type and to abstract its interface.

### Recognizing Hidden Ducks

Coding patterns indicating the presence of a hidden duck:
• case statements that switch on class
• kind_of? and is_a?
• responds_to?

### Placing Trust in Your Ducks

These patterns ( case statements, is_a?, responds_to? ) are saying “I know who you are and because of that I know what you do.”
This knowledge exposes a lack of trust in collaborating objects.
When you see these code patterns, concentrate on the offending code’s expectations and use those expectations to find the duck type.

### Documenting Duck Types

Duck type and its public interface are a concrete part of the design but a virtual part of the code.
Abstraction makes the duck type less than obvious in the code.
You must create good test to test and document duck types.

### Sharing Code Between Ducks

is covered in "Ch 7 Sharing Role Behavior"

### Choosing Your Ducks Wisely

If creating a duck type would reduce unstable dependencies, do so.
It's safe to rely on ruby base classes, don't monkey patch unless you absolutely need that.

## Conquering a Fear of Duck Typing

### Subverting Duck Types with Static Typing

Some programmers beleive in static typing, so they tend to check object classes, it makes code less flexible.
Duck typing provides stable abstractions that removes the dependencies on class.

### Static versus Dynamic Typing

Static typing:
• The compiler unearths type errors at compile time.
• Visible type information serves as documentation.
• Compiled code is optimized to run quickly.
• Runtime type errors will occur unless the compiler performs type checks.
• Programmers will not otherwise understand the code; they cannot infer an object’s
type from its context.
• The application will run too slowly without these optimizations.

Dynamic typing:
• Code is interpreted and can be dynamically loaded; there is no compile/make cycle.
• Source code does not include explicit type information.
• Metaprogramming is easier.
• Overall application development is faster without a compile/make cycle
• Programmers find the code easier to understand when it does not contain type
declarations; they can infer an object’s type from its context.

### Embracing Dynamic Typing

The compiler cannot save you from accidental type errors.
When you start casting, the compiler excuses itself and you are left to rely on your own wits to prevent type errors.

The code is only as good as your tests.
In the real world, compiler preventable runtime type errors almost never occur.

Dynamic typing allows you to trade compile time type checking, a serious restriction that has high cost and provides little value, for the huge gains in efficiency provided
by removing the compile/make cycle.

# Chapter 6
# Acquiring Behavior Through Inheritance
This chapter teaches how to write code that properly uses inheritance, and decide if you should use it.

## Understanding Classical Inheritance
Inheritance is a a mechanism for automatic message delegation. 

## Recognizing Where to Use Inheritance
Antipattern - if statement that checks an attribute that holds the category of self to determine what message to send to self.
“I know who you are and because of that I know what you do"

### Finding the Embedded Types
Variables like type, category, style may be a cue to notice an underlying pattern.
Highly related types that share common behavior but differ along some dimension is a problem that inheritance solve.

### Choosing Inheritance

Message forwarding via classical inheritance takes place between classes; duck types share code via Ruby modules.

Ruby automatically sets your new class’s superclass to Object.
When an object receives a message it does not understand, Ruby automatically forwards that message up the superclass chain in search of a matching method implementation.
Subclasses are  specializations of their superclasses; every object has an Object public interface + more.

## Misapplying Inheritance

## Finding the Abstraction
Subclasses are specializations of their superclasses.
The objects that you are modeling must truly have a generalization–specialization relationship.
Parent class should not combine general and specific code, but have onlu general code.

### Creating an Abstract Superclass

Parent class is abstact class, Ruby doesn't provide abstract keyword, which prevents creation of instances of such classes.
So use your good sense to do this.
Creating a hierarchy has costs; maybe you should wait for more information to identify the correct abstraction (and tolerate code duplication for a while).

When doing rearrangement it’s easier to promote code up to a superclass than to demote it down to a subclass.

### Promoting Abstract Behavior

If you move code from parent to children, any failure on your part will leave dangerous remnants of concreteness in the superclass.
So promote abstractions rather than demote concretions.

Every decision you make includes two costs: one to implement it and another to change it when you discover that you were wrong.
Take both into account.

### Separating Abstract from Concrete

### Using the Template Method Pattern
Template method pattern is the technique of defining a basic structure in the superclass and sending messages to acquire subclass-specific contributions.

### Implementing Every Template Method
Explicitly state that subclasses are required to implement a certain message - it provides useful documentation for future programmers.

```ruby
class Bicycle
  def default_tire_size
    raise NotImplementedError, "This #{self.class} cannot respond to:"
  end
end
```

This additional information makes the problem inescapably clear. Always document template method requirements by
implementing matching methods that raise useful errors.

## Managing Coupling Between Superclasses and Subclasses
Managing coupling is important; tightly coupled classes stick together and may be impossible to change independently.

### Understanding Coupling

Sending super in subclasses methods creates additional dependency on superclass.
If one of future subclasses forgets to send super, this error can manifest at a time and place far distant from its cause, making it very hard to debug.
Forcing a subclass to know how to interact with its abstract superclass causes many problems.

### Decoupling Subclasses Using Hook Messages

post_initialize method (hook) in initialize method of superclass (to provide defaults to all subclasses)
It removes the initialize method from subclasses, they only implement post_initialize (hook) method if needed (to extend defaults)
Subclasses are responsible for what initialization it needs but is no longer responsible for when its initialization occurs.

Hooks can be used to other methods too:

```ruby
class Bicycle
  def spares
    {tire_size: tire_size,
    chain: chain}.merge(local_spares)
  end
  def local_spares
    {}
  end
end

class RoadBike < Bicycl
  def local_spares
    {tape_color: tape_color}
  end
end
```

# Chapter 7
# Sharing Role Behavior with Modules
This chapter explores an alternative that uses the techniques of inheritance to share a role.

## Understanding Roles
Some problems require sharing behavior among otherwise unrelated objects.
When formerly unrelated objects begin to play a common role, they enter into a relationship with the objects for whom they play the role.

## Finding Roles
Many object-oriented languages provide a way to define a named group of methods that are independent of class and can be mixed in to any object. In Ruby, these
mix-ins are called modules. Methods can be defined in a module and then the module can be added to any object. Modules thus provide a perfect way to allow objects of
different classes to play a common role using a single set of code.

The total set of messages to which an object can respond includes
• Those it implements
• Those implemented in all objects above it in the hierarchy
• Those implemented in any module that has been added to it
• Those implemented in all modules added to any object above it in the hierarchy

## Organizing Responsibilities
Instead of knowing details about other classes, the class should send them messages.

## Removing Unnecessary Dependencies
The fact that the class checks many class names to determine what value to place in one variable suggests that the variable name should be turned into a message,
which in turn should be sent to each incoming object.
### Discovering the Schedulable Duck Type
### Letting Objects Speak for Themselves
StringUtils.empty?(some_string) sounds ridiculous cause String is already an object and manages itself.
Objects should manage themselves; they should contain their own behavior.
If your interest is in object B, you should not be forced to know about object A if your only use of it is to find things out about B.

## Writing the Concrete Code
Pick an arbitrary concrete class and implement the interface method (of a role) directly in that class.
## Extracting the Abstraction
Role Module (e.g. Smthable) may include method that will be also implemented in classes playing this role - it is template method pattern.

The code in Schedulable is the abstraction and it uses the template method pattern to invite objects to provide specializations to the algorithm it supplies.
Schedulables override lead_days to supply those specializations.

Inheritance (is-a) and sharing code via modules (behaves-like-a) are similar because they rely on automatic message delegation.

## Looking Up Methods
### A Gross Oversimplification
An object receives method, look up in the class, look up in parent classes hierarhcy, finally look up in the Object class (top of the hierarchy)
Ruby also gives a second chance sending method_missing with argument method_name.

### A More Accurate Explanation
Modules are included in method lookup path. The module’s methods go into the method lookup path directly above methods defined in the class.

### A Very Nearly Complete Explanation
When a single class includes several different modules, the modules are placed in the method lookup path in reverse order of module inclusion. 
Thus, the methods of the last included module are encountered first in the lookup path.

It is also possible to add a module’s methods to a single object, using Ruby’s extend keyword. Extend adds the module’s behavior directly to an object, extending a class with a module creates class methods in that class and extending an instance of a class with a module creates instance methods in that instance.
Finally, any object can also have ad hoc methods added directly to its own personal
“Singleton class.” These ad hoc methods are unique to this specific object.

Method lookup

0) Methods defined only in this one instance
1) Methods defined in modules with which this instance has been extended.
2) Methods defined in class
3) Methods defined in modules included in class
4) Methods defined in parent class
5) Methods defined in modules included in parent class
6) Methods defined in class Object

## Inheriting Role Behavior

### Recognize the Antipatterns

Antipatterns:
0) a variable with a name like type or category to determine what message to send to self contains two highly related but slightly different types => inheritance
1) a sending object checks the class of a receiving object to determine what message to send => overlooked duck type
All of the possible receiving objects play a common role. This role should be codified as a duck type and receivers should implement the duck type’s interface.
In addition to sharing an interface, duck types might also share behavior, place the shared code in a module and include when needed.

### Insist on the Abstraction
All of the code in an abstract superclass should apply to every class that inherits it.
Superclasses should not contain code that applies to some, but not all, subclasses.
This also applies to modules.
Subclasses that override a method to raise an exception like “does not implement” are a symptom of the problem.

### Honor the Contract
Subclasses agree to a contract; they promise to be substitutable for their superclasses.
They must respond to every message in that interface, taking the same kinds of inputs and returning the same kinds of outputs. They are not permitted to do anything that forces others to check their type in order to know how to treat them or what to expect of them.
Subclasses may accept input parameters that have broader restrictions and may return results that have narrower restrictions, all while remaining perfectly substitutable for
their superclasses.

Liskov Substitution Principle (LSP) (from SOLID)
Let q(x) be a property provable about objects x of type T. Then q(y) should be true for objects y of type S where S is a subtype of T.
(subtypes must be substitutable for their supertypes) 

### Use the Template Method Pattern
It's fundamental coding technique for creating inheritable code.
The abstract code defines the algorithms and the concrete inheritors of that abstraction contribute specializations by overriding these template methods.
These methods represent the parts of the algorithm that vary and creating them forces you to make explicit decisions about what varies and what
does not.

### Preemptively Decouple Classes
Writing code that requires subclasses to send super adds an additional dependency; avoid this if you can.
Hook methods solve the problem of sending super, but only for adjacent levels of the hierarchy.

### Create Shallow Hierarchies
Deep, wide hierarchies should be avoided. 
A deep hierarchy has a large set of built-in dependencies, each of which might someday change; programmers tend to be familiar
with just the classes at their tops and bottoms; the classes in the middle lack attention - that increase a chance of introducing errors.

# Chapter 8
# Combining Objects with Composition
Composition is the act of combining distinct parts into a complex whole such that the whole becomes more than the sum of its parts.
The larger object is connected to its parts via a has-a relationship. E.g. the parts are contained within a bicycle.

## Composing a Bicycle of Parts
Bicycles have parts, the bicycle–parts relationship quite naturally feels like composition. If you created an object to hold all of a bicycle’s parts, you could delegate the spares message to that new object. This object represents a collection of parts, not a single part
Bicycle "has-a" Parts (line starting with black diamond in UML)
parts would be attr_reader in bicycle.

### Creating a Parts Hierarchy
Create Parts subclasses, move code from Bicycle subclasses to them.

## Composing the Parts Object
It’s time to add a class to represent a single part.

### Creating a Part
Parts becomes a simple wrapper around Part objects.
```ruby
chain = Part.new(name: 'chain', description: '10-speed')
road_tire = Part.new(name: 'tire_size', description: '23')
road_bike_parts = Parts.new([chain, road_tire])
```
or
```ruby
road_bike = Bicycle.new(size: 'L',
            parts: Parts.new([chain,
            road_tire]))
```
### Making the Parts Object More Like an Array
```ruby
mountain_bike.spares.size # -> 3
mountain_bike.parts.size # -> NoMethodError:
```
You can fix the proximate problem by adding a size method to Parts but other array methods (each, sort) will be unavailable.
You can also make Parts a subclass of Array.
But there are some problems: there are many methods in Array that return new arrays and they won't respond Parts methods.
The other solution is to include/extend some modules of Array in Parts class.
```ruby
require 'forwardable'
class Parts
  extend Forwardable
  def_delegators :@parts, :size, :each
  include Enumerable

  def initialize(parts)
    @parts = parts
  end

  def spares
    select {|part| part.needs_spare}
  end
end
```
Parts now responds methods like each, size, etc.
Method + is not implemented.

```ruby
mountain_bike = Bicycle.new(size: 'L', 
                            parts: Parts.new([chain, mountain_tire,
                            front_shock,
			    rear_shock]))
```

## Manufacturing Parts
Somewhere in your application, some object had to know how to create these Part objects. That knowledge is unnecessary and it can easily leak all over your application.
Everything would be easier if you could describe the different bikes and then use your descriptions to manufacture the correct Parts object for any bike.
It’s easy to describe the combination of parts that make up a specific bike in a 2-dimensional array.

### Creating the PartsFactory
The word factory does not mean difficult, or contrived, or overly complicated. It's just a word to describe an object that creates other objects.
PartsFactory module, it's job is to create Part objects.
First version of PartsFactory takes three arguments, a config , and the names of the classes to be used for Part, and Parts.
This factory knows the structure of the config array (expects name, description and need_parts to be on certain places)
Usage:
```ruby
road_parts = PartsFactory.build(road_config)
```
PartsFactory , combined with the new configuration arrays, isolates all the knowledge needed to create a valid Parts .
### Leveraging the PartsFactory
You can now replace the whole Part class with a simple OpenStruct.
(Struct takes position order initialization arguments while OpenStruct takes a hash for its initialization and then derives attributes from the hash)
PartsFactory must be solely responsible for manufacturing Parts.
PartsFactory returns a Parts that contains an array of OpenStruct objects, each of which plays the Part role.

## The Composed Bicycle
Bicycle has-a Parts , which in turn has-a collection of Part objects. The object think of Parts and Part as roles. The role of Part is played by an OpenStruct.
Now it’s very easy to create a new kind of bike with creating new config.

### Aggregation: A Special Kind of Composition
Delegation is when one object receives a message and merely forwards it to another - it creates dependencies.
A composed object is made up of parts with which it expects to interact via well-defined interfaces (involves delegation).
Composition describes a has-a relationship. The composed object depends on the interface of the role.
Usually composition is just a has-a relationship between 2 models - but it also means that contained object has no life without it's container

Aggregation is exactly like composition except that the contained object has an independent life.
E.g. The university–department relationship is one of composition (in its strictest sense) and the department–professor relationship is aggregation.

## Deciding Between Inheritance and Composition
Inheritance: for the cost of arranging objects in a hierarchy, you get message delegation for free
Composition: structural independence, but at the cost of explicit message delegation (objects stand alone)

Use composition if it's possible, inheritance is a better solution when its use provides high rewards for low risk.
## Accepting the Consequences of Inheritance
Hierarchies are exemplary; by their nature they provide guidance for writing the code to extend them.
Classical inheritance’s greatest strength and biggest weakness: subclasses are bound, irrevocably and by design, to the classes above them in the hierarchy.

## Accepting the Consequences of Composition
When using composition, the natural tendency is to create many small objects that contain straightforward responsibilities that are accessible through clearly defined interfaces.
Composed objects do not depend on the structure of the class hierarchy, and they delegate their own messages, they are immune from suffering side effects as a
result of changes to classes above it in the hierarchy.
At its best, composition results in applications built of simple, pluggable objects that are easy to extend and have a high tolerance for change.
Cost:
The composed object must explicitly know which messages to delegate and to whom, identical delegation code may be needed by many different objects there is no way to share this code.

## Choosing Relationships
Cites:
“Inheritance is specialization.”
“Inheritance is best suited to adding functionally to existing classes when you will use most of the old code and add relatively small amounts of new code.”
“Use composition when the behavior is more than the sum of its parts.”

### Use Inheritance for is-a Relationships
### Use Duck Types for behaves-like-a Relationships
(roles like schedulable, preparable, printable, or persistable)
### Use Composition for has-a Relationships

# Chapter 9. Designing Cost-Effective Tests
## Intentional Testing
Tests reduce bugs and provide documentation, and most important is that they improves application design.
Intentions of testing:
- finding bugs
- supplying documentation
- deferring design decisions
- supporting abstractions
- exposing design flaws 
  if testing is hard, design is bad
  write loosely coupled tests about only the things that matter
### Knowing what to test
Writing too many tests leads to giving up testing.
Write fewer tests, deal with each object as a black box. Rely on stable things in your tests.
What to test:
Incoming messages should betested for the state they return. Outgoing command messages should be tested to ensure they get sent. Outgoing query messages should not be tested.
### Knowing When to Test
Write tests first, whenever it makes sense to do so.
Novice programmers gain more benefits from writing tests first then experienced ones.
### Knowing How to Test
Mainstream test frameworks: Minitest (bundled with Ruby), Rspec.
Testing styles:
TDD - inside-out approach, usually starting with tests of domain objects and then reusing these newly created domain objects in the tests of adjacent layers of code.
BDD - outside-in approach, creating objects at the boundary of an application and working its way inward, mocking as necessary to supply as-yet-unwritten objects.

## Testing Incoming Messages
### Deleting Unused Interfaces
Unused code costs more to keep than to recover - so delete it!
### Proving the Public Interface
Prove that it returns the correct value in every possible situation.
If an application is constructed of tightly coupled, problems appear:
you will need to create a large network of dependent objects to test a certain object and any of them might break in a maddeningly confusing way.
### Isolating the Object Under Test
When you can’t test class in isolation, it bodes ill for the future.
Freeing from an attachment to the class of the incoming object opens design and testing possibilities that are otherwise unavailable (rely on role, not specific class)
### Injecting Dependencies Using Classes
If you inject a class, the code is concrete, it's simple but only works for a specific case.
It only one class acts as role, you can inject class, but if there are lots of such classes and they are costly, you need other solution.
### Injecting Dependencies as Roles
The whole point of dependency injection is that it allows you to substitute
different concrete classes without changing existing code (roles are more stable than concrete classes)
If all objects playing role are expensive, you may want to fake a cheap one to make your tests run faster.
#### Creating Test Doubles
```ruby
# Create a player of the ‘Diameterizable’ role
class DiameterDouble
  def diameter
    10
  end
end
```
A test double is a stylized instance of a role player that is used exclusively for testing.
DiameterDouble is not a mock.
#### Living the Dream
If the interface changes, it'll break the app, but tests will still pass cause the double still implements old method.
When the interface of a role changes, all players of the role must adopt the new interface.
But it's easy to overlook doubles.
#### Using Tests to Document Roles
This problem occurs because the role is nearly invisible.
One way to raise the role’s visibility is to assert that the object plays it (check if object responds to certain method of the interface, that is implemented in the double).
But this solution is incomplete: it cannot be shared with other role players and it doesn't prevent doubles from becoming obsolete.

Roles need tests on their own. Replacing coupling with an injected dependency on role isolate the object under test but create a dilemma about whether to inject a real or a fake object.
Injecting the "real" objects is costly, injecting doubles can speed tests but leave them vulnerable to constructing a fantasy world.
The act of testing did not, by itself, force an improvement in design. Reducing the coupling is up to you and relies on your understanding of the principles of design.

## Testing Private Methods
In the pristine world of idealized design they need not be tested, but in the real world you may need tests.
### Ignoring Private Methods During Tests
Reasons to omit private method testing:
- private methods are invoked by public methods that already have tests (testing is redundant)
- private methods are unstable
- encourages users to break encapsulation (tests should hide private methods, not expose them).
### Removing Private Methods from the Class Under Test
An object with many private methods probably has too many responsibilities.
You can extrat some methods into new object with it's own public interface and test its' methods.
But these methods may be unstable and it's costly to couple to unstable methods.
### Choosing to Test a Private Method
Only test private methods if it makes sense.
E.g. you defer a design decision and write some messy code and end up having private methods that are wildly unstable, it’s reasonable to compound your sins by testing these unstable methods.

## Testing Outgoing Messages
Outgoing messages are either queries or commands.
Queries matter only to the object that sends them.
Commands have effects that are visible to other objects in your application.
### Ignoring Query Messages
### Proving Command Messages
Sometimes it does matter that a message get sent; other parts of your application depend on something that happens as a result.
You shouldn't duplicate tests by checking the message returned, just need to check that the message is sent.
You need a mock. Mocks are tests of behavior, as opposed to tests of state.
test:
- tells the mock what message to expect
- triggers the behavior that should cause this expectation to be met
- asks the mock to verify that it indeed was
#TODO - see rspec code
The mock can be configured to return an appropriate value. They are meant to prove messages get sent, they return results only when necessary to get tests to run.
In a well-designed application, testing outgoing messages is simple.
## Testing Duck Types
### Testing Roles
Your tests should document the existence of the role, prove that each of its players behaves correctly, and show that other objects interact with them appropriately.
The role’s test should be written once and shared by every player (include it as a module in every role player test)
The shared module serves as a test and as documentation.
- role players responds to method
- message is sent correctly (you can use a mock)
### Using Role Tests to Validate Doubles
Solving "Living in a dream" problem:
Extract checking that object responds_to a certain message into a module of its own.
You can use the module to prevent test doubles from silently becoming obsolete.
```ruby
module DiameterizableInterfaceTest
  def test_implements_the_diameterizable_interface
    assert_respond_to(@object, :width)
  end
end
```
When you treat test doubles as you would any other role player and test them to prove their correctness, you avoid test brittleness and can stub without fear of consequence.

## Testing Inherited Code
### Specifying the Inherited Interface
The first goal of testing is to prove that all objects in the hierarchy follow the Liskov Substitution Principle (subtypes should be substitutable for their supertypes).
Write a shared test (BicycleInterfaceTest) for the common contract and include this test in every object (objects should respond to methods of a common interface).
### Specifying Subclass Responsibilities
#### Confirming Subclass Behavior
The abstract Bicycle superclass imposes requirements upon its subclasses, they should share a common test to prove that each meets the requirements (BicycleSubclassTest)
(test requeired methods like hooks, thay may be implemented in the subclass or inhereted from parent, just to check that subclass does nothing too crazy)
These classes allow novices to create new subclasses safely, they can just include these tests when they write new subclasses.
#### Confirming Superclass Enforcement
Checking that NotImplementedError is raised when one of the methods that should be implemented in subclasses is missing.
This behavior is in parent class (Bicycle), so we test it in parent class test (BicycleTest).
Notice that an instance of this abstract class is created in this test (it may need providing some "fake" arguments to make it work).
### Testing Unique Behavior
Tests specializations supplied by individual subclasses.
#### Testing Concrete Subclass Behavior
It’s important to test these specializations without embedding knowledge of the superclass into the test.
#### Testing Abstract Superclass Behavior
Creating an instance of the superclass (Bicycle) is not only hard but the instance might not have all the behavior you need to make the test run.
Because superclass used template methods to acquire concrete specializations you can stub the behavior that would normally be supplied by subclasses.
You can even manufacture a testable instance of superclass by creating a new subclass for use solely by this test.
```ruby
class StubbedBike < Bicycle
  def default_tire_size
    0
  end
  def local_spares
    {saddle: 'painful'}
  end
end
```
It remains convenient to sometimes create an instance of the abstract superclass directly, even though this requires passing the
Creating a subclass to supply stubs can be helpful in many situations.
If you fear that StubbedBike will become obsolete include subclass test to it.























