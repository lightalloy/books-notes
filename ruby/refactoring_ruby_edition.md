# Refactoring. Ruby Edition

# Chapter 1. Refactoring, a First Example
## The First Step in Refactoring
Build the solid tests for this section of the code.
The rhythm of refactoring: test, small change, test.

# Chapter 2. Principles in Refactoring
## Defining Refactoring
2 definitions:
1) A change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior.
2) To restructure software by applying a series of refactorings without changing its observable behavior.

### The Two Hats
2 distinct activities: adding function and refactoring.
When you adding function, you shouldn't be changing any existing code.
When you refactor, you shouldn't add any function.

## Why should you refactor?
- improves the design
- makes software easier to understand
- helps to find bugs
- faster programming

## When should you refactor?
Don't aside time for refactoring, it's something you do all the time in little bursts.
### The Rule of Three
Fist time: do it.
Second time doing similar: duplicate.
Third time: refactor.

### Refactor when you add function
Helps to understand the code you need to modify.
Refactoring is smth you do all the time in little bursts.

- to help understanding the code.
- change the design so that adding a new feature would be easier

### Refactor when you need to fix a bug
- refactor to help improve your understanding

### Refactor as you do a code review
When you're refactoring while reviewing someone's code, it helps you to understand the code better.
It also helps the codereview have more concrete results.
Best group: reveiwer and the author of the code.
The reviewer suggest changes, they both decide if the changes can be easily refactored, then make the changes if needed (pair programming)

### Refactoring for better understanding
When a developer starts working on existing project, it may be worth to spend couple of days on refactoring (to the same thing).
That'll be a great benefit in the future.

## Why refactoring works
Refactoring is the process of taking a program and adding to it value without changing its behaviour.
The value is added by adding qualities to help us work easier and faster in the future.

## What do I tell my manager?
If the manager is technically savvy, it'll be easy to introduce the subject.
If not, don't tell.

## Indirection and refactoring
Indirection is a two-edged sword.
Refactoring tends to break one thing into 2 pieces, so there it's more to manage, indirection increases.
Indirectios pays for itself:
- sharing logic
- explain intention and implementation separately
- isolate change
- encode conditional logic

If you create careful up-front design, it's too easy to guess wrong.
With refactoring you are never in danger of being completely wrong.

When you find parasitic indirection (not paying off), take it out.

## Problems with Refactoring
### Changing Interfaces
You can change the implementation without changing the interface safely.
But many refactorings change an interface.
E.g. simple Rename Method.
If the refactoring changes the published interface, keep the old one.
(The old interface calls the new one)
You should also create some deprecation facility.

Protecting interfaces is doable but hard, so don't publish interfaces unless you absolutely need it.

### Databases
Use migrations, temporary tables, tests for migrations.
You can add a layer between your object model and your database model.
Such layer adds complexity but gives you a lot of flexibility.
(book 'Refactoring Databases')

### Design Changes That Are Difficult to Refactor
Ask yourself how difficult would it be to refactor from one design to another.
If it's easy don't worry and pick the simplest design.
If not, put more effort into the design.

### When Shouldn’t You Refactor?
When it would be easier to rewrite from scratch.
E.g. when code doesn't work:
A compromise:
- refactor-versus-rebuild decision for one component at a time
- Extract Class and Move Method on coherent pieces of behavior, write tests for it

Another reason to avoid refactoring: being close to deadline.

Refactoring for academic purposes: disagreeing with implementation is not a good enough reason to refactor code.

## Refactoring and Design
Up-front design + construction coding vs refactoring.
“With design I can think very fast, but my thinking is full of little holes.” Alistar Cockburn
One argument: no up-front design, code the first approach, get it working, then form it into shape.
Alternative: some up-front design, then code and refactor.
You still think about potential changes and consider flexible solutions. But before implementing them think how easy would it be to refactor a simple solution to a flexible solution. Implement if it's easy.
Refactoring can lead to simpler designs without sacrificing flexibility and makes the process less stressful.

## It Takes A While to Create Nothing
Even if you know exactly what is going on in your system, measure performance; don’t speculate.

## Refactoring and Performance
To make the software easier to understand, you often make changes that will cause the program to run more slowly.
But refactoring also makes the software more amenable to performance tuning.
Write tunable software first and then to tune it for sufficient speed.
3 approaches for writing fast software:
- Decompose, each component must not exceed a budget. Used for hard real-time systems. Overkill for other types of systems.
- Constant attention approach. Every programmer tries to write only high performance code. If you do this, the time will be lost and there'll be a lack of clarity.
- Build a program in a well-factored manner w/o paying attention to performance at first. Then begin a (late) performance optimization stage. Run a profiler and focus on the performance hot spots.
After each change test and rerun the profiler, if the performande is not improved, undo the change.
(Code Complete book)

This approach slows the software in the short term while I’m refactoring, but it makes the software easier to tune during optimization.

## Optimizing a Payroll System
Before Jim provided a tool that measured the system in actual operation, we had good ideas about what was wrong.
The first thing that we should've been implemented was string creation, but not our ideas.
It was a long time before our good ideas were the ones that needed to be implemented.

# Chapter 3. Bad Smells in Code
Deciding when to start refactoring, and when to stop, is just as important to refactoring as knowing how to operate the mechanics of a refactoring.

We will give you indications that there is trouble that can be solved by a refactoring.
But you will have to develop your own sense.

Use this chapter to give you inspiration when you’re not sure what refactorings to do.
You may not find the exact smell, but it should point you the right direction.

## Duplicated Code
Same expressions in 2 methods in the same class: Extract method + invoke code.
Same expressions in 2 sibling classes: Extract Method + Pull Up method
Slightly different expressions: Form template method, Substitute Algorithm
Duplication in the middle of the method: Extract Surrounding method
2 unrelated classes: Extract Class or Extract Module

## Long Method
The longer a procedure is, the more difficult it is to understand.
99% - Extract Method

Replace Temp with Query or Replace Temp with Chain to eliminate the temps.
Parameter Object and Preserve Whole Object to eliminate long lists of parameters.

Still too many temps: Replace Method with Method Object
Comments signal that what it is doing can be replaced by a method whose name is based on the comment.

Conditionals and loops also give signs for extractions.
Conditionals: Decompose Conditional
Loops: Collection Closure Methods, consider using Extract Method on the call to the closure
method and the closure itself.

## Large Class
Too many instance variables.
Extract Class to bundle a number of the variables.
E.g. common prefixes or suffixes for the some subset of variables in a class can suggest the opportunity fo a component.

Options: Extract Subclass, Extract Module, Extract Class.
Useful trick: Extract Module for each of the uses, that may give you ideas about further refactoring.

## Long Parameter List
In object-oriented programs parameter lists tend to be much smaller than in traditional programs.
Long parameter lists are hard to understand, they become inconsistent and difficult to use.

Replace Parameter with Method - when when you can get the data in one parameter by making a request of an object.
Preserve Whole Object - to take a bunch of data gleaned from an object and replace it with the object itself.
Introduce Parameter Object, Introduce Named Parameter - clump several data items with no logical object together.

Exception: when you don't want to create a dependency from the called object to the larger object.
Sending a large parameter list may be reasonable, but pay attention to the pain involved.

## Divergent Change
When we make a change we want to be able to jump to a single clear point in the system and make the change.
Divergent Change happens when one class is commonly changed in different ways for different reasons.

Any change to handle a variation should change a single class or module, and all the typing in the new class/module should express the variation.
Use Extract Class to put them all together.

## Shotgun Surgery
The opposite of the divergent change.
Every time you have to make a kind of change, you have to make a lot of little changes to many different classes.
Hard to find, easy to miss.

Solutions: Move Method, Move Field to put all changes in a single class. Use existing or create a new class.
Often you can create an Inline Class, deal with a small dose of Divergent Change.

## Feature Envy
A classic smell is a method that seems more interested in a class other than the one it actually is in.
The most common focus of the envy is the data.
Solution: Move Method, Extract Method + Move Method.

Often a method uses features of several classes. Then determine which class has most of the data and put the method with that data.

Several sophisticated patterns break this rule: e.g. Strategy and Visitor (Gang of Four), Self-Delegation (Kent Beck). Data and the behavior that references that data usually change together, but there arePrimitive Obsession exceptions.

## Data Clumps
The same three or four data items together in many places.
Ought to be made into their own object.
1) look where clumps appear as instance variables.
Use Extract Class to turn them into an object.

2) Look for clumps as method signatures.
Use Introduce Parameter Object or Preserve Whole Object to slim them down.

A test: If you delete one of data values, would the others make any sense? If they don’t, it’s a sure sign that you should create a new object.

## Primitive Obsession
Using String or Fixnums instead of special object (e.g. Money)

Solutions:
Replace Data Value with Object
Conditionals that depend on a type code => use Replace Type Code with Polymorphism, Replace Type Code with Module Extension, or Replace Type Code with State/Strategy.
A group of instance variables that should go together => use Extract Class.
Primitives in parameter lists => Introduce Parameter Object.
Picking apart an array => use Replace Array with Object

## Case Statements
The same case statement scattered about a program in different places, to add a new clause you need to modify all of them.
Consider polymorphism. Often the case statement matches on a type code.
You need the method or class that hosts the type code value.
- Extract Method + Move Method
- Replace Type Code with Polymorphism, Replace Type Code with Module Extension, or Replace Type Code with State/Strategy

If you only have a few cases that affect a single method, and you don’t expect them to change, then polymorphism is overkill.
Use Replace Parameter with Explicit Methods.
If one of your conditional cases is a null, try Introduce Null Object.

## Parallel Inheritance Hierarchies
A special case of shotgun surgery. Every time you make a subclass of one class, you also have to make a sub-
class of another.
Make sure that instances of one hierarchy refer to instances of the other.
Use Move Method and Move Field and the hierarchy on the referring class will disappear.

## Lazy Class
The class that doesn't do enough.
Use Collapse Hierarchy, Inline Class or Inline Module.

## Speculative Generality
"We need the ability to do this kind of thing someday” and create different hooks and special cases that are not required.
The result is harder to understand and maintain.
If it's not used, move it away.

Solutions:
Lazy Clases => Collapse Hierarhy
Unnecessary delegation => Inline Class
Methods with unused parameters => Remove Parameters
Methods named with odd names => Rename Method

Can be spotted when the only use of the class are only test cases, except the helpers for the test cases.

## Temporary Field
An object in which an instance variable is set only in certain circumstances.
It's difficult to understand.
Use Extract Class to create a home for such variables, put there all code, which refers to that variables.
You may use Introduce Null Object to create an alternative component for when the variables aren’t valid.

For complicated algorithm that needs several variables:
Create a Method Object: Extract Class with these variables and the methods that require them

## Message Chains
A client asks one object for another object, which the client then asks for yet another object, which the client then asks for yet another object, and so on.
The client is coupled to the structure of the navigation.
Solutions:
Hide Delegate, but doing this often turns every intermediate object to a middle man.
Better alternative:
See what the result are used for.
Extract Method, then Move Method to push it down the change.
If several clients of one of the objects in the chain want to navigate the rest of the way, add a method to do that.

## Middle Man
Delegating messages can go too far. E.g. you look at a class’s interface and find half the methods are delegating to this other class.

Use Remove Middle Man and talk directly to the object.
If only a few methods aren’t doing much, use Inline Method to inline them into the caller.
If there is additional behavior, use Replace Delegation with Hierarchy to turn the real object into a module and include it in the middle man.

## Inappropriate Intimacy
Classes that spend too much time delving in each others private parts.
Use Move Method and Move Field to separate the pieces to reduce the intimacy.
Maybe you can Change Bidirectional Association to Unidirectional.
If the classes do have common interests, use Extract Class or use Hide Delegate to let another class act as go-between.
Inheritance often can lead to over-intimacy.
Apply Replace Inheritance with Delegation if needed.

## Alternative Classes with Different Interfaces
Use Rename Method on any methods that do the same thing but have different signatures for what they do.
Keep using Move Method to move behavior to the classes until the protocols are the same.
Then you may be able to use Extract Module or Introduce Inheritance.

## Incomplete Library Class
Reuse is overrated (we just use).
Building libraries is a tough job cause the developers are not omniscient.
Move Method can be used to move method to a library class.

## Data Class
Classes that have nothing else but attributes.
Use Remove Setting Method on any instance variable that should not be changed.
Apply Encapsulate Collection for collection instance variables.
Try to use Move Method to move behavior into the data class or Extract Method.
After a while you can start using Hide Method on the getters and setters.
Data Classes are ok at starting point, but later they need to take some responsibility.

## Refused Bequest
The Subclasses inherit the methods and data from their parents that they don't need.
This usually means that the hierarchy is wrong.
Create a new sibling class and use Push Down Method to push all the unused methods to the sibling, so that the parent contains only the common.
That's not a strong smell. but it's much stronger if the subclass is reusing behavior but does not want to support the public methods of the superclass. Refusing implementatons is not so bad as refusing public methods.
In this case Replace Inheritance with Delegation.

## Comments
Comments are a sweet smell, like a deodorant.
When we remove actual bad smells with refactoring, the comments are no longer needed. 
Try Extract Method, Rename Method.
If you need to state some rules, use Introduce Assertion.

A good time to use a comment is when you don’t know what to do. While describing what's going on, you'll indicate areas in which you are not sure.

## Metaprogramming Madness
Some metaprogramming techniques can result in obfuscated code.
Using method_missing hook oftern results in code that is difficult to understand.
This hook can be useful, but if it's not absolutely needed use Replace Dynamic Receptor with Dynamic Method Definition or even a simple Extract Method to remove the method_missing definition.
If the method_missing definition is truly needed, you might use Isolate Dynamic Receptor to separate concerns.

## Disjointed API
Too much flexibilty with many configuration options. Often the same configuration is used over and over again.

Introduce Gateway to interact with the API in a simplified way.
Introduce Expression Builder can be applied to both internal and external APIs to interact with the public interface in a more fluent manner.

## Repetitive Boilerplate
The easiest solution: Extract Method and call it from multiple places.
Some cases are so common that they need to go further. E.g. attr_reader in ruby.

Introduce Class Annotation - annotate a class by calling a class method from the class definition in the same way that attr_reader is called.

# Chapter 4. Building Tests
If you want to refactor, the essential precondition is having solid tests

## The Value of Self-Testing Code
Life is much easier if you have a test suit to lean on.
Since those days, it’s become a standard to follow the Red/Green/Refactor movement.
Write a failing test, make it pass, refactor.
This process is done many times a day, at least once with each new feature added.
You can complete large refactorings with confidence.
The time spent debugging is also decreased.

## Adding More Tests
Your tests will not find every bug, but they can find bugs.
When you refactor, you'll add more tests and understand the code better.
You don't always have to test all combinations.
Don’t let the fear that testing can’t catch all bugs stop you from writing the tests that will catch most bugs.

# Chapter 6. Composing Methods
Too long methods cause problems: lots of information, complex logic.
Extract Method: biggest problem - local variables.
Use Replace Temp with Query for to get rid of any temporary variables that you can.
If the temp is used in many places, use Split Temporary Variable to make the temp easier to replace.
If the temp is tangled and is hard to remove use Replace Method with Method Object.

Inline Method (the opposite of Extract Method)

Parameters are less of a problem than temps, you can use Remove Assignments to Parameters if needed.

After breaking the method down you can use Substitute Algorithm to introduce the clearer algorithm.

Introduce Named Parameter to improve the fluency of code. If not needed, remove it with Remove Named Parameter.

Remove Unused Default Parameter if a default parameter becomes unused.

## Extract Method
Short methods are better, because they are more understandable and more likely to be reused.
Pay attention to the naming.
The key is not the length, but semantic distance between the name and the method body.

Comments often identify pieces of a method that can be extracted.
The comment itself can be a potential name for the extracted method.

What if more than 1 variable needs to be returned?
Parallel assignment can be used, but should be avoided.
Try to do multiple extractions that return single values.

## Inline Method
A method’s body is just as clear as its name, needless indirection is irritating.
Put the method’s body into the body of its callers and remove the method.
Or you have a group of badly factored methods.
Inline them in 1 large method and re-extract them.

## Inline Temp
You have a temp that is assigned to once with a simple expression, and the temp is getting in the way of other refactorings.
Most of the time Inline Temp is used as part of Replace Temp with Query.

## Replace Temp with Query
By replacing the temp with a query method, any method in the class can get at the information.
Replace Temp with Query often is a vital step before Extract Method.
Local variables make code difficult to extract.
You may need to use Split Temporary Variable or Separate Query from Modifier first to make things easier for complex cases.
- mark the method as private
- ensure the extracted method is free of side effects

Temp are often used in loops.
Sometimes you can use Decomposing and Redistributing the Statement Method.
When this is the case, duplicate the loop for each temp so that you can replace each temp with a query.
The performance won't matter in 9 of 10 cases.

## Replace Temp with Chain
Change the methods to support chaining, thus removing the need for a temp.
```ruby
mock = Mock.new
expectation = mock.expects(:a_method_name)
expectation.with("arguments")
expectation.returns([1, :array])
```
=>
```ruby
mock = Mock.new
mock.expects(:a_method_name).with("arguments").returns([1, :array])
```

It provides a more fluent interface and improves maintainability.
It's different from Hide Delegate, cause it involves only one object.

You'll need to return self from methods that you want to allow chaining from.

```ruby
select = Select.with_option(1999).and(2000).and(2001).and(2002)
```

## Introduce Explaining Variable
If you have a complicated expression: put the result of the expression, or parts of the expression, in a temporary variable with a name that explains the purpose.

Extraneous temporary variable can clutter the code and distract the reader.
When you're tempted to introduce a temporary variable, think about another options.
Use Extract Method if possible.
But sometimes it's difficult to do it, so you can use Introduce Explaining Variable.

## Split Temporary Variable
You have a temporary variable assigned to more than once, but it is not a loop variable nor a collecting temporary variable.
Make a separate temporary variable for each assignment.

## Remove Assignments to Parameters
The code assigns to a parameter.
Use a temporary variable instead.

Assigning to parameters leads to lack of clarity and to confusion between pass by value and pass by reference.
Ruby uses pass by value exclusively, those who have used pass by reference will probably find this confusing.

Create a temporary variable for the parameter, replace all the references.

## Replace Method with Method Object
You have a long method that uses local variables in such a way that you cannot apply Extract Method.
Turn the method into its own object so that all the local variables become instance variables on that object.
You can then decompose the method into other methods on the same object.

Small methods, extract pieces => more comprehensible code.

- create a class
- add attributes for each temp variable and each parameter
- a constructor that takes the source object and each parameter
- create a #compute method
- copy the body of the original method into compute
- replace the old method with the new code that creates the new object and calls compute

- (further refactoring)

## Substitute Algorithm
You want to replace an algorithm with one that is clearer.
Replace the body of the method with the new algorithm.

Replace the complicated way with the clearer way.

## Replace Loop with Collection Closure Method
You are processing the elements of a collection in a loop.
Replace the loop with a collection closure method.

Examples:
loop => .select, .each, .map

If you need to do something in a loop that produces a single value, such as a sum, consider using the inject method.
```ruby
total = employees.inject(0) {|sum, e| sum + e.salary}
```

## Extract Surrounding Method
Two methods that contain nearly identical code, difference in the middle.

Extract the duplication into a method that accepts a block and yields back to the caller to execute the unique code.

```ruby
def connect
  ...
  connection = CreditCardServer.connect(...)
  yield connection
  ...
end

def charge(amount, credit_card_number)
  connect do |connection|
    connection.send(amount, credit_card_number)
  end
end
```

## Introduce Class Annotation
You have a method whose implementation steps are so common that they can safely be hidden away.
Declare the behavior by calling a class method from the class definition.

```ruby
class SearchCriteria
  hash_initializer :author_id, :publisher_id, :isbn

  def self.hash_initializer
    ...
  end
  ...
end
```
When the purpose of the code can be captured clearly in a declarative statement, Introduce Class Annotation can clarify the intention of your code.

Consider using Extract Module on the class method.

## Introduce Named Parameter
Convert the parameter list into a Hash, and use the keys of the Hash as names for the parameters.
Improves readability, useful for optional parameters.

## Remove Named Paramete
If the fluency recieved is no longer worth the complexity, convert the named parameter Hash to a standard parameter list.

## Remove Unused Default Parameter
A parameter has a default value, but the method is never called without the parameter => remove the default parameter.

## Dynamic Method Definition
You have methods that can be defined more concisely if defined dynamically.
```ruby
[:failure, :error, :success].each do |method|
  define_method method do
    self.state = method
  end
end
```

or implement #def_each

### Defining Instance Methods with a Class Annotation
```ruby
class Post
  def self.states(*args)
    args.each do |arg|
    define_method arg do
      self.state = arg
    end
  end
  states :failure, success, :error
end
```
### Defining Methods By Extending a Dynamically Defined Module
Ruby allows to define anonymous modules.
```ruby
class Hash
  def to_module
    hash = self
      Module.new do
        hash.each_pair do |key, value|
          define_method key do
            value
          end
        end
      end
    end
  end
end

class PostData
  def initialize(post_data)
    self.extend post_data.to_module
  end
end
```

## Replace Dynamic Receptor with Dynamic Method Definition
You have methods you want to handle dynamically without the pain of debugging method_missing.
Debugging classes that use #method_missing can be painful - you often get a NoMethodError or even stack level too deep.
Sometimes you can avoid the use of #method_missing.

### Dynamic Delegation Without method_missing
The problems can be avoided by using the available data to dynamically define methods at runtime using #class_eval.

### Using User-Defined Data to Define Methods

Checking if attribute is empty.

```ruby
class Person
  def self.attrs_with_empty_predicate(*args)
    attr_accessor *args
    args.each do |attribute|
      define_method "empty_#{attribute}?" do
        self.send(attribute).nil?
      end
    end
  end

  attrs_with_empty_predicate :name, :age
end
```

## Isolate Dynamic Receptor
A class utilizing method_missing has become painful to alter.
=>
Introduce a new class and move the method_missing logic to that class.

If it’s possible to know all valid method calls ahead of time, prefer Replace Dynamic Receptor with Dynamic Method Definition.
If not, use method_missing.

- create a new class whose sole responsibility is to handle the dynamic method calls
- copy the #method_missing body
- create a method on the original class to return an instance of the focused class
- change the calls to the dynamic method
- remove the #method_missing from original object

## Move Eval from Runtime to Parse Time
You want to limit the number of times eval is necessary.
Move the use of eval from within the method definition to defining the method itself.

In the cases where eval is necessary, it’s often better to move an eval call from runtime to parse time.
- expand the scope of the string being eval’d.

Evaluating the entire method definition allows you to change the define_method to def, it'll be faster.

# Chapter 7. Moving Features Between Objects
Deciding where to put responsibilities is hard to get right the first time.
Use Move Field and Move Method to change it.

If classes become bloated with too many responsibilities, use Extract Class or Extract Module to separate responsibilities.

Irresponsible class => use Inline Class to merge it into another class.

If another class is being used, it often is helpful to hide this fact with Hide Delegate.

Sometimes hiding the delegate class results in constantly changing the owner’s interface, in this case use Remove Middle Man.

## Move Method
A method is, or will be, using or used by more features of another class than the class on which it is defined.
Move the method, change the old method to use delegation or remote it.

If it's hard to choose, look at other methods. Moving the other methods can make the desicion easier.
If it's still hard to choose, that probably doesn't matter much.

## Move Field
A field is, or will be, used by another class more than the class on which it is defined.
Create a new attribute reader (and if necessary, a writer) in the target class, and change all its users.

Consider moving a field if more methods on another class using the information in the field than the class itself.

### Using Self-Encapsulation
```ruby
attr_accessor :interest_rate
```
=>
account_type - AccountType instance
```ruby
def interest_rate
  @account_type.interest_rate
end
```
### extend Forwardable
```ruby
extend Forwardable
def_delegator :@account_type, :interest_rate, :interest_rate=
```

## Extract Class
You have one class doing work that should be done by two.
You add a responsibility to a class feeling that it’s not worth a separate class, as the responsibility grows and breeds, the class becomes too complicated.

A good sign is that a subset of the data and a subset of the methods seem to go together.
Other sign: subset of data change together and are dependent of each other.

- create a new class to express the split-off responsibilities
- rename the old class if needed
- make a link from the old to the new class
- use Move Field for the needed fields
- use Move Method
- review and reduce interfaces

You can hide it by providing delegating methods for its interface, or I can expose it.
If expose, consider dangers of aliasing.

Options:
- ccept that any object may change any part of the attribute, consider Change Value to Reference
- make the attr value immutable
- clone and then freeze the attr value before passing it

Concurrent programs: extracting allows you to have separate locks on the two resulting classes.
But that's a danger if you have to ensure that both objects are locked together - it's a more complex topic.

## Inline Class
If a class isn’t doing very much, move all its features to another class and delete it.
Reverse of Extract Class, fold the class into another class.

## Hide Delegate
A client is calling a delegate class of an object, create the methods to hide delegate.
Create a simple delegating method:
```ruby
def manager
  @department.manager
end
```
Or extend Forwardable and declare the delegating method:
```ruby
extend Forwardable
def_delegator :@department, :manager
```
Forwardable is a module included within the Ruby Standard Library.

## Remove Middle Man
A class is doing too much simple delegation
=>
Get the client to call the delegate directly.

There are advantages of delegation, but there is also a price for it:
Every time the client wants to use a new feature of the delegate, you have to add a simple delegating method to the server.
It becomes painful, the server class is just a middle man and it's time to call the delegate directly.

# Chapter 8. Organizing Data
Simple data value => object
Get rid of magic number

## Self Encapsulate Field
Use attr_reader instead of accessing the instance variables directly.

Two schools of thought: access via getters/setters vs direct access to the variables.
indirect variable access => flexibility
direct variable access => code is easier to read (?)

Use direct access from the constructor or a separate initialization method.

## Replace Data Value with Object
Simple data items become not simple (e.g. phone number)

- create a class for the value
- change the attribute reader in the source class to call the reader in the new class

```ruby
def customer
  @customer.name
end
```
- in the constructor assign the field using the constructor of the new class
```ruby
def initialize(customer)
  @customer = Customer.new(customer)
end
```
- change the attribute writer to create a new instance of the new class
```ruby
def customer=(value)
  @customer = Customer.new(value)
end
```
- you may now need to use Change Value to Reference on the new object

Further refactoring:
- use Rename Method:
```ruby
def customer_name
  @customer.name
end
```
- change the constructor and the writer to use customer_name

Further refactoring may cause to add a new constructor and attribute writer that take an existing customer.
But you'll need to apply Change Value to Reference to the customer so that all orders for the same customer share the same customer object.

## Change Value to Reference
You have a class with many equal instances that you want to replace with a single object.
E.g. Orders belong to Customer

Value objects are things like date or money, they are defined entirely through their data values.
You don’t mind that copies exist.

The decision between reference and value is not always clear.
Start: a simple value with a small amount of immutable data,
Next: it needs some changeable data and the changes should affect all referring objects.
At this point you'll need to switch to the reference.

- use Replace Constructor with Factory Method

```ruby
class Customer
  def self.create(name)
    Customer.new(name)
  end
end
```

replace the calls to constructor with calls to the factory:
```ruby
class Order
  def initialize(customer_name)
    @customer = Customer.create(customer_name)
  end
end
```
- decide what object is responsible for providing access to the objects
  a hash or a registry object
  may be more than 1 object
- decide whether the objects are precreated or created on the fly
  if precreated, make sure they are loaded before they are needed
- alter the factory method to return the reference object
  decide how to handle errors if someone asks for and object tha doesn't exist
- may want to use Rename Method on the factory

```ruby
class Customer
  Instances = {}

  def self.load_customers
    new('blah').store
    ...
  end

  def store
    Instances[name] = self
  end

  def self.with_name(name)
    instances[name]
  end
end

class Order
  def initialize(customer_name)
    @customer = Customer.with_name(customer_name)
  end
end
```

## Change Reference to Value
You have a reference object that is small, immutable, and awkward to manage.

Reference objects have to be controlled, you need to ask for appropriate object.
The memory links can also become awkward.

Value objects are useful for the distributed and concurrent systems.
They should be immutable, it it's true, there is no problem having many objects represent the same thing.

If the value is mutable, you have to ensure that changing any object also updates all the other objects that represent the same thing, that's a lot of pain. The easiest way is to create a reference object.

Immutable: to change value you must replace the existing value object with a new value object.

- if the object isn’t currently immutable, use Remove Setting Method until it is.
- if the candidate cannot become immutable, you should abandon this refactoring.
- create an == method and an eql? method
- create a hash method
- consider removing any factory method and making a constructor public

Before refactoring:
```ruby
class Currency
  attr_reader :code
  def initialize(code)
    @code = code
  end
  def self.get(code)
    @code = code
  end
end
```
Object is already immutable.
```ruby
class Currency
  attr_reader :code

  def initialize(code)
    @code = code
  end

  def eql?(other)
    self == other
  end

  def ==(other)
    other.equal?(self) ||
    (other.instance_of?(self.class) && other.code == code)
  end
end

Currency.send(:new, "USD") == Currency.new("USD") # returns true
Currency.send(:new, "USD").eql?(Currency.new("USD")) # returns true
```

TODO:
If I define eql? , I also need to define hash.

## Replace Array with Object
You have an Array in which certain elements mean different things.
Replace the Array with an object that has a field for each element.

```ruby
row = []
row[0] = "Liverpool"
row[1] = "15"
```
=>
```ruby
row = Performance.new
row.name = "Liverpool"
row.wins = "15"
```
- create a new class to represent the information in the Array
- create methods [](index) and []=(index, value)
- construct the new object wherever the Array was instantiated
- add attr_readers, change the clients
- add attr_writers, change the clients
- remove the [] and []=

### Refactor with Deprecation
If you are developing a lib, consumed by others, ou may want to deprecate methods instead of removing them.
You can create Module#deprecate method and deprecate a method of any class (deprecate :method_name)

## Replace Hash with Object
You have a Hash that stores several different types of objects, and is passed around and used for more than one purpose.
Replace the Hash with an object that has a field for each key.

```ruby
new_network = { nodes: [node], old_networks: [node.network], name: some_code.name }
```
=>
```ruby
new_network = NetworkResult.new
new_network.nodes = [node]
#... etc
```
Almost same as in Replace Array with Object.

The true benefit comes when you can move behavior onto the newly created object.

## Change Unidirectional Association to Bidirectional
You have two classes that need to use each other’s features, but there is only a one-way link.
Add back pointers, and change modifiers to update both sets.

- add a field for the back pointer
- decide which class will control the association
  one to many: object that has a one reference is the controller
  if one object is a component of the other, the composite should control the association
  many to many => doesn't matter
- create a helper method on the noncontrolling side (restricted)
- create a modifier on the controlling side
- modifier on the controlled side calls the modifier on the controlling side

```ruby
class Customer
  def friends_orders
    @orders
  end

  def add_order(order)
    order.customer = self
  end
end

class Order
  attr_reader :customer
  def customer=(value)
    customer.friend_orders.subtract(self) unless customer.nil?
    @customer = value
    customer.friend_orders.add(self) unless customer.nil?
  end
end
```
Many to many:

Order: #add_customer, #remove_customer
Customer: #add_order, #remove_order+-

## Change Bidirectional Association to Unidirectional
You have a two-way association but one class no longer needs features from the other.
Drop the unneeded end of the association.

Bidirectional associations are useful, but they add complexity of maintaining links and objects creation/removing.
They also force an interdependency between the two classes.
It may lead to a highly coupled system.

- examine all the readers of the pointer that you want to remove to find out if the removal is feasible
  look at direct readers and methods that call those methods
  find out if it's possible to determine the other object without using the pointer:
  consider using the attr_reader without a pointer or adding and object as an argument

- attr_reader => Self Encapsulate Field, Substitute Algorithm on the attribute reader
- attr_reader not needed => change each user of the field so that it gets the object in the field another way

- remove all updates to the field, and remove the field

## Replace Magic Number with Symbolic Constant
You have a literal number with a particular meaning.
Create a constant, name it after the meaning, and replace the number with it.
```
GRAVITATIONAL_CONSTANT = 9.81
```
Magic numbers are not obvious. When you use it in many places it's a nighmare to change and maintain.
Before doing this refactoring consider other options.
Sometimes you can use Replace Type Code with Polymorphism or just array.size (if applicable).

## Encapsulate Collection
A method returns a collection.
Make it return a copy of the collection and provide add/remove methods.

Person: #courses, #courses=()
=>
Person: #courses, #add_course(), #remve_course()

At this point I’ve encapsulated the collection. 
No one can change the elements of the collection except through methods on the Person

## Replace Record with Data Class
You need to interface with a record structure in a traditional programming environment (legacy app, traditional API)
- сreate a class to represent the record
- give the class a field with an attribute_accessor for each data item
- you have a dumb data object, further refactoring will add the behaviour

## Replace Type Code with Polymorphism
You have a type code that affects the behavior of a class.
Replace the type code with classes: one for each type code variant.

This situation can be indicated by the presence of case -like conditional statements or if/else constructs.

### Removing Conditional Logic
3 refactorings:
- Replace Type Code with Polymorphism
  The simplest, takes advantage of Ruby Duck typing
  Use if the methods that use the type code make up a large portion of the class,
  Removes the original class and creates new classes for each type.
- Replace Type Code with Module Extension
  Extend a module, mixing in the module’s behavior onto the object.
- Replace Type Code with State/Strate
  Uses delegation: The parent object delegates to the state object for state-specific behavior.

The great thing about Ruby is that you can do Replace Type Code with Polymorphism without inheritance or implementing an interface.

- create a class to represent each type code variant
- change the class that uses the type code into a module. Include the module into each of the new type classes
- change the callers of the original class to create an instance of the desired type instead
- override one of the methods on one of the type classes
- -||- on the other type classes, removing the methods on the module
- repeat for the other methods
- remove the module

## Replace Type Code with Module Extension
You have a type code that affects the behavior of a class.
Replace the type code with dynamic module extension.

By extending a module, we can change the behavior of an object at runtime.
This removes some headache with traditional state/strategy pattern.

But modules can't be unmixed easily, the behavior is hard to remove.
Use Replace Type Code with Module Extension when you don’t care about removing behavior.
If you do care, use Replace Type Code with State/Strategy.

- perform self-encapsulate Field on the type code
- create a module for each type code variant
- make the type code writer extend the type module appropriately
- choose one of the methods, override the method on one of the type modules
- do the same to other modules
- repeat for other methods
- pass the module into the type code setter instead of the old type code

```ruby
class MountainBike
  attr_reader :type_code

  def type_code=(value)
    @type_code = value
    case type_code
      when :front_suspension: extend(FrontSuspensionMountainBike)
      when :full_suspension: extend(FullSuspensionMountainBike)
    end
  end
end

bike = MountainBike.new(type_code: :rigid)
```
I can remove the case statement I created by getting the callers to pass in the appropriate module.
```ruby
class MountainBike
  def type_code=(mod)
    extend(mod)
  end
end

bike = MountainBike.new
bike.type_code = FrontSuspensionMountainBike
```

## Replace Type Code with State/Strategy
You have a type code that affects the behavior of a class and the type code changes at runtime.
Replace the type code with a state object.

Use Replace Type Code with State/Strategy when the type code is changed at runtime and the type changes are complex enough that I can’t get away with Module Extension.

- perform Self-encapsulate Field on the type code
- create empty classes for each of the polymorphic objects. Create instance variables to represent the type.
- use the old type code to determine which of the new type classes should be assigned to the type instance variable.
- take one of the polymorphic methods. Add a method with the same name on one of the new type classes and delegate to it from the parent object.
- repeat for all other types
- repeat for all polymorphic methods

```ruby
class MountainBike...
  extend Forwardable
  def_delegators :@bike_type, :off_road_ability, :price

  def initialize(bike_type)
    @bike_type = bike_type
  end

  def add_front_suspension(params)
    @bike_type = FrontSuspensionMountainBike.new({
            :tire_width => @bike_type.tire_width,
            :base_price => @bike_type.base_price,
            :commission => @bike_type.commission
            }.merge(params))
  end

  def add_rear_suspension(params)
    unless @bike_type.is_a?(FrontSuspensionMountainBike)
      raise "You can't add rear suspension unless you have front suspension"
    end
    @bike_type = FullSuspensionMountainBike.new({...}).merge(params)
  end


end

class RigidMountainBike
  ...
  def off_road_ability
    @tire_width * MountainBike::TIRE_WIDTH_FACTOR
  end

  def price
    (1 + @commission) * @base_price
  end
end
class FrontSuspensionMountainBike
  ...
  def off_road_ability
    @tire_width * MountainBike::TIRE_WIDTH_FACTOR + @front_fork_travel *
    MountainBike::FRONT_SUSPENSION_FACTOR
  end
  def price
    (1 + @commission) * @base_price + @front_suspension_price
  end
end
class FullSuspensionMountainBike
  ...
  def off_road_ability
    @tire_width * MountainBike::TIRE_WIDTH_FACTOR +
    @front_fork_travel * MountainBike::FRONT_SUSPENSION_FACTOR +
    @rear_fork_travel * MountainBike::REAR_SUSPENSION_FACTOR
  end

  def price
    (1 + @commission) * @base_price + @front_suspension_price +
    @rear_suspension_price
  end
end

bike = MountainBike.new(FrontSuspensionMountainBike.new(
                        :type => :front_suspension,
                        :tire_width => @tire_width,
                        :front_fork_travel => @front_fork_travel,
                        :front_suspension_price => @front_suspension_price,
                        :base_price => @base_price,
                        :commission => @commission))


```
Rather than reach into the type object when we’re upgrading, we can use Extract Method to encapsulate the upgradable parameters.
```ruby
class MountainBike...
  def add_front_suspension(params)
    @bike_type = FrontSuspensionMountainBike.new(@bike_type.upgradable_parameters.merge(params))
  end
  def add_rear_suspension(params)
    unless @bike_type.is_a?(FrontSuspensionMountainBike)
      raise "You can't add rear suspension unless you have front suspension"
    end
    @bike_type = FullSuspensionMountainBike.new(@bike_type.upgradable_parameters.merge(params))
  end
end
```
## Replace Subclass with Fields
You have subclasses that vary only in methods that return constant data.
Change the methods to superclass fields and eliminate the subclasses

Before: Person parent class and 2 child classes (Female, Male), that contain methods #femail? and #code
After:
```ruby
class Person
  def initialize(female, code)
    @female = female
    @code = code
  end

  def female?
    @female
  end

  def self.create_female
    Person.new(true, 'F')
  end

  def self.create_female
    Person.new(false, 'M')
  end
end
```

## Lazily Initialized Attribute
Initialize an attribute on access instead of in the constructor.

```ruby
class Employee
  def initialize
    @emails = []
  end
end
```
=>
```ruby
class Employee
  def emails
    @emails ||= []
  end
end
```
Refactoring for readability purposes.

### Example Using instance_variable_defined?
||= is a common idiom but it fails when nil or false are valid values for the attribute.
Then you may use instance_variable_defined? :

```ruby
class Employee...
  def assistant
    unless instance_variable_defined? :@assistant
      @assistant = Employee.find_by_boss_id(self.id)
    end
    @assistant
  end
end
```

## Eagerly Initialized Attribute
Initialize an attribute at construction time instead of on the first access.
Opposite of Lazily Initialized Attribute

Lazily initialized attributes can be problematic to debug because their values change upon access.
Eager initialization leads to encapsulating all initialization logic in the constructor.

Eager vs Lazy is discussable.

# Chapter 9. Simplifying Conditional Expressions
Conditional logic can get tricky, here is a set of refactorings to simplify it.

## Decompose Conditional
You have a complicated conditional (if-then-else) statement.
Extract methods from the condition, “then” part, and “else” parts.
- extract the condition into its own method
- extract the “then” part and the “else” part into their own methods.

See if you can Replace Nested Conditional with Guard Clauses.
If not - Decompose Conditional.

## Recompose Conditional
You have conditional code that is unnecessarily verbose and does not use the most readable Ruby construct.
```ruby
parameters = params ? params : []
# =>
parameters = params || []
```
### Example: Replace Conditional with Explicit Return
```ruby
def reward_points
  return 2 if days_rented > 2
  1
end
```
## Consolidate Conditional Expression
You have a sequence of conditional tests with the same result.
Combine them into a single conditional expression and extract it.

```ruby
return 0 if @seniority < 2
return 0 if @months_disabled > 12
return 0 if @is_part_time
# =>
return 0 if ineligable_for_disability?
```
## Consolidate Duplicate Conditional Fragments
The same fragment of code is in all branches of a conditional expression.
Move it outside the expression.

## Remove Control Flag
You have a variable that is acting as a control flag for a series of boolean expressions.
Use a break or return instead

## Replace Nested Conditional with Guard Clauses
A method has conditional behavior that does not make clear the normal path of execution.
Use guard clauses for all the special cases.

```ruby
def pay_amount
  return dead_amount if @dead
  return separated_amount if @separated
  return retired_amount if @retired
  normal_pay_amount
end
```

Conditional expressions are of one of 2 forms:
1) if and else legs are equal => use if-else
2) one leg is normal behavior, the other is unusual situation.
=> use guard clause

The guard clause either returns, or throws an exception.

## Replace Conditional with Polymorphism
You have a conditional that chooses different behavior depending on the type of an object.
Move each leg of the conditional to a method in an object that can be called polymorphically.

```ruby
def price
  case @type_code
    when :rigid
      (1 + @commission) * @base_price
    when :front_suspension
      (1 + @commission) * @base_price + @front_suspension_price
    when :full_suspension
      (1 + @commission) * @base_price + @front_suspension_price +
    @rear_suspension_price
  end
end
# =>
RigidMountainBike#price
FrontSuspension#price
FullSuspension#price
```
Ruby’s duck typing makes it easy to introduce polymorphism.
In languages like Java you would need implementing interface or using inheritance.

Simplest: implement the same method signature on multiple objects and call these methods polymorphically.
Or: introduce a module hierarchy and have the method that is to be called polymorphically on the module.
Or: introduce an inheritance hierarchy and have the method that is to be called polymorphically on the subclasses.

## Introduce Null Object
You have repeated checks for a nil value.
Replace the nil value with a null object.

```ruby
plan = customer ? customer.plan : BillingPlan.basic
```
=>
Customer#plan + NullCustomer#plan

Most common use - for display information.

Ruby allows us two main options for implementing the null object:
1) traditional: create a subclass of the source class
the null object will respond to all the messages that the source class responds to.
But there may be bugs when the default behaviour of the parent class is not desired.
2) create the new class and define only methods that you need.

You can also implement a message-eating null object.
But that may be dangerous.

3) Add methods to NilClass itself
But you shouldn't do it in most cases.

You should find a meaningful name for your null object. E.g. UnknownCustomer or MissingPerson.

### Other Special Cases
There may be other special cases (like MissingCustomer and Unknown Customer), then you need to create several classes.

## Introduce Assertion
A section of code assumes something about the state of the program.
Make the assumption explicit with an assertion.

```ruby
def expense_limit
  assert { (@expense_limit != NULL_EXPENSE) || (!@primary_project.nil?) }
  (@expense_limit != NULL_EXPENSE) ? \
  @expense_limit : \
  @primary_project.member_expense_limit
end
```
Sometimes the section of code works only if some of conditions are true.
Such assumptions are not always obvious - you need to look through the algorythm.
An assertion is a conditional statement that is assumed to be always true, failure will cause raising an exception.
Assertions act as communication and debugging aids.

Beware of overusing assertions. Use them only if things absolutely need to be true.
Always ask whether the code still works if an assertion fails. If the code does work, remove the assertion.
Beware of duplicate code in assertions. Use Extract Method if needed.

Assertions should be easily removable.

# Chapter 10. Making Method Calls Simpler
This chapter explores refactorings that make interfaces more straightforward.

## Rename Method
First copy the old body of code over to the new name.
Change the body of the old method so that it calls the new one.
Change all the references to refer to the new one.

## Add Parameter
## Remove Parameter
## Separate Query from Modifier
You have a method that returns a value and also changes the state of an object.
Create two methods, one for the query and one for the modification.

It is a good idea to clearly signal the difference between methods with side effects and those without.
That means observable side effects.
E.g. cache the value of a query in a field so that repeated calls go perform better changes state of the object, but these changes are not observable.

### Concurrency Issues
For multithreadal systems it is still valuable to have separate query and modifier operations, but you need to retain a third method that does
both.

## Parameterize Method
Several methods do similar things but with different values contained in the method body.
Create one method that uses a parameter for the different values.

## Replace Parameter with Explicit Methods
You have a method that runs different code depending on the values of an enu- merated parameter.
Create a separate method for each value of the parameter.

The reverse of Parameterize Method.

Switch.turn_on is a lot clearer than Switch.set_state(true)

You shouldn’t use Replace Parameter with Explicit Methods when the parameter values are likely to change a lot.

## Preserve Whole Object
You are getting several values from an object and passing these values as parameters in a method call.
```ruby
low = days_temperature_range.low
high = days_temperature_range.high
plan.within_range?(low, high)
# =>
plan.within_range?(days_temperature_range)
```

A down-side: Passing the object means dependency on that object, not just on the values.
Also consider Move Method as an alternative.

You may not already have the whole object defined. In this case you need Introduce Parameter Object.

## Replace Parameter with Method
An object invokes a method, then passes the result as a parameter for a method.
The receiver can also invoke this method.

## Introduce Parameter Object
You have a group of parameters that naturally go together (a data clump)
Replace them with an object.

- Create a new class to represent the group of parameters you are replacing.
Make the class immutable.
- Add Parameter Object to the end of methods' parameters list, provide the default value
- Remove old parameters one by one and replace references to them with references to the parameter object
- Remove the default for the parameter object

You can also get more value from this refactoring by moving behavior from other methods to the new object.

## Remove Setting Method
A field should be set at creation time and never altered.
Remove any setting method for that field - e.g. remove attr_writer or explicit setter.

## Hide Method
A method is not used by any other class.
Make the method private.

## Replace Constructor with Factory Method
You want to do more than simple construction when you create an object.
Replace the constructor with a factory method.

```ruby
# TODO: format
class ProductController...
  def create
    ...
    @product = Product.create(base_price, imported)
    ...
  end
  class Product
    def self.create(base_price, imported=false)
      if imported
        ImportedProduct.new(base_price)
      else
        if base_price > 1000
          LuxuryProduct.new(base_price)
        else
        sProduct.new(base_price)
    end
  end
end
```

Use when you have conditional logic to determine the kind of object to create.

## Replace Error Code with Exception
A method returns a special code to indicate an error.
Raise an exception instead.

```ruby
def withdraw(amount)
  raise BalanceError.new if amount > @balance
  @balance -= amount
end
```

## Replace Exception with Test
You are raising an exception on a condition the caller could have checked first.
Change the caller to make the test first.

```ruby
def execute(command)
  command.prepare rescue nil
  command.execute
end
# =>
def execute(command)
  command.prepare if command.respond_to? :prepare
  command.execute
end
```

## Introduce Gateway
You want to interact with a complex API of an external system or resource in a simplified way.
Introduce a Gateway that encapsulates access to an external system or resource.

The majority of Rails applications use ActiveRecord as a Gateway to a relational database.

- introduce a Gateway that uses the underlying API
- change one use of the API to use the Gateway instead
- change others

```ruby
class Gateway
  # ...
  def self.save
    gateway = self.new
    yield gateway
    gateway.execute
  end

  def execute
    Net::HTTP.new(url.host, url.port).start do |http|
      http.request(build_request)
    end
  end
end
```
We can then use Introduce Expression Builder to interact with the Gateway in a more fluent manner.

## Introduce Expression Builders
You want to interact with a public interface in a more fluent manner and not muddy the interface of an existing object.
Introduce an Expression Builder and create an interface specific to your sapplication.

An Expression Builder provides a fluent interface as a separate layer on top of the regular API.
Create a class whose sole responsibility is to provide our desired fluent interface.

# Chapter 11. Dealing with Generalization
## Pull Up Method
You have methods with identical results on subclasses.
Move them to the superclass.

## Push Down Method
Behavior on a superclass is relevant only for some of its subclasses.
Move it to those subclasses.

## Extract Module
## Inline Module
The resultant indirection of the included module is no longer worth the duplication it is preventing.
Merge the module into the including class.

## Extract Subclass
A class has features that are used only in some instances.
Create a subclass for that subset of features.

If you need conditional logic to determine which type to create, consider using Replace Constructor with Factory Method.
One by one use Push Down Method to move features onto the subclass.

## Introduce Inheritance
You have two classes with similar features.
Make one of the classes a superclass and move the common features to the superclass.

An alternative to Introduce Inheritance is Extract Class (or Extract Module): the choice between inheritance and delegation.

- Choose one of the classes to be a superclass.
- One by one, use Pull Up Method

## Collapse Hierarchy
A superclass and subclass (or module and the class that includes the module) are not very different.
Merge them together, 1 class instead of 2.

## Form Template Method
You have two methods in subclasses that perform similar steps in the same order, yet the steps are different.
Get the steps into methods with the same signature, so that the original methods become the same. Then you can pull them up.

- decompose the methods so that the extracted methods are either identical or completely different
- pull up the identical methods into the superclass or to the class that extends the modules
- rename methods so that the signatures for all the methods at each step are the same

### Template method with Inheritance

```ruby
class Statement
...
def value(customer)
  result = header_string(customer)
  customer.rentals.each do |rental|
    result << each_rental_string(rental)
  end
  result << footer_string(customer)
end

HtmlStatement < Statement
header string(customer)
each rental string(rental)
footer string(customer)

TextStatement < Statement
header string(customer)
each rental string(rental)
footer string(customer)
```

### Template method with Extension of Modules

```ruby
class Customer
  def statement
    Statement.new.extend(TextStatement).value(self)
  end
  def html_statement
    Statement.new.extend(HtmlStatement).value(self)
  end
end
```

## Replace Inheritance with Delegation
A subclass uses only part of a superclass interface or does not want to inherit data.
Create a field for the superclass, adjust methods to delegate to the superclass, and remove the subclassing.

- Create a field in the subclass that refers to an instance of the superclass.
- Change each method defined in the subclass to use the delegate field
- Remove the subclass declaration and replace the delegate assignment with an assignment to a new object.
- For each superclass method used by a client, add a simple delegating method.

Instead of inheriting from hash:

```ruby
require 'forwardable’
class Policy
  extend Forwardable

  def_delegators :@rules, :size, :empty?, :[]

  def initialize(name)
    @name = name
    @rules = {}
  end
end
```

## Replace Delegation with Hierarchy
You’re using delegation and are often writing many simple delegations for the entire interface.
Make the delegate a module and include it into the delegating class.

Opposite of the prev. chapter.

Beware of it when the delegate is shared by more than one object and is mutable.

## Replace Abstract Superclass with Module
You have an inheritance hierarchy, but never intend to explicitly instantiate an instance of the superclass.
Replace the superclass with a module to better communicate your intention.

In Java you can designate class as abstract to prevent it to be instantiated explicitly.
In Ruby you can't do this. You could write some code to raise exception in the constructor.
But it's better to replace the abstract superclass with a module.

- if the superclass has some class methods, define and inherited hook
- Make the class a module, and replace the inheritance declaration with an include in each of the base classes
- Make the inherited hook an included hook.

Inherited hook:
```ruby
class Join
  def self.inherited(klass) # => included
    klass.class_eval do
      def self.joins_for_table(table_name)
        table_name.to_s
      end
    end
  end
end
```

# Chapter 11. Big Refactorings
## The Nature of the Game
These refactorings take a long time and are not finished at once.
The whole team has to recognize that one of the big refactorings is “in play” and make their moves accordingly.

## Why Big Refactorings Are Important
By refactoring you can ensure that your full understanding of how the program should be designed is always reflected in the program.
Many small individual actions will be enough to eradicate the problem of accumulation of half-understood design decisions.

## Four Big Refactorings
## Tease Apart Inheritance
You have an inheritance hierarchy that is doing two jobs at once.
Create two hierarchies and use delegation to invoke one from the other.

Tangled inheritance is a problem because it leads to code duplication.
It makes changes more difficult, the code is hard to understand.

If every class at a certain level in the hierarchy has subclasses that begin with the same adjective, you probably are doing two jobs with one hierarchy.

1) Identify the jobs being done by the hierarchy
2) Choose one more important job, it will be retained in the current hierarchy.
3) Create an object for each of the subclasses in the original hierarchy (Extract Class at the common superclass)
4) Add an instance variable on the superclass to hold the new object. Initialize the instance variable to the appropriate new class.
5) Extract a module to keep the common code that will be shared between the new classes. Include this module in the new classes.
6) move the methods from the subclasses to the relevant extracted objects
7) pull up methods if possible

### Example
Deal
children: ActiveDeal, PassiveDeal
1) jobs: dealness, presentation
2) Dealness is more important, so we'll keep it in existing hierarchy and extract the presentation layer.
3) Single Active Presentation Style, Single Passive Presentation Style, Tabular Active Presentation Style, Tabular Passive Presentation Style
4-5) Extract Presentation Module to store common behaviour for these classes.
```ruby
class ActiveDeal
  def initialize
    ...
    @presentation = SingleActivePresentationStyle.new...
  end
end
```
6) Move the presentation-related methods and variables of the deal subclasses to the presentation style classes.
There should be no code left in the classes Tabular Active Deal and Tabular Passive Deal, so we remove them.

It's often possible to dramatically simplify the extracted classes and often further simplify the original object.

Get rid of the active-passive distinction in the presentation style. We're left with just Single Presentation Style and Active Presentation Style classes.

Presentation differences can be handled with a couple of variables, so we can keep only Presentation Style module.

## Convert Procedural Design to Objects
You have code written in a procedural style.
Turn the data records into objects, break up the behavior, and move the behavior to the objects.

- Take each record type and turn it into a dumb data object with accessors
- Take all the procedural code and put it into a single class.
- extract methods from long procedures, move methods to the appropriate data classes
- Continue until you’ve removed all the behavior away from the original class, delete the class if possible

## Separate Domain from Presentation
You have views and controller classes that contain domain logic.
Move the domain logic into the model.

Rails uses MVC, but that doesn't mean that logic is always in correct place.
Often domain logic is in controllers or views.
Controllers should only be responsible for accepting user requests, organizing for the model to do its work, and triggering the appropriate view to be displayed.
- identify functionality in the controllers that doesn't belong there
- determine a domain object to move this code or create new one
- move methods
- do the same for the views

## Extract Hierarchy
You have a class that is doing too much work, at least in part through many conditional statements.
Create a hierarchy of classes in which each subclass represents a special case.

It can take a long time to untangle a design.

1) If the variations can change during the life of the object, extract class to pull that aspect into a separate class.
2) Create a subclass for that special case and replace constructor with factory method on the original. Alter the factory method to return an instance of the subclass where appropriate.
3) copy methods that contain conditional logic to the subclass, simplify the methods so that they represent a certain case.
Extract Method in the superclass if necessary to isolate the conditional parts of methods from the unconditional parts.
4) Isolate special cases until all superclass methods have subclass implementations
5) Delete the methods in the superclass that are overridden in all subclasses.
6) Replace Abstract Superclass with Module if the superclass is not instantiated directly.

BillingScheme

=> extract Disability Billing Scheme

Identify cases in which we can have methods that have the same intention but carry it out differently in the two separate cases.

# Chapter 13. Putting It All Together

Knowing the techniques is just the beginning.
It's important to know when to apply them and when to stop/
You’ll know you’re getting it when you can stop with confidence.

You aren’t refactoring to pursue truth and beauty, you do it to make code easier to understand and regain the control on the program.

Stop when you are unsure.

Replay the actions if the tests are failing, take a step back.

Work with a partner.

When you decide to undertake a large refactoring, try to pick off pieces that can be integrated back into the main development branch as quickly as possible.

Never forget the two hats: The refactoring hat, and the new functionality hat: only wear one at a time.
