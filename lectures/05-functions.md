---
title: "05. Function Palooza "
author: Siddarth Krishnamoorthy, Matt Wang, Einar Balan, Boyan Ding
layout: lecture
parent: Lecture Notes
---

{: .note}
This lecture note covers the Function Palooza deck.

## Table of Contents
{: .no_toc }

{:toc}
- dummy item

## Function Palooza
{: .no_toc }
In Function Palooza, we will be doing a deep dive into functions. 
We'll understand:
 - how functions pass arguments and receive parameters
 - how languages return values from functions
 - how languages communicate errors across functions
 - how functions can be passed as arguments, returned and stored in variables
 - how to design functions to operate on a variety of different types of inputs.

## Parameter Passing Semantics

Parameter Passing Semantics is the term we use to describe the underlying mechanisms that languages use to pass arguments (e.g., `x+y`) to functions (e.g., `f(x+y)`).

First a review of some parameter passing terminology via example:
```cpp
double net_worth(double assets, double debt) { // formal parameters
  return assets – debt;
}

int main() {
 cout << "Your net worth is: " <<
   net_worth(10000, 3500); // actual parameters aka args
}

```

Formal parameters refer to the variables that are bound to values while actual arguments/args are the values that we pass in!

The four most common parameter passing semantics are:

* Pass by value: The formal parameter gets a copy of the argument's value/object
* Pass by reference: The formal parameter is an alias for the argument's value/object
* Pass by object reference: The formal parameter is a pointer to the argument's value/object
* Pass by name: The parameter points to an expression graph that represents the argument

As it turns out, these are closely related to binding semantics we learned!

We'll also discuss two more strategies which are a bit more esoteric:
* Pass by value-result: A copy of the argument gets passed in and changes are copied back to the argument on the way out
* Pass by macro expansion: Every use of the parameter in the function is replaced with the exact text passed in as the argument

### Pass by Value

Approach: Each argument is first evaluated to get a value, and a _copy_ of the value is then passed to the function for local use.

Take a look at the following C++ program, the parameter `n` in `stinkify` is passed by value:

```cpp
void stinkify(string n) {
  string t = n + " stinks!";
  n = t;
} 

int main() {
  string s = "Devan";

  stinkify(s);
  cout << s; // Prints "Devan"
}
```

When `stinkify(s)` is executed, the `n` in `stinkify`'s activation record is initially copied from `s` from `main`. When `n` is modified at the end of the function, it only affects the local copy without affecting the original argument `s` in main.


### Pass by Reference

Approach: _Secretly pass the address_ of each argument to the function. In the called function, all reads/writes to the parameter are directed to the data at the original address.

Let's looked at a slightly altered version of above program, where the parameter `n` is passed by reference instead (notice the "`&`" before `n`):

```cpp
void stinkify(string &n) {
  string t = n + " stinks!";
  n = t;
} 

int main() {
  string s = "Devan";

  stinkify(s);
  cout << s; // Prints "Devan stinks!"
}
```

This time when `stinkify(s)` is executed, the `n` in `stinkify`'s activation record points to the `s` in `main`'s activation record. This enables the formal parameter to act as an alias for the original value/object. Thus, each read/write of the formal parameter is directed to the original variable's storage. So the modification of `n` changes the `s` in `main`.

One byproduct that can be caused by pass by reference is **aliasing**. It occurs when two parameters unknowingly refer to the same value/object and unintentionally modify it. It can cause subtle and difficult bugs.

```cpp
void filter(set<int> &in,
            set<int> &out) {
  out.clear();
  for (auto x: in)
    if (is_prime(x)) out.insert(x);
}

int main() {
  set<int> a;
  // ... fill up a with #s
  filter(a, a);
}
```

The function `filter` is supposed copy all prime numbers from `in` into `out`. Although it seems to do the job perfectly, problem will occur when `in` and `out` refer to the same variable, as demonstrated with the call `filter(a, a)` above.

In the example, when `in` and `out` refer to `a`, `out.clear()` will clear the input `a` before it gets processed. This causes the wrong result to be generated.

### Pass by Object Reference

Approach: All values/objects are passed by (copy of the) pointer to the called function. The called function can use the pointer to read/mutate the pointed-to argument.

```python
class Nerd:
  def __init__(self, name, iq):
    self.name = name
    self.iq = iq

  def study(self):
    self.iq = self.iq + 50

# ... 

def be_a_nerd(n):
  n.study();

def main():
  a = Nerd("Alwyn", 150)
  be_a_nerd(a);
  print(a)
```

In the python code, the call `be_a_nerd(a)` copies the object reference `a` into the formal parameter `n` of function `be_a_nerd`. So within `be_a_nerd`, the local variable n points to our original object, which can be mutated through the object reference.

But there is one thing to remember: when we pass by object reference, we can't use assignment to change the value of the original value/object. Let's see the following example:

```python
def stinkify(n):
  t = n + " stinks!";
  n = t

def main():
  s = "Devan";

  stinkify(s);
  print(s); # Prints "Devan"
```

We might expect the assignment `n = t` to change the original variable `s` in the calling function. But in fact, the assignment only changes the _local_ object reference `n` to point to the storage of `t` in Heap Memory. It has no imact on the original object reference `s` or the object `s` points to.

The takeaway is that assignments of object references never change the passed-in value/object. They just change where the local object reference points to. It contrasts with passing by reference, where assignment can change the original value.

So, in a language that uses Object References like Java or Python, your function can either return a new object with relevant changes (e.g., `x = stinkify(x)`), or use mutator functions.

### Pass by Name/Need

Approach: Each parameter is bound to a pointer that points to an expression graph (a "thunk") which can be used to compute the passed-in argument's value.

Here, a thunk is typically implemented as a lambda function, which can be called to evaluate the full expression graph and produce a concrete result when it is needed.

Haskell is a notable example of pass by need.

```hs
func2 y =
  y^2+7

func1 x =
  func2 (3 + x)

main = do
  let z = func1 5
  print z
```

In pass-by-need, once an expression graph is evaluated, the computed result is _memoized_ (cached) to prevent repeat computation. For example, in the code above, if we do another `print z` after the existing print, the value of `z` will be cached without the need for another evaluation.

### Pass by Value-Result

Pass by Value Result is a less common approach to passing, but we'll still take a look at it here. In Pass by Value Result, we pass a copy of the argument in and  once we return from our function we copy over all changes made to that parameter to the original argument. Here's an example in Ada:

```ada
-- Ada pass by value result example    
procedure Negate( val: IN OUT integer ) 
is
begin
 val := val * (-1); // this updates the local parameter variable
end Negate; // when we return, we copy the value of val back into the original arguments storage
   
-- main body
v : integer;
begin
  v := 42;
  Negate(v);
  Put_Line(natural'image(v)); // upon return -42 is copied in!
end
```

The important thing to realize is that, changes are only made to the original argument after the function returns. The parameter variable and argument are still distinct!

### Pass by Macro Expansion

Here's another less common approach, mainly used in C and C++. You may have heard of macros: essentially these are directives to the compiler to "replace" a snippet of code with anothe snippet. Here's an example:

```cpp
// Define a C++ macro that computes the
// square of a number
#define square(a) a * a

int main() {
  int x = 10;
  int result;

  result = square(++x);
  cout << result;
}
```
In this example, we're indicating to the compiler that we want to replace every instance of the macro function `square` with `a*a`. The cool thing about this is we can also pass in arguments and it will replace those too! After all macro expansion is performed, we get the following program:

```cpp
int main() {
  int x = 10;
  int result;

  result = ++x * ++x;
  cout << result;
}
```

Clearly this code behaves differently than if we had used a traditional function! Here we increment the value of x twice, rather than just once. Pretty cool!

### Parameter Passing by Language
{: .no_toc }
Let's look at some common language and the parameter passing scheme they use:

* C++: Pass by value/reference/object reference (pointer)/macro expansion
* Go: Pass by value (primitives)/object reference
* Haskell: Pass by need
* Java: Pass by value (primitives)/object reference
* JavaScript: Pass by value (primitives)/object reference
* Python: Pass by object reference

### Practice: Classify That Language: Parameter Passing
{: .no_toc }
Consider the following program, which prints:

> q is 110
> x is 20, y is 60

What parameter passing strategy is this language using?

```rust
struct Record {
 x:i32,
 y:i32
}

fn change_value(v: &mut i32) {
  *v += 100;
}

fn change_struct(r: &mut Record) {
  r.x *= 2;
  r.y *= 3;
}

fn main() {
  let mut q:i32 = 10;
  change_value(&mut q);
  println!("q is {}", q);

  let mut rec = Record{x:10,y:20};
  change_struct(&mut rec);
  println!("x is {}, y is {}", rec.x, rec.y); 
}
```

Answer: This language is using a hybrid of pass-by-reference and pass-by-pointer. (This is rust)

```scala
def addIfFirstEven(a: => Int, b: => Int): Int = 
  var sum = a 
  if (a % 2 == 0)
    sum += b
  return sum

def triple(x: Int): Int =
   var trip: Int = x*3
   println("3*"+x+" is: "+trip)
   return trip

object Main {
 def main(args: Array[String]): Unit =
   var v1 = 1
   var v2 = 2
   var result = addIfFirstEven(v1,triple(v2))
   println("The result is: "+result) //prints "The result is: 1"
}
```
In this case, this is doing pass-by-name since the second argument of addIfFirstEven (`triple(v2)`) is only evaluated if the first argument (`v1`) is even. Since it's odd in this instance, the `triple` function never runs!

### Positional and Named Parameters

In languages that support **positional** parameters, the order of the arguments must match the order of the formal parameters. For example,
```cpp
bool sum(float arr[], int n) {
  float sum = 0;
  for (int i=0; i < n; ++i) {
    sum += arr[i];
  }
  return sum;
}

int main() {
 float arr[3] = {78,99,65};

 cout << "The sum is: " << sum(arr, 3);
}
```

In languages that support **named** parameters, the call can explicitly specify the name of each formal parameter (called an *argument label*) for each argument. For example,

```cpp
def net_worth(assets,debt):
    return assets - debt

print("Your net worth is: ", net_worth(assets=10000, debt=3500))

print("Their net worth is: ", net_worth(debt=45000, assets=19000))
```
Many languages (like C++) support a combination of positional and named parameters.
```cpp
print("Your net worth is: ", net_worth(10000, debt=3500))
```

Positional parameters allow for a less wordy syntax since we don't need to specify an argument label for each argument. The disadvantage is that we *have* to pass the arguments in the same order as the formal parameters, and this can lead to bugs when we pass arguments in an incorrect order. With named parameters, you can add parameters and shift their order around more easily since each parameter is named. A change in order won't cause a bug. It also makes code more readable, if a bit more verbose, since you know what each argument is.

### Default Parameters

Most languages let you specify default values for specified formal parameters, making these parameters optional in the function call. For example,

```cpp
double net_worth(double assets, double debt = 0) {
  return assets - debt;
}

int main() {
 cout << "Your net worth is: " << net_worth(10000, 3500);
}
```

One or more formal parameters may have a default value. This makes passing the argument optional. If you decide to omit the associated argument to a formal parameter, the provided default value will be used. In languages (like C++ or Python) which do not have mandatory argument labels, default parameters must all be place at the *end* of the parameter list. This means that a definition like the following would be illegal in a language like Python.

```cpp
double net_worth(double assets = 0, double debt, double inheritance = 0) {
  return assets - debt + inheritance;
}
```

However, in languages with mandatory argument labels (like Swift), default values can be used for any parameter.

```swift
// Swift optional parameters
func net_worth(assets: Double, debt: Double=0,
               inheritance: Double) -> Double
  { return assets-debt+inheritance }

func main() {
 print(net_worth(assets: 10000, inheritance: 500))
}
```

Some languages like Python or FORTRAN allow to have optional parameters without default values! A function can check if a given argument was present when the function was called and act accordingly. Here is an example in Python.

```python
# Python optional parameters
def net_worth(assets, debt,**my_optionals):
    total_worth = assets - debt
    if "inheritance" in my_optionals: // checking if argument exists
        total_worth = total_worth +
                      my_optionals["inheritance"]
    return total_worth

print("Net-worth: ", net_worth(10000, 2000))
print("Net-worth: ",
      net_worth(10000, 2000, inheritance=50000))
```

These types of optionals can make code more terse, but harder to understand -- looking at the function prototype gives you incomplete information on what the parameters mean!

Here is another example, now in FORTRAN.

```fortran
! Fortran function with an optional parameter
real function net_worth(assets,debt,inheritance)
    real :: assets
    real :: debt
    real, optional :: inheritance

    real :: total_worth

    total_worth = assets - debt
    if (present(inheritance)) THEN
        total_worth = total_worth + inheritance
    end if
    net_worth = total_worth  ! specify return value
end function net_worth
```

### Variadic functions

A variadic function can receive an arbitrary number of arguments. The most common example is a language's `print` function:

```python
# Python's print function is variadic
# You can pass any # of arguments to it!
print(1)
print(1,"a")
print(1,3.14159,"c",4,"foobar")
```

To implement variadic functions, most languages gather variadic arguments and add them to a container (e.g., to an array, dictionary or tuple) and pass that container to the function for processing.

A notable exception is C++ which requires use of convoluted library functions to access variadic arguments!

Here are some examples of variadic functions in various languages.

#### Variadics in Java
{: .no_toc }
In Java, you may have zero or more fixed parameters. For the variadic parameters, Java creates an array containing every variadic argument and passes it to the variadic parameter.

```java
// Java variadics use an array-like approach
public class VariadicExample {
 private void f(String regular, int... vparams) {
   System.out.println(regular);
   for (int i = 0; i < vparams.length; i++)
     System.out.println(vparams[i]);
 }

 public void someFunc() {
   f("Four #s",2,4,6,8);
 }
}
```

{: .note }
In Java, all variadic parameters must have the same type. To deal with variadic parameters of differing types in Java, we can take advantage of the fact that all boxed primitives are derived from the `Object` class. So we could make a variadic of type `Object`, and then use type reflection (`instanceof`) to differentiate between types and specialise behaviour by argument.

#### Variadics in JavaScript
{: .no_toc }
In JavaScript we don't specify variadic parameters in the function declaration, you just specify fixed parameters. JavaScript provides a builtin `arguments` array which is populated with *all* arguments (fixed and variadic).

```js
// JavaScript variadics

function f(fixed1, fixed2) {
  console.log("Fixed: " + fixed1 + " " + fixed2)
  console.log("Varargs:")
  for (var i=2; i<arguments.length; i++)
    console.log(arguments[i])
}

f(1,"two","three",4.01);
  // Fixed: 1 two
  // Varargs: three 4.01
```

**Python**

Python created a tuple containing each variadic argument and then passes it to the function. You can then enumerate over the tuple as desired.
```python
# Python variadics
def f(fixed1, *args):
  print("First param: ", fixed1)
  print("Everything after first arg:")
  for arg in args:
    print(arg)

f(1,"two","three",4.0)
```

**C++**

Variadics in C++ are a little more tricky. Variadic arguments are identified by the formal parameter of `...`. C++ doesn't have any way of determining the number of variable arguments, so we have to somehow specify this ourselves (i.e. by passing a fixed parameter with the number of arguments).
```cpp
// C++ variadics are janky ☺
#include <stdarg.h>

void vprint(int count, ...) {
    va_list args;
    va_start(args, count);
    while (count--) {
      cout << va_arg(args, double) << " ";
    }
    va_end(args);
}

int main() {
  vprint(3, 3.14159, 2.718, 32.0);
}

```
To start processing variadic arguments, you have to create a `va_list` argument and then you have to call the `va_start` function with the name of the last *fixed* parameter. You can then call the `va_arg` function to get to each argument, and also advance to the next one. You finally call `va_end` to finish processing the variable arguments. You also have to specify the type of each value in C++, it won't know it otherwise (this is because C++ doesn't have type reflection).

## Return Values and Error Handling

In the yonder years (when Carey was still young), error handling was the wild west - there was no standardized way of catching errors! For instance, functions used enums (success, failure) or sentinel values (`-1`, `null`, or `false`) to report results/errors.

This created a hodge-podge of different error-handling approaches, and introduced its own set of bugs and issues. So as languages have evolved, they've started providing explicit mechanisms for dealing with results and errors.

Since then, languages have innovated and improved to make error handling more consistent and effective. Throughout this section, we'll learn about each of these innovations!

### Definitions: Bugs, Errors, Results
Before we discuss how languages handle bugs, errors, and results -- let's define these terms.

A **bug** is a flaw in a program's logic - the only solution is stopping execution and fixing the bug. The examples of a bug can include:

- out of bounds array access
- derefrencing a `nullptr`
- dividng by zero
- illegal casts
- unmet pre/post-conditions (ex: a factorial function returning a negative number!)

Other than bugs, there also exist **unrecoverable errors**. These are non-bug errors where recovery is impossible, and the program must shut down. Examples include:

- out of memory error
- network host not found
- full disk

Other less severe errors are **recoverable errors**. Under this kind of error, the program may continue the execution. Some examples of this type include:

- file not found
- network service (temporarily) overloaded
- malformed email

Finally, when there is no bug or error, the program will produce a **result**, which is an indication of the outcome/status of an operation.

### Handling Techniques: Overview

Here are the major "handling" paradigms provided by various languages:

- **roll your own**: The programmer must "roll their own" handling, like defining enumerated types (success,error) to communicate results.
- **error objects**: Error objects are used to return an explicit error result from a function to its caller, independent of any valid return value.
- **optional objects**: An "Optional" object can be used by a function to return a single result that can represent either a valid value or a generic failure condition.
- **result objects**: A "Result" object can be used by a function to return a single result that can represent either a valid value or a specific Error Object.
- **assertions/conditions**: An assertion clause checks whether a required condition is true, and immediately exits the program if it is not.
- **exceptions and panics**: f() may "throw an exception" which exits f() and all calling functions until the exception is explicitly "caught" and handled by a calling function or the program terminates.

Many languages blend multiple of these approaches, though some are more opinionated than others!

### Error Objects

Error objects are language-specific objects used to return an explicit error result from a function to its caller, independent of any valid return value.

Languages with error objects provide a _built-in error class_ to handle common errors; error objects are returned along with a function's result as a separate return value.

Let's look at a real example written in Go:

```go
func circArea(rad float32) (float32, error) {
  if rad >= 0 {
    return math.Pi*rad*rad, nil
  } else {
    return 0, errors.New("Negative radius")
  }
}

func cost(rad float32, cost_per_sqft float32)
         (float32, error) {
  area, err := circArea(rad)
  if err != nil { return 0, err }
  return cost_per_sqft * area, nil
}
```

The function `circArea` returns both a `double` and an `error` result, `error` is a built-in type in the second place of result tuple.

- if the radius is valid, then we return the circle's area and `nil` for the error result, meaning no error occurs.
- otherwise, we return 0 for th area and an _error object_. We specify what went wrong with the message passed into the constructor `error.New`

The function `cost` gets bot the area and error result from `circArea`. When an error occurs, it propages the error up, and does normal computation otherwise.

Besides using the built-in error type, you can define custom error classes with fields that are specific to your error condition, and even wrap (nested) errors to provide more context.

### Optionals

An "Optional" object can be used by a function to return a single result that can represent either a valid value or a generic failure.

You can think of an Optional as a struct that holds two items: a value and a Boolean indicating whether the value is valid.

{: .note }
Alternatively, you can think of an optional as an ADT, which contains either "nothing", or "someting" (with a value). This is closer to how they are thought of in programming language theory.

Since you only have a simple Boolean to indicate success or failure (not a detailed error description), you only want to use Optional if there's an obvious, single failure mode.

Let's first see an example in C++ (C++17)

```cpp
std::optional<float> divide(float a, float b) {
  if (b == 0)
    return std::nullopt;  // error result!
  else
    return std::optional(a/b);
}

int main() {
 auto result = divide(10.0,0);
 if (result)
    cout << "a/b is equal to " << *result;
 else
    cout << "Error during division!";
}
```

The return type `std::optional<float>` of function `divide` indicates that this function returns a `float` value if it's successful.

- if the function can't compute a valid result, it can return nullopt which explicitly indicates an error.
- otherwise, we construct and return an optional object containing our valid floating-point value.

C++ allows us to treat the optional object like a Boolean when checking it. If it contains a valid result, it'll evaluate to true. Then we can then use the overloaded `*` operator to get the actual float value embedded in the optional object.

{: .warning }
You can only use the `*` operator on C++ optional object when the result is valid. Otherwise, it will result in unspecified behavior (because there is no result when error occurs).

In some languages, optionals are a built-in part of the language with dedicated syntax! Let's look at an equivalent example in Swift.

```swift
func divide(a: Float, b: Float) -> Float? {
    if b == 0 {
      return nil;
    }
    return a/b;
}

var opt: Float?;
opt = divide(a:10, b:20);

if opt != nil {
  let a_div_b: Float = opt!;
  print("Result was: ", a_div_b);
} else {
  print ("Error result!")
}
```

As we can see, the `?` as in `Float?` is used for optional return value.

- when returning:
 - `nil` is used for error result
 - any other value directly creates the optional that represents a valid value.
- when using: `!` is used to extract the value from the optional if it is valid.

### Result Object

A "Result" object can be used by a function to return a single result that can represent either a **valid value** or a **distinct error**.

You can think of a Result as a struct that holds two items: a value and an Error object with details about the nature of the error.

{: .note }
Or, like an ADT with a value variant, and an error variant :)

```swift
enum ArithmeticError: Error {
  case divisionByZero
  // add other error types here as necessary
}

func my_div(x: Double, y: Double) ->
       Result<Double, ArithmeticError> {
  if y == 0  { return .failure(.divisionByZero) }
  else       { return .success(x / y) }
}

let result = my_div(x:10, y: 0)
switch result {
  case .success(let number):
    print("Successful division: ", number)
  case .failure(let error):
    dealWithTheError(error)
}
```

Different from optional objects that we discussed earlier, the error result of result type carries detailed information using error object. You can use it when there are multiple distinct failure modes that need to be distinguished and handled differently.

### Assertions

An assertion is a statement/clause inserted into a program that verifies assumptions about your program's state (its variables) that must be true for correct execution.

We typically use assertions to verify:

- preconditions: something that must be true at the start of a function for it to work correctly.
- postconditions: something that the function guarantees is true once it finishes.
- invariants: An invariant is a condition that is expected to be true across a function or class's execution.

Consider the states to verify in selection sort: `void selection_sort(int *arr, int n);`.

- preconditions: `arr` must not be `nullptr`, `n` must be `>= 0`
- postconditions: For all `i`, `1 <= i < n`, `arr[i] > arr[i-1]`
- invariants: At the end of iteration `j`, the first `j` items of `arr` are in ascending order

An assertion tests a particular condition and terminates the program if it's not met. An assertion states what you expect to be true. Your program aborts if it's not true. Here are a few examples:

```cpp
// C++ assertions
void selection_sort(int *arr, int n) {
  assert(arr != nullptr);
  assert(n >= 0);
  ...
}
```

Some languages enable you to provide a message to explain what went wrong.

```java
// Java assertions
public class SelectionSort {
  public void sort(int arr[]) {
    assert arr != null : "Invalid arr";
    ...
  }
}
```

A few languages (Eiffel, Ada) let you explicitly specify pre- and post-conditions for each function.

```eiffel
-- Eiffel pre and post conditions
selection_sort (arr: ARRAY [G]): ARRAY [G]
  require
    arr_not_void: arr /= Void
  local
    -- locals go here ...
  do
    -- sorting code here sorts the numbers
    -- and stores results in out_arr variable
  ensure
    result_is_set: out_array /= Void
    result_sorted: is_ascend(out_array) = True
  end
```

{: .note }
While Eiffel is not commonly used, this idea - encoding pre and post-conditions *within* the type system / compiler - is becoming really popular. Google "dependent typing"!

### Exception Handling

With other error handling approaches, error checking is woven directly into the code, making it harder to understand the core business logic.

With exception handling, we separate the handling of exceptional situations/unexpected errors from the core problem-solving logic of our program. Thus, errors are communicated and handled independently of the mainline logic. This allows us to create more readable code that focuses on the problem we're trying to solve and isn't littered with extraneous error checks that complicate the code.

#### Participants of Exception Handling
{: .no_toc }
There are two participants with exception handling: a catcher and a thrower.

A **catcher** has two parts:

1. A block of code that "tries" to complete one or more operations that might result in an unexpected error.
2. An "exception handler" that runs if (and only if) an error occurs during the tried operations, and deals with the error.

```cpp
void f() {
  try {
    g();
    h();   // Might have an unexpected error
    i();
  }
  catch (Exception &e) {
    deal_with_issue(e);   // Deals with the error if one occurs
  }
}
```

In the C++ code above. The block of code within `try` belongs to the first part of the catcher, which contains the call to function `h`, which might result in an error. On the other hand, the following `catch` block is the second part. It contains the error handler, and it will only be executed in case an error happens.

The **thrower** is a function that performs an operation that might result in an error. If an error occurs, the thrower creates an exception object containing details about the error, and "throws" it to the exception handler to deal with.

A C++ example might look like the following:

```cpp
void h() {
  // Next command might fail!
  if (some_operation() == failure)
    throw runtime_error("details..");

  op_succeeded_so_do_other_stuff();
  do_even_more_stuff();
  finish_with_more_stuff();
}
```

If any operation performed in the try block results in a thrown exception, the exception is immediately passed to the exception handler (in the catch block) for processing.

Let's say in the function `f`, an failure occurs inside function `h` as shown in the code, it throws an exception at the `throw` in the code above.

- when the exception is thrown within `h`, it is immediately exited, and the remaining statements are skipped. It acts as if the function `h` immediately returns.
- all remaining statements in the try block within `f` are also skipped.
- the execution flow goes into the exception handler within the `catch` block instead. The exception handler processes the exception and figures out how to proceed.
- finally, when the exception handler completes, execution continues normally.

#### Execution Flow
{: .no_toc }
Let's use another exemple to illustrate the execution flow with exception handling.

{: .note }
There's a neat graphic in the slides for this; I'd check it out.

```cpp
void f() {
  do_thing0();
  try {
    do_thing1();
    do_thing2();
    do_thing3();
    do_thing4();
    do_thing5();
  }
  catch (Exception &e) {
    deal_with_issue1(e);
    deal_with_issue2(e);
  }
  do_post_thing1();
  do_post_thing2();
}
```

If we assume there is an exception generated in `do_thing3()`, the execution flow of function `f` is:

* `do_thing0`
* `do_thing1`
* `do_thing2`
* `do_thing3`
* `deal_with_issue1`
* `deal_with_issue2`
* `do_post_thing1`
* `do_post_thing2`

On the other hand, when no exception is generated, the execution flow will be:

* `do_thing0`
* `do_thing1`
* `do_thing2`
* `do_thing3`
* `do_thing4`
* `do_thing5`
* `do_post_thing1`
* `do_post_thing2`

#### What is An Exception?
{: .no_toc }
An exception is an object with one or more fields which describe an exceptional situation.

At a minimum, every exception has a way to get a description of the problem that occurred, e.g., "division by zero."

But programmers can also use subclassing to create their own exception classes and encode other relevant info. The following C++ code demonstrates the creation of a custom exception.

```cpp
class WebsiteException:
           public std::exception {
public:
 WebsiteException(const string& website,
                  int http_error) {
    website_ = website;
    http_error_code_ = http_error;
 }

 string get_website_url() const
    { return website_; }
 string get_http_error_code () const
    { return http_error_code_; }
 const char* what() const noexcept
    { return "Unable to connect to URL\n";}
private:
  string website_;
  int http_error_code_; // eg, 404 access denied
};

// ...
HttpStatus s = connect(url);
if (s != OK)
  throw WebsiteException(url, s.getHTTPStatus());
```

#### The Exception Hierarchy
{: .no_toc }
In fact, an exception may be thrown an arbitrary number of levels down from the catcher.

```cpp
void h() {
  if (some_op() == failure)
    throw runtime_error("deets");

  do_other_stuff();
}

void g() {
  some_intermediate_code();
  h();
  some_more_code();
}

void f() {
  try {
    cout << "Before!\n";
    g();
    cout << "After!\n";
  }
  catch (Exception &e) {
    deal_with_issue(e);
  }
}
```

In the C++ example above, when function `h` throws an exception, it will automatically terminate every intervening function (`g` in the example) on its way back to the handler (`f`).

Additionally, an exception handler can specify exactly what type of exception(s) it handles. A thrown exception will be directed to the _closest_ handler (in the call stack) that _covers its exception type_.

```cpp
void h() {
  throw out_of_range("negative index");
}

void g() {
  try {
    h();
  }
  catch (invalid_argument &e) {
    deal_with_invalid_argument_err(e);
  }
}

void f() {
  try {
    g();
  }
  catch (overflow_error &e) {
    deal_with_arithmetic_overflow(e);
  }
  catch (out_of_range &e) {
    deal_with_out_of_range_error(e);
  }
}
```

In the code above, the `out_of_range` exception thrown in `h` is caught by the second `catch` block in `f`.

If an exception is thrown for which there is no compatible handler, the program will just _terminate_. This is the equivalent of a "panic" which basically terminates the program when an unhandle-able error occurs.

It turns out that different types of exceptions are derived from more basic types of exceptions. For example, C++ has an exception hierarchy, where `std::exception` is the base class of all exceptions.

So if you want to create a catch-all handler, you can have it specify a (more) basic exception class. That handler will deal with the base exception type and all of its subclassed exceptions.

#### Finally
{: .no_toc }
Some languages, like Java, introduced a third component to a catcher to make error handling simpler. This third component is called a `finally` block and it's guaranteed to run whether the try block succeeds or throws. This enables you to place your "cleanup" code in a single place, yet guarantee it runs in both error and success situations. Here is an example of some code that uses `finally` in Java.

```java
// Java "finally" block
public class FileSaver {
  public void saveDataToFile(String filename,
				String data) {
    FileWriter writer = null;
    try {
      // Next 2 lines might throw IOExceptions
      writer = new FileWriter(filename);
      writer.write(data);
    }
    catch (IOException e) {
      e.printStackTrace();  // debug info
    }
    finally {
      if (writer != null)
        writer.close();
    }
  }
}
```
In the above example, the code in the `finally` block always executes, regardless of whether or not the code in the `try` block throws an exception.

#### Exceptions and memory safety
{: .no_toc }
Exceptions can create all sorts of bugs if not used appropriately. To see how that can happen, consider the following example in C++.
```cpp
void h() {
  if (some_op() == failure)
    throw runtime_error("deets");

  do_other_stuff();
}
void g() {
  int* arr = new int[100];
  h();
  // code uses array here... 
  delete[] arr;
}
void f() {
  try {
    g();
  }
  catch (Exception &e) {
    deal_with_issue(e);
  }
}
```
What happens if `h` throws a `runtime_error`? Notice that in `g`, the memory allocated using `new` is never freed, thus causing a memory leak! So you should be wary of using exceptions in languages *without* automatic memory management (like C++).

{: .note }
In languages like C++, you can still use exceptions without worrying about memory leaks if you use [Smart pointers](https://en.cppreference.com/w/cpp/memory).

Some languages like Java force you to annotate functions which throw exceptions with a special token (in the case of Java, you must annotate functions that throw with the `throws` keyword). This way, you will know if you need to catch an exception when you call a function.
```java
// Java functions must explicitly 
// declare their exceptions!
public class MyClass {
  void h() throws IOException {
    if (some_op() == fail_to_open_file)
      throw new IOException("File mising!");
  }
    
  void g() throws IOException {
    h();
    // other code
  }
    
  public void f() {
    try {
      g();
    }
    catch(IOException e) {
      deal_with_issue(e);
    }
  }
}
```

{: .note }
Another big drawback of exceptions is that they are extremely slow when they are thrown!

Here are a few more examples of exceptions in other languages.
* **Swift**

```swift
// Swift exception handling example
enum VendErr: Error {   
    case invalidSelection(selection: String)
    case invalidBill(billValue: Int)
}

class VendingMachine { 
  var dollarsInserted: Int

  func insertMoney(dollars: Int) throws {
    if (dollars != 1 && dollars != 5 && dollars != 10) 
      { throw VendErr.invalidBill(billValue: dollars) }
    dollarsInserted += dollars 
  }

  func buyItem(itemCode: String) throws -> String {
     if itemCode == "A1" && dollarsInserted >= 2 {
        dollarsInserted -= 2 
        return "KitKat"
    } 
    // We got nuthin left!
    throw VendErr.invalidSelection(selection: itemCode)
  } 
}
...
var vm = VendingMachine()
do {
    try vm.insertMoney(dollars:10);
    let item = try vm.buyItem(itemCode: "b3");
    print("Ate \(item). Yum!")
} catch VendErr.invalidSelection {
    print("Invalid Selection.")
} catch VendErr.invalidBill(let billSize) {
    print("Counterfeit $\(billSize) bill!")
}
```

* **Python**

```python
# python exception handling example
def getNumbers(howMany): 
  if howMany < 0:   
    raise ValueError("howMany was < 0")
  return [str(i) for i in range(howMany)]

def saveNums(filename, howMany): 
  try:   
    f = open(filename, "w")   
    try:
       nums = getNumbers(howMany)
       f.write(' '.join(nums))
    except ValueError as ve: 
       print("Had a value error:",ve)
    except IOError:
      print("Badness saving to file")   
    finally:
      f.close()
  except OSError:   
     print("Badness opening file")
```

#### Exception handling guarantees
{: .no_toc }
As we've seen, exception handling can be tricky and difficult to understand. As such, engineers have produced some rules of thumb to ensure exceptions are used in a safe manner. When writing a function, always make sure you meet one of the following guarantees:
* No throw guarantee: A function guarantees it will not throw an exception. If an exception occurs in/below the function, it will handle it internally, and not throw. For example, this is required of destructors, functions that deallocate memory, swapping functions, etc.
* Strong exception guarantee: If a function throws an exception, it guarantees that the program's state will be "rolled-back" to the state just before the function call. For example, `vec.push_back()` ensures the vector doesn't change if the `push_back()` operation fails.
* Basic exception guarantee: If a function throws an exception, it leaves the program in a valid state (no resources are leaked, and all invariants are intact). Here, state may not be rolled back, but at least the program can continue running.

### Panics
Like an exception, a panic is used to abort execution due to an exceptional situation which cannot be recovered from (e.g., an unrecoverable error). Essentially, you can think of a panic as an exception which is never caught, and thus which causes the program to terminate. Panics contain both an error message and a stack trace to provide the programmer with context as to why the software failed.

Here is an example of a panic in C#.
```cs
// C# FailFast panic
class WorstCaseScenario
{
  public void someFunc()
  {
    if (somethingUnrecoverableHappens())
      Environment.FailFast("A catastrophic failure has occurred.");
  }
}
```

### Classify that language
{: .no_toc }
Consider the following program, which computes `a/b` given two values `a` and `b`. What result or error handling approach does this language use?
```rust
use std::io::{Error, ErrorKind};

fn my_div(a:f64, b:f64) -> Result<f64, Error> {
  if b != 0.0 { Ok(a/b) }
  else { Err(Error::new(ErrorKind::Other, 
             "You divided a by zero!"))  }
}

fn main() {
  match my_div(50.0,20.0) {
    Ok(number) => println!("Got {}",number),
    Err(e) => println!("Got an error {}",e),
  };
  match my_div(10.0,0.0) {
    Ok(number) => println!("Got {}",number),
    Err(e) => println!("Got an error {}",e),
  };
}
```
This is an example of a language using result variables for error handling. The language lets you specify either a value or an error type/message for a failure.

{: .note }
The language is actually Rust!

## First Class Functions
In languages with First Class Functions:
* Functions can be passed/returned to/from other functions
* Variables can be assigned to functions
* Functions can be stored in data structures
* Functions can be compared for equality
* Functions can be expressed as anonymous, literal values

Functions here are called "first-class citizens" because they are data objects and can be maniupulated like any other data in a program.

Here are some examples of first class functions in some languages:
* C++

In C++, first-class functions are implemented with function pointers.
```cpp
int square(int val) { return val * val; }
int fivex(int val) { return val * 5; }

using IntToIntFuncPtr = int (*)(int val);

void apply(IntToIntFuncPtr f, int val) {
  cout << "f(" << val << ") is " << f(val) << endl;
}

IntToIntFuncPtr pickAFunc(int r) {
  if (r == 0) return square;
         else return fivex;
}

int main() {
 IntToIntFuncPtr f = pickAFunc(rand() % 2);
  if (f == square) cout << "Picked square\n";
               else cout << "Picked fivex\n";
  apply(f, 10);
}
```

* Go

```golang
func square(val int) int { return val * val }
func fivex(val int) int  { return val * 5 }

type IntToIntFuncPtr func(int) int

func apply(f IntToIntFuncPtr, val int) {
  fmt.Println("f(", val, ") is ", f(val))
}

func pickAFunc(r int) IntToIntFuncPtr {
  if r == 0 {
    return square
  } else {
    return fivex
  }
}

func main() {
  var f = pickAFunc(rand.Intn(2))
  if f != nil {
    apply(f, 10)
  }
}
```

### Anonymous (Lambda) Functions
A lambda function is a function that does not have a function name – it's anonymous. Typically we pass a lambda function as an argument to another function or store it in a variable. Lambdas are used when a short, temporary function is needed just once in a program, and it doesn't make sense to define a general-purpose named function.

Every lambda has three different parts:
* Free Variables to Capture - What variables defined outside the lambda function should be available for use in the lambda function when it is later called.
* Parameters & Return Type - What parameters does the lambda function take and what type of data does it return.
* Function Body - The body of the lambda function that performs its operations.

### Capture by Value
As the name suggests, in capture by value, only the values of the free variables are captured by the lambda. This means that any changes made to the captured variable inside the lambda will not be reflected outside it. Here is an example of capture by value in C++.
```cpp
// C++ lambda function
auto create_lambda_func() {
  int m = 5;  
  int b = 3;
  return [m,b](int x) -> int { return m*x + b; };
}

int main() {
  auto slope_intercept = create_lambda_func();
  cout << "5*100 + 3 is: " << slope_intercept(100);
}
```

### Capture by Reference
C++ (and some other languages) can also capture values by reference. Here is an example.
```cpp
// C++ lambda function – capture by reference
auto fun_with_lambdas() {
  int q = 0;  
  auto changer = [&q](int x) { q = x; };
  changer(5);
  cout << "q is now: " << q; // Outputs "q is now: 5"
  return changer;
}

int main() {
  auto f = fun_with_lambdas();
}
```

### Capture by Environment
In capture by environment, an object reference to the *lexical environment* where the lambda was created is added to the closure. The *lexical environment* is a data structure that holds a mapping between every in-scope variable and its value. That includes all variables in the current activation record (locals, statics), and all global variables. Python uses capture by environment semantics. Here is an example.
```python
def foo():
  q = 5
  f = lambda x: print("q*x is: ", q*x)
  f(10)
```
When you define the lambda, it creates a closure containing:
* the lambda function itself
* an object reference to the current lexical environment

When running the lambda, it looks up each free variable in the lexical environments to obtain its value. Now consider this slightly modified code.
```python
def foo():
  q = 5
  f = lambda x: print("q*x is: ", q*x)
  f(10) # outputs "q*x is: 50"
  q = q + 1
  f(10) # outputs "q*x is: 60"
```
This outputs both `50` and `60`. This is because even though the lambda was defined when `q = 5`, when the lambda runs, it looks up the latest value of the variable in the lexical environment, and therefore outputs `60`.

### Classify that language
Consider the following program which prints out `7`. What capturing strategy does this language employ?
```js
function make_lambda() {
 let puppies = 4;

 // define lambda function with 1 param
 temp = function (kittens) { 
   s = "pup"
   s = s + "pies + kit"
   s = s + "tens" 

   // eval(s) interprets s as if it were a
   // regular statement in the program
   return eval(s)
  }
  return temp
}

f = make_lambda()
console.log(f(3))
```

Notice that our lambda function never explicitly references `puppies`! So the language can't be capturing by value or reference, so it must use capture by environment. The lambda then constructs the string `"puppies + kittens"`. Then the `eval` function takes it as a parameter and treats it as code, and runs it! So running `puppies+kittens` computes `4+3` and the code outputs `7`.

{: .note }
The language is actually JavaScript!

## Polymorphism

Polymorphism is the process of defining a function or class that can operate on many different types of values. Our goal is to express algorithms in their most abstract sense - making our code more interoperable and generalizable. While this might sound complicated, you've already used many polymorphism features in your CS journey - and in this class!

One common example is a vector. A `vector<int>`, `vector<str>`, and any other `vector` share the same operations - how can we reduce code duplication (making a `VectorInt`, `VectorStr`, ...) without taking a huge performance hit? By the end of this class, you'll be able to answer that quesiton!

We'll explore a few polymorphism approaches, focusing on ad-hoc and parametric polymorphism in statically-typed languages. We'll then talk about subtype polymorphism in OOP palooza!

### Ad-hoc Polymorphism

Ad-hoc polymorphism is when we (as the programmer) define specialized versions of a function for each type of object we wish it to support. The language decides which version of the function to call based on the types of the arguments.

You've likely seen this with operator overloading:

```cpp
bool greater(Dog a, Dog b) {
  return a.bark() > b.bark();
}
bool greater(Student a, Student b) {
  return a.gpa() > b.gpa();
}
int main() {
  Dog spot, penny;
  if (greater(spot, penny))
    cout << "Spot wins!\n";

  Student carey, david;
  if (greater(carey, david))
    cout << "Carey wins!\n";
}
```

We call it "ad-hoc" since there's no formal structure in the language to support it.

{: .note }
Ad-hoc polymorphism isn't possible in dynamically typed languages. This is because we usually don't specify types for formal parameters in dynamic languages, so there's no way to define multiple versions of a function with different parameter types. Instead, such languages can specialise behaviour using [type reflection](https://en.wikipedia.org/wiki/Reflective_programming).

### Parametric Polymorphism

With parametric polymorphism, we define a single, parameterized version of a class or function that can operate on many, potentially unrelated types. Parameteric polymorphism is implemented in two major ways, templates or generics. The syntax for the two may look similar, but they're implemented in entirely different ways - with big implications!

#### Templates

Templates do almost all of the work at compile-time.  In languages that use templates (like C++ or Rust), the compiler will see all the types that are used in each template. For each invocation, the compiler will do some type-checking. Then, it will generate a concrete version of the function/class by substituting in the type parameter. Finally, it'll continue compiling the code "as normal". After template substitution, the code is "normal" C++ - there's nothing special about it!

Here's one such example:

```cpp
template <typename T>
void takeANap(T &x) {
  x.sleep();
}
class Dog {
  void sleep() { /* ... */ }
};

class Person {
  void sleep() { /* ... */ }
};

int main() {
  Dog puppers;
  takeANap(puppers); // OK!

  Person carey;
  takeANap(carey); // OK!

  string val;
  takeANap(val); // error: no member named 'sleep' in 'string'
}
```

In the above example, since `takeANap` uses the `sleep()` method internally, only types that support `sleep()` can be used. If you try to use a type that doesn't support `sleep()`, you'll get a compile time error.

When you create a new template, e.g., vector<string>, does the compiler ensure the templated code is type-safe? If so, how?

<details markdown="0"><summary>Answer</summary>

Yes - since the compiler basically generates a concrete version of the function/class with the specified type, and then compiles it as it would any other class, type safety is guaranteed!
</details>

Is templated code more or less run-time efficient than an equivalent function that doesn't use templates, but otherwise has the same logic? Why?

<details markdown="0"><summary>Answer</summary>

Both implementations have the same runtime efficiency since a custom version of the function/class is generated for each distinct type, and it can be optimized just as if you wrote a dedicated function for that type.
</details>

{: .note }
C-style macros are sort of the OG templates. The pre-processor would basically do a textual search-and-replace of the arguments. But, the compiler doesn't generate a new function for each parameterized type.
```cpp
// C macros used to implement template-like functionality

#define swap(T,a,b) { T temp; temp = a; a = b; b = temp; }

int main() {
 int p = 5, q = 6;
 swap(int, p, q);

 std::string s = "foo", t = "bar";
 swap(std::string, s, t);
}
```

{: .note }
In Eggert's 131, we would have learned quite a bit more about macros (especially in the context of Scheme). If you're interested in macros, take a look at the concept of [Hygienic macros](https://en.wikipedia.org/wiki/Hygienic_macro), which demonstrate why C macros are *not* the exact same as templating.

#### Generics

Generics take a different approach. Instead of generating a new function for each parametrized type, we compile just one "version" of the generic function or class - independent of the types that actually use our generics.

Because of this, the generic can't make assumptions about what types might be using it! So, you can only do "generic operations" - hence the name.

Here's an example with C#; note how similar it looks to templates:

```cs
// C# generic container

class HoldItems<T> {
  public void add(T val)
    { arr_[size_++] = val; }
  public T get(int j)
    { return arr_[j]; }
 public void beADuck(int j)
    { arr_[j].quack(); } // ILLEGAL!!

  T[] arr_ = new T[10];
  int size_ = 0;
}

HoldItems<Duck> ducky =
          new HoldItems<Duck>();
ducky.add(new Duck("Daffy"));
Duck d = ducky.get(0);
```

Calling `quack()` on the generic type would be illegal, even though we only ever use `HoldItems` with `Duck` (which implicitly has a `.quack()` method).

Later, code that *uses the generic* is then checked to make sure that the generic's interfaces are used correctly.

Alone, this seems useless - there are very few things you can do on all types. That's why we can **bound types**: adding restrictions on what types are allowed.

Here is an example of bounding in C#:

```cs
interface DuckLike {
  public void quack();
  public void swim();
}

class HoldItems<T> where T: DuckLike {
  public void add(T val)
    { arr_[size_++] = val; }
  public T get(int j)
    { return arr_[j]; }
 public void beADuck(int j)
    { arr_[j].quack(); } // LEGAL!!

  T[] arr_ = new T[10];
  int size_ = 0;
}
```

Here's another example in Haskell!

```hs
qsort :: (Ord t) => [t] -> [t]
qsort [] = []
qsort (x:xs) =
  let leq = qsort [j | j <- xs, j <= x]
      geq = qsort [k | K <- xs, k > x]
  in leq ++ [x] ++ geq
```

Haskell type classes, such as `Ord` and `Eq`, place *bounds* on what types the `t` type variable can take on.


#### Why Parametric Polymorphism?
{: .no_toc }
This all seems unnecessarily complicated. Why do we need this? Couldn't we just use inheritance?

Let's take a look at an instance where *just* inheritance fails us. For context, languages like C# and Java have an `Object` class that superclasses *all other* classes. Let's assume we can do polymorphism just with subclassing:

```cs
class Duck : Object {
  public void quack() {
    Console.WriteLine("Quack");
  }
  public void swim() {
    Console.WriteLine("Paddle");
  }
}

class HoldItems {
  public void add(Object val)
    { arr_[size_++] = val; }
  public Object get(int j)
    { return arr_[j]; }

  Object[] arr_ = new Object[10];
  int size_ = 0;
}
```

*Any* object can be stored in `HoldItems`! So, we could add both a `string` and a `Duck` to a container, since both subclass `Object`. So, this code would be possible...

```cs
HoldItems items = new HoldItems();
items.add(new Duck("Daffy"));
items.add("more stuff");

string s = (string)items.get(0); // Should be illegal!
```

Which is really bad! And, there's no way for us (or C#, at compile-time) to detect a bug! So, we can't allow behaviour like this.

With generics, we tell the compiler which types our generics will use:

```cs
HoldItems<Dog> items = new HoldItems();
items.add(new Duck("Daffy")); // type error!!
```

#### Parametric Polymorphism in Dynamically Typed Languages
{: .no_toc }
We can't *really* have "type-parameterized" classes or functions, since variables don't specify types!

But, duck typing occupies the same feature space. Here is an example in Python:

```python
def qsort(lst):
  if len(lst) <= 1:     # base case
      return lst

  pivot = lst[0]
  rest  = lst[1:]
  lessEq =  [x for x in rest if x <= pivot]
  greater = [x for x in rest if x >  pivot]
  return qsort(lessEq) + [pivot] + qsort(greater)

strs = ["b","c","a"]
ints = [1,5,3,2,4]

qsort(strs)    # ["a","b","c"]
qsort(ints)    # [1,2,3,4,5]
```

But, note that this isn't as safe - we have no idea at compile-time if it'll work!

#### Classify That Langauge
{: .no_toc }
Consider the following parameterized class and some code that uses it below.
```typescript
class StudyList<T> {
    private values: T[] = [];

    public constructor (values: T[]) {
        this.values = values;
    }

    public add(value: T): void {
        this.values.push(value);
    }

    public get(index: number): T {
        return this.values[index];
    }

    public studySession(): void {
        for (var v in this.values)
            v.study();
    }
}
...
// The Nerd class has a study() method.
const nerds = new StudyList<Nerd>([]);

nerds.add(new Nerd("Carey"));
nerds.add(new Nerd("Paul"));
nerds.studySession();
```
The code results in a compiler error
```console
TypeError: v.study is not a function
```
Is the languages using templates or generics?

<details markdown="0"><summary>Answer</summary>

Generics! This is because even though the `Nerd` class has a `study()` method, the code does not compile. If this were a template, a version of `StudyList` would be generated with `T=Nerd` and this should work just fine.

This is TypeScript!
</details>