## Chapter 2. Tokenization and Parsing
Ruby converts your code into two different for-
mats. First, it converts the text in your Ruby program into a series of tokens.
Next, it uses an LALR parser to convert the input stream of tokens into a
data structure called an abstract syntax tree.

## Chapter 3. Compilation
Ruby converts your code into a third format:
a series of bytecode instructions that are later used when your program is actually executed.

Ruby 1.8 executes your program directly from the AST.

Starting with version 1.9, Ruby compiles your code before executing it.
Translate your code from one programming language to another

Ruby compiler translates your Ruby code into another language that Ruby’s virtual machine understands.

But unlike C or Java Ruby’s compiler runs automatically without you ever knowing.

Ruby’s compiler works by iterating through the AST produced by the tokenizing and parsing processes, generating a series of bytecode instructions along the way.

Ruby code is compiled to the YARV virtual machine language.

When you run any Ruby program, you are actually using a virtual
machine designed specifically to execute Ruby programs.

## Chapter 4. Control Structures and Method Dispatch
YARV’s control structures
internally Ruby categorizes methods into 11 types.
ISEQ - def
Ruby labels its own built-in method as `CFUNC`
Ruby uses a hash to track the argument labels and default values.

## Chapter 5. Objects and Classes
Ruby uses the `RObject` structure to represent instances of any custom
classes you define in your code and of some classes predefined by Ruby
itself.
> Every Ruby object is the combination of a class pointer and an array of instance variables.

Ruby uses special C structures to represent instances of many commonly used, built-in Ruby classes called ”generic” objects.

Ruby uses the RString structure to represent an instance of the
String class, RArray for an instance of the Array class, or RRegexp for an
instance of the Regexp class.

The `RClass` structure working with the rb_classext_struct structure saves a large set of information

> A Ruby class is a Ruby object that also contains method definitions,
> attribute names, a superclass pointer, and a constants table.

Ruby saves both instance and class variable types in the same hash table.
Classes also contain a series of hash tables that store their methods.
Each Ruby class records its superclass using the super pointer.

## Chapter 6. Method Lookup and Constant Lookup


## Chapter 12 Garbage Collection
Clean up objects your program no longer uses.
MRI uses the same GC algorithm John McCarthy invented over 50 years ago: mark-and-sweep garbage collection.

JRuby and Rubinius use copying garbage collection, they also employ `generational garbage collection` and `concurrent garbage collection`.

Garbage Collectors Solve Three Problems:
- allocate memory for use by new objects
- identify which objects your program is no longer using.
- reclaim memory from unused objects.

When you create a new Ruby object, the garbage collector allocates memory for that object. Later, Ruby’s garbage collector determines when your program has stopped using the object so it can reuse that memory to create new Ruby objects.

### Mark and Sweep
Internally all Ruby objects are represented by a C structure called RVALUE .

MRI sets the length of the initial free list to about 10,000 RVALUE structures, which means that MRI can create 10,000 Ruby objects without allocating more memory.

Ruby divides the allocated memory into subsections known as heaps in the MRI source code, each about 16k in size, creates a free list for each of these heaps.

Eventually MRI uses up all remaining objects on the free list. At that point, the GC system stops your program, identifies objects that your code is no longer using, and reclaims their memory for allocation to new objects. I

If no unused objects are found, Ruby asks the operating system for more memory. No memory ==> throw `OutOfMemory`` exception.

The Lazy Sweeping algorithm reduces the amount of time a program is stopped by the garbage collector, sweeps only enough garbage objects back to the free
list to create a few new Ruby objects.

Main disadvantage -- the program has to stop.

### Copying Garbage Collector
JRuby and Rubinius use another GC:
- they allocate memory for new objects and reclaim memory from garbage objects using an algorithm called copying garbage collection.
- handle old and young Ruby objects differently using generational
garbage collection.
- use concurrent garbage collection to perform some GC tasks at the
same time that your application code is running.

Copying garbage collectors allocate memory for new objects from
a single large heap or memory segment.

When that memory segment is used up, these collectors copy only the live objects over to a second memory segment, leaving the garbage objects behind.

### Generational Garbage Collection
- a technique that treats new objects differently than older ones (young vs mature)

#### Using the Semi-Space Algorithm for Young Objects
Young objs become garbage frequently.
When the Eden heap fills up with new objects, the garbage collector identifies most of them as garbage because new objects usually die young.
Fewer live => fewer copying.

When an object has survived several copyings, it's promoted to "mature".

#### Garbage Collection for Mature Objects
It's needed to run GC against them much less frequently.

#### Concurrent Garbage Collection
the garbage collector runs at the same time as your application code.
Concurrent garbage collectors run in a separate thread from the pri-
mary application.
Most computers today contain microprocessors with multiple cores, which allow different threads to run in parallel, 1 core is dedicated to the GC thread.

> MRI Ruby 2.1 also supports a form of concurrent garbage collection by performing the sweep portion of the mark-and-sweep algorithm in parallel while your Ruby code continues to run.
























