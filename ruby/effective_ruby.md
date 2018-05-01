## Spaceship <=>
- Implement object ordering by defining a “ <=> ” operator and including the Comparable module
- The <=> operator should return nil if the left operand can’t be compared
with the right.
- If you implement `<=>`` for a class you should consider aliasing eql? to
`==`

## Prefer Class Instance Variables to Class Variables
Class variables (those which begin with “ @@ ”) are handled differently. They’re attached to a class and visible to all instances of that class.

Classes are objects and therefore have their own private set of instance
variables.

## Duping Cloning
`dup` returns unfrozen object
`clone` can return frozen (if the caller is frozen)

dup and clone return shallow copies!

## Manage Resources with Blocks and ensure
Write an ensure clause to release any acquired resources.
Use the block and ensure pattern with a class method to abstract away
resource management.
```ruby
class Lock
  def self.acquire
    lock = new # Initialize the resource
    lock.exclusive_lock!
    yield(lock) # Give it to the block
  ensure
  # Make sure it gets unlocked.
    lock.unlock if lock
  end
end
```
## Know the Difference Between the Variants of eval
Passing binding:
```ruby
def glass_case_of_emotion
  x = "I'm in a " + __method__.to_s.tr('_', ' ')
  binding
end
x = "I'm in a test"
p eval('x')
p eval('x', binding)
p eval("x", glass_case_of_emotion)
```
`instance_eval`:

The object you invoke `instance_eval` on becomes the context for the evaluation. This allows you to reach into an object and access its private methods and instance variables.

```ruby
Widget.instance_eval do
  def table_name; "widgets"; end
end
Widget.table_name
Widget.singleton_methods(false)
```
`class_eval` evaluates a string or a block in the context of a class.

```ruby
Widget.class_eval do
  attr_accessor(:name)
  def sold?; false; end
end
```
`class_eval` is defined in the Module module as a singleton method which means it can only

`instance_exec`, `class_exec`. `module_exec`
The eval versions accept strings or blocks the exec variants only accept blocks.

















be used on modules and classes.
























