# Lost In Translation
## Modeling a Business
Mental models are often incomplete, unstable and unscientific, so developers are often approached with an incomplete idea.
There may be reasons why one aspect or another goes unnoticed until later in the development.

## Our First Bugs
One of the greatest challenges in writing software is communication.
Mental models are both difficult to communicate and often misunderstood - that's how our first bugs enter the program.
E.g. typos or nil lead to incorrect results.
Over time incorrect expectations and incomplete mental models lead to change - that's why Agile and Lean are so popular.
Responding to change is more valuable than following a plan (Agile Manifesto)
We can confidently say: We'll be wrong.

To avoid frustration and to prevent bugs we can separate our business processes from our domain models.
Our app architecture should respond well to changes in our understanding of what we're building.

> A key, longstanding hallmark of a good program is that it separates what is stable from what changes in the interest of good maintenance

# Object Orientation
## Where we are
OOP is pervasive, but the definition of “object-oriented” seems to have a number of interpretations:
Let's pick one:
Object-oriented programs execute algorithms by sending messages among objects, where the objects are an encapsulation of data and functions that perform operations.

So we represent things with objects and those objects know how to perform actions. These objects talk to each other by sending messages.

### What Are Objects
The name of the object reveals the intention of what it represents.
In a typical ruby app we create an object Account with an amount of cents.

### Doing More
We need to add money to an account, so we add that ability.
```ruby
class Account
  def add_money(cents)
    @cents += cents
  end
end
```
Next we need to transfer money from one account to another.
We add a method #transfer_to(account, amount)
One account in sending a message to another.

This is where we begin to complicate our code, but it isn't obvious.

### Restricted OO
We’ve gone from a class that represents a thing, to a class that represents a thing as well as defining its actions.
We built a class to represent objects that store information (how much money) as well as perform actions (transfer money).

With OO, the perspective the programmer takes is from within the object itself.
The traditional approach allows you to see the state of the object, the public methods, the private and protected methods, the attributes.
The problem is that we need to understand a lot about the class of an object, but still know little about the actual execution of a program.
We know that methods may be called and the state may be changed but we're still guessing what the actual behaviour is.

If we add more methods to Account, that'll exaggerate the problem easily.

```ruby
> account.public_methods(false)
=>
[:add_money, :transfer_to, :recalculate, :hold_funds, :subtract,
:transactions, :split, :close]
```
If we only care about transfers, these extra features are noise.

We can call this “Restricted OO” because our goal of readable code is supported only by the restrictions that we place on our architecture.

Our perspective is from within an individual class => we don't have a broad understanding of how and when domain models interact

We are forced to understand increasingly more about our system in its entirety to make sense of specific scenarios.

### Full OO
With Full OO we observe our objects from a perspective of being among them, not within them.
We can see exactly what object sends what message to what other object and we can clearly see and specify where those abilities are assigned.

> A very important goal for DCI is to give the user an illusion of working directly with her mental model when working with the computer.

This mental model we seek is the network of interacting objects. It is the representation of how objects interact with what other objects and when.
DCI elevates our perceived network of objects to a readable, executable algorithm.

## Where We've been
What did the first OO languages intend?
Simula was the first language to introduce objects, it was intended to simulate the real world.

> A computer program is a series of instructions which contains all  the information necessary for a computer to perform some task.

Reading code as if it’s a series of instructions is pedantic, no one does it this way.

## Where We're Going
But couldn’t they? Shouldn’t they?
We can and we should.

Step-by-step instructions are often how we think about our business process.
That's a reason why BDD became so popular. The goal of BDD is writing better code and executable specification.

DCI is similar to BDD. It encourages to focus on real world scenarious.
You're describing interactions with the actual objects that interact.

DCI pushes us to wite software that we better understand in terms of object collaboration and runtime execution.

# Being and Doing
Traditionally we create classes of things and then add behavior.
One problem is that classes grow like weed.

We need to define what the things are and then focus on programming what they do.

Example: a User class with #activate method
Over time we add more features that represent other aspects of behavior.

The actual activation is something that a user will do, not what it is.
Combining what the user is (representation of data) with what it does complicates our mental model.
The classes should represent data or model the behavior of the data object, but not both.

Let's create module Activator with the #activate method.

Often refactoring is just moving the code around, well intentioned, but accomplishes little.

If we just include Activator to User class, we'll still have the discussed problems.

## Managing Change
Let's include a User class from typical rails app with name, email and password validations.
This class is unlikely to change.
If we follow SRP, new methods shouldn't belong here.

Making changes outside of the user we'll have a clearer separation of concerns, better test targeting and better reusability. 
We'll create an Activator object:

```ruby
class Activator
  def initialize(user)
    @user = user
  end

  def activate
    ...
    @user.activated_at = Time.now
    @user.save
  end
end
```

It's probably better to call it UserActivator - depending on your domain.

### Implicitness, Explicitness, and Encapsulation
The OO purists may claim that we've broken encapsulation - Activator changes the user's state.
We've created an implicit contract - the Activator assumes that the user has a certain interface.

But explicit structure is more obvious.
That's the point for discussion.

## Simplifying the Code
Wouldn't be lots of bits and pieces, that we create to separate data and interaction harder to understand and maintain?
If you build an app in Rails you've probably seen controllers, that do a lot of things: find records, call methods on them, send emails, etc, etc.

Then you realize, that you need to extract some functionality, e.g. to avoid the code duplication.

One solution is skinny controllers, fat models.
You might decide to use callbacks.
```
after_create :send_welcome, :notify_admin_of_new_member
```

## Wrong Turn
Later in your development you realize that the administrators prefer to create users themselves.
Now you don't need one of callbacks in this situation.

You might add attr_accessor :dont_send_to_admin? and check it for sending callbacks.

This option is awful.
We could create an alternate class method #create_without_admin_notification! and call skip_callback! inside it, but it's even more terrible.

Callbacks are often a developers first foray into spaghetti code.

Setting flags and adding more methods to a global namespace are just two ways that we increase the cognitive load on ourselves.

# Objects in Context
DCI is an approach that removes the distraction of unrelated code - the objects don't have distracting abilities, the classes aren't cluttered with extra features.
Let’s look at our earlier example of transferring money between accounts to see how we can apply the idea of a DCI context.

We set yhr router to map a POST to our TransfersController:
```ruby
resources :accounts do
  resources :transfers, :only => [:index, :create]
end
```
Handling the request:
```ruby
def create
  account = Account.find(params[:account_id])
  account.transfer_to(params[:destination_id], params[:amount])
  redirect_to account_transfers_path(account)
end
```
Let’s alter this example to add the ability to alter only now that we need it.
```ruby
def create
  source = Account.find(params[:account_id])
  destination = Account.find(params[:destination_id])
  source.extend(Transferrer)
  source.transfer_to(destination, params[:amount])
  redirect_to account_transfers_path(source)
end
```
This example assumes that you’ve got a Transferrer module that defines
transfer_to.

There is a possibility of duplicating code. For now we aint gonna need it.
But will our application architecture help us or hurt us when we finally do need to reuse this code?
Reusing the code in the controller as it is will not be easy.

## Clearer Perspective
__The context__ is the object responsible for organizing the data and roles for carrying out your use cases.
It’s a powerful model for encapsulating your business logic.

Where is the business logic in common approaches? It's everywhere: controllers, models, split into modules and so on.
We can build a good system in such way but it'll be more difficult to understand.

The power of the DCI context object will get you closer to the intention-revealing code.

## Describing the system
The "D" is for data modeling: what we know as traditional objects, though bereft of knowledge about scenarios. It captures the static structure of objects[...]
The "C" is for context: the mapping of roles onto objects on a peruse-case basis[...]
The "I" is for interaction: an algorithm of a stateless role, written in terms only of other roles, that defines in readable terms what the system does, and how it does it.

We’ve explored the separation of data objects and their responsibilities but we pulled them together in our controller.
That's logical, but if the controllers become bloated by responsibilities such a way.

We can limit the controllers to handling requests and responses, and put our business logic into contexts.

```ruby
class MoneyTransfer
  def initialize(source, destination)
    @source, @destination = source, destination
  end

  def execute(amount)
    @source.transfer_to(@destination, amount)
  end
end
```

We will pass accounts to this context when it’s initialized, we want to refer to them in terms relevant to our context.
We call these __Methodless roles__ .

Our concern with DCI is with the interactions between and among roles.
Objects play different roles at different times during its execution.

A Methodless Role is the role an object plays in the execution of an algorithm.
Naming is very important for this context (source and destination, not account1, account2)

Back in our context, we’re still missing the ability to do anything.

Let’s alter the context:
```ruby
class MoneyTransfer
  def initialize(source, destination)
    @source = source
    @destination = destination
    assign_transferrer(@source)
  end

  def execute(amount)
    @source.transfer_to(@destination, amount)
  end

  private

  def assign_transferrer(account)
    account.extend(Transferrer)
  end

  module Transferrer
    def transfer_to(destination, amount)
      self.balance -= amount
      destination.balance += amount
    end
  end
end
```
We have just added what we call a __Methodful Role__.
In prev example objects playing roles in this context as source and destination had no special roles.
The Transferrer is the Methodful Role, it adds methods to the data object.

## Implementing Your System
Added tests for the MoneyTransfer.
Now we have something useful and reusable, but what does our controller look like?
```ruby
# TransfersController#create
  source = Account.find(params[:account_id])
  destination = Account.find(params[:destination_id])
  transfer = MoneyTransfer.new(source, destination)
  transfer.execute(params[:amount])
...
```
Although we’ve put our business logic in a reusable container, this controller action is still complicated.
It would be great to have code like this:
```ruby
transfer = MoneyTransfer.new(params[:account_id], params[:destination_id])
account = transfer.execute(params[:amount])
```

Alter the tests and modify MoneyTransfer:
```ruby
class MoneyTransfer
  def initialize(source_id, destination_id)
    @source = find_account(source_id)
    @destination = find_account(destination_id)
    assign_transferrer(@source)
  end

  private

  def find_account(id)
    Account.find(id)
  end
end
```

## All Set Up
We've put our business logic in a place that's reusable and easily tested.
When explaining our implementation we can say 'this is how it works' instead of 'What accounts do'.

With our focus on the program’s duties we can more quickly react to changes.
The MoneyTransfer context in this example centralizes our understanding of the system.

We won't do this for every procedure that our application needs to perform, but we have a powerfult tool to separate being from doing.

To better manage our flow we need a way to organize related scenarios.

# Gazelles And Gazebos
__User stories__ describe an end user’s behavior that will be later translated into code.
A __use case__ helps us to describe multiple scenarios for a business requirement.
For use case you have a specific preconditions, specific actors and specific goals.

> You can fit a gazelle inside a gazebo just as you can fit a user story inside of a use case, otherwise would be cruel.

A use case describes at least two exit points for a business process: the success and the failure.
Often there are many variations and multiple exit points in related scenarios.

Use cases DO:
- Hold Functional Requirements in an easy to read, easy to track text format
- Represents the goal of an interaction between an actor and the system.
  The context is an object that encapsulates this concept.
- Records a set of paths (scenarios) that traverse an actor from a trigger event (start of the use case) to the goal (success scenarios).
- Records a set of scenarios that traverse an actor from a trigger event toward a goal but fall short of the goal (failure scenarios).
  A use case describes a complete interaction between the user and your system and it is the responsibility of your context to implement this.
- Are multi-level: one use case can use/extend the functionality of another.
  A context can trigger other contexts within.

Using DCI goal is to implement business logic in an organized set.

In Rails, your controllers should handle the user actions that trigger these use cases.
You might have multiple ways to trigger a use case.
E.g. user can interact with objects and admin can interact with them in admin view.

Put your use cases in executable code and trigger them from wherever your interface requires it.
Begin by writing your use cases.

# Crossing the Use Chasm
We can write use cases to clarify the requirements of our program. Creating a context helps us to implement those requirements in executable code.
With this approach we can leave the decisions like data persistence for later.

## Framing the problem
Let’s create a scenario for this use case: a user submits a question and receives an answer from an expert.
```bash
rails new ask_the_experts
touch app/models/expert_questioning.rb
```

expert_questioning is the context for describing the use case.

### A Note On Naming And Code Use

expert_questioning_context.rb seems attractive as a filename.
We use _controller appendage for rails controllers.

The convention for the _controller appendage in Rails simplifies the setup of routing requests to the appropriate class to handle them
A UsersController will be automatically initialized when a request matches the :users route

But choosing to add Context to our context classes and _context to their filenames does nothing to simplify our code nor make magic happen. I

A gerund is often an intention revealing choice for a context. Our domain models are __objects__ represented by nouns, and our object interactions are __methods__ represented by verbs.
Our context is something __happening__ represented by a gerund.

Think about how you will describe your process in spoken language before you put it into terms of a programming language.

### Expanding Our Use Case
Our __Primary Actor__ will be a user of the system.

```ruby
# Primary Actor: a regular user
```
Specify a __Goal__
```ruby
# Goal: user gains information about a specific finance question
```
List __Supporting Actors__ (additional users, other parts of your system, or other systems)
```ruby
# Supporting Actors: a financial expert
```
__Preconditions__
```ruby
# Preconditions: user is authenticated and authorized
```
So our questions are:
Who is doing this? What is the goal? Who else is involved? And what else do I need for this to work?

Without creating any working code, we’ve begun with a clear understanding of what this program is going to do.

## Preparing the Context
Let's begin defining what we need to get things working.
```bash
touch spec/models/expert_questioning_spec.rb
```
We specify the requirements of the initialization method where we define what arguments we’ll need.
```ruby
# spec/models/expert_questioning_spec.rb
it 'initializes with a questioner and a question' do
...
end
...
it 'errors without a questioner and question' do
...
end
...
```
The described_class method in RSpec will return the class used in the current describe block.
It'll be easy to change the name of the class without changing the spec.

Let's create a simpler helper file giving us only what we need.
```bash
touch spec/spec_helper_lite.rb
```
We don't need booting the Rails app for now (using spec/spec_helper.rb)
For our spec_helper_lite we’ll add references to RSpec’s libraries, we'll add references for our necessary files to our expert_questioning_spec.rb

Let's define the class and the initialize method, so that the tests would pass:
```ruby
# Primary Actor: a regular user
# Goal: user gains information about a specific finance question
# Supporting Actors: a financial expert
# Preconditions: user is authenticated and authorized
class ExpertQuestioning
  def initialize(questioner, question_text)
  end
end
```
## Decoupling the File System
require_relative is usefult, but it couples us to the file system. We want to move files aroumd easily.
Instead we can use a simple require to by a #needs helper in spec_helper_lite.rb, which will add a certain directory to $LOAD_PATH.
We'll be able to load our model with:
```ruby
needs 'models'
require 'expert_questioning'
```
We could add app/models (and other needed dies) to load path, but that would cause an overhead. It's better to keep things simple and small.

## Defining Success
We'd like to nofify experts about the new question and the users about the question answered.

The __Trigger__ for this use case is the submitting of a question by the __Primary Actor__:

Actions, triggered by the primary actor:
- A notification is sent to the user that her question will be answered soon.
- An available expert is assigned to the question.
- The expert is notified of the unanswered question.

Actions, triggered by the supporting actor:
- The expert submits an answer to the question.
- The user is notified of the answer to her question.

There are no extra details (implementation), just what needs to be done.
Our implementation can be as simple as throwing paper airplanes around for a notification delivery system.

The five steps we’ve created represent just one scenario.
There are also __Extensions__ to this use case that we’ll see where we have alternate outcomes and failures.

## Brainstorming
As our program is used, we’re more likely to run into edge cases and alternate outcomes that our simple mental model didn’t cover.
Failures, alternatives, other actors.
__Extentions__ for our procedure:
- no expert is available
  => the user is notified that no expert is available for her question
- the expert determines that the question is answered already.
  => the expert assigns the question to an existing answer

The Lean approach teaches us that determining the aspects of this use case should be exhaustive.
We already have a problem with sending the user 2 contradictory notifications if there are not experts.

We should send a user a notification that her question will be answered soon after assinging an expert.
By using this approach we can avoic such errors before writing any actual code.

## Setting Boundaries
Our ExpertQuestioning will begin when a user submits a question. We’ll need a break in the procedure where we wait for the expert to answer
a question.
Now we’re venturing into mental models that are more complex, so it’s important to find good names.
An #execute method will probably not reflect our understanding.

Following naming conventions limits required information to understand a system. But there is a downside too: we should focus on communicating the
business process in our code not fitting our process to our code.

Let’s go with “start” and “finish” since we could say that we’d start with asking and finish by answering.
```ruby
class ExpertQuestioning
  def start
    # 1. An available expert is assigned to the question.
    # 1a. No expert is available.
    # 1a.1 The user is notified that no expert is available.
    # 2. A notification is sent to the user that her question
    will be answered soon.
    # 3. The expert is notified of the unanswered question.
  end
  def finish
    # 4. The expert submits an answer to the question.
    #
    4a. The expert determines that the question has already
    been answered.
    #
    4a.1 The expert assigns the question to an existing
    answer.
    # 5. The user is notified of the answer to her question.
  end
end
```

Create tests:
```ruby
# when starting:
  # it assigns an available expert to the question
  # it notifies the questioner that the answer is queued
  # it notifies the assigned expert of the questio
# when finishing:
  # answers the questio
  # notifies the questioner of the answer
```
All of these tests are pending for now, and each one can be a point of discussion for the implementation
Each one could become another use case.

# Screenplay In Action
We've set up the context. Now we need to decide how to implement the parts.

## Finding the Expert
For now details of selecting the expert is a distraction.
Our main concern is the asking and answering of a question. so let's leave a note for ourselves:
```ruby
def expert
  # TODO: Expert selection requires consideration
  Expert.first_available
end
# bundle exec rake notes
```

# Responding to Rails
We could create a responder to offload the behavior of redirection, rendering and other controller responsibilities.
When the flow of our code is typical, it’s OK for the details to be hidden somewhere else.
But when we need to understand special decisions or other elements of our program we want them to be as clear as possible.

If we create a responder:
```
def create
  transfer = MoneyTransfer.new(params[:account_id], params[:destination_id])
  respond_with transfer.execute(params[:amount])
end
```

The behavior (rendering view, redirection) depends on the return value of the execute method.
Where does this responsibility belong? Should the responder be a part of the context?

## Considering Dependencies
So does framework depends on business or vice versa?

The framework depending on the business logic might look like this:
```ruby
class SomeController < ActionController::Base
  def action
    if BusinessClass.logic_method
      redirect_to :some_path
    else
      render :some_other_action
    end
  end
end
```
When our needs are simple, this approach is quick and easy to get an application going.
But when we want to understand what process BusinessClass and logic_method represent, we have knowledge of the behavior in both BusinessClass and SomeController.
Our controller acts as the connection to the framework and it knows much about the BusinessClass.

To understand our program we need to keep more things in our minds and piece them together by matching up methods and behavior.

We can simplify our program and ease our understanding by writing a controller like the following:
```ruby
class SomeController < ActionController::Base
  def action
    respond_with BusinessClass.logic_method
  end
end
```
Now the controller only needs to know two things (BusinessClass and logic_method).

## Alternate Dependency Approaches
By using respond_with , our controller expects an object that behaves like a Responder.
Rails will automatically create a Responder for us and follow the conventions.

Relying on Rails to create a default responder for us might be hiding important information - we'd need to dig through documentation and framework code.
For Rails responder all we need is an object that responds to call:
```ruby
class SomeController < ActionController::Base
  def action
    respond_with BusinessClass.logic_method,
  :responder => lambda{ ... }
  end
end

# or

class SomeController < ActionController::Base
  self.responder = lambda{ ... }
  def action
    respond_with BusinessClass.logic_method
  end
end
```
We can keep a responder in the Business class. It might help us to keep the concerns in one place, but we either give up any useful behavior from our framework or inherit from ActionController::Responder creating an explicit dependency.

Instead of implementing the interface that our framework expects, we can tell the framework related parts what to do.
Dependency Injection:
```ruby
class TransfersController < ApplicationController
  def create
    transfer = MoneyTransfer.new(params[:account_id], params[:destination_id])
    transfer.command(self)
    transfer.execute(params[:amount])
  end
end
```
We are telling the transfer object to command the current controller what to do. This method could be named set_listener, guide, or any other term that best reveals the intentions.

## Framework Glue
We need to implement #go_to method.
We use this method instead of built-in redirect_to or render for 2 reasons:
- better naming for our domain
- 





