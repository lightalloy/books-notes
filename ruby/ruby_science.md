# Ruby Science

## Code smells

### Long Method
### Large Class
### Feature Envy
A method (or a method-to-be) that would work better on another class.
- repeated references to the same object.
- parameters or local variables that are used more than methods and in-
stance variables of the class in question.
- methods that include a class name in their own names (such as
`invite_user`).
- private methods on the same class that accept the same parameter.
- Law of Demeter violations.
- Tell, don’t ask violations.

Solutions:
- extract method, move method, inline class

Prevention:
- follow the Lod
- tell, don't ask

### Case Statement
#### Symptoms:
- Case statements that check the class of an object.
- Case statements that check the type code
- Multiple if-elsif statements
- Divergent change caused by changing or adding when clauses.
- Shotgun surgery caused by duplicating the case statement.

#### Solutions
- Replace `type` code with subclasses
- Replace conditional with polimorphism
- Use CoC

### Shotgun Surgery
- You have to make the same small change across several different files
- difficult to manage and keep track of the changes

#### Solutions
- Replace conditional with polimorphism
- Replace conditionals with Null Object
- Extract Decorator
- Introduce parameter object
- Use CoC to eliminate small steps that can be inferred based on a convention.
- Inline class

#### Prevention
- try inverting control
- DRY
- follow LoD
- tell, don't ask

### Divergent Change
- a class changes for multiple reasons

#### Symptoms
- You can’t easily describe what the class does in one sentence.
- Changed more frequently than other classes in the app
- Different changes to the class aren’t related to each other.

#### Solutions
- Extract class, Move Method, Extract Validator, Introduce Form Object
- Use CoC

#### Prevention
- Single Responsibility
- Open/Closed
- Composition over Inheritance
- Dependency Inversion

#### Long Parameter List
- you can't easily change the method args
- 3+ args
- the method is coomplex due to number of args
- the method requires large amounts of setup for isolated testing

##### Solutions
- introduce Parameter Object
- Extract Class if the method is complex due to the number of collaborators

##### Anti-solution
Grouping parameters into a hash
Connascense of position => Connascense of name
It'll help a bit, but doesn't help with the number of params

#### Duplicated Code
- copy-paste, shortgun surgery

##### Solutions
- extract method, extract class
- extract partial
- Replace Conditional with Polymorphism
- Replace Conditional with Null Object

#### Uncommunicative Name
- difficulty understanding
- similar names, dissimilar functionality
- redundant names (e.g. include type)

##### Solutions
Rename methods, extract class, extract method, inline class

#### STI
##### Symptoms
- you need to change from one subclass to another
- behaviour is shared among some of the subclasses, but not others
- 1 subclass is a fusion of some other subclasses
- STI models have to understand that the're implemented through STI
- hard to understand

##### Solutions
- Replace subclasses with strategies
- Switch to polymorphic association

##### Prevention
Composition over Inheritance

#### Comments
##### Symptoms
- comments within method bodies
- 1+ comment per method
- todo comments
- commented out dead code

##### Solutions
- introduce explaining variable
- extract method
- move todos into task management system
- delete commented code (rely on vcs to get it back)


#### Mixin
Multiple inheritance.

Drawbacks:
- use the same namespace as classes, can cause naming conflicts
- can't easily accept initializing args, can't have their own state
- inflate the number of class' methods
- not easy to add/remove at runtime
- difficult to test in isolation

##### Symptoms
- methods in mixin that accept the same params
- methods in mixin that don't reference the state
- logic that can't be used w/o a mixin
- classes that have few public methods except from the ones from a mixin
- inverting dependencies is difficult

##### Solutions
- Extract Class
- Replace mixin with Composition

##### Prevention
- composition over inheritance
Reserve mixins for reusable framework code like common associations and callbacks.

#### Callback
Models with lots of callbacks are prone to bugs and are hard to refactor.

##### Symptoms
- callbacks with business logic, e.g. payments processing
- attributes that allow callbacks to be skipped
- methods such as `save_without_send_email`
- conditionally invoked callbacks

##### Solutions
- replace callbacks with method (if the callback logic is unrelated with the persistence)

### Solutions
#### Replace Conditional with Polymorphism
Removes divergent change, shotgun surgery, feature envy
Makes easier to reuse code, avoid duplication.

##### Replace Type Code with Subclasses
##### STI
```ruby
"#{object.to_partial_path}_form"
```
If you add behaviour much more often then add types, consider using observers or visitors.

#### Replace Conditional with Null Object
Removes multiple checks for nil, removes shotgun surgery.
Replaces conditional logic with simple checks.

##### Drawbacks
- May cause confusion - developer may expect instance of "real" object, not a of a NullObject.
- Some methods may need to distinguish NullObject from real ones, you may need to implement `present?` methods on your null objects.
- NullObject may need to reimplement large part of the object interface.

#### Extract Method
##### Replace Temp with Query

#### Rename Method
If you have a large number of references, add an alias to keep and old name working.

#### Extract Class
decreases the amount of complexity of each class, but increases the overall complexity of the app.
Extract classes in response to pain and resistance.

#### Extract Value Objects
```ruby
# app/models/recipient_list.rb
class RecipientList
  include Enumerable
  
  def initialize(recipient_string)
    @recipient_string = recipient_string
  end
  
  def each(&block)
    recipients.each(&block)
  end
  
  def to_s
    @recipient_string
  end
  
  private
  
  def recipients
    @recipient_string.to_s.gsub(/\s+/, '').split(/[\n,;]+/)
  end
end

# app/controllers/invitations_controller.rb
def recipient_list
  @recipient_list ||= RecipientList.new(params[:invitation][:recipients])
end
```

Immutable, so no writer methods.

#### Extract Decorator
E.g. base case - returning the real summary for the questions' answers
Decorated case - return a summary with a hidden answer

- move decorated case to decorator
- move conditional logic into the decorator
- move body into decorator
- promote parameter to instance variables
- change decorator to follow component interface
  so the decorator should have the same component as the decoratable object

- invert control
```ruby
def decorated_summarizer
  if include_unanswered?
    summarizer
  else
    UnansweredQuestionHider.new(summarizer, current_user)
  end
end

@summaries = @survey.summaries_using(decorated_summarizer)
```

##### Drawbacks
- Decorators must keep up to date with their component interface
- might be harder to understand
- classes with lots of public methods are difficult to decorate
- decorators can modify methods in the component interface easily but adding new methods won't work with multiple decorators w/o metaprogramming like `method_missing`

#### Extract Partial
Remove complex or duplicated view code.

#### Extract Validator
Removes complex validations from models.
Removes duplication for classes that use the same validations, easier to reuse.
Follows SRP

#### Introduce Explaining Variable
Break up complex, hard to read statement by placing part of it into a local variable.

#### Introduce Form Object
```ruby
class SurveyInviter
  include ActiveModel::Model
  attr_accessor :recipients, :message, :sender, :survey
  validates :message, presence: true
  validates :recipients, length: { minimum: 1 }
...
  def recipients=(recipients)
    @recipients = RecipientList.new(recipients)
  end
  
  def invite
    if valid?
      deliver_invitations
    end
  end
 
 private

 def create_invitations
 ...
 end

 def deliver_invitations
 ...
 end
end

# controller:
@survey_inviter = SurveyInviter.new(survey_inviter_params)

@survey_inviter.invite
```

#### Introduce Parameter Object
Reduce the number of input parameters to a method.

- groups params that are naturally fit together
- encapsulates behavior between related params

E.g. first_name, last_name, email => recipiemt

#### Use Class as Factory
knows how to build something, such as one of possible strategies for summarizing answers to questions in a survey.

An object, that holds a reference to an abstract factory doesn't need to know the resulting object's class, it knows only that that object implements a certain interface.

- removes shotgun surgery and case statements

#### Move Method
##### Dangerous: Move and Extract at the Same Time
Better break refactoring into 2 steps, it'll reduce the amount of time during
which something is broken.

#### Inline Class

#### Replace Subclasses with Strategies
- composition over inheritance
- eliminates large classes
- makes it easier to change parts of the structure

Big refactoring, break it down to smaller steps.

- extract a strategy class from each subclass and move (and delegate) as many methods as you can to the new class

```ruby
class OpenSubmittable
  def score(text)
    0
  end
end
```
Change the old subclass method to delegate to the new method:

```ruby
class OpenQuestion < Question
  def score(text)
    submittable.score(text)
  end
  ...
  def submittable
    OpenSubmittable.new
  end
end
```
Repeat the same process for each of the subclasses.
At this point things may get worse temporarily.
It's always better to refactor on a branch, apart from any new features.

#### Pull Up Delegate Method into Base Class
```ruby
# app/models/question.rb
delegate :score, to: :submittable
```

#### Move Remaining Common API into Strategies
Repeat the first two steps for every non-Railsy method that the subclasses implement.

The `breakdown` method requires state from the subclasses, so we'll pass it:
```ruby
# app/models/multiple_choice_question.rb
def submittable
  MultipleChoiceSubmittable.new(self)
end
```

#### Move Remaining Non-Railsy Public Methods into Strategies
Move public methods that are only implemented in one subclass.
This time, the delegator can live directly on the subclass,
rather than the base class:
```ruby
# app/models/scale_question.rb
def steps
  submittable.steps
end
```

#### Remove Delegators from Subclasses
Our subclasses now contain only delegators, code to instantiate the submittable, and framework code.

Find where the delegators are used and change the code to directly use the strategy instead.
You may need to pass the strategy in where the subclass was used before.

Then remove delegators.

#### Instantiate Strategy Directly from Base Class
The submittable method simply instantiates a class based on its own class name and passes itself to the initialize method:
```ruby
# app/models/open_question.rb
def submittable
  OpenSubmittable.new(self)
end
```
Let's pull up the method up to base class:
```ruby
# app/models/question.rb
def submittable
  submittable_class_name = type.sub('Question', 'Submittable')
  submittable_class_name.constantize.new(self)
end
```
We can then delete `submittable` from each of the subclasses.

Now subclasses contain only Rails-specific code, like associations and validations.

#### A Fork in the Road
You can’t move the association to a strategy class without making that strategy an ActiveRecord::Base subclass.

Also, one of our submittable strategies has state specific to that strategy.
The fields `min` and `max` are in the questions table, but are used only for scale questions. We can’t remove this pollution without creating a table for scale questions.

2 ways to proceed:

1) Don't make strategies AR subclasses, keep the association for multiple choice questions and the minimum and maximum for scale questions on the `Question` class, and use that data from the strategy.
That may result in divergent change and large `Question` class.

2) Make strategies AR subclasses. Move the association and state specific to strategies to those classes.
Create a table for each strategy.
Add polymorphic association to `Question`.

#### Convert Strategies to ActiveRecord Subclasses
```ruby
# app/models/open_submittable.rb
class OpenSubmittable < ActiveRecord::Base
  has_one :question, as: :submittable
  ...
end

# app/models/question.rb
def submittable
  submittable_class = type.sub('Question', 'Submittable').constantize
  submittable_class.new(question: self)
end
```

#### Introduce a Polymorphic Association
```ruby
# app/models/question.rb
belongs_to :submittable, polymorphic: true
```
Add polymorphic fields to questions.

Change the submittable method (overriding an association)
```ruby
# app/models/question.rb
def build_submittable
  submittable_class = type.sub('Question', 'Submittable').constantize
  self.submittable = submittable_class.new(question: self)
end

# app/controllers/questions_controller.rb
def build_question
  @question = type.constantize.new(question_params)
  @question.build_submittable
  @question.survey = @survey
end
```

#### Pass Attributes to Strategies

We’re persisting the strategy as an association, but the strategies don't have any state.
But scale submittables need min and max.
```ruby
# app/models/question.rb
def build_submittable(attributes)
  submittable_class = type.sub('Question', 'Submittable').constantize
  self.submittable = submittable_class.new(attributes.merge(question: self))
end
```
Move the `minimum` and `maximum` fields over to the `scale_submittables` table.

```ruby
# app/controllers/questions_controller.rb
def build_question
  @question = type.constantize.new(question_params)
  @question.build_submittable(submittable_params)
  @question.survey = @survey
end
```

#### Move Remaining Railsy Behavior Out of Subclasses
Move the association and behavior over to the strategy class.

#### Backfill Strategies for Existing Records
Move data from `questions` table to strategies' tables.

#### Pass the Type When Instantiating the Strategy
Remove the dependence on the type column.
Accept a `type` when building the submittable:
```ruby
# app/models/question.rb
def build_submittable(type, attributes)
  submittable_class = type.sub('Question', 'Submittable').constantize
  self.submittable = submittable_class.new(attributes.merge(question: self))
end
```
#### Always Instantiate the Base Class
```ruby
# app/controllers/questions_controller.rb
@question = type.constantize.new(question_params)
# =>
@question = Question.new(question_params)
```
Change `type` to `submittable_type` in the form.
Revisit views that rely on polymorphic partials.

- Remove `type` column
- Remove subclasses

#### Simplify the `type` switching

```ruby
# app/models/question.rb
def switch_to(type, attributes)
  old_submittable = submittable
  build_submittable type, attributes
    transaction do
      if save
        old_submittable.destroy
      end
  end
end
```
- easy to change types
- reduced coupling
- clear boundary in the API for questions and submittables, easier to test
But:
- increases overall complexity
- need 2 queries to the db to get data
- introduced 2 useless tables

### Replace Mixin with Composition

### Replace Callback with Method

### Use Convention Over Configuration

## Principles
### DRY
> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

Many techniques are just based on preventing or eliminating duplication.
When knowledge is duplicated, changing it means making the same change in several places.

#### Duplicated Knowledge vs. Duplicated Text
The principle doesn't state that code shouldn't be repeated.

Making behavior easy to reuse is essential to avoiding duplication.
Developers won't be tempted to copy/paste smth that's easy to reuse.

### Single Responsibility Principle
> A class should have only one reason to change

Classes with fewer responsibilities are more likely to be reusable, easier to understand and faster to test.

#### Reasons to Change
E.g. invitation class has many reasons to change:
- The format of invitation tokens changes
- A bug in email validation
- other delivery mechanism
- Another way to persist invitations
- an app uses new framework
- the interface of AR or ActiveSupport changes

#### Stability
Not all reasons to change are created equal.
You should anticipate how likely the changes are.
The less confident you are about a decision, the more you should isolate that
decision from the rest of your application.

#### Cohesion
One of the primary goals of SRP is to promote cohesive classes.
More related methods and properies => more cohesive class.

Classes with high cohesion are easier to understand, change and reuse.

#### Responsibility Magnets
Every application develops a few black holes that like to suck up as much re-
sponsibility as possible, slowly turning into God classes.
E.g. `User`

It’s easy to get sucked into a responsibility magnet by falling prey to “Just-One-More Syndrome.” Don’t feed the problem; add a new class instead.

#### Tension with Tell, Don’t Ask
E.g. Purchase model that knows how to charge a user:
```ruby
class Purchase
  def charge
    purchaser.charge_credit_card(total_amount)
  end
end
```
It follows "tell, don't ask", but it violates SRP.
If you extract a class `PurchaseProcessor`.

```ruby
class PurchaseProcessor
  def initialize(purchase, purchaser)
    @purchase = purchase
    @purchaser = purchaser
  end
  def charge
    @purchaser.charge_credit_card @purchase.total_amount
  end
end
```
It'll follow SRP, But violate "tell, don’t ask", because it must ask the @purchase for its total_amount in order to place the charge.

These two principles are often at odds with each other and you must make a
pragmatic decision about which direction works best for your own classes.

#### Drawbacks
- tension with "tell, don't ask"
- increase in number of classes
- may introduce additional indirection, harder to understand high-level behavior

### Tell, Don’t Ask
```ruby
class Order
  def charge(user)
    # doesn't folloe the principle: asks user if she has a valid credit card
    if user.has_valid_credit_card?
      user.charge(total)
    else
      false
    end
  end
end
```
We can move this decision into `User#charge`
`Order#charge` can simply delegate to `User#charge`, passing its own relevant
state (total)

#### Encapsulation of Logic
encapsulates the conditions under which an operation can be performed in one place
#### Encapsulation of State
Referencing another object’s state directly couples two objects together based
on what they are, rather than on what they do.
By following tell, don’t ask, we encapsulate state within the object that uses it, exposing only the operations that can be performed based on that state and hiding the state itself within private methods and instance variables.

#### Minimal Public Interface
TdA leads to minimal public interface. Public methods are a liability.
Before they can be changed, moved, renamed or removed, you will need to find every consumer class and update each one accordingly.

#### Tension with MVC
The view needs to ask the objects.
You could obey tell, don’t ask by making the user know how to render the form, but that violated MVC.

### Law of Demeter
A method of an object should invoke only the methods of the following kinds of objects:
- itself
- its parametes
- any objects it creates/instantiates
- its direct component objects

Is an attempt to help developers to manage dependencies.
It restrict how deeply a method can reach into another object dependency graph, it prevents tight coupling.

#### Multiple Dots
The quickest way to deal with it is delegation.
If you use a lot of delegators, consider changing the consumer class to a different object.
E.g. the code referencing `User` should actually reference to the `Account`.

#### Multiple Assignments
The method can not have multiple dots on one line, but still violate LoD.
(using local variable to "hide" violation)

#### The Spirit of the Law
Fixing the violation itself shouldn't be your objective, but removing the problem that caused the violation should.
- many delegator methods may the relationships are not accurately represented in your code
- check if you really need prefix for the delegated methods if you need it.
- avoid multiple prefixes for delegated methods
- avoid assigning to instance variables to work around violations.

#### Objects vs. Types
The first formulation of LoD was in terms of types.
This formulation allows some more freedom when chaining using a fluent syntax.
It allows chaining as long as each step of the chain returns the same type.

#### Duplication
The LoD violations often duplicate knowledge of dependencies.

#### Application
Smell, causing LoD violations:
- Feature Envy
- Shotgun Surgery

Solutions:
- Move Methods to the owner of the dependency
- Inject Dependencies so that methods have direct access to the dependencies they need
- Inline class if it adds to the dependency chain w/o providing enough value

### Composition Over Inheritance
If there isn't a strong case for using inheritance, use composition.

#### Inheritance
Example: Base `Inviter` class and `MessageInviter`, `EmailInviter` inherited from it.
There's no clear boundary between a base class and subclasses.
The subclasses access reusable behaviour by invoking private methods from the base class.

#### Composition
Example: `InvitationMessage` class to implement common logic. `EmailInviter` and `MessageInviter` use that class to reuse the common behavior.

#### Dynamic vs. Static
In the inheritance model the components are assembled statically.
In the composition - dynamically.

With inheritance you can’t :
- swap out a superclass once it’s assigned.
- easily add and remove behaviors after an object is instantiated.
- inject a superclass as a dependency.
- easily access an abstract class’s methods directly.

With composition you can:
- easily change out a composed instance after instantiation.
- add and remove behaviors at any time using decorators, strate-
gies, observers and other patterns.
- easily inject composed dependencies.
- use their methods anywhere

#### Dynamic Inheritance
In ruby you can:
- reopen and modify classes at runtime
- extend objects with modules after they’re instantiated to add
behaviors
- call private methods by using `send`
- create new classes at run-time by calling `Class.new`

These features make it possible to overcome the rigidity of inheritance models, but it's easier to perform all these operations with objects instead of classes.

#### The Trouble with Hierarchies
Your classes follow hierarchy when you use inheritance, but you may want to combine the behaviour the other way.
Composition provides several ways to achieve it: e.g. using a decorator to add functionality.
Once you build objects with a reasonable interface, you can combine them endlessly with minimal modification to the existing class structure.

#### Mixins
Ruby’s answer to multiple inheritance.
But it's much easier to combine behavior with composition instead of creating a lot of classes for different combinations with mixins.
Ruby does allow dynamic use of mixins using the extend method. This technique does work, but it has its own complications.

#### Single Table Inheritance
STI - persist inheritance
Polimorphic associations - persist composed objects

##### Drawbacks
There're situations when composed objects can hurt more than iheritance trees.
- inheritance cleanly represents hierarchies
- subclasses know what their superclass is so they're easier to instantiate. For composition you have to know both the composed and composing object.
- composition is more abstract, so you need to name it, it may lead to vocabulary overload

##### Application
After you replace inheritance with composition you may want to extract decorators and inject dependencies.

### Open/Closed Principle
> Software entities (classes, modules, functions, etc.) should be
open for extension, but closed for modification.

Benefits:
- easy mofifying a class w/o a risk of breaking it and classes that depend on it
- change of class' interface => need to change lots of classes that depend on it

#### Strategies
##### Inheritance
Create a subclass for another behavior
(if classes don't require persistance)

##### Decorators
```ruby
# app/models/unsubscribeable_invitation.rb
class UnsubscribeableInvitation < DelegateClass(Invitation)
  def deliver
    unless unsubscribed?
      super
    end
  end

  private
  def unsubscribed?
    Unsubscribe.where(email: recipient_email).exists?
  end
end
```
Similar to subclass, but may be applied at runtime:
Easier for persisting and testing.

##### Dependency Injection
We can modify our `Invitation` class slightly to allow client classes to inject a mailer:
```ruby
# app/models/invitation.rb
def deliver(mailer)
  body = InvitationMessage.new(self).body
  mailer.invitation_notification(self, body).deliver
end
# app/mailers/unsubscribeable_mailer.rb
class UnsubscribeableMailer
  def self.invitation_notification(invitation, body)
  ...
  end
...
end

# app/models/survey_inviter.rb
invitation.deliver(UnsubscribeableMailer)
```

##### Everything is Open
We found ways not to modify `Invitation`, but still we had to modify other classes.
You can design your code so that most new or changed behavior takes place by writing a new class, but something, somewhere in the existing code will need to reference that new class.

It's hard to determine what to leave open when writing a class.
Rather than guessing what will require extension in the future, pay
attention as you modify existing code. After each modification, check to see if
there’s a way you can refactor to make similar extensions possible without
modifying the underlying class.

##### Monkey Patching
```ruby
# app/monkey_patches/invitation_with_unsubscribing.rb
Invitation.class_eval do
  alias_method :deliver_unconditionally, :deliver
  def deliver
  ...
  end
end
```
Although monkey patching doesn’t literally modify the class’s source code, it
does modify the existing class.
Monkey patching has most of the drawbacks of modifying the original
class without any of the benefits of following the open/closed principle.

#### Drawbacks
Code may be easier to change bug difficult to understand

### Dependency Inversion Principle
> A. High-level modules should not depend on low-level modules.
> Both should depend on abstractions.
> B. Abstractions should not depend upon details. Details should
> depend upon abstractions.

#### Inversion of Control
- assigning dependencies at run-time, rather than statically referencing dependencies at each level.

##### Where To Decide Dependencies
If you push your dependency decisions up until they reach the layer that contains the information needed to make those decisions, you will prevent changes from affecting several layers.

##### Drawbacks
Following this principle results in more abstraction and indirection, as it’s often difficult to tell which class is being used for a dependency.

You can mitigate this issue by using naming conventions and well-named classes.
However, each abstraction introduces more vocabulary into the application.



