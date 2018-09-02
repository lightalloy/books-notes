## Chapter 10. Collections central: Enumerable and Enumerator

Mixing `Enumerable` to your own classes:
```ruby
class C
  include Enumerable
end
```
Enumerators are objects that encapsulate knowledge of how to iterate through a particular collection.

### 10.1 Gaining enumerability through each
`each` yields items to a supplied code block, one at a time.
`each` does different things for different types of objects.
```ruby
class Rainbow
  include Enumerable
  def each
    yield "red"
    yield "orange"
    yield "yellow"
    yield "green"
    yield "blue"
    yield "indigo"
    yield "violet"
  end
end

r = Rainbow.new
r.each do |color|
  puts "Next color: #{color}"
end

y_color = r.find {|color| color.start_with?('y') }
Enumerable.instance_methods(false).sort
```

Defining each , together with mixing in Enumerable , buys you a great deal of functionality for your objects. Much of the searching and querying functionality you see in Ruby arrays, hashes, and other collection objects comes directly from Enumerable - `map`, `select`, `find`, etc

### 10.2 Enumerable Boolean queries
`any?`, `one?`, `none?`, `include?`

```ruby
states.any? {|state| state =~ / / }
states.one? {|state| state =~ /West/ }
```

For hashes:
```ruby
states.include?("Louisiana") # consults keys
states.all? {|state, abbr| state =~ / / }
```
### 10.3 Enumerable searching and selecting
`find`, `find_all`, `reject`

`grep`
```ruby
colors = %w{ red orange yellow green blue indigo violet }
colors.grep(/o/)
```
`group_by`:
```ruby
colors.group_by {|color| color.size }
```

`partition`:
```ruby
 enum.partition {| obj | block }  => [ true_array, false_array ]
```

### 10.8 Sorting enumerables
1 Define a comparison method for the class ( <=> ).
2 Place the multiple instances in a container, probably an array.
3 Sort the container.


```ruby
class Painting
 ...
  def <=>(other_painting)
    self.price <=> other_painting.price
  end
end

paintings.sort
```
#### 10.8.2 Defining sort-order logic with a block
```ruby
year_sort = [pa1, pa2, pa3, pa4, pa5].sort do |a,b|
  a.year <=> b.year
end
```

### 10.9 Enumerators and the next dimension of enumerability
An iterator is a method that yields one or more values to a code block. An enumerator is an object, not a method.

An enumerator is a simple enumerable object. It has an each method. It has an each method, and it employs the Enumerable module to define all the usual methods— select , inject , map , and friends—directly on top of its each .

#### 10.9.1 Creating enumerators with a code block
```ruby
e = Enumerator.new do |y|
  y << 1
  y << 2
  y << 3
end
```
`y` is a yielder, an instance of `Enumerator::Yielder`, automatically passed to your block.
y.yield(1) - same

Upon being asked to iterate, the enumerator consults the yielder and makes the next move—the next yield—based on the instructions that the yielder has stored.

The enumerator e is an enumerating machine
