---
title: "07. Control Palooza "
author: Carey Nachenberg, Einar Balan, Ashwin Ranade
layout: lecture
parent: Lecture Notes
---

{: .note}
This lecture note covers the Control Palooza deck.

## Table of Contents
{: .no_toc }

{:toc}
- dummy item

## Control Palooza
{: .no_toc }
Everything you never learned about how code executes!

## Expressions

An expression is a combination of values, function calls and operations that, when evaluated, produces an output value. An expression may or may not have side effects.

Which of the following are expressions?
- f(x)*5
-- Yes!
- if (foo)
   cout << "bar";
-- No
- a = 5
-- In some languages, yes. Like in C++ we can do b = a = 5, so a = 5 is an expression that has a side effect and produces a result
- a && (b || c)
-- Yes!
- if a > 5 then a*3 else a*4
-- Yes - in functional languages (or languages with functional features) this if code is an expression, which produces a result of either a*3 or a*4

## Expression Evaluation Order

Consider the following C++ code:

```cpp
int c = 1, d = 2;

int f() {
  ++c;
  return 0;
}

int main() {
  cout << c - d + f() ;
}
```

Or this code:

```cpp
int c = 0; 

int f() {
  ++c;
  return c;
}

void h(int a, int b)
  { cout << a << " "<< b; }

int main() {
  h(c, f());
}
```

Challenge: What will these C++ programs print?  
The first program produces either -1 or 0
The second program produces either 0 1 or 1 1

Why? C++ doesn't specify what order the terms of an arithmetic expression must be evaluated! Nor does it specify what order parameters must be evaluated! Some languages mandate a left-to-right eval order (C#, Java, JavaScript, Swift), while others don't (C++, Rust, OCaml).

So in general, if your expressions are calling functions, those functions shouldn't have side effects that could impact the execution of the other functions called in the expression. Otherwise you'll get unpredictable behavior depending on the compiler and potentially each time you compile the code it could do a different thing!

Why avoid specifiying ane explicit evaluation order? The reason that an ordering is not specified is to give the compiler latitude to optimize the code.

## Associativity

Regardless what order each of the terms are evaluated... Most languages use left-to-right associativity when evaluating mathematical expressions. Which is the way we all learned to do it in school.

So an expression like:

```cpp
cout << c - d + f() ;
```

Would be evaluated as:

```cpp
cout << (c - d) + f() ;
```

Instead of:

```cpp
cout << c - (d + f());
```

There are some notable exceptions, like evaluating exponentiation (e.g., 2^3^2 == 512, since we often evaluate exponentiation from right-to-left).  Assignment is also right-associative, so we can do something like this:

```cpp
  a = b = c = 5;  
  // equivalent to:
  a  = (b = (c = 5));   // so c = 5 runs first, then b = c runs, then a = b runs
```

## Short Circuiting

In any boolean expression with AND /  OR, most languages will evaluate its sub-expressions from left-to-right. The moment the language finds a condition that satisfies or invalidates the boolean expression, it will skip evaluating the rest of the sub-expressions. This can dramatically speed up your code!

For example, this code here:

```cpp
if(is_cute() || is_smart() || is_rich()) {
  date_person();
}
```

Really executes like this:

```cpp
if (is_cute()) 
  date_person();
else if (is_smart())
  date_person();
else if (is_rich())
  date_person();
```

And this code here:

```cpp
if (good_salary() && fun_work() && good_culture()) {
  take_job();
}
```

Really executes like this:

```cpp
if (good_salary()) {
  if (fun_work()) {
    if (good_culture()) {
      take_job();
    }
  }
}
```

Some languages, like Kotlin, have the ability to use boolean expressions with short circuiting and without!  e.g.

```kotlin
fun rich(salary: Int): Boolean {
  if (salary > 200000) return true;
  print("Not in silicon valley!\n")
  return false
}

fun main() {
  val cute = true; val smart = false;
  val salary = 200000;
    
  if (cute == true || smart == true || rich(salary) == true)  // uses short circuiting
    print("You're my type!\n")

  if (cute == true or smart == true or rich(salary) == true)  // no short-circuiting
    print("I still want to date you!\n")
}
```

## Iteration

There are three different types of iteration that we'll study:

Counter-controlled: We use a counter, like we would in a for-loop in C++ or using a range in Python:

```
    for i in range(1,10):
        do_something()
```

Condition-controlled: We use a boolean condition to decide how to long to loop - this is almost always a while() or do-while() loop.

```
    while (condition)
       do_something()
```

Collection-controlled: We're enumerating the items in a collection

```
   for (x in container)
        do_something(x)
```

## Iteration in loops

In languages like C++ with simple counter-controlled loops (e.g., for(int i=0;i<10;i++) { ... }) it''s easy to see how looping works. But how does looping work when we use a collection-controlled loop (e.g., iterating thru a vector of objects) or use a range()? There''s more going on under the hood than you think!  There are two primary approaches: using an iterator (of which there are two types of iterators), or using first-class functions.  Let''s learn both!

These kinds of loops are implemented using "iterators":

```python
for n in student_names:
 print(n)

for i in range(0,10):
  print(i)
```

These kinds of loops are implemented using first-class functions:

```swift
fruits.forEach
	{ f -> println("$f") }
```

### Iterator-based loops

An iterator is an object that enables enumeration of a sequence of values. An iterator provides a way to access the values of a sequence without exposing the sequence's underlying implementation.

- Iterators can be used to enumerate the values held in a container (e.g., a list, set)
- Iterators can be used to enumerate the contents of external sources like data files
- Iterators can be used to enumerate the values of an abstract sequence. An abstract sequence is one where the values in the sequence aren't stored anywhere, but generated as they're retrieved via the iterator

Iterators used with containers and external sources are typically distinct objects from the objects that contain the sequences they refer to.

#### Iterable Objects vs Iterators

- An Iterable Object is an object like a container (e.g., vector, list, set) which can have its items iterated over.

- An Iterator is a thing that can refer to an Iterable Object and be used to iterate over the vaulues it holds or represents.

To get an iterator, you request one from an iterable object. The Iterable Object then returns the iterator to you. You may then use the iterator to refer to the items in the iterable object.

Examples of getting an iterator from an iterable object:

```kotlin
// Kotlin: iterator into container
val numbers = listOf(10,20,30,42)

val it = numbers.iterator()  // get an iterator into the numbers list
```

```swift
// Swift: abstract sequence
var range = 1...1000000

var it = range.makeIterator()  // get an iterator to the first # of the abstract seq.
```

```cpp
// C++
vector<string> fruits = {"apple", "pear", ... };

auto it = fruits.begin();  // get an iterator into a C++ vector
```

While iterators differ by language, most* tend to have an interface that look like one of these (* C++ has a notably different approach):

##### An interface with hasNext() and next() methods

These types of iterators are used with containers and external sources like data files. They have two methods:

iter.hasNext(): We ask the iterator (NOT the iterable object) whether it refers to a valid value that can be retrieved?

iter.next(): We ask the iterator (NOT the iterable object) to get the value that it points to, and advance to the next item

```kotlin
// Kotlin: iterator into container
val numbers = listOf(10,20,30,42)

val it = numbers.iterator()
while (it.hasNext()) {
  val v = it.next();
  println(v)
}
```

##### An interface with just next() methods

These types of iterators are used with Abstract Sequences, like ranges: range(1,10)

iter.next(): The iterator (NOT the iterable object) generates and returns the next value in the sequence, or indicates the sequence is over via a return code or by throwing an exception

```swift
// Swift: abstract sequence
var range = 1...1000000

var it = range.makeIterator()
while true {
  var v = it.next()  // in Swift, the next() method returns either nil or a valid value
  if (v == nil) { break }
  print(v)
}
```

#### C++ has unique iterators

C++ models its iterators after C++ pointers, which can be incremented, decremented, and dereferenced.  They're a notable exception to iterators in most languages!

```cpp
// C++ has unusual iterator syntax!
vector<string> fruits = {"apple", "pear", ... };

for (auto it = fruits.begin(); it != fruits.end(); ++it)
  cout << *it;
```

### So how does looping work under the hood?

When you loop over the items in an iterable object , the language secretly uses an iterator to move through the items! This is a fantastic example of Syntactic Sugar!

When you do this:

```kotlin
// Kotlin: iterate over list of #s
val numbers = listOf(10,20,30,42)

for (v in numbers) {
  println(v)
}
```
Here's what's really happening (the language generates this code secretly for you):

```kotlin
val numbers = listOf(10,20,30,42)

val it = numbers.iterator()
while (it.hasNext()) {
  val v = it.next()
  println(v)
}
```

Or when you do this:

```swift
// Swift: iterate over range

for v in 1...1000000 {
  print(v)
}
```

Here's what's really happening:

```swift
var range = 1...1000000

var it = range.makeIterator()
while true {
  var v = it.next()
  if (v == nil) { break }
  print(v)
}
```
### How are iterators implemented?

There are two primary approaches for creating iterators:

- With Traditional Classes: An iterator is an object, implemented using regular OOP classes.
- "True Iterators" aka Generators: An iterator is implemented by using a special type of function called a generator.

#### Iterators built using traditional classes

Let's see an iterator that is built using a traditional class - Python makes it easy.

First we must define an iterable class (e.g., like a vector class) which we can iterate over. To make a class iterable, we must define an \_\_iter\_\_() method that creates and returns an iterator object. The ListsOfStrings class is an _iterable_ class - NOT an iterator.  If its \_\_iter\_\_ method is called, it will return an iterator that can be used to iterate over its items.

```python
class ListOfStrings:
  def __init__(self):
    self.array = []

  def add(self,val):
    self.array.append(val)

  def __iter__(self):
    it = OurIterator(self.array)   // calling __iter__ returns a new OurIterator object
    return it
```
Second we can define our OurIterator class. Notice that this iterator class is uniqely tied to our ListOfStrings class above. The two work hand-in-hand.

```python
class OurIterator:
  def __init__(self, arr):
    self.arr = arr
    self.pos = 0

  def __next__(self):
    if self.pos < len(self.arr):
      val = self.arr[self.pos]
      self.pos += 1
      return val
    else:
      raise StopIteration
```

Our iterator class must have a method called \_\_next\_\_() that gets the value of the item pointed to by the iterator, advances the iterator, and returns the value. As you can see, if the iterator has run out of items to iterate over (ther are no more items in the iterable object), the iterator (in Python) will throw an exception to tell the user of the iterator that there are no more items left.

If we define our iterable class and iterator classes this way, Python will give us first-class looping support over our iterable class!

```python
nerds = ListOfStrings()
nerds.add("Carey")
nerds.add("David")
nerds.add("Paul")

for n in nerds: 
  print(n)
```

And here''s an example of an iterator in Java:

```java
import java.util.Iterator;  

class ListOfStrings implements Iterable<String> {
  private String[] array;
  private int numItems = 0, maxSize = 100;

  public ListOfStrings() 
    { array = new String[maxSize]; }
  public void add(String val) 
    { array[numItems++] = val; }
  @Override public Iterator<String> iterator() {      
    Iterator<String> it = new OurIterator();
    return it;
  }

  class OurIterator implements Iterator<String> {
    private int iteratorIndex = 0;
    @Override public boolean hasNext() 
      { return iteratorIndex < numItems; }
    @Override public String next() 
      { return array[iteratorIndex++]; }
  }
}
```

Things to note: 
- The ListOfStrings class implements Java's Iterable interface, which makes this clas an Iterable class (it can have its items iterated over). The class must then implement the iterator() method which can be used to obtain an iterator into the iterable object.
- The Iterator class is a nested class within ListOfStrings, which lets it access the memer variables of ListOfStrings.  Notice how it implements the Iterator interface. To do so, it must provide a hasNext() method and a next() method.

And here's how we'd use our class.
```java
public void testOurList() {         
  ListOfStrings nerds = 
    new ListOfStrings();

  nerds.add("Carey");
  nerds.add("David");
  nerds.add("Paul");

  Iterator it = nerds.iterator();  
  while (it.hasNext()) { 
    String value = (String)it.next();    
    System.out.println(value);
  } 

  for (String s: nerds)
    System.out.println(s);
}
```
Notice how above we can manually get the iterator then use hasNext() and next() with it. Or we can use Java's looping syntax, and Java will use syntactic sugar, hiding the calls to get the iterator and use hasNext() and next().

Ok, next let's see how to create a range() iterable using a class like we have in python (and many other languages):

```python
class OurIter:  # iterator class
  def __init__(self, start, end):
   self.cur = start
   self.end = end

  def __next__(self):
    if self.cur < self.end:
      val = self.cur
      self.cur += 1
      return val
    else:
      raise StopIteration()

class OurRange:  # iterable class
  def __init__(self, end):
    self.end = end

  def __iter__(self):
    iter = OurIter(0, self.end)
    return iter
```

Notice the OurRange class, which is an iterable.  We construct it with the maximum value to iterate to, so it will iterate from [0,N).  The class implements an \_\_iter\_\_() method which is called to obtain an iterator into the iterable.  You can see that this method creates a new OurIter iterator, specifying the starting and ending range values.

The OurIter class simply implements a \_\_next\_\_ method, just like any other iterator in python. This method obtains the next value if one is available,  then advances so the next call will return the next value in the range, and finally returns the value. If no more values are available in the range, the method raises a StopIteration exception, which is the way that Python indicates the iterator has reached the end of its iteration.

#### "True Iterators" aka Generators

A "true iterator" is an iterator that is implemented by using a special type of function called a generator.

A generator is essentially a closure that can be paused and resumed over time. Many languages have generators, but we'll use python to illustrate the concept since the syntax is pretty simple. But first let's start with pseudocode:

```python
def foo(n):
  for i in range (1, n):
    print(f'i is {i}')
    pause
    print('woot!')

def main():
  p = create_pausable(func=foo, n=4)  # Line A
  print('CS')
  start(p)    # Line B
  print('is')
  resume(p)  # Line C
  print('cool!')
  resume(p)
```

Here's the general idea - remember the above is pseudocode:

- Line A: You create a special closure (called a generator), which has one extra piece of data - an instruction pointer. The instruction pointer starts off on the first line of the function, and the function starts in a suspended state. It's not running at this point. We've just created the closure.
- Line B: We wake up the generator/closure begin running it. It will run until it hits the pause command.  At that point, the instruction pointer is saved in the closure so we know where to continue running, and all changes made to the local variables in the closure are also retained. The generator returns to the next line of the main function and keeps running there. Later when we resume the generator, it continues running where it left off, with the same variables/values.
- Line C: We resume the generator, and it continues running where it left off last time

So the above pseudocode would print:
```
CS
i is 1
is
woot!
i is 2
cool!
woot!
i is 3
```

Now here's the real python syntax:

```python
def foo(n):
  for i in range (1, n):
    print(f'i is {i}')
    yield
    print('woot!')

def main():
  p = foo(4)
  print('CS')
  next(p)
  print('is')
  next(p)
  print('cool!')
  next(p)
```

Here's what changed:
- Instead of "pause" we use the keyword "yield" to pause the function
- Instead of calling create_pausable to create our closure, we just call the function and pass in the arguments, then store the return value in a variable. This does NOT run the function. It just creates the closure, and then returns an object reference to the closure. We then store the object reference to the closure inside our variable p. How does Python know not to run the function but just to create a closure? Because it has a "yield" command inside of it. This implicitly tells Python this is a generator function and not a traditional function, and that the first call should not run the function, but just create a closure
- Instead of calling "start" to start the generator running, and "resume" to resume its execution, we just use next() on the closure that was returned during the initial call.

By the way, to use our earler iterable/iterator nomenclature, the generator function is an "iterable", and the closure that is returned when it's first called behaves like an iterator.

Now let's see a more complex generator - it can not only yield its execution, but every time it yields it can return a value to the caller!

```python
def bar(a, b):
  while a > b:
    yield a  # Line A
    a -= 1

f = bar(3, 0)
print('Eat prunes!')
t = next(f)  # Line B
print(t)
t = next(f)
print(t)
t = next(f)
print(t)
print(f'Explosive diarrhea!')
```

Notice that on Line A, the generator "yields a value". So not only does it pause itself, but it also returns a value at this time. Notice that on Line B, we wake up the generator and let it run until it yields a value. The yielded value is returned by next() and that value is stored in the t variable.

This is a more idiomatic use of a generator – they're used to  "generate" a sequence of values that can be retrieved one at a time – the sequence might be finite, or infinite!

Since a generator is an iterable object, and produces an iterator... You can use it like any other iterable object in loops! Another name for a generator function is a "true iterator."

```python
def our_range(a, b):
  while a > b:
    yield a
    a -= 1

print('Eat prunes!')
for t in our_range(3,0):
  print(t)
print(f'Explosive diarrhea!')
```
Here's a Javascript generator:

```javascript
// JavaScript defines generators with a * character after the function name
gen = function*(n) {
  yield 'I';
  yield 'want';
  yield n;
  yield 'cookies';
};

g = gen(10);
while (true) {
  r = g.next();
  if (r.done) break;
  console.log(r.value);
}
 
for (word of gen(3)) {
  console.log(word);
}
```

And here's one in Kotlin:

```kotlin
// Kotlin generators care called "sequences"
fun vals(start: Int) = sequence {
  var i = start
  while (true) {
    yield(i)
    ++i
  }
}

fun main(args: Array<String>) {
  val iter = vals(42).iterator()
  var count = 3
  while (count-- > 0) {
    var n = iter.next()
    println(n)
  } 
  
  for (n in vals(7).take(5))  
    println(n)     // 7, 8, 9, 10, 11
}
```

#### Analysis of Class-based Iterators and Generators
Some things are easier to do with class-based iterators, and some things are easier to do with generators. But the two are isomorphic and functionally equivalent - anything we can do with a class we can do with a generator, and visa-versa.

### First-class function-based iteration

Alright, we'll end this lecture by looking at how some languages use first-class functions to do iteration. The syntax often looks something like this:

```
var fruits = ... // some list of fruits

fruits.forEach { f -> println("$f") }
```
Basically, the container (e.g., vector, list or set) contains a forEach() method, which accepts one parameter.  We pass in a { lambda } function to the forEeach method as this parameter.  The forEach method iterates  over each item in the iterable, and passes each item as the parameter to the lambda function (e.g., to the parameter f shown in the lambda above).

Here are some examples:

```rust
// Rust
(0..10).for_each(|elem| println!("{}", elem));

let items = vec!["fee","fi","fo","fum"];
items.iter().for_each(|elem| println!("{}", elem));
```

```ruby
# Ruby
(0..9).each do |elem| 
  print elem, " " 
end

items = ["fee","fi","fo","fum"]

items.each do |elem|
    print(elem,"\n")
end
```

```kotlin
// Kotlin
(0..9).forEach({ elem -> println("$elem") })

var items = arrayOf("fee","fi","fo","fum")
items.forEach({ elem -> println("$elem") })
```

When we use this syntax, we're NOT using an iterator! Instead, we're passing a function as an argument to a forEach()/each() method that loops over the iterable's items!

Let's build a forEach() method for our earlier Python container:

```python
class ListOfStrings:
  def __init__(self):
    self.array = []

  def add(self,val):
    self.array.append(val)

  def forEach(self, f):   # define our own for-each method
    for x in self.array:
      f(x)

yummy = ListOfStrings()
... # add items to yummy

def like(x):
 print(f'I like {x}'))

yummy.forEach(like)
```

Notice how the iterable object (a ListOfStrings) iterates over its own items, and calls the provided f(). This is essentially a map() operation like we saw in functional programming!

## Introduction to Concurrency

Concurrency is when we decompose a program into simultaneously executing tasks.

The tasks can:
- run in parallel on multiple cores, OR run multiplexed on a single core
- operate on independent data, OR operate on shared mutable data/systems
- be launched deterministcally through program flow, OR be lanuched to due external event (button click in UI)

Concurrency can make our programs more efficient!

**Question:** If a concurrent program only runs on a single core, why would it be any faster than a program that runs serially?
**Answer:** Some tasks involve lots of waiting, not a lot of CPU. During that waiting time, a concurrent program can use the CPU for other useful tasks! For example, downloading multiple photos.

## Models for Concurrency

Two approaches for implementing concurrency:

**Multi-threading model:** a program creates multiple "threads" of execution that run concurrently (potentially in parallel).

The programmer explicitly "launches" one or more functions in their own threads, and the OS schedules their execution across available CPU cores.

```cpp
// Multi-threaded program (pseudocode)
void handle_user_purchase(User u, Item i) {
  if (bill_credit_card(u) == SUCCESS) {
    create_thread(schedule_shipping(u, i));
    create_thread(send_confirm_email(u, i));
  }
}
```

**Asyncronous Model**: A single-threaded loop manages a queue of tasks, executing them one at a time as they become ready.

When an event occurs (e.g., user clicks a button) the event results in a new function `f()` being added to the queue which eventually runs and handles the event.

```js
// Event-driven program (pseudocode)
function process_payment() { ... }

setup_event_associations() {
 button = create_new_button("Pay Now!");
 button.set_func(ON_CLICK, process_payment);
}
```

### Multi-threading Model

There are three key features of the multithreading model:
1. **Thread management**: each thread allows us to run operations concurrently
2. **Synchronization**: ensures that threads do not interfere when sharing resources
3. **Message passing**: allows different threads to safely send messages to each other

### Thread Management

Most languages provide libraries for thread management. Consider this example in C++:

```cpp
// C++ threads
#include <thread>

void f(int n) {
  for (int i=0;i<n;++i)
    cout << "Hi!";
}

int main() {
  std::thread t(f, 10);   
  ... // this keeps running concurrently
}
```

Some languages, like Go, even include threads as primitive objects!

```go
// Go's "goroutines" which are also threads
func f(n int) {
  for i := 0; i < n; i++ {
    fmt.Println("Hi!")
  }
}



func main() {
  go f(10)
  ... // this keeps running concurrently
 }
```

#### Fork Join

Fork Join is a common pattern for multithreading. It's essentially a concurrent version of divide and conquer:

1. first, we "fork" one or more tasks so they execute concurrently...
2. second, we wait for all those tasks to complete (aka "join") and then proceed.

{: .note }
If you're following at home ... this also sounds like a `map`-`fold` or `map`-`reduce`. Interesting...


**Example:** A parallel sort would be a good example of a problem suitable for fork-join.

```js
function sort_in_parallel(array, n) {
  t1 = run_background(sort(array[0:n/3]));
  t2 = run_background(sort(array[n/3:n*2/3));
  t3 = run_background(sort(array[n*2/3:n]));

  wait_for_all_tasks_to_finish(t1, t2, t3);
  merge_sorted_subarrays(array);
}
```

```js
function task1(param) { ... }
function task2(param) { ... }
function task3(param1, param2) { ... }

function launch_in_parallel() {
  t1 = run_background(task1(42));
  t2 = run_background(task2("suss"));
  t3 = run_background(task3(3.14, 2.71));

  wait_for_all_tasks_to_finish(t1, t2, t3);
  print("All tasks completed, proceeding...");
}
```

Fork-join is often used recursively! For huge datasets, we may choose to split a data source in two (forking). But, if it's still too big, we can split it again! You can imagine this repeating until each chunk is small enough to deal with independently.

#### Fork-Join In Different Languages

```cpp
// C++
#include <thread>

void task1(int n)  { ... }
void task2(int n)  { ... }
void task3(int n)  { ... }

int main() {
  std::cout << "Forking threads!\n";
  //To launch a function in the background, we simply create a new thread object.

  std::thread t1(task1, 10);
  std::thread t2(task2, 20);
  std::thread t3(task3, 30);

  // do other processing here...

  t1.join(); //To join, we simple call the thread.join() method. This blocks the caller until the thread finishes.

  t2.join();
  t3.join();
  std::cout << "All threads joined!\n";
}
```

```cs
// C#
using System;
using System.Threading;
using System.Threading.Tasks;

class Program {
  static int task1(int param) { ... }
  static int task2(int param) { ... }
  static int task3(int param) { ... }

  public static void Main(string[] args) {
    int r1 = 0, r2 = 0, r3 = 0;
    Console.WriteLine("Forking threads!");
    var t1 = Task.Run(() => r1 = task1(10));
    var t2 = Task.Run(() => r2 = task2(20));
    var t3 = Task.Run(() => r3 = task3(30));
    Task.WaitAll(t1, t2, t3);
    Console.WriteLine("Joined: {0}, {1}, {2}",
                      r1, r2, r3);
  }
}
```


```py
# Python
import threading

def task(n):
  while n > 0:   # do some computation
    n = n - 1

print("Forking threads!")

#In Python, we must create a thread object, and then do thread.start() before it runs.

t1 = threading.Thread(target=task, args=(100000000,))
t2 = threading.Thread(target=task, args=(100000000,))
t3 = threading.Thread(target=task, args=(100000000,))
t1.start()
t2.start()
t3.start()

# do other processing here...

t1.join()
t2.join()
t3.join()
print("All threads joined!")
```

#### "Multi-threading" in Python

**Question for the Python program:**
Assuming looping 100 million times takes 5s, how long will this program take to run on a multicore PC?

**Answer**: 15s
We'd expect all three tasks to run in parallel... taking 5s total. But it takes 15s!
Why? Because when each Python thread runs, it claims exclusive access to Python's memory/objects.

So only one thread generally does computation at a time!

Why? Python's garbage collection system was never designed to be thread-safe!

Python has something called a "Global Interpreter Lock" or GIL. The GIL is like a hot potato – it can only have one owner at a time. Once a thread takes ownership of the GIL it's allowed to read/write to Python objects. After a thread runs for a while, it releases the GIL (tosses the potato) to another thread and gives it ownership. If a thread is waiting for the GIL, it simply falls asleep until it gets its turn.

**Question:** So why even support multiple threads? Are there any cases where multithreading speeds things up?

**Answer:** Answer: I/O operations like downloading data from the web or saving a file to disk DON'T need the GIL to run! So these kinds of operations can still make progress if launched in the background!

### Concurrency With Shared Mutable State

**Question:** What will the value of karma be if I run both functions concurrently until completion? Why?

```cpp
int karma = 0;

void do_good_deeds() {        // runs in thread t1!
  for (int i=0;i<100000;++i)
    karma++;
}

void be_naughty() {           // runs in thread t2!
  for (int i=0;i<100000;++i)
    karma--;
}
```

**Answer:** Given that each "thread" of execution can be interrupted at any time to run the other, it's impossible to know!

The undefined behavior is due to both threads using a _shared mutable state_; `karma` is a mutable variable that's shared between multiple threads. To take a closer look, let's consider the most basic operations that make up `karma++`.

1. Reading the `karma` variable from memory
2. Incrementing the value that is read
3. Storing the new incremented value back into `karma`'s memory address

If we have two threads accessing `karma`, these basic operations can be interleaved differently, resulting in unpredictable, nondeterministic behavior. In short: this is not good! Consider the following potential sequences accross two threads:

```
do_good_deeds: read karma (value: 0)
do_good_deeds: increment 0, the value that was read (value: 1)
be_naughty:    read karma (value: 0)
do_good_deeds: write 1 to karma
be_naughty:    decrement 0, the value that was read (value: -1)
be_naughty:    write -1 to karma
```

Here the final value stored in `karma` is -1.

```
do_good_deeds: read karma (value: 0)
do_good_deeds: increment 0 (value: 1)
be_naughty:    read karma (value: 0)
be_naughty:    decrement 0 (value: -1)
be_naughty:    write -1 to karma
do_good_deeds: write 1 to karma
```

Here the final value stored in `karma` is 1. Each thread reads in the intitial value of `karma` and performs some operation on it. The unpredictability comes from the fact that the order in which we write the resulting value back to `karma` in each thread might vary. This is called a race condition and clearly it's a big problem! How might we avoid it?


### Safe Concurrency With Shared Mutable State

Most modern languages now have built-in language features to make it safer to use shared mutable state. To list a few: mutexes, semaphores, spinlocks, barriers, conditions, and read write locks. Lots of funny words but they are very useful!

Here are a few examples:

```cpp
// C++ example of using mutex objects
#include <thread>
#include <mutex>

int counter = 0;
std::mutex mtx;

void increment(int times) {
  for (int i = 0; i < times; ++i) {
    mtx.lock();      // Lock mutex
    counter++;       // Critical section
    mtx.unlock();    // Unlock mutex  
  }
}

int main() {
  // Create two threads running increment(1000)
  std::thread thread1(increment, 1000);
  std::thread thread2(increment, 1000);
  ...
}
```
In C++, we "lock" mutexes to indicate a section of our code that only one thread can access at a given time. This is useful for managing shared mutable state across threads, such as a counter variable!

Here's another example in Ruby:

```ruby
// Ruby example of using mutex objects
require 'thread'

counter = 0
mutex = Mutex.new

increment = ->(times) {
  times.times do
    mutex.synchronize { 
      counter += 1 
    }
  end
}

# Create two threads running increment(1000)
thread1 = Thread.new { increment.call(1000) }
thread2 = Thread.new { increment.call(1000) }
...
```

Some languages have built in synchronization primitives! Consider Java's `synchronized` keyword:

```java
// Java has a built-in keyword for synchronization
public class Counter {
  private Object syncObj = new Object();
  private int counter = 0;

  public void increment(int times) {
    for (int i = 0; i < times; i++) {
      synchronized(syncObj) {   
        counter++;
      }
    }
  }

  public static void main(String[] args) throws Exception {
    Counter counter = new Counter();

    Thread thread1 = new Thread(() -> counter.increment(1000));
    Thread thread2 = new Thread(() -> counter.increment(1000));
    ...
  }
}
```

Here the `synchronized` keyword requests exclusive ownership of `syncObj`. If another thread reaches this point while the original thread still has ownership, it will be blocked until ownership is released. 

### Message Passing

To reduce the use of shared mutable state and race conditions, some languages use built-in message queues to enable threads to safetly communicate.

Consider this example in Go:

```go
// Go "channels" used for message passing
func genPrimes(messages chan int) {
  for i := 2; ; i++ { // Infinite loop  
    if isPrime(i) {
      messages <- i // Send message
    }
  }
}

func main() {
  messages := make(chan int)

  go genPrimes(messages) // launch thread

  for {
      prime := <-messages // dequeue next prime
      fmt.Println(prime)
  }
}
```

Here's another example in Erlang:

```erlang
// Erlang message passing example
gen_primes(pid, N) ->
  if is_prime(N) -> pid ! {prime, N}; 
     true        -> ok
  end,
  gen_primes(LoopPid, N + 1).

loop() ->
  receive
    {prime, Prime} ->
      io:format("~p~n", [Prime]),
      loop()
  end

start() ->
  pid = self(),
  spawn(fun() -> gen_primes(pid, 2) end),
  loop().
```

### Asyncronous Programming

Let's spend a bit of time to see the other approach -- asyncronous programming!

In asyncronous programming, statements in a program are not executed from top to bottom. Instead, a module called a **runtime** maintains a queue of **coroutines** to execute. A coroutine is a function that once run, can be paused and resumed. The runtime repeatedly dequeues, suspends, and resumes coroutines resulting in a concurrent execution of each of them. I/O operations can even be performed outside the queue while a coroutine is running.

Async programming becomes very useful when we have an IO bound task, such as reading from a databse or downloading web data. While waiting for an IO operation to be completed, the runtime can suspend a coroutine and enqueue other coroutines. Coroutines are also very lightweight in terms of memory usage compared to threads! 

Carey's slides have a great animated example that we can't really do justice here. Go check it out!

Here's an example in Python:

```python
from asyncio import run, create_task


async def g(x):
  return x + "!"


async def f(x):
  y = await g(x)
  return "Hello " + y


async def main():
  task1 = create_task(f("world"))
  task2 = create_task(g("async"))
  r1 = await task1
  r2 = await task2
  print(f"Results: {r1} {r2}")
```

Here's a more practical example in which a database query is performed on a web server:

```python
from aiohttp import web
import asyncpg

async def handle_request(request):
  product_name = request.query.get('product_name')
  query = "SELECT product, price FROM products WHERE product_name = $1"


  connection = await asyncpg.connect(dsn='postgres://user:password@localhost/mydatabase')
  result = await connection.fetch(query, product_name)
  await connection.close()

  if result:
    response_text = f"{result['product']}: ${result['price']}"
  else:
    response_text = "No matching product found."

  return web.Response(text=response_text)

app = web.Application()
app.add_routes([web.get('/prod_search', handle_request)])

web.run_app(app)
```

### Multithreading vs. Async

Why use one model over another? Let's see!

| Multithreading | Asynchronous Programming |
|----------------|--------------------------|
| Capable of true parallelism across cores | Concurrency without true parallelism |
| Preemptive multitasking | Cooperative mulitasking |
| Uses more memory; each thread has extensive execution context | Uses less memory; each coroutine is lightweight |
| Race conditions are common, requiring synchronization | Race conditions are less common due to single threaded nature |
| Best suited for CPU bound tasks | Best suited for IO bound tasks |
| Complex error handling | Easy error handling |

In summary: concurrency is a super powerful tool in programming! We explored a few different ways it is implemented and the implications of each.