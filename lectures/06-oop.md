---
title: "05. OOP Palooza "
author: Carey Nachenberg
layout: lecture
parent: Lecture Notes
---

{: .note}
This lecture note covers the OOP Palooza deck.

## Table of Contents
{: .no_toc }

{:toc}
- dummy item

## Object-Oriented Programming
{: .no_toc }
Object-Oriented Programming is a programming paradigm based on the concept of **objects**. While classes are often used, they're not required (like in JS!). The core idea is that objects talk to each other with "messages" - a fancy word for method calls.

### A Brief History
We covered a brief history of OOP languages in class, including:

- Sketchpad: Developed by Ivan Sutherland, it was an oscilloscope-based CAD program from 1963. It pioneered concepts like classes, objects, inheritance, and late binding/polymorphism. It wasn't a language but a CAD program, but it embodied many OOP concepts.
- The record: The notion of a record or struct was developed by Tony Hoare in 1966. He noticed that many things in the real world can be modeled as objects with a set of related fields, and that objects of the same category or type (e.g., Dogs) often had similar sets of fields, defining classes of objects.
- Simula: Between 1962-1966, Ole-Johan Dahl and Kristen Nygaard, two scientists in Norway developed the Simula language to make it easier to build military simulations.  One big leap with Simula was that it defined objects as having both data fields (like Hoare) and functions/methods to operate on that data. Simula pioneered the following items:
  - Classes have both functions and data
  - Objects can be instantiated from Classes
  - Use of the dot notation for object member access (e.g., doggo.bark())
  - Inheritance
  - Virtual functions/procedures to facilitate polymorphism/dynamic binding
  - Dynamic binding (we'll talk about this later)

Take a look at some Simula classes to see how close they look to modern C++, Java, etc.:

```
CLASS POINT(X,Y); REAL X, Y;
  COMMENT***CARTESIAN REPRESENTATION
BEGIN
  BOOLEAN PROCEDURE EQUALS(P); REF(POINT) P;
    IF P =/= NONE THEN
      EQUALS := ABS(X-P.X) + ABS(Y-P.Y) < 0.00001;
  REAL PROCEDURE DISTANCE(P); REF(POINT) P;
    IF P == NONE THEN ERROR ELSE
      DISTANCE := SQRT( (X-P.X)**2 + (Y-P.Y)**2 );
END***POINT***
```

```
POINT CLASS COLOREDPOINT(C); COLOR C;
BEGIN
  BOOLEAN PROCEDURE EQUALS(Q); REF(COLOREDPOINT) Q;
    ...;
END***COLOREDPOINT**
```

```
REF(POINT) P; REF(COLOREDPOINT) CP;
P :- NEW POINT(1.0,2.5);
CP :- NEW COLOREDPOINT(2.5,1.0,RED);
CP.C := BLUE;
```

- Smalltalk: Meanwhile, Alan Kay (who is a UCLA emeritus prof!) was at Xerox PARC developing the DynaBook - a precursor to the modern laptop. Alan wanted to design a language that could be used to implement an OS, apps, everything. Inspired by his own background in math and biology, SketchPad, Hoare's records, and Simula, he came up with a broad conception of OOP, and developed SmallTalk. While most languages have adopted Simula-like syntax, Kay's ideas about OOP have continued to shape the field, and Kay is credited with coining the term OOP. In Smalltalk:
  - Everything is an object
  - Objects send messages to other objects to request behaviors (like cells in an organism, or nodes on the arpanet - the precursor to the internet)
  - Each object decides how/if to respond/behave when it receives a message
  - Objects may pass messages to other objects to get their work done
  - Methods may return objects as their result to a sender

Here's an example smalltalk program below.

```smalltalk
"Snippet of Smalltalk code that sings '99 bottles of beer'"
99 to: 1
   by: -1
   do: [:j | Transcript show: (j printString),' bottles of beer on the wall ';cr].

Transcript show: 'No more bottles of beer on the wall!;cr.
```

This code implements a loop by sending the object 99 (EVERYTHING is an object in Smalltalk) a message with three parameters: (1) to, (2) by, and (3) do. The first parameter tells the 99 object what it should iterate to. The second parameter tells 99 how much to increment during each iteration, and the final parameter is a block of code to run during each iteration.

Think of this code as being:
```cpp
Number n(99);
n.to_by_do(1,-1,lambda to print stuff)
```

To briefly conclude our history, in the 70s and 80s, Bjarne Stroustrup created C With Classes, later known as C++, by marrying ideas from Simula with C.  And then Java, C#, Scala, Kotlin and Swift followed in the 2000s and 2010s.

## Essential Components of OOP
{: .no_toc }
So what are the essential components of OOP?

- *Classes*: A class is a blueprint for creating new objects – it defines a public interface, code for methods, and data fields.
- *Interfaces*: An interface is a related group of function prototypes that we want one or more classes to implement.
- *Objects*: An object represents a particular "thing" - like a circle of radius 5 at location (1,3). Each object  has its own interface, code, and field values.
- *Inheritance*: A derived class inherits either the code, the interface, or both from a base class.
The derived class can override the base class's code or add to its interface.
- *Subtype Polymorphism*: Code designed to operate on an object of type T (e.g., Person) can operate on any object that is a subtype of T (e.g., Student).
- *Dynamic Dispatch*: The actual code that runs when a method is called depends on the target object, and can only be determined at runtime.
- *Encapsulation*: Encapsulation is the bundling of a public interface and private data fields/code together into a cohesive unit.


### Encapsulation

Encapsulation is the guiding principle behind the OOP paradigm. There are two facets to encapsulation:
- We bundle related *public interface*, *data*, and *code* together into a cohesive, problem-solving machine.
- We *hide data/implementation* details of that machine from its clients – forcing them to use its public interface.

Encapsulation has many benefits:
- Simpler programs: We have reduced coupling between modules, simplifying our programs
- Easier Improvements: We can improve implementations without impacting other components
- Better Modularity: We can build a class once and use it over and over in different contexts

A big part of software engineering is anticipating future changes. One reason for OOP's success is that it helps people build software that is resilient to many types of change.


### Classes, Interfaces, Objects

- Class: A class is a blueprint that specifies a public interface, code, and fields that make up a type of object.
- Interface: An interface is a related group of function prototypes that describes behaviors that we want one or more classes to implement.
- Object: An object is a distinct value/entity, often created from a class blueprint – each object has its own logical copy of data and methods.

### Deep Dive: Classes, Interfaces, Objects
{: .no_toc }
- Class: A class is a blueprint that specifies a public interface, code, and fields that make up a type of object.
- Interface: An interface is a related group of function prototypes that describes behaviors that we want one or more classes to implement.
- Object: An object is a distinct value/entity, often created from a class blueprint – each object has its own logical copy of data and methods.

### Classes
{: .no_toc }

Here'a C++ class which shows most of the major component of a typical class:

```cpp
// Class Declaration
class Nerd { 
 public:
  Nerd(const string& name);
  ~Nerd();
  string talk() const;

 private:
  string name_;
  int IQ_;

  void study(int hrs);
};

// Implementation
Nerd::Nerd(const string& name) {
  name_ = name;
  IQ_ = 100;
  study(3);
}

Nerd::~Nerd() { IQ_ = 0; }
string Nerd::talk() const  { return name_ + " likes closures."; }
void Nerd::study(int hrs) { IQ_ += hrs; }
```

Key elements of classes include: the class name, constructors, destructors, public and private member variables/functions, the public interface, and method implementations.

#### Every time you define a class you define a new type
{: .no_toc }
When you define a new class, like Nerd above, it also defines a new type. If the class is a concrete class (i.e., all of its methods have { definitions }) and not an *abstract class* (which is missing one or more method { implementations }) then you can use this new type to define all of the following: objects, object references/pointers, and references. A class is a class, and a type is a type, and they're different entities. 

### Interfaces

An interface is a related group of function prototypes (declarations without { bodies }) that describes behaviors that we want one or more classes to implement. An interface specifies what we want one or more classes to do, but not how to implement these interfaces.

```java
// Java interface definition for shapes
interface Shape {
  public double area();
  public double perimeter();
  public void scale(double factor);
}
```

And here's an example from C++:

```cpp
// C++ interface definition for shapes
class Shape {
 public:
   virtual double area() = 0;
   virtual double perimeter() = 0;
   virtual void scale(double factor) = 0;
};
```

As you can see above, C++ we use pure-virtual functions within a class to define an interface.

Once we define an interface, we can ask a class to *implement* or *inherit* from it. Here's an example from Java that implements the Shape interface we defined above. You can see that the Square class provides { definitions/implementations } for each function declared in the interface. We can now say that a Square *supports*, *implements*, or *inherits* the Shape interface (that's the terminology you'd use).

```java
class Square implements Shape {
 ...
 public double area()
   { return w_ * w_; }
 public double perimeter()
   { return 4 * w_; }
 ...
}
```

Interfaces enable otherwise-unrelated classes to provide a common set of behaviors without actually inheriting any code like we learned about in CS32.

#### Every time you define an interface you define a new reference type
{: .no_toc }
When you define a new interface, like Shape above, it also defines a new type - specifically a *reference* type. Since an interface lacks any function { definitions/code }, you can't use a reference type to define full objects:

```java
Shape x = new Shape();  // Trying to allocate a "new Shape()" doesn't work!
```

But you can use reference types to define all of the following: object references, pointers and references, depending on which of the above your language supports. So we can use reference types like this:

```java
  void someFunc(Shape s) { ... }  // s is an object reference in Java, so this works
```

So here's a question: If each interface defines a type, and each class defines a type, and a class implements an interface – does the class have two types? The answer is Yes! For example our Square class actually has TWO types - it is of the Square type AND it is of the Shape type!  And you can pass a Square to any function that accepts either a Square or a Shape object reference, pointer or reference!

In contrast to an interface, when you define a class with all of its methods { implemented }, you create what's called a "value type" or a concrete type - and you may define all of the following: objects, object references/pointers, and references.  

### Objects

An object is a distinct value, often created from a class blueprint – each object has its own copy of fields and methods.

In most languages, objects are *instantiated* by using a class, e.g.:

```cpp
class Circle { ... };

Circle x;  // instantiates a Circle object named x in C++
```

A class serves as a blueprint for creating new objects.  But this is not always the case! In some languages, like JavaScript, there are no classes, only objects! Here's some code to create a JavaScript object containing fake information about famous computer scientist Alan Kay.  It has a bunch of fields (key-value pairs) as well as a method, fullName().  You can also see that we can update our object to add a function - we add tellJoke() to our object. Finally, we use our object to ask Alan to tell a joke.  No classes are involved in creating this object.

```js
// Javascript had objects but not classes!
var student1 = {
  first: "Alan",
  last: "Kay",
  phone: "818-555-1212",
  student_id: 958333245,
  fullName: function() 
    { return this.first + " " + this.last; }
};

student1["tellJoke"] = function() { return "Two mice walk into a bar"; }

console.log(student1.fullName() + " says " + student1.tellJoke());
```
### Classes, Interfaces, Object Challenges

Q: Why even support the notion of a class in a language? What are the pros and cons of using a model like JavaScript with just objects?

A: Classes enable us to create a consistent set of objects, all generated in a consistent way based on the class specification. Classes enable us to define a new type, which we'll see is useful for polymorphism in statically typed languages

Q: JavaScript can dynamically add methods to objects, is there any reason we can't dynamically add/remove methods to/from classes?

While most/all static languages don't allow this, dynamically typed languages absolutely allow this, Python being one example. In a statically typed language, adding a new method would potentially change the class interface which would complicate compilation/interpretation, but technically this could be done too.

## Shallow-dive Into Classes

Let's talk about some key aspects of classes that you may not have known before.

### Class Fields and Class Methods

Normally, each object has its own copy of every member variable (e.g., each Stack object has its own array/count). But sometimes we want to allow the overall class to have its own fields. A "class field" is one that's associated with the overall class – it's stored once in memory and that single copy is used by all of the class's objects. A "class method" is similarly associated with the overall class, and may only operates on class field.

What are the uses for class fields/methods?

- Defining Class-level Constants: If we have a constant that's shared by all objects of a class, we make it a class field.
- Counting the Number of Objects: If we want to keep a count of how many objects have been instantiated, the class can track this with a class field.
- Assigning Each Object a Unique ID: If we want to assign each object a unique ID number, we can use a class field to hold the next ID to be assigned.

#### Class Fields
{: .no_toc }
Here's an example from C++. We'll update our Nerd class to not only track names/IQs for individual nerds, but to track how many total nerd objects there are overall. We'll do this tracking with a class field that keep the total count (num_nerds_) and use our constructor/destructor to up/down the count. 

```cpp
// nerd.h
class Nerd { 
 public:
  Nerd(const string& name) {
   name_ = name;
   IQ_ = 100;
   ++num_nerds_;
  }
  ~Nerd() {
   --num_nerds_;
  }
  string talk() const { ... }

 private:
  string name_;
  int IQ_;
  static int num_nerds_;

  void study(int hrs) { ... }
};
```

```cpp
// nerd.cpp
// initialize class fields in the cpp; initializes value of our class field.
int Nerd::num_nerds_ = 0;
```

In C++, when we define a member variable as *static* that means that it's a class-level field, stored in memory just once for the class. Each object/instance doesn't store this field within their memory! Instead, the field is stored once in the static data area of the process's RAM per class and all object methods just refer to this singleton value. So all objects share the same singleton variable `num_nerds_`.

#### Class Methods
{: .no_toc }
A class method is one that can ONLY access class fields - it cannot access ANY instance fields (e.g., name_ or IQ_) inside of an object, because it's not associated with any particular object. It's associated instead with the overall class.  Here's an example in C++:

```cpp
class Foo { 
 public:
  Foo(int v) {
    val_ = v;
    ++num_foos_;
  }
  ~Foo() { --num_foos_; }
  int inc()
   { ++val_; return val_; }

  static int getActiveFooCount()
   { return num_foos_; }

 private:
  int val_;
  static int num_foos_;
};

int main() {
  Foo a(5);

  cout << "Count of foos: " << a.getActiveFooCount() << endl;
  cout << "Count of foos: " << Foo::getActiveFooCount();
}

```

In the above example, the `getActiveFooCount()` method is a class-level method, because it's defined as static.  It may only access class-level fields or call other class-level methods, since it knows nothing about instance-level fields/methods. As you can see, in C++ we have two ways of calling a class-level method: using an instance variable name, like *a* followed by a dot, or by using the class name, like *Foo*, followed by two colons. Other languages vary in the syntax used to call class-level methods.

Here's an example from python - you can see we use the class name (e.g., Foo) followed by a period to call a class method or access a class variable:

```python
# Python class variables/functions
class Foo:
 num_foos_ = 0  # class variable

 def __init__(self, v):
   val_ = v
   Foo.num_foos_ = Foo.num_foos_ + 1

 def __del__(self):
   Foo.num_foos_ = Foo.num_foos_ - 1

 def inc(self):
   val_ = val_ + 1
   return val_

 @classmethod
 def get_num_foos(cls):
   return cls.num_foos_

a = Foo(5)
b = Foo(6)
print("Number of foos:", Foo.get_num_foos())

```

And here's an example from Java. It looks pretty similar to C++ in that it uses static to define class methods/fields:

```java
// Java class variable/method example
public class Nerd {
  private String name;
  private int IQ;
  private static int num_nerds;
  
  static {
    num_nerds = 0;
  }
  public Nerd(String name) {
    this.name = name;
    this.IQ = 100;
    num_nerds++;
  }
  public void finalize() {
    this.IQ = 0;
    num_nerds--;
  }

  public static int getNumNerds() {
    return num_nerds;
  }
}
```

```java
// Java calling a class method getNumNerds()
public static void main(String args[]) {
  Nerd n = new Nerd("Carey");
  System.out.println("There are " + Nerd.getNumNerds() + " nerds.");
}

```

### Class Field and Class Method Challenge
{: .no_toc }
Q: When would we want to use class-level methods?

A: We use class-level methods when a method only accesses class-level variables and thus doesn't need access to member variables. We might also use class-level methods when defining a class that contains a bunch of related functions that aren't associatee with individual objects. For example, imagine we're defining a Math class with functions like sqrt(), exp(), tan(), cos(), etc. Those functions could all be class functions and placed into a Math class.

```cpp
class MathLibrary {
   static double sqrt(double x); // doesn't need access to member variables
   static double exp(double x);  // ditto!
}; 
```

Q: Can a class-level method call a traditional instance-level method? What about visa-versa? Why?

A: No, a class method can't call an instance method, because an instance method by definition is associated with an instance, but a class method is not. So when the class method runs, it has no idea what object it's associated with, so it can't call an intance method - which would be associated with a particular object. On the other hand, an instance method can call a class method just fine.

### Classify That Language: Class Functions
{: .no_toc }
Which, if any of the methods in this class are class methods (as opposed to instance methods)?

```rust
struct Nerd {
  name: String,  
  iq: i32
}
 
impl Nerd {
  // constructor
  pub fn new(name: String) -> Nerd {
    return Nerd { name: name, iq: 100 }
  }
    
  pub fn study(&mut self, hrs: i32) {
    self.iq += hrs;
    println!("{} studied {} hours.", self.name, hrs);
    println!("My incredible IQ is now {}.", self.iq)
  }
}
 
fn main () {
  let mut n = Nerd::new("Carey".to_string());
  n.study(3);
}

```

Notice how the study() method has a self parameter (like we see in python, or "this" in C++).  That identifies study() as an instance method. But notice that the new() method - a constructor in this language - does not have a self parameter. That is an indication it's a class method. It has no way of identifying a particular instance/object without a self field. So constructors in this langauge are class-level methods, and they must construct and then return a new object.  This language is Rust!

### This and Self

When a method is called, it must be told what object it is to operate on (so it can find the object's member fields). In most languages, this is done transparently when you call a method on an object p, e.g., p.printMyAddress().  The language passes a reference to p as a hidden extra argument, and it may be referenced in the method as self or this.  Self/this is an object reference or pointer in most languages.

Here's an illustration of how `this` is implemented in c++:

```cpp
// The code you write:
class Nerd {
 private:
   string name;
   int IQ;

 public:
   Nerd(const string& name) {
     this->name = name;
     IQ = 100; // this->IQ is optional
   }

   string talk() {
     return this->name + " likes PI."; 
   }
};

int main() {
 Nerd n("Carey");
 cout << n.talk();
}
```

```cpp
// What's actually happening for the code above:
struct Nerd {
  string name;
  int IQ;
};

void init(Nerd *this, const string& name) {
  this->name = name;
  this->IQ = 100;  
}

string talk(Nerd *this) {
  return this->name + " likes PI.";
}

int main() {
 Nerd n;
 init(&n, "Carey");
 cout << talk(&n);
}

```

And here's an example in Python that uses self:

```python
# Python Nerd class
class Nerd:
 def __init__(self, name):
   self.name = name
   self.IQ = 100

 def talk(self):
   return self.name + " likes closures."
```

In python, using the `self.` prefix is mandatory, unlike in many other languages where self/this is usually optional (unless we need to bypass shadowing of a local variable).

Here's an example in go:

```golang
// Golang Nerd class
type Nerd struct {
  name string
  IQ   int
}

func New(name string) *Nerd {
 ...
}

func (self *Nerd) Talk() string {
  return self.name + " likes closures."
}

func (self *Nerd) Study(hrs int) {
  self.IQ += hrs
}

...

n := New("Carey")
fmt.Println(n.Talk())
```

It's interesting to note that in Go, the self parameter is explicit and it's placed *before* the function name, all by itself. Then the usual parameters are placed after the function name. 

### This and Self Challenges
{: .no_toc }
Q: Would a class method (as opposed to an instance method) also have a this/self parameter? Why or why not?

A: Class-level methods operate on the overall class and don't even know what instance they're operating on (if any), so no this/self pointer or object reference is passed to them. 


Q: In Python, could you make the `self.` prefix optional?  If not, why not?

A: In Python as it currently stands the `self.` prefix is required. While a reference to read a variable could figure out if a variable is a local or a member variable, assigning a variable (without `self.`) would be problematic. Would `a = 5` create a new member variable (`self.a`) or would it create a new local variable?  Without explicit member variable declarations, which Python doesn't have, assignment without the self prefix would be ambiguous.

### Classify That Language: This and Self
{: .no_toc }
```perl
package Employee;   # Equivalent to a class def.

sub new_employee {  # constructor function
  # Parameters held in $_[0], $_[1], ... $_[n-1]
  # employee name in $_[0], and income in $_[1]
  my $obj = {};   
  $obj->{"name"} = $_[0];   # param 0 is name
  $obj->{"income"} = $_[1]; # param 1 is income
  bless $obj, "Employee";
  
  return $obj;  # return the object
} 

sub compute_ytd_income { # func. to compute income
  my $emp = $_[0];  # 1st param is employee obj
  my $months_worked = $_[1]; 
  return $emp->{'income'} * $months_worked/12;
} 

...

# Code to create/use an employee
use Employee;

$e = new_employee("Ed", 100000);
print "My name: ", $e->{"name"};
print "My income thru Nov is: ",
      $e->compute_ytd_income(11);
```

Q: Does this language have a notion of this/self for objects? Explain!

A: Yes! When we call a method, e.g.,` $e-\>compute_ytd_income()`, this passes the object reference `$e` as the first argument to the method (as `$_[0]`). In this language, function parameters are placed in array called `$_` and you can index them. This is Perl.

### Access Modifiers

As we learned, a key goal of encapsulation is to hide the data/implementation details from users of a class. Languages provide "access modifier" syntax to enable a class to designate what parts of a class are publicly visible and to hide the class's data/implementation.

There are generally three different access modifiers:

- *public*: A public method or data member in class C may be accessed by any part of your program.
- *protected*: A protected method or data member in class C may be accessed by C or any subclass derived from C.
- *private*: A private method or data member in class C may only be accessed by methods in class C.

Public, protected and private generally have the same semantics across languages, though there are exceptions. For instance, Java's protected keyword not only lets subclasses access the member/method, but also lets any other class defined in the same source file.

If you omit an explicit access modifier, different languages have different defaults – some default fields/methods to private, some default to public, etc. Python defaults to public, for example, while C++ defaults to private. 

Here's an example from Swift:

```swift
// Swift access modifiers
class Person {
  private var name: String

  public init(name: String) {
    self.name = name
  }
  public func talk() {
   print("Hi, my name is "+self.name)
  }
  private func poop() {
    print("Grunt... plop!")
  }
}

class Student: Person {
  ... 
  public func dostuff() {
     self.talk()     // Works – talk is public
     self.poop()     // Error – poop private
  }
}

var p = Person(name:"Irna")
p.talk()     // Works – talk is public
p.poop()     // Error – poop private

```

Note: Swift doesn't have a protected keyword! It does have something called *internal* which is similar but slightly different, however.

Here's how we do public/protected/private in Python. It uses underscores to indicate protected/private methods:

```python
# Python access modifiers
class Person:
 def __init__(self, name):
   self.__name = name

 def talk(self):         # Public method
   print("Hi, my name is "+ self.__name)

 def _daydream(self):    # Protected method
   print("Rainbow narwhals")
  
 def __poop(self):       # Private method
   print("Grunt... plop!")


# Derived class
class Student(Person):
 ...
 def do_stuff(self):
   self.talk()           # Works fine - public
   self._daydream()      # Works fine - protected
   self.__poop()      

a = Student("Cindi")
a.talk()          # Works fine - public
a._daydream()     # This works!! - Python doesn't enforce protected methods
a.__poop()        # Generates error
```

#### Encapsulation Best Practices
{: .no_toc }
Here are some things to think about when using encapsulation.

- Design your classes such that they hide all implementation details from other classes.
- Make all member fields, constants, and helper methods private. 
- Avoid providing getters/setters for fields that are specific to your current implementation, to reduce coupling with other classes.
- Make sure your constructors completely initialize objects, so users don't have to call other methods to create a fully-valid object.

#### Classify That Language: Access Modifiers
{: .no_toc }
Our Comedian class has both public and private methods... but no explicit public or private keywords!! Assuming comedians are only serious in private, how does this language hide private members?

```golang
package comedian

type Comedian struct {
  name string
  joke string
}



func (c *Comedian) TellJoke(times int) {
  for i := 1; i < times; i++ {
    fmt.Println(c.joke)
  }
}

func (c *Comedian) GetName() string {
  return c.name
}

func (c *Comedian) beSerious() {
  fmt.Println(c.name + " saves for retirement!\n")
}

func main() {
  c1 := Comedian{"Carey", "Knock, knock..."}
  c1.TellJoke(5)
}
```

Answer: In this language, methods that begin with a capital letter are public, and those that begin with a lower-case letter are private.  This is GoLang!

### Properties, Accessors and Mutators

While our goal in OOP is encapsulation, sometimes we want to expose a field (aka property) of an object for external use. For example, in a Person object we might want to expose a person's name externally.

- A property is some attribute of an object, e.g., a person's age or name.
- An accessor is a method that gets the value of such a property.
- A mutator is a method that changes the value of a property.

Below is how we'd create accessors/mutators in C++ - you have to roll your own.  But many programming languages have syntax to define accessors and mutators more easily.

```cpp
class Person {
public:
  string getName() const;       // accessor
  void setName(string name);    // mutator
};
```

Why use accessors? Accessors and mutators for a property enable us to hide a field's implementation and more easily refactor. We can define a function which abstracts the underlying implementation of a given field/property. Some languages actually support built-in syntax to create accessors/mutators for properties.  Let's look at a few examples:

```python
# Python's accessors/mutators
class Person: 
 ...

 @property
 def age(self): 
   return self._age

 @age.setter
 def age(self, age):
   if age >= 0 and age <= 130:
     self._age = age
   else:
     raise ValueError("Invalid age!")

a = Person("Archna", 22)
print(a.age)
a.age = 23
```

In python, the first age() method is an accessor method. The @property designation is python's way of indicating that this function is an accessor. Simply referring to the function by name, just like you would a member variable - causes it to be called. Notice how we simply refer to `a.age` in the call to print(`a.age`). This looks just like its reading the value of a member variable but it's really calling a function, whose return value is treated like the value of the member variable. Similarly, `@age.setter` designates that the second age method (which takes two parameters) is a mutator method. Simply using assignment, as in `a.age = 23`, causes this method to be called and the value of 23 to be passed in as the age argument.

Here's another example from Kotlin, which defines accessors/mutators:

```kotlin
// Kotlin's accessors/mutators
class Temperature {
  var celsius: Double = 0.0
    get() { return field }
    set(value) { field = value }
  var fahrenheit: Double  
    get() {
      return ((celsius * 1.8) + 32.0)
    }
    set(value) {
      celsius = (value - 32)/1.8
    }
}

fun main(args: Array<String>) {
  var b = Temperature()
  b.fahrenheit = 98.6
  println(b.celsius)  // 37
}
```

In Kotlin, we may define one or more properties (celsius, fahrenheit) and then specify a getter and setter for each such property. And in fact, the property might just be synthetic, as fahrenheit is above (in other words, it doesn't have a concrete member variable). Any reference to fahrenheit really uses the backing celsius field for its computations.

#### Property Best Practices
{: .no_toc }
So when should you write a property vs. defining a traditional method (in a language with explicit support/syntax for properties)? Here's the rule of thumb if your language supports syntax for properties: Use properties when you're exposing the state of a class (even if exposing that state requires minor computation). Use traditional methods when you're exposing behaviors of a class or exposing a state that requires heavy computation (e.g., a database query that might take tens of milliseconds or more). In general, you don't want to use properties to expose implementation details of a class that might change if you update your implementation. So exposing a property of a name would be fine, but exposing a property for some internal member variable that is specific to your current implementation would be bad.

## Inheritance

You learned about one type of inheritance in CS32, but there are actually four types:

- Interface Inheritance: This is about creating many classes that share a common public base interface `I`. A class supports interface `I` by providing implementations for all `I`'s functions.
- Subtype Inheritance: A base class provides a public interface and implementations for its methods. A derived class inherits both the base class's interface and its implementations.
- Implementation Inheritance: This is about reusing a base class's method implementations. A derived class inherits method implementations from a base class.
- Prototypical Inheritance: One object may inherit the fields/methods from a parent object. This is used in JavaScript.

Let's learn all of them!

### Interface Inheritance

Here's how it works. You define a public interface, which is basically a collection of related function prototypes (aka declarations). Here's an example of an interface in C++ - C++ just uses fully-abstract classes to define interfaces (where none of the functions has an implementation):

```cpp
// C++ pure interface inheritance example
class Shape {
public:
  virtual double area() const = 0;
  virtual double perimeter() const = 0;
  virtual void scale(double factor) = 0;
};
```

Notice that these are pure-virtual methods - there is no implementation specified. They specify "what" to do, but not "how." Then you can define a class that "inherits" (aka) implements this interface:

```cpp
class Circle : public Shape {
public:
 Circle(double radius) { rad_ = radius; }
 virtual double area() const
   { return PI * rad_ * rad_; }
 virtual double perimeter() const
  { return 2 * PI * rad_; }
 virtual void scale(double factor)
  { rad_ *= factor; }
private:
  double rad_;
};
```

With interface inheritance we can have entirely unrelated/different classes that support a common interface:

```cpp
class Washable {
public:
  virtual void wash() = 0;
  virtual void dry() = 0;
};

class Car: public Washable {
public:
  void drive() { cout << "Vroom vroom"; }
  void wash() { cout << "Use soap"; }
  void dry()  { cout << "Use rag"; }
};

class Person: public Washable {
public:
  void talk() { cout << "Hello!"; }
  void wash() { cout << "Use shampoo"; }
  void dry()  { cout << "Use blowdryer"; }
};
```

Notice that even though Car and Person are unrelated, they both support a Washable interface, and can both be washed!

With interface inheritance we can also have a single class support multiple interfaces. For example, in the example Below a Car can be both washed and driven:

```cpp
class Washable {
public:
  virtual void wash() = 0;
  virtual void dry() = 0;
};

class Drivable {
public:
  virtual void accelerate() = 0;
  virtual void brake() = 0;
};

class Car: public Washable, public Drivable {
public:
  // Car methods
  void turnOn() { cout << "The car is on!"; }

  // Washable methods
  void wash() { cout << "Use soap"; }
  void dry()  { cout << "Use rag"; }

  // Drivable methods
  void accelerate() { cout << "Vroom vroom"; }
  void brake() { cout << "Screeeeeech"; }
};
```

Once we define an interface we can find functions that specify the interface as the type of a parameter:

```cpp
// We can use the interface as a reference type, so now w can refer to any washable object
void cleanUp(Washable& w) {
  w.wash();
  w.dry();
}

// We can use the interface as a reference type, so now w can refer to any drivable object
void driveToUCLA(Drivable& d) {
 d.accelerate();
 // waste 1 hour in traffic
 d.brake();
}

int main() {
  Car forrester;
  cleanUp(forrester);
  driveToUCLA(forrester);
}

```

Neat, right? Now our above Car class has three personas. It is a Car, it is a Drivable thing, and it is a Washable thing.

#### Examples in Other Languages
{: .no_toc }
Here's an example from Swift; it uses the `protocol` keyword to mean interface:

```swift
protocol Shape {
  public func area() -> Double
  public func perimeter() -> Double
  public func scale(factor: Double)
}

class Circle: Shape {
  init(radius: Double)
    { rad_ = radius; }
  public func area() -> Double
    { return PI * rad_ * rad_; }
  public func perimeter() -> Double
    { return 2 * PI * rad_; }
  func scale(factor: Double)
    { rad_ *= factor; }

  private var rad_: Double
}
```

Here's a similar example in Java, which uses `implements`:

```java
public interface Shape {
  public double area();
  public double perimeter();
  public void scale(double factor);
};

class Circle implements Shape {
  public Circle(double radius)
    { rad_ = radius; }
  public double area()
    { return PI * rad_ * rad_; }
  public double perimeter()
   { return 2 * PI * rad_; }
  public void scale(double factor)
   { rad_ *= factor; }

  private double rad_;
};
```

Go has an interesting approach: they use *structural* inheritance, where you don't need to explicitly say that a `Circle` implements a `Shape`; all that needs to be true is that it implements the relevant functions.

```go
type Shape interface {
  Area() float64
  Perimeter() float64
  Scale(f float64)
}

type Circle struct {
  x      float64
  y      float64
  radius float64
}

func (c Circle) Area() float64 {
	return PI * c.radius * c.radius }
func (c Circle) Perimeter() float64 {
	return 2 * PI * c.radius }
func (c Circle) Scale(f float64) {
	c.radius *= f }
```

This is also how languages like Rust work!

#### Common Use Cases
{: .no_toc }
Interfaces are ubiquitous in object-oriented programming. Here are a few places you might encounter them:

1. comparing objects - ex Java's `Comparable`
2. iterating over an object - ex Java's `Iterable`
3. (from class: serializing an object!)


First, let's take a look at Java's `Comparable`:

```java
// This interface is part of the Java language distribution:
public interface Comparable<T>
{
  // Compares two objects, e.g., for sorting
  public int compareTo(T o);
}

public class Dog implements Comparable<Dog> {
  private int bark;
  private int bite;
  Dog(int bark, int bite) {
    this.bark = bark;
    this.bite = bite;
  }

    public int compareTo(Dog other) {
    int diff = this.bark - other.bark;
    if (diff != 0) return diff;
    return this.bite - other.bite;
  }
}

ArrayList<Dog> doggos = new ArrayList<>();
doggos.add(new Dog(10, 20));
doggos.add(new Dog( 3, 48));
doggos.add(new Dog(15, 4));

doggos.sort(Comparator.naturalOrder()); // sort() uses the Comparable interface to compare objects
```

We can also use interfaces to make objects iterable:

```java
// This interface is part of the Java language distribution:
public interface Iterable<T>
{
  Iterator<T> iterator();
}

public class Dog { ... }

class Kennel implements Iterable<Dog> {
    private ArrayList<Dog> list;

    public Kennel() {
      list = new ArrayList<Dog>();
    }
    public void addDog(Dog doggo)
      { list.add(doggo); }

    public Iterator<Dog> iterator()
      { return list.iterator(); }
}

...
Kennel kennel = new Kennel();
kennel.addDog(new Dog("Fido",...));
kennel.addDog(new Dog("Nellie",...));

for (Dog d : kennel) {  // Java's for loop automatically uses the Iterable interface to iterate
  d.bark();
}
```

#### Best Practices (Pros & Cons)
{: .no_toc }
Use Interface Inhertance when you have a "can-support" relationship between a class and a group of behaviors.

- The Car class can support washing.
- The kennel class can support iteration.

Use Interface Inhertance when you have different classes that all need to support related behaviors, but aren't related to the same base class.

- Cars and Dogs can both be washed, but aren't related by a common base.

Pros:

- You can write functions focused on an interface, and have it apply to many different classes.
- A single class can implement multiple interfaces and play different roles

Cons:

- Doesn't facilitate code reuse

### Subclassing Inheritance

This is the inheritance we learned about in CS32, and the one that most people think of when they think of inheritance.

Here's how it works:

- We create a base class `B`, which provides a public interface and method implementations
- We create a derived class `D` which inherits `B`'s public interface and its method implementations
- Derived class `D` may override `B`'s methods, or add its own new methods, potentially expanding `D`'s public interface.

Here's an example from C++:

```cpp
// C++ subclassing example
class Shape {
public:
  Shape(float x, float y)  { x_ = x; y_ = y; }
  void setColor(Color c)   { color_ = c; }
  virtual void disappear() { setColor(Black); }
  virtual float area() const = 0;
...
protected:
  void printShapeDebugInfo() const { ... }
};

class Circle: public Shape {
public:
  Circle(float x, float y, float r) :  Shape(x,y)
  	{ rad_ = r; }
  float radius() const { return rad_; }
  virtual float area() const { return PI * rad_ * rad_; }
  virtual void disappear() {
    for (int i=0;i<10;++i)
    	rad_ *= .1;
        Shape::disappear();
  }
};

```

Notice that that the derived class, Circle, not only inherits the public interface of Shape (e.g., setColor, disappear, and area), but also inherits the implementation/code  of some of these functions. Because the derived class inherits the base's public interface and its implementation, it can do anything the base class can do! And you can pass the derived class anywhere the base class is expected (because the derived class is a subtype of the base class). And since the derived class's interface is expanded, it can also do its own specialized derived things.

#### Examples in Other Languages
{: .no_toc }
Here's an example from Java:

```java
// Java class for Shapes
abstract class Shape {
  Shape(float x, float y)
    { x_ = x; y_ = y; }
  public final void setColor(Color c)
    { color_ = c; }
  public void disappear()
    { setColor(Black); }
  public abstract float area();

  protected final void printShapeDebugInfo()
    { ... }

  private float x_;
  private float y_;
  private Color color_;
}

class Circle extends Shape {
  Circle(float x, float y, float r) {
    super(x, y);
    rad_ = r;
  }
  public float radius()
    { return rad_; }
  public void disappear() {
    for (int i=0;i<10;++i)      rad_ *= .1;    super.disappear();
  }
  public float area()
    { return PI*rad_*rad_; }

  private float rad_;
}
```

Here's a similar example in Swift:

```swift
class Shape {
  init(x : Double, y : Double) {
    x_ = x
    y_ = y
  }
  public func setColor(c : Color)
    { color_ = c }
  public func disappear()
    { setColor(c:Black) }
  public func area() -> Double
    { return 0 }
  func printShapeDebugInfo()
     { ... }

  private var x_ : Double;
  private var y_ : Double;
  private var color_ : Int = White;
}

class Circle:  Shape {
  init(x : Double, y : Double, r : Double) {
    rad_ = r
    super.init(x : x, y : y)
  }
  public func radius() -> Double
    { return rad_  }
  override public func disappear() {
    for i in 1...10 {
      rad_ *= 0.1
    }
    super.disappear()
  }
  override public func area() -> Double
    { return PI * self.rad_ * self.rad_ }

  private var rad_ : Double;
}
```

Some quick differences:

- Swift doesn't support abstract methods (this is a holdover from Objective-C)
- Swift requires the `override` keyword, rather than implicit behaviour (like in Java)

And, we even have subclassing in Python!

```py
class Shape:
  def __init__(self, x, y):
    self.x = x
    self.y = y
  def set_color(self,c):
    self.color = c
  def disappear(self):
    self.setColor(Black)
  def area(self):
    pass
  def _printShapeDebugInfo(self):
    # ...

class Circle(Shape):
  def __init__(self, x, y, r):
    super().__init__(x, y)
    self.rad = r
  def radius(self):
    return self.rad
  def disappear(self):
    for i in range(10):
      self.rad *= 0.1
    super().disappear()
  def area(self):
    return PI * self.rad * self.rad
```

Some interesting things:

- we've used the `pass` keyword to specify an abstract method (though, it turns out there are a few ways to do it)
- we need to explicitly call the constructor of the superclass!

#### Subtype Inheritance Best Practices
{: .no_toc }
Use subtype inheritance when:

- When there's an is-a relationship:
	- A Circle is a type of Shape
	- A Car is a type of Vehicle
- When you expect your subclass to share the entire public interface of the superclass AND maintain the semantics of the super's methods
- When you can factor out common implementations from subclasses into a superclass:
	- All Shapes (Circles, Squares) share x,y coordinates and a color along with methods to get/set these things.

When should you not use subtype inheritance? Consider this example:

```cpp
// C++ Collection class
class Collection {
public:
  void insert(int x) { ... }
  bool delete(int x) { ... }
  void removeDuplicates(int x) { ... }
  int count(int x) const { ... }
  ...

private:
  int *arr_;
  int length_;
};

// C++ Set class
class Set: public Collection {
public:
  void insert(int x) { ... }
  bool delete(int x) { ... }
  bool contains(int x) const { ... }
  ...

private:
  int *arr_;
  int length_;
};
```

Why wouldn't we want to use it in this case? Because:

- Reason #1: Our derived class doesn't support the full interface of the base class.
- Reason #2: The derived class doesn't share the same semantics as the base class.

In these cases, where you'd like to inherit the functionality of a base class but don't want to inherit the full public interface (because it doesn't make sense to do so in your derived class) you can use composition and delegation. Composition is when you make the original class a member variable of your new class. Delegation is when you call the original class's methods from the methods of the new class. Here's an example:

```cpp
class Collection {
public:
  void insert(int x) { ... }
  bool delete(int x) { ... }
  int count(int x) const { ... }
};

class Set
public:
  void insert(int x) {
    if (c_.howMany() == 0) c_.insert();   // delegation
  }
  bool delete(int x) {
    return c_.delete(x)
  }
  bool contains(int x) const {
    return c_.count(x) == 1;
  }
  ...
private:
  Collection c_;    // composition
};
```

Composition/delegation has the benefit of hiding the public interface of the original class (Collection) and at the same time the second class can leverage all of its functionality.

#### Pros and Cons
{: .no_toc }
OK, let's sum up the pros/cons of subtype inheritance:

Pros:
- Eliminates code duplication/facilitates code reuse
- Simpler maintenance – fix a bug once and it affects all subclasses
- If you understand the base class's interface, you can generally use any subclass easily
- Any function that can operate on a superclass can operate on a subclass w/o changes

Cons:
- Often results in poor encapsulation (derived class uses base class details)
- Changes to superclasses can break subclasses (aka Fragile Base Class problem, which we'll learn about in week 9)

### Implementation Inheritance

In Implementation Inheritance, the derived class inherits the method implementations of the base class, but NOT its public interface. The public interface of the base class is hidden from the outside world. This is often done with private or protected inheritance

```cpp
// C++ Collection class
class Collection {
public:
  void insert(int x) { ... }
  bool delete(int x) { ... }
  int count(int x) const { ... }
  ...
};

// C++ Set class
class Set: private Collection	// private inheritance
public:
  void add(int x)
    { if (howMany() == 0) insert(x); }
  bool erase(int x)
    { return delete(x); }
  bool contains(int x) const
    { return count(x) > 0; }
  ...
};
```

Since Set *privately* inherits from Collection, it inherits all of its public/protected methods, but it does NOT expose the public interface (insert(), delete(), count(), ...) of the base Collection class publicly. And for that matter, Set is NOT a subtype of Collection. The Set class hides the fact that it has anything to do with the Collection base class. So you CAN'T pass a Set to a function that accepts a Collection, since as far as the language is concerned, they're unrelated types.

In general, implementation inheritance is frowned upon. If you want to have a class "inherit" all of the functionality of a base class, but hide the public interface of the base class, it's always better to use composition + delegation to do so.

### Classify That Language: Inheritance
{: .no_toc }
```rust
// Equivalent of a Dog class, with bark method.
struct Dog {
  name: String,
  stomach: String
}

impl Dog {
  fn bark(&self) { println!("Woof!") }
}

trait Ingests {
  fn eat(&mut self, food: &str);
  fn drink(&mut self, liquid: &str);
}

impl Ingests for Dog {
  fn eat(&mut self, food: &str) {
    println!("{} eats {}", self.name, food);
    self.stomach = food.to_string()
  }
  fn drink(&mut self, liquid: &str) {
    println!("{} laps {}", self.name, liquid);
    self.stomach = liquid.to_string()
  }
}

fn main() {
  let mut fido = Dog {...};
  fido.bark();
  fido.eat("grass");
  fido.drink("toilet water")
}
```

What type of inheritance is this program using?

- Interface Inheritance
- Subclassing/Hybrid Inheritance
- Implementation Inheritance

Answer: It's using interface inheritance. In this language (Rust), the name for an interface is a "trait."

### Prototypal Inheritance
{: .no_toc }
This one's mostly for completeness, but we won't test you on it. Prototypal Inheritance is used in languages that only have objects, not classes. An example would be JavaScript. In this case, an object may inherit the fields and methods from one or more "prototype" objects.

As it turns out, every JavaScript object has a hidden reference to a parent (aka "prototype") object. An object may specify that it inherits the fields/methods of some other object. Any fields/methods of the same name in the "derived" object as in the "base" object, shadow those in the "base" object:

```javascript
obj1 = {
  name: "",
  job: "",
  greet: function()
          { return "Hi, I'm " + this.name; },
  my_job: function()
             { return "I'm a " + this.job; }
}

obj2 = {
  name: "Carey",
  job: "comedian",
  tell_joke: function()
    { return "USC education"; },
  __proto__: obj1
}
```

Notice the __proto__ field in obj2?  This says that obj2 inherits all of the fields/methods of obj1. So you could call obj2.greet(), or obj2.my_job() and it works fine. Since obj2 redefines name to "Carey", any functions called on obj2, including those originally defined in obj1, like greet(), will use obj2's version of the field! So a call to obj2.greet() would print out "Hi I'm Carey".

### Shallow Dive into Inheritance

Here, we'll focus more on the *mechanics* of inheritance, rather than comparing paradigms.

#### Construction

Construction follows the same basic pattern in all languages! When you instantiate an object of a derived class, this calls the derived class's constructor. Every derived class constructor does two things:

- It calls its superclass's constructor to initialize the immediate superclass part of the object
- It initializes its own parts of the derived object

In most languages, you must call the superclass constructor first from the derived class constructor, before performing any initialization of the derived object:

```java
// Construction with inheritance in Java
class Person {
 public Person(String name) {
   this.name = name;
 }
 private String name;
}

class Nerd extends Person {
  public Nerd(String name, int IQ) {
    super(name);  // calls superclass constructor first
    this.IQ = IQ; // then initializes members of the drived class
  }
  private int IQ;
}

class ComedianNerd extends Nerd {
  public ComedianNerd(String name, int IQ, String joke) {


  }
  private String joke;
}
```

In other languages, like Swift, the derived constructor initializes the derived object's members first, then calls the base class constructor.

Some languages will auto-call the base-class constructor if it's a default constructor (has no parameters), but other languages, like Python *always* require you to call the base constructor explicitly:

```python
# Python inheritance+construction
class Person:
  def __init__(self):
    self.name = "anonymous"

  ...

class Nerd(Person):
  def __init__(self, IQ):
    super().__init__()  // if we leave this out, we simply won't initialize our Person base part!
    self.IQ = IQ
  ...

```

#### Destruction

When destructing a derived object, the derived destructor runs its code first and then implicitly calls the destructor of its superclass. This occurs all the way to the base class.

#### Finalization

With Finalization, some languages require an explicit call from a derived class finalizer to the base class. In other languages, the derived finalizer automatically/implicitly calls the base class finalizer. This varies by language, so make sure to figure out what your language does.

### Multiple Inheritance and Its Issues

Multiple inheritance is when a derived class inherits implementations from two or more superclasses, e.g.:

```cpp
class Person { ... };
class Robot { ... };

class Cyborg: public Person, public Robot
{ ... }
```

Multiple inheritance is considered an anti-pattern, and forbidden in most languages! In cases where you want to use multiple inheritance, have your class implement multiple interfaces instead.

Here's an example where there's a problem:

```cpp
class CollegePerson {
public:
  CollegePerson(int income) { income_ = income; }
  int income() const { return income_; }
private:
    int income_;
};

class Student: public CollegePerson {
public:
  Student(int part_time_income) :
    CollegePerson(part_time_income)
  {/* Student-specific logic */ }
};

class Teacher: public CollegePerson {
public:
  Teacher(int teaching_income) : 
    CollegePerson(teaching_income)
  { /* Teacher-specific logic */ }
};

class TA: public Student, public Teacher  {
public:
    TA(int part_time_income, int ta_income) : 
      Student(part_time_income),
      Teacher(ta_income) 
    { /* TA-specific logic */ }
};

int main() {
 TA amy(1000, 8000);
 cout << "Amy's pay is $"
      << amy.income();
}
```

Notice that in this example, both Student and Teacher inherit from the same base class, CollegePerson. Also notice that CollegePerson has an income() method that returns the CollegePerson's income\_ field.  But here's the problem, since a TA class inherits from both Student *and* Teacher, and each of these has a CollegePerson base part (each with their own copy if the income_ field), the TA class inherits TWO copies of the income_ field, and two copies of the income() method!  So when you ask for a TA's income(), does this call Student's version of income() or Teacher's version of income()?  Given the example above, if we called Student's version, we'd get a result of $1000, but if we called Teacher's version we'd get a result of $8000. This is called the "diamond pattern" - it occurs when a derived class D inherits from a base class B twice by inheriting from two distinct classes X, Y that both on their own are derived from B. It actually inherits the base class B twice, and thus has two copies of every member field, etc.

Needless to say, this is very confusing for the programmer.  Some languages, like C++ will generate an error for this: "error: non-static member income' found in multiple base-class subobjects of type 'CollegePerson'". Other languages have a mechanism to pick one of the two functions to always call, or perhaps let the programmer specify which base version to call (e.g., TA.income()).  Regardless, you want to avoid the diamond pattern, and multiple inheritance more generally.

We can even have issues with multiple inheritance without a diamond pattern. Consider what would happen if two base classes define a method with the same prototype, and then a third class inherits from both base classes. When calling that method, which version should be used?  It gets complicated.

TL;DR - don't use multiple subclass inheritance; inherit from multiple interfaces instead

### Abstract Methods and Classes

Abstract methods are methods that define an interface, but don't have an implementation. For instance, area() and perimeter() are abstract methods in the example below:

```cpp
class Shape { 
public:
  double area() const = 0;
  double perimeter() const = 0;
};
```

An abstract method defines "what" a method is supposed to do along with its inputs and outputs – but not "how." An abstract class is a class with at least one abstract method - it defines the public interface and partial implementation for subclasses.

When defining a base class, most languages allow the programmer to specify one or more "abstract methods", which are just function prototypes without implementations.

```cpp
class Shape { // C++ class
public:
  // area and rotate are "abstract" methods
  virtual double area() const = 0;
  virtual void rotate(double angle) = 0;
  virtual string myName() 
    { return "abstract shape"; }
};
```

```java
abstract class Shape {  // Java class
 // area and rotate are "abstract" methods
 public abstract double area();
 public abstract void rotate(double angle);
 public String myName()
   { return "abstract shape"; }
};
```
Question: Can we instantiate an object of an A.B.C. like Shape?
```cpp
int main() {
  Shape s;  // ?????
  cout << s.area(); // ????
}
```
Answer: You can't instantiate an object of an ABC because it has an incomplete implementation.  What if the user tried to call s.area()? There's no method implementation to call!

So why define abstract methods at all? Why not just provide dummy implementations in the base class?

```cpp
class Shape { // C++ class
public:
  // area is an "abstract" method
  virtual double area() const { return 0; }   // dummy version returns 0
  virtual void rotate(double angle) { }        // dummy version does nothing
  virtual string myName() 
    { return "abstract shape"; }
};
```

Answer: An abstract method forces the programmer to redefine the method in a derived class, ensuring they don't accidentally use a dummy base implementation in a derived class!

OK what about abstract classes and types? Just as with concrete classes, when you define a new abstract class, it automatically creates a new type of the same name! More specifically, it defines a _reference type_, which can ONLY be used to define pointers, references, and object references... not concrete objects.

#### When should you use abstract methods/classes
{: .no_toc }
Use abstract methods/classes under the following circumstances.
- Prefer an abstract method when there's no valid default implementation for the method in your base class.
- Prefer an abstract class over an interface when all subclasses share a common implementation (one or more concrete methods or fields), and you want the derived classes to inherit this implementation

In contrast, you would inherit/implement an interface when you have a "can-do" relationship, e.g., a Car can do Driving, so a car might implement a Drivable interface.

### Inheritance and Typing

When we use inheritance to define classes, we automatically create new supertypes and subtypes! The base class (e.g., Person) defines a supertype. The derived class (e.g., Student) defines a subtype. And given the rules of typing, we can then use a subtype object anywhere a supertype is expected.

As we learned earlier, any time you define a class or an interface, it implicitly defines a new type.

The type associated with a concrete class is specifically called a value type.  Value types can be used to define references, object references, pointers, and **instantiate objects**.

The type associated with an interface OR abstract class is called a reference type. Reference types can ONLY be used to define references, object references, and pointers.

Ok, so how does typing work with inheritance? Consider these two classes:

```cpp
class Person {
public:
  virtual void talk()
    { cout << "I'm a person"; }
  virtual void listen()
    { cout << "What'd you say again?"; }
};

class Student: public Person {
public:
 virtual void talk() 
   { cout << "I hate finals."; }
 virtual void study() 
   { cout << "Learn, learn, learn"; }
}; 
```

Well... the Person base class definition implicitly defines a new type called Person. And the Student subclass definition implicitly defines a new type called Student. And since Student is a subclass of Person the Student type is a subtype of the Person type!

Ok, and how does typing work with interface inheritance? Consider this example:

```java
// Java interface inheritance
public interface Washable {
  public void wash();
  public void dry();
}

public class Car implements Washable {
  Car(String model) { ... }
  void accelerate(double acc) { ... }
  void brake(double decel) { ... }

  public void wash() {
   System.out.println("Soap & bucket");
  }
  public void dry() {
    System.out.println("Dry with rags");
  }
}
```
Well... the Washable interface definition implicitly defines a new type called Washable. And the Car class definition implicitly defines a new type called Car. And since Car is an implementer of Washable the Car type is a subtype of the Washable type!

#### Inheritance and Typing Challenges
{: .no_toc }
Question: When would a derived class not be a subtype of its base class?
Answer: When we privately inherit from a base class, the derived class is NOT a subtype of the superclass's type, nor does it share the interface of the base class!  

Question: When could a class have a supertype, but not a superclass?
Answer: A class could have a supertype of an interface type, without having a superclass, since an interface is not a class (at least in some languages). 

### Subtype Polymorphism

Subtype Polymorphism is the ability to substitute an object of a subtype (e.g., Circle) anywhere a supertype (e.g., Shape) is expected. More technically, S.P. is where code designed to operate on an object of type T operates on an object that is a subtype of T.

```cpp
class Shape { ... };
class Circle: public Shape { ... };

void display(Shape& s) {    // display accepts a Shape (supertype) object
  ...
}

int main() {
  Circle circ(0,0,5);
  display(circ);            // but we're passing in an object that's asubtype of Shape
}
```

As we learned in datapalooza, typing rules state that if a function can operate on a supertype, it can also operate on a subtype! The same holds true for supertypes and subtypes associated with inherited classes! And for classes that implement interfaces!

This also works because the  class associated with the subtype supports the same public interface as the class associated with the supertype:

```cpp
class Washable { // Interface
  virtual void wash() = 0;
  virtual void dry() = 0;
};
class Car: public Washable { ... }

void washAThing(Washable& w) {  // this function  accepts a supertype
  w.wash();
}

void cleaningCrew() {
  Car Subaru;
  washAThing(subaru);  // we're passing a subtype object to the function above
}
```
So we've seen that we can substitute a subtype object anywhere a supertype is expected. This works because the subclass inherits/implements the same interface as its superclass. But that's not enough! We also expect the methods of the subclass to have the same semantics as those in superclass.

Consider these base and derived classes:

```cpp
class Shape {
public:
  virtual double getRotationAngle() const {
      // returns angle in radians
  } 
  ...
};

class Square: public Shape {
public:
  virtual double getRotationAngle() const {
    // returns angle in degrees
  }
  ...
};
```

Notice that our derived class returns the result in degrees, but the base class returns the result in radians?  This would change the semantics of the interface defined by the base class, and is a bad idea.  Imagine if you passed a Square object to a function that operated on Shapes?  Any time it operated on a Square and called getRotationAngle() it would get a result in degrees, when it expected results in radians. This would cause tons of bugs. To ensure this doesn't happen, a derived class should always implement the same semantics as the base class. If a subclass adheres to the interface and semantics of the base class, we can truly substitute any subtype object for a supertype, and our code will still work. This is called the Liskov Substitution Principle, named after computer scientist Barbara Liskov.

#### Subtype Polymorphism in Dynamically-typed Languages?
{: .no_toc }
So far, all of our examples have been with statically-typed languages! 

Question: Can we have subtype polymorphism in dynamically-typed languages like Python? Consider this example:

```python
class Car:
  def speedUp(self, accel):
    ...
  def slowDown(self, decel):
    ...
  def steer(self, angle):
    ...

class HybridCar(Car):
  def speedUp(self, accel):
    ...
  def slowDown(self, decel):
    ...

def drive_car(c):
  c.speedUp(5)

def main():
  prius = HybridCar()
  drive_car(prius)  # is this using subtype polymorphism?

```
Answer:  Nope. Subtype polymorphism occurs when we process a subtype object as if it were an object of the supertype. But in dynamically-typed languages, variables have NO type at all!  For instance, the parameter c in the drive_car(c) class above has no type. So how can we be treating the prius (a subtype) as if it's a supertype (a Car)?  We can't, and we're not.  So we're NOT using subtype polymorphism! This code will work, however (using duck typing)! It's just not subtype polymorphism!

#### Classify That Language: Subtype Polymorphism
{: .no_toc }
Consider the following code. In this language, a "trait" is the name for an interface. Does this program use subtype polymorphism? Why or why not?

```scala
trait Shape {
  def getArea(): Double
  def getPerim(): Double  
}

class Square(side: Double) extends Shape {
  override def getArea(): Double = side * side
  override def getPerim(): Double = 4 * side
}
class Circle(r: Double) extends Shape {
  override def getArea(): Double = Math.PI * r * r
  override def getPerim(): Double = 2 * Math.PI * r
}

def printStats[T <: Shape](s: T): Unit = { 
  println("The area is: " + s.getArea()) 
  println("The perimeter is: " + s.getPerim()) 
}

val square = Square(10.0)
val circle = Circle(1.0)
printStats(square)
printStats(circle)
```
Answer: This program does use subtype polymorphism. We pass a subtype object like square/circle, and treat it like a supertype, using type T, which is designated to be some subtype of a Shape.

### Dynamic Dispatch

Imagine we have a variable var that refers to some object:

```cpp
void foo(Shape& var) { // c++
  cout << var.area();
}
```
or:

```python
def foo(var):
  print(var.area()) 
```

With Dynamic Dispatch, when you call a method on var, e.g.: var.area(), the actual method that gets called is determined at run-time based on the target object that var refers to. The determination is made in one of two ways:
- Based on the target object's class, or
- By seeing if the target object has a matching method (regardless of the object's type or class), and calling it if one was found - in languages without classes and just objects

So dynamic dispatch is the process of figuring out which method to call when a call is made to a virtual method at runtime. Do we call Circle's area() method, Square's area() method, or what?

#### Dynamic Dispatch in Statically-typed Languages
{: .no_toc }
In statically typed languages, the language examines the class of an object at the time of a method call and uses this to "dynamically dispatch" to the class's proper method. Typically this is implemented by adding a hidden pointer into each object which points at a vtable for the class. A vtable is an array of function pointers which points to the proper method implementation to use for the current class.  Every class has its own vtable. Only virtual methods are included in the vtable.  So a Circle object would have a pointer to a vtable pointing to Circle's area() and perimter() methods. And a Square object would have a pointer to a vtable pointing to Square's area() and perimeter() methods.  When the call happens, e.g. var.area(), the vtable associated with the object is consulted and the pointer to the area() function is used to call the proper function.

Dynamic dispatch is NOT used to call regular instance or class methods. Instead a standard function call is used in these cases. For regular instance functions (that are non-virtual) or class/static functions, a function call to them is called "static dispatch," since the proper function to call can be determined at compile time (statically), not at runtime (dynamically).

#### Dynamic Dispatch in Dynamically-typed Languages
{: .no_toc }
In dynamically typed languages, things get more interesting! While a class may define a default set of methods for its objects the programmer can add/remove methods at runtime to classes or sometimes even individual objects! So when a method is called, the language can't necessarily rely upon an object's class or type to figure out what method to use! Here's an example from javascript:

```javascript
// Javascript dynamic dispatch example
function makeThemTalk(p) {
  console.log(p.name + " says: " + p.talk());
}

var person = {
  name: "Alan Kay",
  talk: function() { return "Java is meh"; },
  change: function()
    { this["talk"] = function() { return "I <3 Smalltalk"; } }

};

var bird = {
  name: "bluebird",
  talk: function() { return "Chirp chirp"; }
};

makeThemTalk(person)
person.change()
makeThemTalk(person)
makeThemTalk(bird)
```

So how might a language that allows you to customize methods for individual objects determine which method to call? Answer: The language stores a unique vtable in every object!

#### Dynamic Dispatch Challenge Questions
{: .no_toc }
Question: What is the difference between subtype polymorphism and dynamic dispatch?

Answer: Subtype polymorphism is a statically-typed language concept that allows you to substitute/use a subtype object (e.g., HighPitchedGeek) anywhere code expects a supertype object (e.g., a Geek).  Since the subtype class shares the same public interfaces as the supertype, the compiler knows that the types are compatible and substitution is allowed. 

In contrast, dynamic dispatch is all about determining where to find the right method to call at run-time for virtual/overridable methods. Dynamic dispatch occurs whether or not we have subtype polymorphism, as we've seen in dynamically-typed languages like Ruby and Python.

Question: Is dynamic dispatch slower, faster, or the same speed as static dispatch? Why?

Dynamic dispatch is slower than static dispatch, since we need to look up function pointers in the vtable at runtime before performing the call.  Static dispatch calls are hard-wired at compile time into the generated machine code, and thus much faster (a direct machine language call).

#### Classify That Language: Dynamic Dispatch
{: .no_toc }
Consider the following classes...

```cpp
class Geek {
public:
  void tickleMe() { 
    laugh(); 
  }
  virtual void laugh()
    { cout << “ha ha!”; }
};

class HighPitchGeek: public Geek {
public:
  virtual void laugh()
    { cout << “tee hee hee”; }
};

int main() {
  Geek *p = new HighPitchGeek;
  
  p->tickleMe(); // ?

  delete p;
} 
```

Question: Will dynamic dispatch be used in this program? Also, what will print, and why?
Answer: Yes, it's using dynamic dispatch. This will print "tee hee hee". Why? Because dynamic dispatch always selects the most derived-version of a function to call.  So even though the laugh() method is called by the tickleMe() method which is defined in Geek, the laugh() method has been redefined in HighPitchedGeek, and our object is of type HighPitchedGeek, and so this version is called..