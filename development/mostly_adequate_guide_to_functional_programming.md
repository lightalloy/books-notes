# Mostly Adequate Guide to Functional Programming
## Chapter 02: First Class Functions
We can treat functions like any other data type and there is nothing particularly special about them.

functions are callable in JavaScript
with `()` => run and return a value
without `()` => returns the function stored in variable

### Why Favor First Class?
a needlessly wrapped function changed => need to change the wrapper
```js
// go back to every httpGet call in the application and explicitly pass err along.
httpGet('/post/2', (json, err) => renderPost(json, err));
// renderPost is called from within httpGet with however many arguments it wants
httpGet('/post/2', renderPost);
```

## Chapter 03: Pure Happiness with Pure Functions
A pure function is a function that, given the same input, will always return
the same output and does not have any observable side effect.

In functional programming, we dislike unwieldy functions that mutate data.

### Side Effects May Include...
- changing the file system
- writing to the db
- making an http call
- mutations
- printing to the screen / logging
- obtaining user input
- querying the DOM
- accessing system state

The philosophy of functional programming postulates that side effects are a primary cause of incorrect behavior.

Side effects disqualify a function from being pure.

### 8th Grade Math
A function is a special relationship between values: Each of its input values
gives back exactly one output value.
Since functions are simply mappings of input to output, one could simply jot down object literals and run them with [] instead of ():
```
const isPrime = {
1: false,
2: true,
3: true,
4: false,
5: true,
6: false,
};
isPrime[3]; // true
```
functions with multiple arguments => bundle them up in an array or just think of the arguments object as the input.

Pure functions are mathematical functions and they're what functional programming is all about.

### The Case for Purity
#### Cacheable
For starters, pure functions can always be cached by input.
Memoization technique.
Simplified implementation:
```
const memoize = (f) => {
  const cache = {};
  return (...args) => {
    const argStr = JSON.stringify(args);
    cache[argStr] = cache[argStr] || f(...args);
    return cache[argStr];
  };
};
```

You can transform some impure functions into pure ones by delaying evaluation:
```
const pureHttpCall = memoize((url, params) => () => $.getJSON(url, params));
```
We don't actually make the http call - we instead return a function that will do so when called.

#### Portable / Self-documenting
Pure functions are completely self contained.
A function's dependencies are explicit and therefore easier to see and understand.

We're forced to "inject" dependencies, or pass them in as arguments.
It makes our app more flexible.
In a JavaScript setting, portability could mean serializing and sending functions over a socket.

#### Testable
Pure functions make testing easier
QuickCheck - a combinator library originally written in Haskell, designed to assist in software testing by generating test cases for test suites.

#### Reasonable
Referential transparency.
Equational reasoning -- replace the functions with their actual value.

#### Parallel Code
We can run any pure function in parallel since it does not need access to shared memory and it cannot have race conditions or side effects.

## Chapter 04: Currying
You can call a function with fewer arguments than it expects. It returns a function that takes the remaining arguments.

```js
const add = x => y => x + y;
const increment = add(1);
```
We can use a special helper function called `curry` to make defining and calling functions like this easier.

```js
const match = curry((what, s) => s.match(what));
const replace = curry((what, replacement, s) => s.replace(what, replacement));

match(/r/g, 'hello world');
const hasLetterR = match(/r/g); // x => x.match(/r/g)
```
Demonstrates the ability to "pre-load" a function with an argument or two in order to receive a new function that remembers those arguments.

### More Than a Pun / Special Sauce
Currying is useful for many things:
E.g. making new functions by giving our base functions some arguments

Function that works on single elements => function that works on arrays (wrap with map):
```js
const map = curry((f, xs) => xs.map(f));
const getChildren = x => x.childNodes;
const allTheChildren = map(getChildren);
```

Giving a function fewer arguments than it expects is typically called partial
application.

Currying: each single argument returns a new function expecting the remaining arguments. So no matter if the output is another function - it qualifies as pure.

## Chapter 05: Coding by Composing
```
// compose
const compose = (...fns) => (...args) => fns.reduceRight((res, fn) => [fn.call(null, ...res)], args[0];

// simplified compose
const compose2 = (f, g) => x => f(g(x));

const toUpperCase = x => x.toUpperCase();
const exclaim = x => `${x}!`;
const shout = compose(exclaim, toUpperCase);
shout('send in the clowns'); // "SEND IN THE CLOWNS!"
```
Composition is associative:
```
compose(f, compose(g, h)) === compose(compose(f, g), h);
```
It doesn't matter how we group them.
That allows us to write a variadic (with variable number of args) compose:
```
const loudLastUpper = compose(exclaim, toUpperCase, head, reverse);
```

The variadic compose can be found in libraries like lodash, underscore, and ramda.

Any group of functions can be extracted and bundled together in their very own composition.

### Pointfree
- functions that never mention the data upon which they operate.

```js
// not pointfree because we mention the data: name
const initials = name => name.split(' ').map(compose(toUpperCase, head)).join('. ');
// pointfree
const initials = compose(join('. '), map(compose(toUpperCase, head)), split(' '));
```

Pointfree code can help us remove needless names and keep us concise
and generic. But it can sometimes obfuscate intention.

### Debugging
A common mistake is to compose something like arguments, without first partially applying it.

```
// wrong
const latin = compose(map, angry, reverse);
// right
const latin = compose(map(angry), reverse);
```
You can use impure `trace` function if you have trouble debugging a composition.

### Category Theory
-- an abstract branch of mathematics that can formalize concepts from several different branches such as set theory, type theory, group theory, logic, and more.

A category is:
- A collection of objects
  will be data types
- A collection of morphisms
  our standard every day pure functions
- A notion of composition on the morphisms
  `compose`
- A distinguished morphism called identity
  a useful function called `id`

```
const id = x => x;
```
it's just like the identity property on numbers.

Other categories:
1) objects - directed graphs with nodes
morphisms - edges
composition - path concatenation

2) objects - numbers
morphisms - >=

### In Summary
Composition connects our functions together like a series of pipes.
Data will flow through our application.

## Chapter 6. Example Application

### Declarative Coding
```js
// imperative
const makes = [];
for (let i = 0; i < cars.length; i += 1) {
  makes.push(cars[i].make);
}
// declarative
const makes = cars.map(car => car.make);
```

```js
// homegrown `prop`
const prop = curry((property, object) => object[property]);
```

### A Principled Refactor
```js
// map's composition law
compose(map(f), map(g)) === map(compose(f, g));
```

## Chapter 07: Hindley-Milner and Me
Types and their implications:
```js
// head :: [a] -> a
const head = xs => xs[0];

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));
```

### Narrowing the Possibility
Parametricity - this property states that a function will act on all types in a
uniform manner.
```js
// reverse :: [a] -> [a]
```
Can it sort? No. Can it rearrange? Yes.

### Free as in Theorem
Besides deducing implementation possibilities, this sort of reasoning gains us
free theorems.

```js
// head :: [a] -> a
compose(f, head) === compose(head, map(f));
// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```

The first one says that if we get the head of our array, then run some
function f on it, that is equivalent to, and incidentally, much faster than, if we first map(f) over every element then take the head of the result.

That's common sense, but the computes don't have one.

### Constraints
We can constrain types to an interface.
```js
// sort :: Ord a => [a] -> [a]
```
`A` must implement an `Ord` interface.
We call these interface declarations type constraints.
```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Two constraints: `Eq` and `Show`:
Those will ensure that we can check equality of our a s and print the difference if they are not equal.

### Summary
Hindley-Milner type signatures are ubiquitous in the functional world.
Simple to read and write, but it takes time to master the technique of
understanding programs through signatures alone.

## Chapter 08: Tupperware
### The Mighty Container
First we will create a container, that can hold any type of value.
```js
class Container {
  constructor(x) {
    this.$value = x;
  }
  static of(x) {
    return new Container(x);
  }
}
Container.of(3);
// Container(3)
Container.of('hotdogs');
// Container("hotdogs")
Container.of(Container.of({ name: 'yoda' }));
// Container(Container({ name: 'yoda' }))
```
Once data goes into the Container it stays there. We could get it
out by using .$value , but that would defeat the purpose.

### My First Functor
We'll need a way to run functions on the data in the container:
```js
// (a -> b) -> Container a -> Container b
Container.prototype.map = function (f) {
  return Container.of(f(this.$value));
};

Container.of(2).map(two => two + 2);
// Container(4)
```
We can work with our value without ever having to leave the Container .
Wait a minute, if we keep calling map , it appears to be some sort of
composition:
```js
Container.of('bombs').map(concat(' away')).map(prop('length'));
// Container(10)
```

> A Functor is a type that implements map and obeys some laws

A Functor is simply an interface with a contract. Functors come from
category theory.

By asking the container to apply functions for us, we gain abstraction of function application.

### Schrödinger's Maybe
`Container` is usually called `Identity` and has about the same impact as our id function.
There are other functors, which provide useful behavior whilist mapping.
```js
class Maybe {
  static of(x) {
    return new Maybe(x);
  }
  get isNothing() {
    return this.$value === null || this.$value === undefined;
  }
  constructor(x) {
    this.$value = x;
  }
  map(fn) {
    return this.isNothing ? this : Maybe.of(fn(this.$value));
  }
  inspect() {
    return this.isNothing ? 'Nothing' : `Just(${inspect(this.$value)})`;
  }
}
```
`Maybe` looks a lot like `Container`, but it will first check to see if it has a value before calling the supplied function.
```js
Maybe.of({ name: 'Boris' }).map(prop('age')).map(add(10));
// Nothing
Maybe.of({ name: 'Dinah', age: 14 }).map(prop('age')).map(add(10));
// Just(24)
```
Our app doesn't explode with errors as we map functions over our null values - `Maybe` takes care of them.

Pointfree style:
`map` is fully equipped to delegate to whatever functor it receives:
```js
// map :: Functor f => (a -> b) -> f a -> f b
const map = curry((f, anyFunctor) => anyFunctor.map(f));
```
`Functor f =>` notation - f must be a functor.

### Use Cases
Used in functions which might fail to return a result.
Sometimes a function might return a Nothing explicitly to signal failure => the latter functions won't be run.

### Releasing the Value
There will always be an end of the line; some effecting function that sends JSON along, or prints to the screen, or alters our filesystem, or what have you.
We cannot deliver the output with return , we must run some function or another to send it out into the world.

A common error is to try to remove the value from our Maybe one way or another.

An escape hatch:
```js
// maybe :: b -> (a -> b) -> Maybe a -> b
const maybe = curry((v, f, m) => {
  if (m.isNothing) {
    return v;
  }
  return f(m.$value);
});
// getTwenty :: Account -> String
const getTwenty = compose(maybe('You\'re broke!', finishTransaction), withdraw(20));
getTwenty({ balance: 200.00 });
// 'Your balance is $180.00'
getTwenty({ balance: 10.00 });
// 'You\'re broke!'
```

With `maybe`, we are witnessing the equivalent of an if/else statement whereas with map, the imperative analog would be: `if (x !== null) { return f(x) }`

### Pure Error Handling
`throw/catch` is not very pure.
`Either`: we can respond with a polite message instead of declaring a war on input.

```js
class Either {
  static of(x) {
    return new Right(x);
  }
  constructor(x) {
    this.$value = x;
  }
}
class Left extends Either {
  ...
}
class Right extends Either {
    ...
}
const left = x => new Left(x);
```

`Left` and `Right` are two subclasses of an abstract type we call `Either`.

```
Either.of('rain').map(str => `b${str}`);
// Right('brain')
left('rain').map(str => `It's gonna ${str}, better bring your umbrella!`);
// Left('rain')
Either.of({ host: 'localhost', port: 80 }).map(prop('host'));
// Right('localhost')
left('rolls eyes...').map(prop('host'));
// Left('rolls eyes...')
```
We are short-circuiting our app when we return a `Left`. But now we have a clue as to why our program has derailed.

```js
error message or the age back.
// fortune :: Number -> String
const fortune = compose(concat('If you survive, you will be '), toString, add(1));
// zoltar :: User -> Either(String, _)
const zoltar = compose(map(console.log), map(fortune), getAge(moment()));
```
It reads as one linear motion from right to left rather than climbing through the curly braces of a conditional statement.

We use `_` in the right branch's type signature to indicate it's a value that should be ignored

`fortune` is completely ignorant of any functions.

A function can be surrounded by map , which transforms it from a non-functory function to a functory one.

This process is called lifting.

Functions tend to be better off working with normal data types rather than container types, then lifted into the right container as deemed necessary.

`Either` is great for casual errors like validation as well as more serious, stop the show errors like missing files or broken sockets.

`Either` captures logical disjunction (a.k.a || ) in a type. It also encodes the idea of a Coproduct from category theory. It is the canonical sum type (or disjoint union of sets).

There are many things `Either` can be, but as a functor, it is used for its error handling.

We have little `either`, which behaves similarly to `maybe`, but takes two functions instead of one and a static value.

```js
// either :: (a -> c) -> (b -> c) -> Either a b -> c
const either = curry((f, g, e) => {
  let result;
  switch (e.constructor) {
    case Left:
      result = f(e.$value);
      break;
    case Right:
      result = g(e.$value);
      break;
     // No Default
  }
  return result;
});

// zoltar :: User -> _
const zoltar = compose(console.log, either(id, fortune), getAge(moment()));
zoltar({ birthDate: '2005-12-12' });
// 'If you survive, you will be 10'
// undefined
zoltar({ birthDate: 'balloons!' });
// 'Birth date could not be parsed'
// undefined
```

`id` parrots back the value in the Left to pass the error message to console.log

### Old McDonald Had Effects...
Reach inside of the container and get at its contents => `IO`.
When we map over our IO , we stick that function at the end of a composition which, in turn, becomes the new `$value` and so on.

Let's rename `$value` property to `unsafePerformIO` to remind our users of its volatility.

```js
class IO {
  constructor(io) {
    this.unsafePerformIO = io;
  }
  map(fn) {
    return new IO(compose(fn, this.unsafePerformIO));
  }
}
```

`findParam('searchTerm').unsafePerformIO()` instead of
`findParam('searchTerm').$value`

### Asynchronous Tasks
Instead of callback hell:
`Folktale` `Data.Task`

`Task` playing the role of our promise. The function `map` works as `then`.

Like IO , Task will patiently wait for us to give it the green light before running. To run our `Task`, we must call the method fork. This works like `unsafePerformIO`.

```js
// -- Pure application -------------------------------------------------
// blogPage :: Posts -> HTML
const blogPage = Handlebars.compile(blogTemplate);
// renderPage :: Posts -> HTML
const renderPage = compose(blogPage, sortBy('date'));
// blog :: Params -> Task Error HTML
const blog = compose(map(renderPage), getJSON('/posts'));
// -- Impure calling code ----------------------------------------------
blog({}).fork(
  error => $('#error').html(error.message),
  page => $('#main').html(page),
);
$('#spinner').show();
```

We just read bottom to top, right to left, instead of bouncing between callbacks.

Even with Task , our IO and Either functors are not out of a job.
Task takes care of the impurities of reading a file asynchronously, but we still deal with validating the config with Either and wrangling the db connection with IO .

### A Spot of Theory
Useful properties:
```js
// identity
map(id) === id;
// composition
compose(map(f), map(g)) === map(compose(f, g));
```
In category theory, functors take the objects and morphisms of a category and
map them to a different category. This new category must have an identity and the ability to compose morphisms.

You can think of a category as a network of objects with morphisms that connect them.

If an object `a` is in our source category `C`, when we map it to category `D` with functor `F`, we refer to that object as `F a`.

`Maybe` maps our category of types and functions to a category where each object may not exist and each morphism has a `null` check. (in code it's achieved by surrounding each function with map and each type with our functor)

If we have identity and associative composition we have a category.

## Chapter 09: Monadic Onions

> A pointed functor is a functor with an `of` method

The motivation for this interface is a common, consistent way to place a value into our functor without the complexities and specific demands of constructors.
E.g. `IO` and `Task` constructors require a function as their arg, and `Maybe` and `Either` don't.

`Left.of` doesn't make any sense. Each functor must have one way to place a value inside it and with `Either` , that's `new Right(x)`, so we define `Right.of`.

`pure` , `point` , `unit` , and `return` -- synonyms of `of`.

### Mixing Metaphors
Monads are like onions.
Sometimes we get a nested functor situation like `IO(IO('['))` or `Maybe(Maybe(Maybe({name: 'Mulburry', number: 8402})))`

We can use `join`, if we have 2 layers of the same type:
```js
const mmo = Maybe.of(Maybe.of('nunchucks'));
// Maybe(Maybe('nunchucks'))
mmo.join();
// Maybe('nunchucks')
```

> Monads are pointed functors that can flatten
Any functor which defines a join method, has an of method, and obeys a few
laws is a monad.

```js
Maybe.prototype.join = function join() {
  return this.isNothing() ? Maybe.of(null) : this.$value;
};
```
For `IO`:
`IO.prototype.join = () => this.unsafePerformIO();`


### My Chain Hits My Chest
We often end up calling `join` right after a `map`.
Let's abstract this into a function called chain .
```js
// chain :: Monad m => (a -> m b) -> m a -> m b
const chain = curry((f, m) => m.map(f).join());
// or
// chain :: Monad m => (a -> m b) -> m a -> m b
const chain = f => compose(join, map(f));
```

`chain` is also called `>>=` (bind) or `flatMap`.
So we can refactor `map/join` pair with `chain`.

Remember to map when returning a "normal" value and chain when we're returning another functor.

As a reminder, this does not work with two different nested types.

### Power Trip
We sometimes struggle to understand how many containers deep a value is or if we need `map` or `chain`.

Declarative `upload`:
```js
// readFile :: Filename -> Either String (Task Error String)
// httpPost :: String -> Task Error JSON
// upload :: String -> Either String (Task Error JSON)
const upload = compose(map(chain(httpPost('/uploads'))), readFile);
```
we protect against 3 errors.
Imperative `upload` would look much more complex.

### Theory
```js
// associativity
compose(join, map(join)) === compose(join, join);
```
```js
// identity for all (M a)
compose(join, of) === compose(join, map(of)) === id;
```
It states that, for any monad `M`, `of` and `join` amounts to `id`

Identity and associativity are the laws of the category.

```js
const mcompose = (f, g) => compose(chain(f), g);
// left identity
mcompose(M, f) === f;
// right identity
mcompose(f, M) === f;
// associativity
mcompose(mcompose(f, g), h) === mcompose(f, mcompose(g, h));
```

## Chapter 10: Applicative Functors
The ability to apply functors to each other.
E.g. `add` `Container.of(3)` to `Container.of(2)`
We can achieve this with `chain`. `map` and partially applied `add(2)`, but we are stuck in the sequential world of monads wherein
nothing may be evaluated until the previous monad has finished its business.
```js
Container.of(2).chain(two => Container.of(3).map(add(two)));
```
It would be lovely if we could succinctly apply one functor's contents to
another's value without these needless functions and variables.

### Ships in Bottles
`ap` is a function that can apply the function contents of one functor to the value contents of another.

```js
Container.of(add(2)).ap(Container.of(3));
// Container(5)
Container.of(2).map(add).ap(Container.of(3));
```

> An applicative functor is a pointed functor with an `ap` method

F.of(x).map(f) === F.of(f).ap(F.of(x));

Mapping `f` is equivalent to `ap`-ing a functor of `f`.

`Task` - prime situation where applicative functors pull their weight.

### Coordination Motivation
```js
Task.of(renderPage).ap(Http.get('/destinations')).ap(Http.get('/events'));
// Task("<div>some page with dest and events</div>")
```
2 calls instantly + `renderPage` when they're resolved.

### A pointfree way to write these applicative calls.
```js
const liftA2 = curry((g, f1, f2) => f1.map(g).ap(f2));
const liftA3 = curry((g, f1, f2, f3) => f1.map(g).ap(f2).ap(f3));
```

### Operators
`<$>` is `map` (aka `fmap`)
`<*>` is just `ap`

### Free Can Openers
`of/ap` is equivalent to `map`



### Laws
#### Identity
```js
// identity
A.of(id).ap(v) === v;
```

#### Homomorphism
```js
// homomorphism
A.of(f).ap(A.of(x)) === A.of(f(x));
```

#### Interchange
```js
// interchange
v.ap(A.of(x)) === A.of(f => f(x)).ap(v);
```

#### Composition
```js
// composition
A.of(compose).ap(u).ap(v).ap(w) === u.ap(v.ap(w));
```

## Chapter 11: Transform Again, Naturally
### Curse This Nest
Nesting -- having two or more different types all huddled together around a value.
```js
Right(Maybe('b'));
IO(Task(IO(1000)));
[Identity('bee thousand')];
```
We should keep our types organized, or our code will be too complicated.

### All Natural
A Natural Transformation is a "morphism between functors", that is, a function
which operates on the containers themselves.
```js
// nt :: (Functor f, Functor g) => f a -> g a
compose(map(f), nt) === compose(nt, map(f));
```
We can run our natural transformation then map or map then run our natural transformation and get the same result.

### Principled Type Conversions
Like usual type conversion (e.g. Integer to Float), but with algebraic containers.

```js
// idToMaybe :: Identity a -> Maybe a
const idToMaybe = x => Maybe.of(x.$value);
// idToIO :: Identity a -> IO a
const idToIO = x => IO.of(x.$value);
```

We're just changing one functor to another.
We are permitted to
lose information along the way so long as the value we'll map doesn't get lost.
map must carry on, even after transformations.
One way to look at it is that we are transforming our effects.
`ioToTask` - converting sync to async
`arrayToMaybe` - nondeterminism to possible failure
But we cannot convert asynchronous to synchronous in js -- no `taskToIo`.

### Feature Envy
Using some features from another type like `sortBy` on a `List`.

```js
// arrayToList :: [a] -> List a
const arrayToList = List.of;
const doListyThings_ = compose(sortBy(h), filter(g), map(f), arrayToList); // law applied
```

### Isomorphic JavaScript
*isomorphism* - we can go back and forth without loosing any information.

We say that two types are isomorphic if we can provide the "to" and
"from" natural transformations as proof.

```js
// promiseToTask :: Promise a b -> Task a b
const promiseToTask = x => new Task((reject, resolve) => x.then(resolve).catch(reject));
// taskToPromise :: Task a b -> Promise a b
const taskToPromise = x => new Promise((resolve, reject) => x.fork(reject, resolve));
const x = Promise.resolve('ring');
taskToPromise(promiseToTask(x)) === x;
const y = Task.of('rabbit');
promiseToTask(taskToPromise(y)) === y;
```

### A Broader Definition
These structural functions aren't limited to type conversions by any means.
```js
reverse :: [a] -> [a]
join :: (Monad m) => m (m a) -> m a
```
The natural transformation laws hold for these functions too.

### One Nesting Solution

### In Summary
Natural transformations are functions on our functors themselves.

## Chapter 12: Traversing the Stone
### Types n' Types
Sometimes you end up with an array of functors, which you want to join.

### Type Feng Shui
`sequence`
```js
sequence(List.of, Maybe.of(['the facts'])); // [Just('the facts')]
sequence(Either.of, [Either.of('wing')]); // Right(['wing'])
sequence(IO.of, Either.of(IO.of('buckle my shoe'))); // IO(Right('buckle my shoe'))
sequence(Task.of, left('wing')); // Task(Left('wing'))
```

The inner functor is shifted to the outside and vice versa.
```js
const sequence = curry((of, x) => x.sequence(of));
```
### Effect Assortment
Different orders have different outcomes where our containers are concerned.
`[Maybe a]` - collection of possible values, keep "good values"
`Maybe [a]` - possible collection, all or nothing

Types can be swapped to give us different effects:
```js
// fromPredicate :: (a -> Bool) -> a -> Either e a
// partition :: (a -> Bool) -> [a] -> [Either e a]
const partition = f => map(fromPredicate(f));
// validate :: (a -> Bool) -> [a] -> Either e [a]
const validate = f => traverse(Either.of, fromPredicate(f));
```
`partition` will give us an array of `Left`s and `Right`s according to the predicate function.
`validate` - give us the first item that fails the predicate in `Left` , or all the items in `Right`

```js
traverse(of, fn) {
  return this.$value.reduce(
    (f, a) => fn(a).map(b => bs => bs.concat(b)).ap(f),
    of(new List([])),
  );
}
```

Code in `List.traverse` is accomplished with `of` , `map` and `ap` , so will
work for any Applicative Functor.

### Waltz of the Types
Using `traverse` instead of `map` , we've successfully herded those unruly `Task`s into a nice coordinated array of results.
```js
// readFile :: FileName -> Task Error String
// firstWords :: String -> String
const firstWords = compose(join(' '), take(3), split(' '));
// tldr :: FileName -> Task Error String
const tldr = compose(map(firstWords), readFile);
traverse(Task.of, tldr, ['file1', 'file2']);
// Task(['hail the monarchy', 'smash the patriarchy']
```
Instead of `map(map($))` we have `chain(traverse(IO.of, $))` which inverts our types as it maps then flattens the two `IO`s via `chain`.

```js
// getAttribute :: String -> Node -> Maybe String
// $ :: Selector -> IO Node
// getControlNode :: IO (Maybe Node)
const getControlNode = compose(chain(traverse(IO.of, controls')), $);
```

### No Law and Order
#### Identity
```js
const identity1 = compose(sequence(Identity.of), map(Identity.of));
const identity2 = Identity.of;
// test it out with Right
identity1(Either.of('stuff'));
// Identity(Right('stuff'))
identity2(Either.of('stuff'));
// Identity(Right('stuff'))
```

#### Composition
```js
const comp1 = compose(sequence(Compose.of), map(Compose.of));
const comp2 = (Fof, Gof) => compose(Compose.of, map(sequence(Gof)), sequence(Fof));
// Test it out with some types we have lying around
comp1(Identity(Right([true])));
// Compose(Right([Identity(true)]))
comp2(Either.of, Array)(Identity(Right([true])));
// Compose(Right([Identity(true)]))
```
If we swap compositions of functors, we shouldn't see any surprises since the composition is a functor itself.

#### Naturality
```js
const natLaw1 = (of, nt) => compose(nt, sequence(of));
const natLaw2 = (of, nt) => compose(sequence(of), map(nt));
// test with a random natural transformation and our friendly Identity/Right functors.
// maybeToEither :: Maybe a -> Either () a
const maybeToEither = x => (x.$value ? new Right(x.$value) : new Left());
natLaw1(Maybe.of, maybeToEither)(Identity.of(Maybe.of('barlow one')));
// Right(Identity('barlow one'))
natLaw2(Either.of, maybeToEither)(Identity.of(Maybe.of('barlow one')));
// Right(Identity('barlow one'))

traverse(A.of, A.of) === A.of;
```
If we first swing the types around then run a natural transformation on the outside, that should equal mapping a natural transformation, then flipping the types.

### In Summary
Traversable is a powerful interface that gives us the ability to rearrange our
types with the ease of a telekinetic interior decorator.









































































