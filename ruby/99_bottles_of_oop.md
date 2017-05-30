# 1. Rediscovering Simplicity
Design choices have cost, it makes sense to pay this cost if you also accrue benefits.
Design is about picking right abstractions. But abstractions are hard.
Overanticipating, premature abstractions => difficult situations.
You should resist abstractions until they absolutely insist on being created.
This book is about finding the right abstraction.

## 1.1 Simplifying Code
Contradictory Goals: Code must remain concrete enough to be understood and be abstract enough to allow for change.
There’s a sweet spot that represents the perfect compromise between comprehension and changeability,
and it’s your job to find it.

### 1.1.1. Incomprehensibly Concise
First solution: hard to understand because it is inconsistent and duplicative, contains hidden concepts w/o names.

## Method versus Message
A method is defined by an object and contains behavior.
A message is sent by an object to invoke behavior.

Questions to help you assess value and cost of the bit of code:
- How difficult was it to write?
- How hard is it to understand?
- How expensive will it be to change?

### 1.1.2. Speculatively General
Difficult to write, too many levels of indirection.

Questions to ask:
- how many verse variants are there?
- which verses are most alike and in what way?
- which verses are most different and in what way?
- rule to determine what verse is sung next?

Value/cost questions:
- How difficult was it to write?
- How hard is it to understand?
- How expensive will it be to change?

### 1.1.3. Concretely Abstract
Too many individual methods, several classes.
Code is difficult to write. Methods are is easy to understand, but it's hard to get the point of entire song.
A small change can lead to need to do many changes.

Small methods + DRYness should lead to code that is easy to change.
But there wrong abstractions (name, level) lead to high cost of change.

### 1.1.4. Shameless Green
Easy to write, understand and change.
But disturbing, cause it's not DRY.

If you DRY out duplication or create methods to extract code, you'll create levels of indirection and make code more abstract.
In theory it makes code easier to understand and change, but in practice it often achieves th opposite.

## 1.2. Judging Code
We constantly judge code and make choices.
But defining what code is good code is not an easy thing.

### 1.2.1. Evaluating Code Based on Opinion
There are lots of definitions of good code.
Good code not only works, but is also simple, understandable, expressive and changeable.
These definitions tell what good code is like, but don't tell how to achieve this state.
None of these definitions is precise in a measurable way.

### 1.2.2. Evaluating Code Based on Facts
Code metrics:

#### Source Lines of Code
#### Cyclomatic Complexity
Counts the number of unique execution paths through a body of source code
(many nested conditionals => high score, no nested conditionals => 0)
Can be used to compare code, to determine overall complexity, to find out if you have enough tests.
Easy and useful, but views the world through a narrow lens - code does more than just evaluate conditions.
#### Assignments, Branches and Conditions (ABC) Metric
- variable assignments
- function calls and message sent
- conditional logic
Measure of complexity.
Ruby tool 'Flog'.

### 1.2.3. Comparing Solutions
Shameless Green has the lowest total Flog score, the second lowest SLOC, and the second lowest Worst Bit score. If your goal is to write straightforward code, these metrics point you toward Shameless Green.

## 1.3. Summary
As programmers grow they become comfortable with complexity.
But experience programmers write simple code.
Writing Shameless Green is fast, and the resulting code might be "good enough".
It's duplicative and not very object-oriented, but if nothing ever changes the best strategy is to deploy this code and walk away.
But if the change is required, the code may be not so good enough.

# Test Driving Shameless Green
Investigation on how to create tests that lead to Shameless Green.
## 2.1. Understanding tests
TDD works like this:
1) Write a test (red - failing)
2) Make it run (green)
3) Make it right (refactor)

Red/Green/Refactor cycle, the TDD mantra
Get to green quickly, it means safety and simplifies all your following actions.
This chapter concentrates on creating the tests and writing simple code to pass them. Future chapters refactor the resulting code to improve the design.

## 2.2. Writing the First Test

The first test is the hardest to write.
It's important to sketch out an overall plan before you write the first test.
Sketch out a public Application Programming Interface.
It may have methods: verse(n), verses(a, b), song

It makes sense to begin by testing a single verse.
TDD tells you to write the simplest code that will pass this test. So we write just enough code to change the error message.
'''
class Bottles
end
'''
Next steps:
- add an empty method 'verse'.
- add an argument (_)
- add exact verse text to the method

Obviously, the second test will break the code. It may seem pointless to write transitional code, but it's the essence of TDD.
TDD is toggling between wearing 2 hats:
1) Writing tests hat - big picture, work with the overall plan in mind
2) Writing Tests hat - pretend to know nothing other than the requirements, specified by the test.

## 2.3. Removing Duplication
Next test: you should do the simplest, most useful thing that proves that your code is incorrect.
It's verse 3 (verses 3-99 are nearly identical, testing high and low ends)
As the tests get more specific, the code get more generic.

## 2.4. Understanding   Transformations
In the article [Transformation  Priority    Premise](https://8thlight.com/blog/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html) there is a list of transformations that change code from more specific to more generic in priority order, from simpler to more complex.

## 2.5. Tolerating  Duplication
Verses 0,1,2 are unique. It makes sense to test the verse 2.
- write test
- failure
- possibilities:
*new conditional:
'''
if number == 2
...
else
...
end
'''
*interpolated string:
'''
...
"#{number-1} bottle#{'s' unless  (number-1)  ==  1}  of  beer"+
                           "on the wall.\n"
'''

Choice by metrics: interpolated string, but metrics are wrong - can't find 'unless' conditional.

You could also think of #pluralization method. But it's not a correct abstraction - bottle/bottles is duplicated many times, it represents an important concept, but that's not a pluralization.
You can't yet determine the right abstraction. So you have to choose: try to guess abstraction or tolerate duplication for a while.
These questions may help you:
- Does the change make code harder to understand?
- What is the future cost of doing nothing now?
- Will I soon get more information?

For now we'll resist creating an abstraction, cause the 'interpolated string' solution is harder to understand.

## 2.6 Hewing to the plan
So it's sometimes better to remove the duplication and sometimes to keep it.
Think of reaching green quickly, create only unambigious abstractions, refactor later.
Next step: test verse 1.
It's possible to pass the test with interpolated logic, but according to the previous example it's better not to do so.
Adding a case statement.
Next step: test verse 0.
We won't remove duplication currently - the cost of reaching green by adding one more branch is very low.

Now we have tests for all verse variants and code to make all of them pass.
The code is easy to understand, because it doesn't have many levels of indirection.

## 2.7. Exposing Responsibilities
Creating and testing method verses(a, b)
Creating a test for verses 88 and 89.
Duplication is useful when it supplies independent, specific examples of a
general concept that you don’t yet understand.
But here it's not useful - it would duplicate already existing example.
'''
def verses(_,_)
verse(99)+"\n"+verse(98)
end
'''
There are different ways to make tests pass:
- Fake It (till you make it)
- Obvious Implementation
- Triangulate

'Obvious' idea is attractive, but it's also perilous. Sometimes obvious idea turns out to be the wrong guess.
Fake It style may seem awkward, but with practice it becomes natural and speedy. It forces you to write better tests and is the antidote to hubris of thinking you know what's right.
Obvious is for very small leaps.

Triangulate is "conservatively drive abstraction with tests"
It means write several (breaking) tests at once and then one piece of code that will make all of them pass.

Let's test a few verses. And generalize the code.
Choose between adding a conditional and making code more abstract.
Here the best choice is abstraction.

## 2.8. Choosing Name
Creating song method.
Let's reflect API on the whole.
Does song serve it's purpose or is redundant and should be deleted?
The song method imposes a single dependency; to use it, you need only know its name, but to use verses method the caller should know a lot.

## 2.9. Revealing Intentions
The #song is intention and verses(99, 0) is implementation.
Song users don't need to know implementation details of verses.

## 2.10 Writing Cost-Effective Tests
TDD is not free, but benefits outweight the cost.
But the promise of TDD is not universally fulfilled. Often tests are hard to understand and to changed, time-consuming. Sometimes test suites are even abandoned.
Sometimes tests are tied too closely to code.
The tests should confirm what your code does without knowing how.

## 2.11 Avoiding the Echo Chamber
Testing the song method:
You may be tempted to test it using verses(99, 0)
Asserting that the song returns the expected lyrics is very different from testing that the song returns the same as verses.
This test would be too tighly coupled to the implementation. You also could have the wrong code and the test would still pass.
The text needed for 100 verses is fairly lengthy, and you may resist writing
out the full string because of concerns about duplication.

## 2.12 Considering Options
1) Assert that the expected output matches that of some other method.
Dependencies, that could break the song.
2) Assert that the expected output matches a dynamically generated string.
That would require logic, which can break the song in a confusing way.
3) Assert that the expected output matches a hard-coded string.
Only third idea is independent of the current implementation.
DRY is good idea for code, but less useful in tests.

# 3. Unearthing Concepts
Shameless Green is simple and understandable, but it's also kind of procedural and contains duplication.
It's good enough until the change is needed.

## 3.1. Listening to Change
The code is expensive, so improving code based only on aesthetic purposes may be not the best use of your precious time.
Changes should be driven by requirements.
A new requirement tells you how the code should be changed, changes impose higher standarts.
New requirement: output '1 six-pack' in each place where it currently sais '6 bottles'.
If we just add branches to case statement, it'll have 6 branches. This is unacceptable, so we should improve the code.

## 3.2. Starting With the Open/Closed Principle
Open/Closed - open for extension and closed for modification.
'O' in SOLID
Single Responsibility - methods of the class should be cohesive around a single purpose
Open/Closed - open for extension. closed for modification.
Liskov Substitution - subclasses should be substitutable for their superclasses.
Interface Segregation - objects should be forced to depend on methods they don't use
Dependency Inversion - depend on abstractions, not concretions

Open/Closed - you should not conflate the process of refactoring with adding new features
New requirement => first rearrange your code such that it'll be open to the new feature, then add the new code.
Our code in not open for the new requirement, so we should refactor it.
But it's hard to figure out how to do it.
If you don't know how to make the code open, remove the easiest to fix/best understood code smell.

## 3.3. Recognizing Code Smells
The easiest way to unearth the code smells is to make a list of things you dislike about the code.

## 3.4. Identifying the Best Point of Attack
Our code contains the switch statement smell and duplicated code.
Duplicated Code is the easiest.
Removing duplication is not directly connected with the new requirement. The plan is to nibble away, one code smell at a time, in faith that the path to openness will be revealed.

## 3.5. Refactoring Systematically
Safe refactoring relies upon tests.
If tests fail because you’ve broken the code, undo the last change, make a better one and proceed.
If you rearrange code without changing behaviour but tests fail, the tests are flawed (e.g. tests know about implementation). In this situation you should improve the tests first.

## 3.6. Following the Flocking Rules
Current task - removing the duplication.
### Flocking Rules
1) Select the things that are most alike.
2) Find the smallest difference between them.
3) Make the simplest change that will remove that difference.

Steps for code change:
- parse the new code
- parse and execute it
- parse, execute and use the result
- delete unused code

With these steps you'll get very precise error messages when something goes wrong. As you gain experience you'll take larger steps.

For now:
- change only 1 line at a time
- run the tests after every change
- if the tests fail undo and make a better change

Birds flocking is done by 3 simple rules:
1) Alignment - Steer towards the average heading of neighbors
2) Separation - Don’t get too close to a neighbor
3) Cohesion - Steer towards the average position of the flock

## 3.7. Converging on Abstractions
The Flocking rules are atomic and general, they don't yet inspire confidence.
### 3.7.1 Focusing on Difference
When examining complicated problems, the eye is first drawn towards sameness.
Sameness is easier to identify, but difference is more useful, cause it has more meaning.
DRYing the difference has more value.
The difference represents an abstraction.
You don't have to identify abstraction in advance, just follow the rules and the abstraction will appear.

Programmers usually decide a solution and implement it, the next code may be surprising.
Solution crafted by intention (usual way) => taking many small, iterative steps, resulting in solution, discovered by refactoring.

Back to the code:
2 case and else case are most alike, the only real difference is the word "bottle" versus the word "bottles."

### 3.7.2. Simplifying Hard Problems
Now we need to make the most alike strings identical.
It's important to focus on this specific goal, go by horizontal path.
The most difficult parts are the most interesting, but sometimes solving easy problems first transmute hard problems into easy ones.
Next step: replace hard-coded numbers.
Now the only difference is bottle vs bottles.

### 3.7.3. Naming Concepts
It's time to decide what bottle/bottles mean in the context of the song.
The name of the thing should be one level of abstraction higher than the thing itself.
'Unit' is too general, not in the language of the domain.
It may be useful to think of other things that would also be in the same category.
E.g. the container, which is the good name.

### 3.7.4. Making Methodical Transformations
It's tempting to make all changes in one step.
But making a slew of simultaneous changes is not refactoring — it’s rehacktoring. In real world you may end up with the ocean of red, then try to fix them, discard all of them and start anew.
It's better to make a series of tiny changes and run tests after each.
If tests fail, you'll know the exact change that caused a failure.
If the tests pass, the code still works, even when the refactoring in partially complete.

The steps guided by flocking rules:
- create an empty container method.
- container returns 'bottles'
- calling container in the else branch

Container should eventually receive an argument, but if we just add it, there will be ArgumentError: wrong number of arguments.
Real applications can have similar probles but with 10-100-1000 callers instead of one.

### 3.7.5. Refactoring Gradually
A strategy not to edit all callers simultaniously:
Container With Defaulted Argument:
'''
def container(number=:FIXME)
  if number == 1
    'bottle'
  else
    'bottles'
  end
end
'''
Once the refactor is complete, the default should be removed.
Note, that this change is multiline, but it could be represented in single a line form.
For now we'll keep it this way.
Next steps:
- really pass the argument in the else branch
- pass the argument in the 2 branch
- remove FIXME
- remove the 2 branch - now 2 and else cases are identical

We've done 15 steps, 12 of them required changes in code.
This way may seem very slow, but with practice it becomes very fast.
The time is also recouped by avoiding lengthy debugging sessions.
You can later combine steps if circumstances allow.

# 4. Practicing Horizontal Refactoring
Let's iteratively apply the Flocking Rules to get a more abstract sample to produce any verse.

## 4.1. Replacing Difference With Sameness
Next: the 1 and else cases are most alike.

Modifying first and second lines of these 2 branches:
- replacing hard-coded 1 with number
- bottle vs bottles => container(number)

## 4.2. Equivocating About Names
Third line: difference 'it' vs 'one'.
Finding the good name for such fuzzy concept is hard. 'thing' is broad and 'it_or_one' is too narrow.
Let's choose name 'pronoun' by now.
Steps:
- create empty pronoun method
- alter pronoun to return 'one'
- replace 'one' with pronoun
- add defaulted value to pronoun (FIXME)
- add conditional
- pass the argument to pronoun
- alter the pronoun calls to use argument
- remove default value

## 4.3. Deriving Names From Responsibilities
Next difference: last line in 1 and else cases: 'no more bottles' vs "#{number-1} #{container(number-1)}"
- replace 'bottles' with container(number-1)
- 'no more' vs #{number-1}

We code name this concept 'remainder', but the 0 case also starts with 'no more', so we'll choose 'quantity'.

But that's not a general licence to think ahead at such points.
When creating the abstraction, think of your current goal.
First choose the responsibility, then search for a good name which reflects this responsibility.

## 4.4. Choosing Meaningful Defaults
:FIXME default is helpful, but won't work in every case.
- creating quantity method, returning 'no more'
- calling method in 1 case
- add conditional to the quantity method (number == 0 ? 'no_more' : number)
=>
tests fail (FIXME bottles of beer on the wall)
FIXME default works only when you want to execute the false branch, for the true branch we need a more specific default.
- reverting to the prev step
- adding 0 as default
- pass number-1 as an argument in the 1 branch
- use quantity in the else case
- the default can be removed
- send container instead of bottles in 1 branch
The 1 branch can be deleted.

## 4.5. Seeking Stable Landing Points
3 new concepts (quantity, pronoun, container) were defined.
Each of them has a single responsibility, are of identical shape.
It's the result of following specific rules.
If you follow the rules of refactoring, you'll quickly arrive at stable points.
The consistency enables the next refactoring.

## 4.6. Obeying the Liskov Substitution Principle
We have 2 remaining branches.
Let's find the next smallest difference.
It's 'No more' vs number. It looks like the quantity concept, but it would return the lowercase 'no more' and broke the tests. So let's replace 'no more' with quantity(number).capitalize in the 0 case
If we replace the same thing in the else branch, the tests will fail with NoMethodError.
Review the quantity method: the true branch returns a String and the false branch returns Integer.
You may want to change the quantiry method, but let's firstS explicitly call to_s for both results of quantity method, that fixes the tests.
### Liskov Substitution Principle
The official definition is 'the subtypes should be substitutable for their supertypes'
But the LSP also applies to duck types - they should be substitutable for each other.
Liskov prohibits you from doing anything that would force the sender of the message to test the result of this message in order to know how to behave.
So we should modify the quantity method to always return string (to_s for the number) instead of sending to_s to the results.
Next: replace bottles with container in the 0 case. Now the first lines of 2 branches are the same.

## 4.7. Taking Bigger Steps
It's beginning to feel that there's a common refactoring pattern.
If this theory is corrent, it makes sense to combine several steps to speed up.
Let's examine the second lines of the remaining branches.
Replace the corresponding strings with the calls to quantity and container.
Lines 1 and 2 are now identical.
The next difference: 
lines 3 are very different, but they represent a single concept.
We must name a concept, create a method to represent it and replace the strings with a message send.
The concept is an action to take depending on the number of bottles.
We can do it in smallest steps or combine them.
If we take bigger steps, we'll create an action method.
Then replace 3rd lines with action message send.

Combining steps now is different from previous container method creation, cause now you've practiced the Flocking Rules. Now it makes sense to take bigger steps.
But if you combine steps and the tests fail, first return to green and make incremental changes.

## 4.8. Discovering Deeper Abstractions
There is only 1 difference left now - the 4th lines, let's resolve it.
'bottles' is the same as container, so we need to resolve '99' vs qunatity difference.
We may want to alter quantity method and replace if with a case statement.
But smth is deeply wrong eith this solution:
-1 is invalid number of beers.
Questions:
- what is the responsibility of the quantity method?
- is there a way to make 4-th phrases more alike or identical?

99 is not a special case of quantity concept.
You can replace the lines more alike by replacing 99 with quantity(99)
Replace 'bottles' with a call to container method.
'99' must represent the same concept ad number-1.
The concept should know that when the number is 50, result is 49 and when the number is 0, the result is 99.
There are #succ and #pred methods in ruby (4.succ => 5, 'b'.pred => 'a', etc)
In our case the main direction is down, expect the verse 0 case, which is followed by 99.
The term successor is right, here it means following, not higher.
First successor implementation:
'''
def successor(number)
  number - 1
end
'''
Next:
- replace number-1 with successor in the else branch
- make successor open to be used with 0, add if statement
- replace 99 with successor

The 0 and else cases are now identical.

Successor is important and separating it from quantity gives both methods a single responsibility.

## 4.9. Depending on Abstractions

Abstractions are benefitial in many ways: consolidating code, so it becomes a shortcut for an idea, the code can be changed with ease.
Another subtle benefit: they tell where code relies upon an idea. To get this benefit you must refer to an abstraction in every place where it applies.
E.g. container(number-1). This code doesn't actually want the container of number-1, it wants the container for a following verse.
You should replace all occurencies of number-1 with successor(number).
One more refactoring trick to prove that this common template work for all changes:
Add the whole verse text to the end of the method whithout removing case statements.
Now we can remove the case statement.
It's important to ask yourself if the new code is better.
The Flog score is worse, but the code is better.
The new concepts are revealed and isolated in the new code.
If several programmers follow the Flocking Rules, their resulting code will be the same.

# 5. Separating Responsibilities
The impetus of our refactoring was the nex 'six-pack' requirement.

## 5.1. Selecting the Target Code Smell
Code should be opened for extension and closed for modification.
Let's reexamine the current code and find if it's open for the new requirement.
A code is not yet open for it.
The truth about refactoring is that sometimes it makes things worse.
Proper refactoring allowes you to explore a problem domain safely.
We've done the refactoring, but the code is not yet open. Now we must decide whether it's better to proceed with additional modifications or revert to the previous state and try one more time.

The current code is not open but it's improved, so let's keep it.
You must continue to be guided by code smells.

### 5.1.1. Identifying Patterns in Code
### 5.1.1. Spotting Common Qualities
### 5.1.3. Enumerating Flocked Method Commonalities
Questions to ask:
#### Do any methods have the same shape?
  The Flocked Five methods are all of the same shape.
  It’s not yet clear what it means that these methods have the same shape, but it’s important to notice that they do.
##### Squint Test - one easy way to judge the code.
- put the code of interest on the screen
- lean back
- squint your eyes, such as you can still see the code, but can no longer read it or zoom out.
- look for the changes in shape and in color
Changes of shape reveal the presence of conditionals. Two or more levels of indentation result in multiple execution paths and add complexity.
Changes of color indicate differences in the level of abstraction. A method that mixes many colors tells a story that may be difficult to follow.

#### Do any methods have the arguments of same shape?
6 methods take the same argument - the Flocking Five and the verse method.

#### Do arguments of the same name always  mean the same thing?
Verse method - number represents the verse number.
Number in the Flocking Five method is a bottle number.
These methods (verse vs five) use the same argument name to represent different concepts.
This is rarely a good idea.
The naming mistakes make it harder to notice underlying code smells, now you must examine the arguments and clarify the abstractions they represent (verse number - for the verse, bottle number - for the flocking five methods)

#### Where would you put the private keyword in this class?
Before the flocking five

#### How would you break the class into 2 pieces?
Same

For the Flocking Five methods (container, quantity, action, pronoun and successor):
#### Do the tests in the conditionals have anything in common?
Thy all test that number is exactly equal to another value.

#### How many branches do conditionals have?
2

#### Do the methods contain any code rather than the conditionals?
No.

#### Do methods that take number as an argument depend more on number or more on a class as a whole?
The flocked five depend only on the number argument rather than on the rest of the class.

### 5.1.4. Insisting Upon Messages
The flocking five method are flawed from the point of view of the OO practitioner.
This solution is optimized to understandability instead of changeability.
The above pattern means that objects are missing.

If you see the conditional, be suspicious.
Conditionals are useful in OOP apps. Manageable OO applications consist of pools of small objects that collaborate to accomplish tasks. E.g. some object, somewhere, must choose which objects to create, and this    often requires conditionals.

There’s a big difference between a conditional that selects the correct object and one that supplies behavior. The first is acceptable and unavoidable. The second suggests that you are missing objects in your domain.

To preserve code ignorance, minimizing dependencies is required. The container methods yearns to be injected with a smarter object to which it could forward the message:

'''
def container
  smarter_number.container
end
'''

## 5.2. Extracting Classes
A number of methods take the same argument, have the same shape, contain a conditional, could be private and depend more on argument than on the class as a whole.
These are traits that point on a Primitive Obsession code smell. 
Built-in data classes like String, Integer, Array , and Hash are examples of "primitives."
The cure for Primitive Obsession is to create a new class to use in place of the primitive.
The refactoring recipe is Extract Class.

### 5.2.1. Modeling Abstractions
You must now choose a name for a new class.
The primitive you are replacing represents a bottle number.
Not a bottle (physical), but a bottle number (idea).
The power of OO is that you create a virtual world, in which ideas (e.g. Purchase, Discount, etc) are real.
Don't focus on physical things only when designing an app.

Name of our class: there are two most obvious choices - ContainerNumber and BottleNumber.

### 5.2.2. Naming Classes
The rule is: name methods one level of abstraction higher than current implementation.
If we extrapolate this rule to classes, the name should be ContainerNumber.
But the name BottleNumber is good enough.
BottleNumber - less flexible, straightforward. ContainerNumber - more abstract. 
That rule applies more to numbers than to classes.
The classes must be named after the name they are, you can revisit this desicion if things change later.

### 5.2.3. Extracting BottleNumber



 

5 - (185)-222
6 - 222-280


- write out flocking rules and some refactoring algorythms



