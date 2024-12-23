---
title: "02. Essential Python"
author: Einar Balan, Ashwin Ranade
layout: lecture
parent: Lecture Notes
---

{: .note }
This goal of this lecture is to get you ready for the upcoming quarter long project! It covers all the essential stuff about Python that you definitely did not learn in CS 35L... and that's not meant to be sarcastic you literally have not learned about what's covered here.[^1] It's all pretty cool! .

[^1]: Some more CS 131 lore: once upon a time Carey spent a lot more time covering Python, but this year he's decided to cut down on that to spend more time on other aspects of the class.

## Table of Contents
{: .no_toc }

{:toc}
- yo


## Learning Python through Challenges

This lecture is all about Python, which you'll need to be fairly familiar with for the upcoming projects! Through a series of challenges, we'll go through many aspects of Python that you may have overlooked when you first learned it.

### Object Allocation

Consider a Circle class with versions written in both C++ and Python:

```cpp
#include <cmath>

class Circle {
public:
  Circle(double rad)
    { this->rad = rad; }

  double area() const
    { return M_PI * rad * rad; }

  void setRadius(double rad)
    { this->rad = rad; }

private:
  double rad;
};
```

with the Python equivalent:

```py
import math

class Circle:
  def __init__(self, rad):
    self.rad = rad

  def area(self):
    return math.pi * self.rad**2

  def set_radius(self, rad):
    self.rad = rad
```

In C++, there are two ways we can define an object:

```cpp
// on the stack - "no pointer"
Circle c(10);
cout << c.area();

// on the heap - "with a pointer"
Circle *p = new Circle(20);
cout << p->area();
```

In Python, we can only do

```py
c = Circle(10)
print(c.area())
```

How does Python allocate objects - is it directly/on-the-stack, or with pointer indirection? As a hint, the following is valid code:

```py
c = Circle(10)
print(c.area())
c = None # similar to nullptr in C++
```
<details>
<summary><b>Answer</b></summary>

Python uses something <i>like</i> pointers, called an <b>object reference</b>. Every Python object is allocated on the heap. Unlike C++, we don't use any special syntax to "dereference" a pointer.
</details>


{: .note }
In Python, *all* variables are object references. This applies to primitives in other languages, like numbers -- `42`. On reassignment, we don't "change" the allocated `42`; instead, we allocate a new number, and point to it. Though, this is mostly abstracted away from you as a programmer!

This affects how we call methods too:

```py
# Circle class in Python
import math

class Circle:

  def __init__(self, rad):
    self.rad = rad

  def area(self):
    return math.pi * self.rad**2

  def set_radius(self, rad):
    self.rad = rad

c = Circle(10)
print(c.area()) # here, "self" points to c!
```

### Class Variables

Consider this module `thing.py`:

```py
# thing.py
class Thing:
 thing_count = 0

 def __init__(self, v):
  self.value = v
  Thing.thing_count += 1
  self.thing_num = Thing.thing_count
```

What will the following print? Why?

```py
import thing

t1 = Thing("a")
print(f"{t1.thing_num} {Thing.thing_count}")


t2 = Thing("b")
print(f"{t1.thing_num} {Thing.thing_count}")
print(f"{t2.thing_num} {Thing.thing_count}")
```

<details>
<summary><b>Answer</b></summary>
<pre>
1 1
1 2
2 2
</pre>
</details>

Why?

- `thing_count` is a *class* variable
- when a class is defined, Python creates a special "class" object that's *shared* by all instances of the class; it represents the *entire* class
- so, we can think of `thing_count` as being "shared" across different instances of `Thing`
- we can access class (member) variables with the class name - like `Thing.thing_count += 1`
- in contrast, `self.value` is an *instance* (member) variable

{: .note }
You can also have class methods, which are methods that are shared across all instances of a class. If you're familiar with static methods in other languages, class methods work similarly.

You generally want to use class variables and methods that only operate on state shared across the *entire* class (or have no state at all), and don't need any information about the instances. 

### Copying of Objects

```py
# Circle class in Python
import math

class Circle:

  def __init__(self, rad):
    self.rad = rad

  def area(self):
    return math.pi * self.rad**2

  def set_radius(self, rad):
    self.rad = rad
```

What happens when we run this code?

```py
c = Circle(10)
print(c.area())
c2 = c
c.set_radius(0)
print(c2.area())  # What happens?!?!?
```

<details>
<summary><b>Answer</b></summary>

It prints 0! This is a consequence of Python's object reference behaviour. Assignment does not copy by default, it simply points the new variable to the same object reference!
</details>

Luckily, there is a way to accomplish the copy behaviour we're hoping for. One way to do that is with the `copy.deepcopy()` function:

```py
import copy

c = Circle(10)
print(c.area())

c2 = copy.deepcopy(c)
c.set_radius(0)
print(c2.area) # not 0!
```

"Deep copying" recurisvely makes a copy of the top-level object, and *every* object it refers to.

In contrast, shallow copying - done with `copy.copy()` - copies just the top level object (but not recursively).

### Object Equality

What does the following print?

```py
# Different types of equality in Python
fav = 'pizza'
a = f'I <3 {fav}!'
b = f'I <3 {fav}!'
c = a

if a == b:
  print('Both objects have same value!')
if c is a:
  print('c and a refer to the same obj')
if a is not b:
  print('a and b refer to diff. objs')
```

<details>
<summary><b>Answer</b></summary>
<pre>
Both objects have same value!
c and a refer to the same obj
a and b refer to diff. objs
</pre>
</details>

Core ideas:

- `==` looks at *values*, not references
- `is` looks at *references*, not values

### Inheritance

One key OOP feature is inheritance. Consider:

```py
class Person:
  def __init__(self, name):
    self.name = name

  def talk(self):
    print('Hi!\n')

class Student(Person):
  def __init__(self, name):
    super().__init__(name)
    self.units = 0

  def talk(self):
    print(f"Heya, I'm {self.name}.")
    print("Let's party! Oh... and ")
    super().talk()
```

```py
def chat(p):
 p.talk()

def cs131_lecture():
 s = Student('Angelina')
 chat(s)
```

Some questions:

1. How does a derived class inherit from a base class?
2. How does a derived class method override a base class method?
3. How does a derived method call a base-class method?

<details>
<summary><b>Answer</b></summary>

<ol>
<li>By placing the base class in the parens (e.g. `Student(Person)`) </li>
<li>"Virtual by default" - as long as they have the same name!</li>
<li>Using the `super()` prefix!</li>
</ol>
</details>



**You need to explicitly call the base class constructor!**

### Strings

In Python, each String is an object just like our Circle objects. However, unlike some other languages, Strings are actually immmutable. This means that they can't be modified once created. Consider the following example:

```python
fact1 = 'Del Taco rules! '
fact2 = 'CS131 is lit! '

truth = fact1 + fact2

truth += 'I have spoken.'
```

Upon first galnce you might think that the last line mutates the `truth` variable, but in reality it creates an entirely new string object. This new object consists of the previous contents of  `truth` and `'I have spoken'`. `truth` is then reassigned to point at this new variable. Pretty neat!

### How are Lists Implemented?

In Python, random access on a list is super fast! We can get any jth element in O(1) time. Based on this, how do you think lists might be implemented?

<details>
<summary><b>Answer</b></summary>

Lists are implemented as dynamically allocated arrays, with each entry containing an object reference. In C++ terms, you can think of it as a vector of pointers!
</details>

You can also have lists of lists just like in any other language. Consider the following program. What do you think it will print?

```python
# Lists of lists (lol) in Python

primes = [1,2,3]
odds = ['carey', 'todd']

lol = [primes, odds]

primes[2] = 7
odds.append('paul')
odds = [1,3,5]

print(lol)
```

<details>
<summary><b>Answer</b></summary>


<pre>
[[1,2,7], ['carey', 'todd', 'paul']]
</pre>
<br>
The final assignment doesn't affect <code>lol</code> because it simply reassigns the object reference odds points to. It does nothing to affect the obejct reference in the list <code>lol</code>!
</details>


### Parameter Passing

If you think back to CS 32, you'll remember that C++ has three different ways to pass parameters into functions: by-value, by-reference, and by-pointer. Each results in drastically different behaviour!

In Python things are a bit simpler. There is only one approach called *pass by object reference* (which is identical to passing by pointer). We'll demonstrate this with the following program:

```python
def compute_cost(r):
 COST_PER_SQFT = 10.50
 return r.area() * COST_PER_SQFT

window = Rectangle(3,4)  # 3' by 4' window
cost = compute_cost(window)
print(f'The cost of your window is ${cost}')
```

When we call `compute_cost` with `window`, Python passes `window`'s object reference to the function and assigns it the name `r`. In other words `r` points to the `window`. Therefore if we modify the **contents** of `r` in any way, these changes will be reflected in `window` outside of the function.

With this in mind, try to determine what each of the following Python programs print:

```python
def nerdify(s):
 s = 'coding ' + s

i_like = 'parties'
nerdify(i_like)
print(i_like)
```

<details>
<summary><b>Answer</b></summary>


<pre>
parties
</pre>
<br>
Reassigning s within nerdify does not affect which object i_like refers to!
</details>

```python
def peachify(f):
 f = f + ['peach']

fruits = ['apple', 'cherry']
peachify(fruits)
print(fruits)
```
<details>
<summary><b>Answer</b></summary>


<pre>
['apple', 'cherry']
</pre>
<br>
Reassigning f within peachify does not affect which object fruits refers to!
</details>

```python
def peachify2(f):
 f.append('peach')

fruits = ['apple', 'cherry']
peachify2(fruits)
print(fruits)
```
<details>
<summary><b>Answer</b></summary>


<pre>
['apple', 'cherry', 'peach']
</pre>
<br>
Here, we're direclty modifying the object within peachify2, so the change is reflected by fruits as well.
</details>

```python
def peachify3(f):
 f[0] = 'peach'

fruits = ['apple', 'cherry']
peachify3(fruits)
print(fruits)
```
<details>
<summary><b>Answer</b></summary>


<pre>
['peach', 'cherry']
</pre>
<br>
Once again, we're directly modifying the object within peachify3, so the change is reflected by fruits as well.
</details>

```python
def largeify(c):
 c = Circle(10)

unit = Circle(1)
largeify(unit)
print(unit.radius())
```
<details>
<summary><b>Answer</b></summary>


<pre>
1
</pre>
<br>
Reassigning c within peachify does not affect which object unit refers to!
</details>

```python
def largeify2(c):
 c.set_radius(10)

unit = Circle(1)
largeify2(unit)
print(unit.radius())
```
<details>
<summary><b>Answer</b></summary>

<pre>
10
</pre>
<br>
We're directly modifying the object within largeify2, so the change is reflected by unit as well.
</details>

### Default Parameters

Python has a cool feature that allows you to set default values for parameters. However, it can lead to nasty bugs if you're not careful. Consider this program:

```python
# A puzzling function
def puzzle(w, words = []):
  words.append(w)
  print(words)

puzzle("This")
puzzle("is")
puzzle("confusing!")
```

It ends up outputting the following, which is a bit unexpected!

```
["This"]
["This", "is"]
["This", "is", "confusing"]
```

Here's what's happening: when you define a function with a default parameter that is mutable (i.e. a list, dictionary, or a set), Python will create a single object for that default parameter before the function is even called. Then, everytime we call that function, that same object will be used for that parameter. So any changes to that object will be reflected in future function calls! Moral of the story: be careful with default parameters for mutable types.

To fix this, we can do something like the following:

```python
# A (less) puzzling function
def puzzle(w, words = None):
  if words is None:
    words = []  
  words.append(w)
  print(words)
```

----

## Appendix

{: .note }
Everything that follows is the old intro to the Python section. Carey didn't cover it this year to try to streamline the Python part of the class. It covers slides 1-36 of [Python Palooza](https://docs.google.com/presentation/d/1Tool2iOyv022hq5uBF8AjR091CZmu2vM/edit?usp=share_link&ouid=106377371757242811146&rtpof=true&sd=true). Take a look if you're curious, but you won't be responsible for the content here.

### Just a Bit on Python
{: .no_toc }

What do people use Python for?

- quick and dirty scripting
- industrial scripts
- web backends (ex Flask and Django)
- data science (pandas, numpy) and machine learning (pytorch, tensorflow)

Generally, people say to avoid python for writing efficient programs. Among other things, this is because Python is "interpreted".

{: .note }
There are alternative implemenations of Python that aren't interpreted, but are rather compiled!

But, people often use it in machine learning, which is a compute-intensive application. What gives? Well, Python can use something called a "C extension" - calling code written in other higher-performance languages like C and C++. This is what powers most data science and machine learning libraries like pytorch and tensorflow do under-the-hood!

#### "Pythonic"
{: .no_toc }

Compared to other languages, Python is relatively opinionated on the right way to do things. Doing things the right way is often called "Pythonic"; we'll use this word throughout these notes.

### Learning Python through Challenges - Basics
{: .no_toc }

Much of this lecture is learning program characteristics through examples. Let's get started!

(implicitly, we're assuming some existing Python knowledge from CS35L and other prerequisites)

#### Program Execution
{: .no_toc }

Consider this following code:

```py
# deltaco.py
def lets_go(person):
  print('Hey, ' + person)
  print("Let's go to Del Taco!")

lets_go('Carey')

def main():
  print('Oh crap, now I have gas!')

print('Nom nom')
```

When we run it, we get:

```
$ python3 deltaco.py
Hey, Carey
Let's go to Del Taco!
Nom nom
```

Pause. What does this tell us about Python?

1. the Python interpreter runs from top-down
2. defining a function with `def` does not run it
3. even if we have a function called `main()`, **it won't always be run**!

So, how would we define a main function? We will use a "magic" piece of code like this:

```py
if __name__ == "__main__":
  main()
```

When we run a python script from the command line, the interpreter will set `__name__` to `"__main__"`. It's a "special variable"!

Why not just do a top-level call to `main()`, like this?

```py
main()
```

While it would fix our immediate problem, it would *also* run the code when we imported the module - which is not what we want!

#### More Variables and Types
{: .no_toc }

Consider the following code: will it generate the error:

- before the program runs
- on line #1
- on line #2

```py
def add_n_print(x,y):
  print(x + y) # Line 1

def main():
  add_n_print('foo', 5) # Line 2
```

...

Answer: it generates an error when the code reaches line #1 *at runtime*. We'd get an error of the form:

```
TypeError: can only concatenate str (not "int") to str
```

Unlike C++, Python's type-checker is at *runtime*. The type-checker detects that the types of values that `a` and `b` refer to are incompatible, and generates an error.

- this is called **dynamic typing** - which we'll talk about in-depth
- however, note that Python *still has types*!
- (and, this makes things harder to debug :/)

{: .note }
Fun fact: Python has recently supported "type hints", which add Haskell-like type annotations to functions. These *aren't* enforced by the interpreter!

#### Variables and Types
{: .no_toc }

Will this `trick_or_treat()` function generate an error, or will it work?

```py
def trick_or_treat(name, age):
  if age < 10:
    goodie = 'candy corn'
    print('Boooooo!')
  else:
    goodie = 'Snickers'
    print('Bwahahahahaha!')

  print(f"{name}, here's your {goodie}!")

trick_or_treat('Felix', 5)
```

...

Answer: it does work! Python is **function-scoped**: when an identifier is declared, it persists until the end of the function.

Contrast this to C++, where the equivalent code *would* generate an error:

```cpp
void trick_or_treat(string name, int age) {
  if (age < 10) {
    string goodie = "candy corn";
    cout << "Boooooo!";
  } else {
    string goodie = "Snickers";
    cout << "Bwahahahahaha!";
  }
  cout << name << ", here's your " << goodie << "!\n";
}
```

#### Looping with Counters
{: .no_toc }

What does this program print?

```py
for i in range(2,5):
 print(f'{i} witches!')

for i in range(7,1,-2):
 print(f'{i} zombies!')

goblins = 100
while goblins > 0:
 if goblins % 97 == 0 or goblins < 62:
   break
 print('Boo!')
 goblins -= 1
print('That was scary!')
```

...

Answer: the following:

```
2 witches!
3 witches!
4 witches!
7 zombies!
5 zombies!
3 zombies!
Boo!
Boo!
Boo!
That was scary!
```

Why bring this up? Python's `range()` behaves *slightly* differently from Haskell's ranges:

- Python's `range()` is semi-open: it includes the first item, but not the last. So, `range(2,5) = [2,5) = [2,3,4]`
- Haskell's `..` is completely closed: `2..5 = [2,5] = [2,3,4,5]`

### Learning Python through Challenges - Object-Oriented Programming
{: .no_toc }

#### Classes
{: .no_toc }

Consider the following Python class:

```py
class Car:
  MILES_PER_GAL = 30
  def __init__(self, gallons):
    self.gas_gallons = gallons
    self.odometer = 0

  def drive(self, miles):
    gals_needed = miles / Car.MILES_PER_GAL
    if self.gas_gallons > gals_needed:
      self.gas_gallons -= gals_needed
      self.odometer += miles

  def get_odometer(self):
    return self.odometer
```

We can insantiate and use the object like so:

```py
def use_car():
  c1 = Car(16)  # 16 gallons of gas
  c1.drive(10)
  print(f'I drove {c1.get_odometer()} mi')
```

Looking at this code, can you identify:

1. how do you define a constructor in a Python class?
2. how do you define member variables in a Python class?
3. how is a method different from a normal Python function?

...

Answer:

1. We define a constructor with the `__init__` function. Notice the "explicit" `self` - this behaves like `this` in C++, but is *always* the first argument!
2. We can define new member variables by just calling `self.[variable_name]`! Note that you can't declare variables without assigning them!
3. We define it "in" the function; it must have a `self` argument; and, it can access member fields/variables.

Now, consider this updated code:


```py
class Car:
  MILES_PER_GAL = 30
  def __init__(self, gallons):
    self.gas_gallons = gallons
    self.odometer = 0

  def drive(self, miles):
    gals_needed = miles / Car.MILES_PER_GAL
    if self.gas_gallons > gals_needed:
      self.gas_gallons -= gals_needed

  def __update_odometer(self, miles):
    self.odometer += miles

  def get_odometer(self):
    return self.odometer
```

and

```py
def use_car():
  c1 = Car(16)  # 16 gallons of gas
  c1.drive(10)
  print(f'I drove {c1.get_odometer()} mi')
```

Three more questions:

1. Changing `get_odometer()`'s return statement to `return odometer` doesn't work --- why?
2. How do you call a method from another method?
3. How do we indicate whether a method is public or private?

...

Answers:

1. Unlike C++, Python requires an explicit `self.` to identify member variables.
2. Again, unlike C++, Python requires an explicit `self.` to identify member functions.
3. Python's access modifiers are built into variable names: two underscores creates a private method (compare `__update_odometer` to `get_odometer`). Other variables are public by default!

{: .note }
Syntactically, the definition of a Python class is a sequence of statements that gets run anytime the class is instantiated. So, you can include any legal Python code in your class definition!

#### Class Method
{: .no_toc }

Consider the following Python code with a class method, which has no `self` parameter:

```py
class Thing:
 thing_count = 0

 def __init__(self, v):
  self.value = v
  Thing.thing_count += 1
  self.thing_num = Thing.thing_count


 def change_val(self, new_val):
  self.value = new_val

 def a_class_method(foo, bar):
  return Thing.thing_count * foo + bar

 def another_class_method(bletch):
  Thing.a_class_method(bletch,20)
```

`a_class_method` and `another_class_method` don't have a `self` parameter - so, they can't access instance (member) variables or functions!

You can call them like this:

```py
import thing

t1 = Thing("a")
Thing.another_class_method(42)
```

### Automatic Garbage Collection
{: .no_toc }

In Python, you don't have to worry about freeing objects when you're done with them, like you have to in C++. Instead, Python keeps track of who's pointing to each object; when nothing points to it, Python cleans it up.

More broadly, this is called "automatic memory management".

{: .note }
We'll cover this much more in-depth in the coming weeks!

In other words: we don't need the `new` and `delete` keywords!

### Classes and Destructors
{: .no_toc }

Python has "destructors", but they're rarely used, and not guaranteed to run. More generally, they're really a type of "finalizer".

It looks like this:

```py
class TextBook:
  ...

class Student:
  def __init__(self):
    self.book = TextBook()

  def study(self):
    self.book.read()

  def __del__(self):
    if self.book.finished_reading():
      print("You graduated!")
```

Destructors are only called when GC'd - but, it might not happen at all! Again, we'll explore this in a few lectures.

(in Python, instead of having your destructor deal with non-memory resources - like temporary files, network connections, etc. - you'll want to explicitly free these)


### Duck Typing
{: .no_toc }

Consider another classical pedagogical example:

```py
class PersonInDuckSuit:
  ... 			  # code omitted for clarity
  def quack(self):
    print('Hi! Err... oops, I mean quack quack.')

class Duck:
  ... 			  # code omitted for clarity
  def quack(self):
    print('Quack quack quack!')

class Vehicle:
  ... 			  # code omitted for clarity
  def drive(self):
    print('Vrooooom!')
```

What does this print?

```py
def quack_please(x):
  x.quack()

p = PersonInDuckSuit()
d = Duck()
v = Vehicle()
quack_please(p)
quack_please(d)
quack_please(v)
```

...

Answer:

```
Hi! Err... oops, I mean quack quack.
Quack quack quack!
AttributeError: 'Vehicle' object has no attribute 'quack'
```

This is a feature called **duck typing** - at runtime (and only when the line runs), Python checks if the input to `quack_please()` has a `quack` method. If it does, it runs that method; if not, an error is run.

This is different from C++, which requires features like inheritance (or interfaces) to make this code works.


### Object Identity
{: .no_toc }

Each distinct object has an ID number that we can access with the `id` function. For example,

```py
# Different types of equality in Python
fav = 'pizza'
a = f'I <3 {fav}!'
b = f'I <3 {fav}!'
c = a

print(id(a)) # 140426129935136
print(id(b)) # 140426129649376
print(id(c)) # 140426129935136 -- the same!
```

{: .note }
In CPython (the reference Python implementation), `id()` returns the (virtual) memory address.

Here's a challenge: are these object IDs going to be the same?

```py
booger = 10
print(id(booger))
booger = booger + 1
print(id(booger))
```

Answer: no! Since, `10` and `11` have different addresses in memory! This is in contrast to C++, where the reference stays the same - the value at that memory just changes.

### None
{: .no_toc }

Python has a keyword called `None`. Based on some code examples, what do you think it does?

```py
def play_with_dog(lst):
  obj = None
  for x in lst:
    if x.is_canine():
      obj = x
      break
  if obj is not None:
    obj.give_bone()

def declare_happiness(what = None):
  if what is None:
    print("I'm happy for no reason!")
  else:
    print(f"{what} makes me happy!")
```

We use `None` to indicate the absence of a value - it's like `nullptr` (in a way). The Pythonic way to validate if an item is/is not `None` is with the `is` and `is not` keywords.

What do you think the following will do?

```py
q = None

if q is False:
  print('Is None the same as False?')

if not q:
  print('Does not work with None?')

if q is None:
  print('Ahhh q is None!')

if q == None:
  print('Ahh q == None!')
```

...

Answer:

```

Does not work with None?
Ahh q is None!
Ahh q == None!
```

Importantly, `None` and `False` *aren't* exactly the same thing! But, it's "falsy" - which is why we get `if not q` working with `None`.

------
## Footnotes
{: .no_toc}