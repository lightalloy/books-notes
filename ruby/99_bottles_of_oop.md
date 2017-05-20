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


2 - (76) - 88
3 - 89-120
4 - 122-170
5 - 170-222
6 - 222-280