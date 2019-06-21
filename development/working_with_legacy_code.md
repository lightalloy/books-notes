# Working Effectively with Legacy Code

# The Mechanics of Change
## Chapter 1. Changing Software

Reasons to change:
- add a feature
- fix bug
- improve the design
- optimize resource usage

There is a big difference between adding new behavior and changing old behavior.
It seems nearly impossible to add behavior without changing it to some degree.

Refactoring - improving design w/o changing its behavior.
            - a series of small structural modifications, supported by tests to make the code easier to change.
Making code more maintainable.

Optimization - making changes, to improve time or memory consumptions.

3 things can change when we work on a system: structure, functionality, resource usage.

In all cases (bug fixes, feature added, refactoring, optimization) we want our preserve most of the existing behavior.

> Preserving existing behavior is one of the largest challenges in software development.

## Chapter 2. Working with Feedback
Strategies: Edit and Pray vs Cover and Modify

Traditionally: tests are developed after the code. The group of testers run the tests afterwards.
The feedback loop is large: software is developed for weeks, then passed to testers, they test, and only afterwards return to developers.
Regression testing is often done in the application interface.

Other scenario: having a number of unit tests, refactoring under a cover. Feedback loop is much shorter.
The next programmers who work on this piece of code will have an easier time.

### What Is Unit Testing?
What are units?
In procedural - usually functions, in OO - objects.
But it's hard to test in isolation.

Unit tests: run fast (instantanious), help localizing errors, help to get test coverage.

It's not a unit test if it talks to a database, accesses network, touches the file system or you have to tweak your config to run it.

### Higher-Level Testing
How do we start making changes to a legacy project?
- have tests

> Dependency is one of the most critical problems in software development.
> Much legacy code work involves breaking dependencies so that change can be easier.

In many cases adding tests w/o changing code is inpractical (unfortunately)

#### The Legacy Code Dilemma
> When we change code, we should have tests in place. To put tests in place, we often have to change code.

We can break the dependencies on `DBConnection` by introducing an interface `IDBConnection`.

> When you break dependencies in legacy code, you often have to suspend your sense of aesthetics a bit.

### The Legacy Code Change Algorithm
1. Identify change points.
2. Find test points.
3. Break dependencies.
4. Write tests.
5. Make changes and refactor.

## Chapter 3. Sensing and Separation
> In systems that weren’t developed concurrently with unit tests, we often have to break dependencies to get classes into a test harness, but that isn’t the only reason to break dependencies.

2 reasons to break dependencies:
- Sensing — to sense when we can’t access values our code computes.
- Separation - to separate when we can’t even get a piece of code into a test harness to run.

Example:
Class `NetworkBridge`, which accepts an array of `Endpoints`.
We could sniff packets across the network, get some hardware and set up a local cluster, but that would take a lot of time.
We can’t sense the effect of our calls to methods on this class, and we can’t run it separately from the rest of the application.

We can’t sense the effect of our calls to methods on this class, and we can’t run it separately from the rest of the application.

### Faking Collaborators
Fake Object - an object that impersonates some collaborator of your class when it is being tested.

Example:
`Sale` uses class `ArtR56Display` to display (deep in the code).
Both `ArtR56Display` and `FakeDisplay` implement `Display` (common interface, which has `showline` method)

A `Sale` object can accept a display through the constructor and hold on to it internally.

Fake Objects Support Real Tests, cause this way we "divide and conquer", test only what is needed.

non-OO languages => define an alternative function, which records values in some global data structure that we can access in tests.

#### Mock Objects
- fakes that perform assertions internally.
You can use mock object frameworks. But when they're not available, you can just create fake objects.

## Chapter 4. The Seam model
Existing code is often poorly suited for testing. The solution is to write tests as you develop or spend time to design for testability.

We are often told to design software to consist of small reusable peices.
But even when pieces of software look independent, they often depend upon each other in subtle ways.

### Seams
Pulling out classes from existing projects to test changes your perspective on what's a good design.

How do we call the global function (communicating with another subsystem) in production, but don't do it in test?

*Seam* - A seam is a place where you can alter behavior in your program without editing in that place.
*Object Seam* - subclassed the class for test, redefined function for the test.
We were able to test w/o changing the calling code.

Replace behavior at seams => selectively exclude dependencies in our tests.

### Seam Types
#### Preprocessing Seams
Program code is read by a compiler, the compiler emits object code or bytecode instructions.
Only a couple of languages have a build stage before compilation, C and C++
Can be used to overcome the testing ostacles when testing in C and C++.

we can introduce a header file called `localdefs.h` and within t provide a definition for `db_update` and some helpful variables.

*Enabling Point (of a Seam)* - a place where you can make the decision to use one behavior or another.

#### Link Seams
Linkers - resolve each of the calls so that you can have a complete program at runtime.

Create classes with the same names, put them into a different directory, and alter the classpath to link to the different files.

C and C++ (static linking) -- Often the easiest way to use the link seam is to create a separate library for any
classes or functions you want to replace.

You can write more code in you fake functions to sense, e.g. to record calls.
The enabling point for a link seam is always outside the program text, e.g. in the deployment script.

> If you use link seams, make sure that the difference between test and production environments is obvious.

#### Object Seams
Most useful in OO.

`cell.Recalculate();`
If we can change which `Recalculate` is called in that line of code without changing the code around it, that call is a seam.

Not all method calls are seams. If you're creating an instance of a certain class and then call the method, it's not a seam. There's no enabling point.
```java
Cell cell = new FormulaCell(this, "A1", "=A2+A3");
...
cell.Recalculate();
```

Passing cell as an argument:
```java
public Spreadsheet buildMartSheet(Cell cell) {
  ...
   cell.Recalculate();
  ...
}
```

The enabling point is the argument list of buildMartSheet.

Sometimes when testing the legacy code it makes sense to subclass and redefine a method for the test.
That's kind of indirect, but this way you won't have to modify the code to test it, which is safer.

Object seams are the best choice for OOP languages, preprocessing and link seams are useful at times, but they're not as explicit.
Reserve the link and the preprocessing seams for when there're no better alternatives.

## Chapter 5. Tools
### Automated Refactoring Tools
Be careful with automated refactoring - they sometimes change behavior even tough they were not intended to.
Have tests to be sure the behavior was kept the same.

### Test Harnesses
The most effective testing tools I’ve run across have been free.

Ui-based tests are fragile.

#### xUnit:
- tests written in the same language as code
- tests run independently
- grouped into suites that can be run on demand

#### jUnit
In junit you write tests by subclassing a class named TestCase.
junit uses reflection to find the methods of the test and creates a separate object for each one.
So the tests don't affect one another.
+ you can use `setUp` method to use prepare the test data.
`tearDown` - to clean after each test.

#### CppUnitLite
C++ lacks reflection

#### NUnit
For the .NET languages.

### General Test Harnesses
Frameworks for Integrated Tests
FIT and Fitnesse (acceptance testing framework)

# Part 2. Changing Software
## I Don’t Have Much Time and I Have to Change It
Often writing tests takes more time than implementing a change. Does it worth it?
> With tests around the code, nailing down functional problems is often easier.
(It'll be easier to find an error later, and find if there's and error)

In the best case: it'll start to recoup in the next iteration.
The worst case: we won't need to change it for years.

Typically, changes cluster in systems.
Change today => you'll probably need to change it soon again.

It's hard to decide whether to add tests or just add quick dirty changes to the code, if you're under time pressure.
Dilemma: pay now or pay more later.

### Sprout Method
- Identify where you need to make your code change
- Formulate a change as a sequence of statements (if possible), prepare a dummy call to the new method.
- Develop the sprout method using test-driven development

Disadvantages:
- you're giving up on the source method, may leave it in the odd state

Advantages:
- you clearly separate the new code

### Sprout Class
You need to make changes to a class, but you can't create objects of these class in a test harness.
You can create another class, put there your changes and call it from a source class.

Having several report generators, that implement the same interface.

2 cases that lead to a sprout class:
- adding entirely new responsibility to one of your classes
- we have to add a small bit of functionality that could be in the existing class, but we can't get class into a test harness

Disadvantage: sometimes adding a new concept is not a right thing and you only have to do that to put the code into a test harness.

### Wrap Method
Adding behavior to existing methods is easy, but it's also a nasty thing.
Putting actions together because they happen at the same time is not correct, cause later you'll need to do one thing w/o another.

- create a method with the name of the original method
- put the old logic in its own private method
- put the new logic in its own private method
- call these methods from the wrapping method

Downsides:
- the new feature that you add can’t be intertwined with the logic of the old feature.
- have to create a new method for the old code
- may lead to poor names

Advantage:
- you'll have a new tested functionality
- separated logic

### Wrap Class
Similar to wrap method, but put logic into a separate class.

Decorator pattern:
Need to use the current logic (of the method) + add some.
Subclass a class + add behavior.

- create an abstract class that defines the set of operations you need to support
- subclass inheriting from an abstract class, accepts an instance of the class in its constructor, and provides a body for each of those methods (`ToolControllerDecorator`)
- subclass a decorator and put the logic there + `super`

Use sparingly. The code that has decorators decorating decorators can become convoluted.

Or just create a class with the needed behavior and call it.

- identify a method to change
- create a class that accepts the class you are going to wrap as a constructor argument
- create a method on that class, which does the new work
- write another method that calls the new method and the old method on the wrapped class
- instantiate the wrapper class in your code in the place where you need to enable the new behavior

Wrap Class is another issue comparing with the Wrap/sprout method. Use it if the behavior you add is independent or if a class grows too large.

## Chapter 7. It Takes Forever to Make a Change
### Understanding
May take a long time in a legacy system.
It's easier to understand systems that are broken up into small, well-named, understandable pieces.

### Lag Time
The amount of time between a change and the moment you get real feedback about the change.

interpreted language => can get near-instantaneous feedback
compiled => sometimes there're dependencies that we don't want to compile just to be able to compile smth else

### Breaking Dependencies
OO => instantiate the classes you need in a test harness

### The Dependency Inversion Principle
> it is better to depend on interfaces or abstract classes than it is to depend on concrete classes.
Depend on less volatile things.

There is some conceptual overhead in having more interfaces and packages.
More interfaces and packages: build the package will go faster, but the entire system build time will go up slightly.
But the average time for a build based on what needs to be recompiled, can go down dramatically.

- High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces).
- Abstractions should not depend on details. Details (classes) should depend on abstractions.

## Chapter 8. How Do I Add a Feature?
### TDD
- write a failing test case
- get it to compile
- make it pass
- remove duplication
- repeat

Repeat for each case you need.
E.g. checking that you don't divide by zero.

Don't remove duplication too early, just make it compile and tests pass at first.
Just copy/paste and modify is fine. You'll refactor as a next step.

#### TDD + Legacy
One of the most important thing: focus on one thing at a time, you're either writing code or refactoring, not both at the same time.

The 0-th tdd step for the legacy code: Get the class you want to change under test.

### Programming by Difference
Old technique that fell out of favor in the 90s with the rise of OOP.
`MessageForwarder`.
Subclass and have `AnonymousMessageForwarder` which overrides `getFromAddress` to return anonymous address.
Easy to pass a test, but introducing inheritance for a small change not wise in the long-term.

If we put features into distinct subclasses, we can only have one of those features at a time.

We could add a feature by adding an option and changing the behavior by condition ()

```java
...
if (configuration.getProperty("anonymous").equals("true")) {
  from = getAnonymousFrom();
}
else {
  from = getFrom(Message);
}
...
```

But the code can become littered with the conditional statements.
You can also create a `Mailing-Configuration` class instead of just passing options.
You can add other methods to a `MailingConfiguration`, like `buildRecipientList` and its name to `MailingList`.

`RenameClass` refactoring changes the way people see code and lets them notice the possibilities they might not considered before.

Adding a feature options:
- subclass (can use only 1 at a time)
- pass options + use if statements (can litter the code, violate SRP)
- add a separate class (could be overkill)

### The Liskov Substitution Principle
Objects of subclasses should be substitutable for objects of their superclasses throughout our code.
Subclasses can be used where superclasses are used (without having to know that they are objects of a subclass).

- avoid overriding concrete methods
- if you do, see if you can call the method you are overriding in the overriding method

In general, code gets confusing when we override concrete methods too often.
Solution: subclass from abstract. E.g. 2 subclasses and and abstract superclass.

## Chapter 9. I Can’t Get This Class into a Test Harness
Problems:
- objects can't be created easily
- The test harness won’t easily build with the class in it
- The constructor we need to use has bad side effects
- Significant work happens in the constructor, and we need to sense it

### The Case of the Irritating Parameter
First, just try to instantiate a class in a test harness.
Construction test - just try to create, no assertions.

`RGHConnection` - an irritating parameter, requires a lot of setup.

The best way to fake is to extract interface.
Then create a class `FakeConnection` that implements `IRGHConnection`.

> Test code doesn’t have to live up to the same standards as production code.

Pass null is often a useful technique in Java.

#### Null Object Pattern
Use it instead of repetitive null checks. E.g. an object that returns nulls, empty stings and so on.

### The Case of the Hidden Dependency
The object is instantiated inside the code.
You could use `Parametrize Constructor` (add a parameter to a constructor).

In languages that allow default arguments, there is a simpler way of doing Parameterize Constructor. We can simply add a default argument to the existing constructor.

### The Case of the Construction Blob
A constructor construct a large number of objects internally or accesses a large number of globals.
That would lead to a very large parameter list.
Or a constructor accepts a few objects and then uses them to create other objects.
Possible refactorings:
- Extract and Override Factory Method (not possible in some languages - C++)
  + it's not a good idea in general
- Supersede Instance Variable (create a setter method to be able replace your dependency, e.g. a cursor)
  have to be very careful cause of destructors/deletion from memory
  be careful not to use supercede logic in production code
- Supersede Instance Variable (better avoid it when possible)
- Prefer Extract and Override Factory when possible

### The Case of the Irritating Global Dependency
Global dependencies are hardest to deal with.

We could parametrize the constuctor, but we have many occurences in the database.

We could use the Singleton Pattern. It's a mechanism people use to make global variables.
Singletons are hard to fake.
We can create a static method to replace an instance of the singleton.
You can also reinitialize singleton for each test (create a method that will set a singleton instance to `null`)

Why do we want only a single instance in the system (so use singleton)?
- there's only 1 such thing in the real world
- we could have serious problems if 2 of these things created (e.g. hardware, like nuclear control)
- If someone creates two of these things, we’ll be using too many resources -- often

The main reason people use singleton is that we don't want to pass a variable around many times.

team rule: everyone on the team should understand that we have one instance of the database in the application and that we shouldn’t have another.

If you have an app where thousands of objects need access to the database, you probably can separate responsibilites so that some object would store and retrieve data and others do other things.

In most cases, variables that are global are globally accessible, but they really aren’t globally used.

### The Case of the Horrible Include Dependencies
Java and C# have optimization: the compiler looks if an imported class is compiled already. If it's not, it compiles it, if it is, it only reads a brief snippet of info from the compiled file.

C++ doesn't have this optimization: the compiler has to reparse the declaration and build up an internal
representation every time it sees that declaration.

Create fakes and include them `#include “Fakes.h“`

Downside: you don't break dependencies (code doesn't become better), you have to create a separate program.
Use this technique only for severe cases.

### The Case of the Onion Parameter
Having to create objects to create objects to create objects to create a parameter for a constructor.
It's possible to Extract Interface or Extract Implementer on a most immediate dependency.

In any language where we can create interfaces or classes that act like inter-
faces, we can systematically use them to break dependencies.

### The Case of the Aliased Parameter
A class accepts an instance of an object that gets an associated value from its hierarchy.
You could extract an interface but that would require extracting interfaces for the whole hierarchy, that's a lot of work.

Another Strategy - Subclass and Override Method to create a `FakeOriginationPermit` for the test.

## Chapter 10. I Can’t Run This Method in a Test Harness

### The Case of the Hidden Method
Test a private method:
- test through a public method (preferred)

To test a private method we have to make it public. If making public bothers you, probably, your class does too much, consider refactoring. The method could be moved to a new class and be public there.

Possible solution: change to `protected` + subclass. Acceptable in some cases, but violates incapsulation.
In many OO languages you can use reflection to access private variables/methods. But that's a hack!

### The Case of the “Helpful” Language Feature
E.g. you can't instantiate `HttpPostedFile` that has no public constructor (C#).
+ the class is sealed (can't subclass)
It's done for the security reasons.

We can only use `Adapt Parameter`. Change method to accept `OurHttpFileCollection` instead of `HttpFileCollection`.
(they both inherit from non-sealed `NameObjectCollectionBase`).

What do we need from `Http-PostedFiles`? Properties `FileName` and `ContentLength`.
Extract interface `IHttpPostedFile` and write a wrapper `HttpPostedFileWrapper` that has those properties.
We can create `FakeHttpPostedFile` for testing.

The only annoyance is that we have to iterate the original `HttpFileCollection` in the production code, wrap each `HttpPostedFile` that it contains, and then add to a new collection that we pass to the method.

### The Case of the Undetectable Side Effect
The class does a lot: creates GUI components, receives notifications from a handler, calculates, displays.
Refactorings: extract methods to separate reponsibilities.

#### Command/Query Separation
A method should be a command or a query, but not both.
Command: modifies state, doesn't return a value.
Query: returns a value, doesn't modify an object.
----------------------

After those extractions, we can `Subclass and Override Method` and test the code that is left in the original method.
We can see the separated responsibilities and extract them to their own classes.

## Chapter 11. I Need to Make a Change. What Methods Should I Test?
Characterization Tests - a test that characterizes the actual behavior of a piece of code.
You write tests for the existing code.

What tests to write? You may write tests for each method to change, but when you work with a tangled legacy code you should first figure out what exactly to test.

### Reasoning About Effects
When a value is passed to a constructor by reference, it could be changed later, and it will change the result of the methods calls.

Effect changes, what affects what, they have a separate bubble for each variable that can be affected and each method whose return value can change.

Check that when a `CppClass` object is created, its declaration list and the list’s contents (argument) aren’t going to change.

It's often difficult to figure out why the results we are looking at are what they are.
And we have to ask ourselves another question: If we make a particular change how could it possibly affect the rest of the results of the program?

### Reasoning Forward
We look at a set of objects and try to figure out what will change downstream if they stop working.

Sketch effects, find all of the clients of the class you are examining.
You can probably sense the results by several methods, and write tests only for them.

We need to find places to test, and the first step is figuring out where change can be detected: what the effects of the change are. When we know where we can detect effects, we can pick and choose among them when we write our tests.

### Effect Propagation
Place where you make change - notice methods that return values. If the values are not being used, they propagate effects to
code that calls them.

Effect can propagate in sneaky ways. E.g. you can modify state somewhere and it affects the app.

### Tools for Effect Reasoning
Know your language, private/public attributes, what's mutable, what's not.

### Learning from Effect Analysis
Implicit/explicit rules to restrict effects.
Functional vs oop.

### Simplifying Effect Sketches
When we remove tiny pieces of duplication, we often end up getting effect sketches with a smaller set of endpoints.
E.g. accessing instance variables through getters.

Often dependency breaking techniques lead to breaking encapsulation.
E.g. you add a parameter, there'll be 1 more path to follow.
But adding tests will still make code easier to understand and reason about. Often it can help me get more encapsulation later.

Encapsulation is important because it makes it easier to reason about code.

## Chapter 12. I Need to Make Many Changes in One Area
Do I have to break dependencies to all the classes involved?

Higher-level - e.g. you write a test for a method that has several dependencies.
Higher-level tests are often useful while refactoring, but you shouldn't just replace unit tests with them.
Getting higher-level tests is the first step to having unit tests.

### Interception Points
- a point in your program where you can detect the effects of a particular change.

It is a good idea to pick interception points that are very close to your change points, because of safety + it's harder to set up tests for the interception points.

#### Higher-level interception points
In most cases, the best interception point we can have for a change is a public method on the class we’re changing: easy to find and easy to use.

*A pinch point* is a narrowing in an effect sketch, a place where tests against a couple of methods can detect changes in many methods.

### Judging Design with Pinch Points
A pinch point is a natural encapsulation boundary.
This is pretty much the definition of encapsulation: We don’t have to care about the internals, but when we do, we don’t have to look at the externals to understand them.
Writing tests at pinch points is an ideal way to start some invasive work in part of a program.

### Pinch Point Traps
- letting unit tests slowly grow into mini-integration tests

Tests at pinch points are kind of like walking several steps into a forest and drawing a line, saying “I own all of this area.”

## Chapter 13. I Need to Make a Change, but I Don’t Know What Tests to Write
Tha main goal of autotesing is preserving behaviour. You'll find bugs with the help of these tests later when changing code.

### Characterization Tests
Algorithm:
- use a piece of code in a test harness
- write a failing test
- see the output
- modify the test to be correct

A characterization test is a test that characterizes the actual behavior of a piece of code.
If you find a bug, mark this test as suspicious for further investigation.
Writing charachterization tests helps us to understand the code.
We can stop adding tests when we're confident with the code behaviour.

#### The Method Use Rule
Before you use a method in a legacy system, check to see if there are tests for it. If there aren’t, write them.

### Characterizing Classes
- search for a tangled piece of logic, write a test for it
- make a list that can go wrong
- write tests for invariants
- use sensing variable

Think about the reader: what info would be nice to have when working with this class?

#### When You Find Bugs
What to do? It depends. In most of the cases, fix bugs as they are found. But if the code is deployed, investigate how fixing will affect users and other behaviour.

### Targeted Testing
When you write a test for a branch, ask yourself whether there is any other way that the test could pass, aside from executing that branch.
Not sure => use a sensing variable or the debugger.
Many characterization tests look like “sunny day” tests: just test the behaviour precense w/o checking special conditions.

Solutions: calculate values manually, sensing variables to verify that a particular path is being covered and conversions are exercised, charachterize a smaller piece of code.

The most valuable characterization tests exercise a specific path and exercise each conversion along the path.

### A Heuristic for Writing Characterization Tests
- write tests to the area you're changing. Write as many tests as you need to understand the code
- take a look at the specific things you are going to change, and attempt to write tests for those
- When extracting/moving code: Verify that you are exercising the code that you are going to move and that it is connected properly. Exercise conversions.

## Chapter 14. Dependencies on Libraries Are Killing Me
Don't over-rely on the libraries.

Avoid littering direct calls to library classes in your code.
Every hard-coded use of a library class is a place where you could have had a seam.

Sometimes the best thing you can do is write a thin wrapper over the classes that you need to separate out.

Library designers sometimes forget that good code runs both in prod and test environments.

`once dilemma` - singletons.
`restricted override dilemma` - code with lots of non-virtual methods is harder to get under the test

> Sometimes using a coding convention is just as good as using a restrictive language feature. Think about what your tests need.
E.g. not override certain virtual methods (which are made virtual to get them under the test)

## Chapter 15. My Application Is All API Calls

Make the code clearer:
- the first step is to identify the computational core of code
- write out a list of the code’s responsibilities
- make a design that shows separate responsibilities

Nearly every system has some core logic that can be peeled away from API calls.

Options:
- Skin and Wrap the API
  - Good when api is small.
  - You want to separate out the dependency on the 3-rd party library
  - you are unable to test via API
- Responsibility-Based Extraction
  - api is complicated
  - you have refactoring tools

You can use both techniques.

## Chapter 16. I Don’t Understand the Code Well Enough to Change It

Techniques that help to understand the tangled code:
- sketching
- making the list of the method responsibilities and possible effects
- scratch refactoring (don't add tests, just move around)
- Delete Unused Code

## Chapter 17. My Application Has No Structure
Often feaures are developed in urgency, which leads to bad structure.
But nobody knows how much better it could be or how much money is being lost because of poor structure.
- it's hard or impossible to get the big picture
- the team is dealing with emergency after emergency

Traditional path: architect worked on the big picture, with the team.
The problem apprears when an architect doesn't communicate with the team often, and she has a completely different opinion from the rest of the team.

The architecture is too important to be left exclusively to a few people.

### Telling the Story of the System
2+ people talk: questions + simplified answers
You have to focus on the important things this way.

### Naked CRC
CRC - Class, Responsibility, and Collaborations.
Writing them on cards, move the responsibilities if you fell like it.

Explain a system by laying cards:
- Cards represent instances, not classes.
- Overlap cards to show a collection of them.

### Conversation Scrutiny
There's smth mesmerizing in large chunks of legacy code. It's tempting not to create abstractions, and so, overlook them.

## Chapter 18. My Test Code Is in the Way
### Class Naming Conventions
E.g.  `Test`, prefix/suffix, `Fake` prefix.
There could be a lot of variations, set your conventions.

## Test Location
It can make sense to separate tests from the deployed code/

## Chapter 19. My Project Is Not Object Oriented. How Do I Make Safe Changes?
Easy case - check the values of the variables.
Hard case - e.g. a function has a bad side effect (e.g. calls a 3-rd party service)

Option - use a link seam and create a fake alternative function for test.
This leads to a kind of messy code, but the procedural languages don't provide better solutions.

In C you can use macro preprocessor:

```c
#ifdef TESTING
#define ksr_notify(code,packet)
#endif
```

Another option - use file inclusion so that the tests and production code are in different files.

### Adding New Behaviour
Use TDD.
C supports function pointers.
We can create a struct that contains pointers to functions.
We can call the functions in a very natural object-oriented style:

```c
struct database{
  void (*retrieve)(struct record_id id);
  void (*update)(struct record_id id, struct record_set *record);
...
};
extern struct database db;
db.update(load->id, loan->record);
```

### Taking Advantage of Object Orientation
Many procedural languages now have a possibility to start using OOP (compiling C++ in C, extensions, etc)
Use `Encapsulate Global References`

All programs are object-oriented but some contain only 1 object.
If the procedural language you are using has an object-oriented successor, I recommend moving toward it.
Object seams are good for getting tests in place.

## Chapter 20. This Class Is Too Big and I Don’t Want It to Get Any Bigger

- hard to find out what you need to change
- if programmers concurrently work on the same class, it can lead to problems
- pain to test

Solutions: sprout class, sprout method.

### Single-Responsibility Principle (SRP)
The class should have 1 responsibility, and there should be 1 reason to change it.

In real world cases, the key is to identify the responsibilities and then figure out a way to incrementally move toward more focused responsibilities.

### Seeing Responsibilities
Method grouping.
The key thing is to be able to see responsibilities and learn how to separate them well.
> Legacy code offers far more possibilities for the application of design skill than new features do.

A set of heuristics to determine responsibilities (not iventing, but discovering them!)

#### Heuristic #1: Group Methods
- look at the similar method names
Write down methods with their access type, and try to group them.

Wait until you have to modify one of the methods you’ve categorized, and then decide whether you want to extract a class at that point.

#### Heuristic #2: Look at Hidden Methods
Pay attention to private and protected methods: a lot of them => may need to get out another class.

"How do I test private methods?"

#### Heuristic #3: Look for Decisions That Can Change
Look for decisions you already have made.

#### Heuristic #4: Look for Internal Relationships
Are certain instance variables used by some methods and not others?

There could be "lumps" of variables that tend to stay together.
Feature sketches - show which methods and instance variables each method in a class uses.
- draw circle for each of the variables
- draw circle for each method
- draw a line from each method circle to the circles for any instance variables and methods that it accesses or modifies.

Similar to effect sketches.
Based on the feature sketch, you can move clusters into another class (check if the new class has a distinct responsibility)

#### Heuristic #5: Look for the Primary Responsibility
Try to describe the responsibility of the class in a single sentence.
SRP can be violated on the interface and implementation level.

Violation at the implementation level:
Responsibilities can be delegated to other classes, the class acts like a facade.
It's easier to manage, but still it violates the SRP.

#### Interface Segregation Principle (ISP)
In a large class we can often see groupings of methods that particular clients use.
Create an interface for each of these groupings and let the class implement those interfaces.

When we have interfaces, we can start moving code from a big class for a new class?
Extract a new class, which implements the interface, and pass the prev class to the extracted one.
You'll need to change the calling code, but the original class will become smaller.

#### Heuristic #6: When All Else Fails, Do Some Scratch Refactoring
If you are having a lot of trouble seeing responsibilities in a class, do some scratch refactoring.

#### Heuristic #7: Focus on the Current Work
It is easy to become overwhelmed. Remember that the changes you currently are making are telling you about some particular way that the software can change.

### Other Techniques
Read books, more important - other people's code, opensource projects.

### Moving Forward
- Don't go on refactoring binge (in most cases)

Best option:
- identify the responsibilities
- make sure that everyone else on the team understands them
- break down the class on an as-needed basis.

### Tactics
SRP at the implementation level: extract classes, delegate to them.
Then introduce SRP in at the interface level.

Actions to Extract Class.

## Chapter 21. I’m Changing the Same Code All Over the Place
Removing duplication is an easier part of refactoring, and you can do it gradually.
But is it worth it? It depends.

### First Steps
> Abbreviations in class and method names are problematic.
(May be ok if used consistently)

E.g. we’ve removed all of this duplication. Has it made things better or worse?

If you want to change existing behavior in your code and there is exactly one place you have to go to make that change, you’ve got **orthogonality** (independence).
Removing duplication helps with it.

### Open/Closed Principle
Code should be open for extension but closed to modification.
When we remove duplication, our code often naturally starts to fall in line with the Open/Closed Principle.

## Chapter 22. I Need to Change a Monster Method and I Can’t Write Tests for It
Kinds of monster methods:
- bulleted methods - no indentation, list of code chunks
- snarled method - dominated by a single large, indented section

### Tackling Monsters with Automated Refactoring Support
When doing automated refactoring without tests, use the tool exclusively.
With the autorefactoring tools you can do a lot of coarse work safely and handle the details after you get other tests in place.

### The Manual Refactoring Challenge
Possible mistakes when extracting methods:
- forgetting to pass an argument
- naming method so that it hides/overrides another method
- mistakes when passing parameters or return values

### Techniques to help with refactoring
#### Introduce Sensing Variable
Sometimes code effects are indirect and they are hard to test.
It's worth to introduce a sensing variable just for the test, write a test using that sensing variable, and remove the variable and the test after a series of refactoring.

#### Extract What You Know
Start small and find little pieces of code that we can extract confidently without tests, and then add tests to cover them.
The coupling count - the number of values that pass into and out of the method you are extracting.
Pay attention to the coupling count, it should be small.
After you extract, write a few tests for the method you extracted.

#### Gleaning Dependencies
- write tests for the logic that you need to preserve
- extract things that the tests do not cover
Powerful when critical behavior is tangled with other behavior.

#### Break Out a Method Object
Create a class whose only responsibility is to do the work of your monster method.

### Strategy
Structural tradeoffs.

- Skeletonize Methods
Extracting conditional and body to the separate methods.
- Find Sequences
Extracting conditional and body together.

Controversial advice, apply in different situations.

- Extract to the Current Class First
Even if it leads to weird method names, extracting to the current class first is less prone to errors.
You'll be able to move it to another class later.

- Extract Small Pieces
- Be Prepared to Redo Extractions

After some time you’ll usually find better ways to accommodate new features more easily, you'll get an insight into old design and a better way of moving forward from the first extractions,

## Chapter 23. How Do I Know That I’m Not Breaking Anything?
Code is pretty fragile!

### Hyperaware Editing
Edit-triggered testing, tdd, pair programming, short feedback loop.

### Single-Goal Editing
Holding a lot in your head is useful, but doesn't lead to good decision-making.
When working on a feature/refactoring, do only on thing at a time. Write other things that come up into your mind and do it later.
Ask yourself (and a partner when pair-programmin) "What are we working on?" and do only that.

### Preserve Signatures

### Lean on the Compiler
1. Altering a declaration to cause compile errors
2. Navigating to those errors and making changes.

E.g. commenting out a method to find if it's used. But there are limits to this. E.g. a method can be defined in a superclass, so be aware of it.

### Pair Programming
Try it! A second set of eyes definitely helps when breaking dependencies.


## Chapter 24. We Feel Overwhelmed. It Isn’t Going to Get Any Better
The key to thriving in legacy code is finding what motivates you.

Sometimes rewriting from scratch seems like a good decesion, but it's hard to rewrite an evolving system.
The rewriting team has to do 2 jobs - rewriting + adding the new features that are added to the old system.

Things to try: Pick the ugliest most obnoxious set of classes in the project, and get them under test.
This is a way to start getting control over your codebase.

The grass isn’t really much greener in green-field development.

# Part 3. Dependency-Breaking Techniques
## Chapter 25. Dependency-Breaking Techniques
Use Adapt Parameter when you can’t use Extract Interface on a parameter’s class or when a parameter is difficult to fake.
E.g. a class accepts an `HttpServletRequest`.
You can use a mocking library and mock and `HttpServletRequest`.
Or we can wrap the parameter that is coming in and break our dependency on the API interface entirely.
Wrap inside the `ParameterSource`.
Fake - `class FakeParameterSource implements ParameterSource`
prod - `class ServletParameterSource implements ParameterSource`

We don't preserve signatures here! Use extra care!

> Move toward interfaces that communicate responsibilities rather than implementation details. This makes code easier to read and easier to maintain.
Code littered with wide interface and unused methods is harder to understand. Narrow interfaces help with understanding.

The goal is to get tests in place, nothing more. Safety first, you'll refactor later.

### Break Out Method Object
Main idea - move a long method to a new class.
The arguments of the constructor should be a reference to the original class, and the arguments to the original method.
Create a method that'll contain a body of the original method (often the method is called `call()`, `run()`, but can have another name)
Sometimes you'll need to extract interface to break the dependencies.

### Definition Completion
In some languages, we can declare a type in one place and define it in another (С, С++)
Include the header containing this class declaration in the test file and provide alternate definitions for the methods before our tests.

We can provide null bodies for methods that we don’t care about or put in sensing methods that can be used across all of our tests.
The downside - 2 different sets of definitions.

### Encapsulate Global References
- fake globals
- link to different things
- encapsulate global references

> If several globals are always used or are modified near each other, they belong in the same class.
So, you can create a new class that encapsulates those globals, and have them as public methods/fields.
While naming, think of the methods of the class.

Referencing a member of a class rather than a simple global is only the first step. Consider further refactoring.

It may seem like a small improvement, but it's an important one.
In places where you want to use fakes, use Introduce Static Setter, Parameterize Constructor, Parameterize Method or Replace Global Reference with Getter.

### Expose Static Method
If a method doesn’t use instance data or methods, turn it into a static method.
The static portions of a class can be seen as a “staging area” for things that don’t quite belong to the class.

- write a test that accesses the method that you want to expose as a public static method of the class.
- move the body to the static method
- make changes so the code runs correctly )
- further refactoring, e.g. move the method to another class

### Extract and Override Call
```java
public class PageLayout {
  protected void rebindStyles() {
    styles = StyleMaster.formStyles(template, id);
  ...
  }
}
```
A method calls a static method on another class => extract the call to a new method and override it in a testing subclass, This is known as Extract and Override Call.

### Extract and Override Factory Method
Hard-coded initialization in constructors makes testing hard.
Instead, instantiate a class in a method and call it in the constructor (but you can't do in this C or C++)
That'll be a factory method. Than subclass and override in inside the test.

### Extract and Override Getter
Workaround for C and C++ (instead of override factory method)
Introduce a getter for the instance variable that you want to replace with a fake object, use the getter in every place in a class.
Then subclass and override the getter to provide alternate objects under test.

a lazy getter - a function that creates the instance variable on the first call. Make sure that all of the code in the class is using the getter (not the variable before it's created)

Make sure that you delete the testing instance in a way that is consistent with how the code deletes the production instance.
When there's only 1 problematic method it's easier to use Extract and Override Call. But Extract and Override getter is better choice when there are a lot of problematic methods.

### Extract Implementer
Turn the class into an interface by subclassing it and pushing all of its concrete methods down into that subclass.
Make all of the remaining public methods pure virtual (abstract).
The initial class at this point is a pure interface.
Make the new class inherit from the initial class (the interface).

We’re replacing the creation of objects of one concrete class with objects of another, so we aren’t really making our overall dependency
situation better. But it's helpful for the possible further refactoring.

It's harder to extract an implementer when there are child or parent classes in the hierarchy.
You'll have to extract implemeters for the classes of the hierarchy, so think of other options.

### Extract Interface
Ways to do:
- automated refactoring
- extract using the steps for extracting method
- cut/copy and paste several methods from a class at once and place their declarations in an interface

In C++, you have to mimic intefaces by creating a class that contains nothing but pure virtual functions.

- create an interface
- make your class implement it, make the fake class implement it
- change the place where you want to use the object so that it uses the interface rather than the original class
- define the method in the interface, implement methods in the class and fake class

> When you extract an interface, you don’t have to extract all of the public methods on the class you are extracting from

It's hard to deal with non-virtual methods, static methods in Java.

### Introduce Instance Delegator
Use of static methods is common.
E.g. for singleton pattern or for utility classes.
Utility classes are used when it's hard to find a clear abstraction. E.g. `Math`.
There's no problem in testing static methods unless they have anything that is hard to depend on.
Then you'll have to implement an object seam (Parametrize method)
```java
public class SomeClass
  {
    public void someMethod(BankingServices services) {
    ...
    services.updateBalance(id,sum);
  }
...
}
```
When you have many methods like this, you'll want to move them to a class and make them instance methods.

### Introduce Static Setter
A technique to be able to fake singletones.
Consider just passing the variable.

### Link Substitution
In OO you can substitute pretty easily of objects implement the same interface or have the same superclass.
In C you can Link Substitution to replace one function with another.

### Parameterize Constructor
Add a parameter to a constructor to be able to pass another object for the test.
In Java you can create a separate constructor with other arguments, and call the original one inside it.
Or: pass an optional argument with a default value.
Disadvantage: When we add a new parameter to a constructor, we are opening the door to further dependencies on the parameter’s class.

### Parameterize Method
You have a method that creates an object internally, and you want to replace the object to sense or separate.
Similar to Parametrize Constructor.

### Primitivize Parameter
This refactoring has many downsides, but still, makes writing test possible.
- Develop a free function that does the work you would need to do on the class
- Add a function to the class that builds up the representation and delegates it to the new function.

### Pull Up Feature
Pull up the cluster of methods into an abstact superclass.
Subclass it and create instances of the subclass in your tests.

### Push Down Dependency
Separates problematic dependencies from the rest of the class.
- Identify which dependencies create problems in the build.
- Create a new subclass with a name that communicates the specific environment of those dependencies.
- Copy the instance variables and methods that contain the bad dependencies into the new subclass, preserving signatures
- Create a testing subclass

### Replace Function with Function Pointer
In procedural languages you can't use many of the techniques you can use in OO.
In C you can use function pointers.

It happens completely at compile time, so it has minimal impact on your build system.

### Replace Global Reference with Getter
Wrap a global inside an instance method. Subclass and overrride for a test.

### Subclass and Override Method
is a powerful technique, but be careful.
Sometimes dependencies are well isolated, sometimes you have to override large methods.

### Supersede Instance Variable
For the languages that disallow overrides of virtual function calls in constructors.

Add a method which accepts a parameter and supersedes it. In the method destroy your previous instance variable and create the one you need.

`Extract and Override Factory Method` is a better choice for languages that allow it

### Template Redefinition
For languages that have generics and a way of aliasing types.
Make `AsyncReceptionPort` a template rather than a regular class. Teplates can operate with generic types.
We can instantiate the template with a different type in the test file.

### Text Redefinition
Redefine methods on the fly, e.g. in ruby.

## Questions
### Chapter 1. Changing Software
- What are 4 reasons to change software?
- How to distinguish adding behavior from changing it?
- feature from a bug?
- What are the goals of refactoring and optimization?
- What's common in changing code for all the 4 reasons?
- How do we mitigate risk of the change?
- Why avoiding changes is not a good strategy?

### Chapter 2. Working with Feedback
- What's the traditional way to do a regression testing?
- What are the problems with it?
- Why is it important to test in isolation?
- What's a unit test?
- Is it always possible to write tests before changing the code?
- How to solve the Legacy Code dilemma?
- What's the legacy code change algorithm?

### Chapter 3. Sensing and Separation
- What are the reasons to break dependencies?
- What are the problems of sensing and separation?
- How do you write the code so that you can use the fake object in the test?
- How should a test object be implemented?
- Why do fake objects support real tests? (not fake tests)
- How do you create fake objects in non-OO languages?
- What are mock objects?

### Chapter 4. The Seam Model
- what are the solutions to make software suitable for tests?
- Does testability depend on good design?
- What's a seam?
- What kinds of seams do you know?
- What's an object seam?
- When calling method of an instance is a seam and when not?
- When it could be better to subclass and redefine a method for a test instead of passing the test object as an argument?
- How to choose the seam type?

### Chapter 5. Tools
- what to pay attention to when you use tools for automated refactoring
- why you shouldn't rely on ui-based tests?
- How do you write tests with JUnit?
- How does it make tests not to affect each other?

## Chapter 6. I Don’t Have Much Time and I Have to Change It
- Is it worth it to write tests if it takes much more time than changing the code?
- Pay now or pay more later?
- how to sprout a method?

- what are the 2 motivations to sprout a class?
- Won't introducing a new concept clutter our code? (when sprouting a class)
- What're advantages and disadvantages of the sprout class?

- Whats a Wrap Method?
- Advantages/disadvantages

- What's a decorator pattern? How do you implement it?
- When to use Wrap Class instead of Sprout/Wrap method?


## Chapter 7. It Takes Forever to Make a Change
- What are additional obstacles when getting feedback in the compiled languages in comparison to the interpreted ones.
- What is the dependency inversion principle and how does it work?
- What are advantages and disadvantages of breaking dependencies with introducing more interfaces and packages?

## Chapter 8.
- What's the steps to add a feature in TDD?
- Is copy/pasting code bad when adding a feature to the aweful code?

- What are your options to alter a class/method behavior? What are the possible disadvantages?
- What's a Liskov Substitution Principle?
- What's the problem with subclassing a concrete implementation?

## Chapter 9. I Can’t Get This Class into a Test Harness
- What are the possible problems you can encounter when trying to get a class into a test harness?
- What is an irritating parameter?
- What are the ways to deal with it?
- Does the test(fake) and production code has to meet the same standarts?
- What's a hidden dependency and how would you deal with it?
- What's a construction blob and how to deal with it?
- What are the dangers of testing singletons?
- How to test a singleton?
- What are the usual reasons to create a singleton?
- If you have an app where lots of objects are accessing the db, how would you refactor not to use singleton or a global variable or pass the variable 100500 times?
- What's the difference of Java/C# and C++ when dealing with(compiling) imported classes?
- What's an onion parameter and how to deal with them?

## Chapter 10. I Can’t Get This Method into a Test Harness
- How to test a private method?
- is it ok to use reflection to access private variables or methods in tests?

## Chapter 11. I Need to Make a Change. What Methods Should I Test?
- How do you call the test you write for the existing code? (to preserve behavior).
- What possible effects can we get if the constructor receives arguments by reference?
- What problems can you meet while figuring out the effects?
- What questions to ask when writing Characterization Tests?
- How to determine what tests to write? (in the first place)
- How to determine code effects in your language?
- Can the objects passed as arguments be mutated in Java?
- How can you have less effects in the code in different languages?
- Is it ok to break encapsulation when breaking dependencies and why?
- Why encapsulation is important?

## Chapter 12. I Need to Make Many Changes in One Area
- what's a higher-level test?
- Are the higher level tests more useful while refactoring?
  Can they substitute unit tests?
- How to choose interception points?
- What's the best interception point in most cases?
- How pinch points relate to encapsulation?

## Chapter 13. I Need to Make a Change, but I Don’t Know What Tests to Write
- What's the main goal when writing tests?
- What's the algorithm of writing a characterization test?
- What to do if you find a bug while writing a characterization test?
- When to stop when writing characterization tests?
- What should you do before using the method in a legacy system?
- What actions are important when Characterizing Classes?

## Chapter 14
- How to protect yourself from the problems with the 3rd-party libraries?

## Chapter 15
- How to start improving the large chunk of untested code?
- What are 2 options to make the design better when you have a lot of calls to a library in your code?

## Chapter 16
- What techniques to use when the code is hard to understand?

## Chapter 17. My Application Has No Structure
- What's the problem when the architecture is only the architect's responsibility?
- How does telling the story of the system help?
- How to describe a system architechture using cards?
- Is it always easy to add abstractions to a legacy codebase? What problems can appear?

## Chapter 18. My Test Code Is in the Way
- What are your options for naming test classes?
- Where to put your test code?

## Chapter 19. My code is not OO
- What options are there to redefine a function for test in the procedural languages?
- Is TDD usable in procedural languages?
- How to use function pointers to call functions in oo-style?
- What are the benefits of using oo successors of the procedural languages?

## Chapter 20
- Why large classes make development complicated?
- What's SRP?
- What would you de to move from giant classes to better design in a real-life application?
- What techniques you can use to determine responsibilities?
- What are feature sketches?
- At what levels can SRP be violated?
- How Interface Segregation Principle helps with refactoring large classes?
- What's the best algorithm to break down a large class after identifying responsibilities?
- How to introduce SRP (on a large class) gradually?

## Chapter 21. I’m Changing the Same Code All Over the Place
- Is it always worth removing duplication?

## Chapter 22. I Need to Change a Monster Method and I Can’t Write Tests for It
- What is a monster method?
- What are bulleted and snarled methods?
- Do the autorefactoring tools help to refactor such methods?
- What mistakes are possible when extracting methods?
- What's a sensing variable?
- What's a coupling count? Why should it be small when extracting?
- What's Gleaning dependencies? When is it useful?

## Chapter 23. How Do I Know That I’m Not Breaking Anything?
- Why should you preserve signatures when refactoring?
- How to lean on the compliler while refactoring?

## Chapter 24. We are overwhelmed.
- Is it always more fun to work on a green-grass project?
- How to deal with dejection when working on a legacy codebase?
- How to start getting control over the large legacy codebase?

## Chapter 25.
- Wide or narrow interfaces?
- What to use when you can't extract interface and need to break a dependency?
- How to break out a method object?
- How to implement different method definitions in C or C++ (when headers and definitions are stored separately)? What are the downsides?
- What are 3 ways to alter globals behaviour?
- When to expose static methods? What's are further refactoring steps?
- What to do when you have hardcoded object instantiation in the constructor?
- What's a lazy getter?
- How to deal with C and C++ when you have object instantiation in the constructor and can't call virtual methods there?
- How to name interfaces (I or not to I)?
- How to create interfaces in C++?
- What problems can appear when testing static methods? How to refactor?
- What are utility classes, is it ok to have them?
- If you parametrize the constructor, do you have to pass an additional argument on every call?
- What are disadvantages of parametrizing a constructor?
