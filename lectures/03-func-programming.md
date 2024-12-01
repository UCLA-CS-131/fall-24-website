---
title: "03. Func. Programming "
author: Einar Balan, Matt Wang, Boyan Ding
layout: lecture
parent: Lecture Notes
---

{: .note }
This covers the Intro to Functional Programming deck.

## Table of Contents
{: .no_toc }

{:toc}
- dummy item

## What is Functional Programming?

Functional programming is defined by several key tenets:

- every function must **take an argument** (or more than one)
    - C-type `main()` functions are forbidden
- every function must return a value
    - C-type functions of type `void` are forbidden
- functions are **pure**: they should have no side effects
  - side effects include changing global variables, input/output, print statements, etc.
  - *the most important tenet!*
- calling a function `f(x)` with the same argument `x` should **always return the same output `y`** [^1]
- variables are **immutable**: after declaration, they cannot be modified
  - ex: `f(x){ x = x + 1; return x }` is not allowed!
- functions are **first-class** - just like other values, they can be stored as variables, passed as arguments, etc.

[^1]: In other words, functions are deterministic.

More rigorously:, a **pure function** has two core properties:

1. It doesn't depend on any data other than its inputs to compute a result
2. It doesn't modify any data beyond initializing local variables required to compute its output

Sometimes, this is called a **referentially transparent function**.

This function is pure: it only depends on `p`, and the only variable it initializes is never re-assigned (i.e., it's just to help us with naming).

```cpp
int f(int p) {
 int q = 5 * p * p;
 return q;
}
```

This function is **not pure**: not only does it depend on a global variable `z` (not an input to the function), but it also modifies it during the computation!

```cpp
int z;
int f(int p) {
  return p * z++;
}
```

There are a few **consequences** resulting from these basic tenets:
- no random numbers can be generated or used
    - since that would violate determinism
- no input is taken in a compiled Haskell programme
    - since that would violate determinism (input will determine result produced)
- no output is produced (aside from the final product) in a compiled Haskell programme
    - since that would not longer be pure (we are altering program state)

### Functional versus Imperative

Let's compare and contrast!

Imperative programs:

- are sequences of statements, loops, and function calls - they operate sequentially
- require changes in variable state
- as a result, multi-threading can be tricky, since there's shared mutable state and thus race conditions
- the order of execution matters!

Functional programs have different paradigms:

- at the core, functional programs are **only composition of functions** - no loops, statements, etc.
- no changes in variable state are allowed
- multi-threading is trivial: there is no shared mutable state (since mutability is not allowed)!
- and, interestingly, **the order of execution is not important**

What does that last point mean? Under the hood, it's all about side effects.

In an imperative language, calling functions in different orders can have side effects. Consider this example:

```cpp
int global = 0;

int func1(int arg) {
 return arg + global++;
}

int func2(int arg) {
  return arg * global;
}

int func(int arg) {
  int y = func1(arg);
  int z = func2(arg);

  if (z == 7)
    return 0;
  else return y;
}
```

In `func`, even though the declarations of `y` and `z` seem to not depend on each other - calling them in different orders matters!

- if we were to run `int y = func1(arg)` first, then `global` gets incremented to `1`, which *changes `z`'s behaviour*!
- if we were to run `int z = func(arg)` first, then `global` stays at `0`, and `z` returns `0`!

Functional programming disallows code like what we just saw; so, we really could evaluate `y` and `z` in any order!

## Intro to Haskell

Haskell is one of the first (and earliest) pure functional languages. This means that it is one of the best embodiments of functional programming design goals with minimal features from other paradigms. While it has many restrictions - like requiring generally pure functions - it really lets us learn functional programming!

### Our First Program

After [installing Haskell](https://www.haskell.org/ghc/download.html), we can run the interactive interpreter with `ghci`:

```sh
$ ghci
```

(you can exit this with `Ctrl+D`)

In a separate window, let's make a file - `hypot.hs` (note the file extension)

```hs
-- hypot.hs
hypot a b = sqrt((a^2) + (b^2))
```

{: .note }
`--` is how you declare a comment in Haskell.

We can now load this in `ghci`:

```console
$ ghci
GHCi, version 9.2.4: https://www.haskell.org/ghc/  :? for help
ghci> :load hypot
[1 of 1] Compiling Main             ( hypot.hs, interpreted )
Ok, one module loaded.
ghci> hypot 3 4
5.0
```

Let's say we update our file:

```hs
-- hypot.hs
square x = x * x
hypot a b = sqrt (square a + square b)
```

We can reload this file with `:r`:

```console
ghci> :r
[1 of 1] Compiling Main             ( hypot.hs, interpreted )
Ok, one module loaded.
ghci> hypot 7 8
10.63014581273465
```

We can also define functions in the interpreter itself:

```console
ghci> :{
ghci> squareArea :: Double -> Double
ghci> squareArea    side   =  side^2
ghci> :}
ghci> squareArea 4.0
16.0
```
Notice how we used `:{` and `:}` to define a multiline input. 

## Basic Haskell: Challenge by Challenge

Let's learn Haskell through a series of challenges. We'll show you some code and your job is to try figure either how it works or what its output is! By the end, you'll know a lot more about Haskell and you'll be ready to start writing your own basic programs. Let's get into it!

As we start, you should be aware that Haskell is very different from anything you've been exposed to in the past. Don't worry if it doesn't make sense at first! We (the entire teaching team) are here to support you in lecture, discussion, and office hours. That being said, if you're curious about how certain parts of the language work, the best way to learn is to simply try it in `ghci`. 

Okay! Now let's really get into it with our first challenge.

### Simple Functions 

Here's a Haskell function that computes 3x^2.

```haskell
-- polynomial.hs
poly :: Double -> Double
poly x = 3 * x^2
```

And here's the C++ equivalent:

```cpp
// polynomial.cpp
double poly(double x) {
  return 3 * x*x;
}
```

We can load `polynomial.hs` into `ghci` as follows:
```console
ghci> :load polynomial
ghci> poly 2
12
```

Some things to notice:
1. We specified the type of the function poly on the line before the actual function definition. In plain English, you can read it as a function that takes an argument of type Double and returns a Double. 
2. We don't include parenthesis around parameters like in other languages.
3. The body of the function is a simple expression following an equal sign!
4. There is no return statement.
5. To call a function we just specify its name and parameters.

In case you were curious, you can also define a main function in Haskell!

```haskell
-- polynomial.hs
poly :: Double -> Double
poly x = 3 * x^2

main :: IO()
main = do 
    let result = poly 5
    putStrLn ("Result: " ++ show result)
```

This is beyond the scope of the class, but it can be useful for things like writing test cases etc. An important note is that the `putStrLn` function is essentially a print function. You might be wondering: *how is that possible if side effects aren't allowed in Haskell?* Output is definitely a side effect!

In short, since we defined main to be a special type `IO()`, Haskell can work around the side effect constraint through some pretty cool and complex math.[^3]

[^3]: If you're curious about learning more, you should look into monads and the IO monad in particular. If you prefer written resources, read the Learn You a Haskell chapters on [IO](http://learnyouahaskell.com/input-and-output) and [Monads](http://learnyouahaskell.com/a-fistful-of-monads), or Galen Wong's wonderful [CS 231 notes](https://galenwong.github.io/blog/2021-08-29-cs231-notes/). [Here](https://www.youtube.com/watch?v=Q0aVbqim5pE) is a good video on monads in general and [here](https://www.youtube.com/watch?v=fCoQb-zqYDI) is a video explaining the IO monad in particular. As a warning, these topics are complicated so it may take a while for them to truly sink in, but it is super interesting! 

But most of the time in this class you'll just be writing functions and calling them from the interpreter directly.

### Parameters 

Consider this function:

```haskell
-- greater.hs
is_greater :: Int ->  Int  -> Bool
is_greater   first  second =  
    first > second
```

And the C++ equivalent:

```cpp
// greater.cpp
bool is_greater(int first, int second) {
  return first > second;
}
```

It determines if the first parameter is greater than the second. Here it is in action:

```console
ghci> :load greater
ghci> is_greater 2 5
False
ghci> is_greater 12 8
True
```

Some things to notice:
1. All but the last type in a function's type signature refer to the types of the parameters.
2. The last type is the return type.
3. We can use multiple lines for a function as long as we indent.
4. There are no parentheses or commas in a function call.

### Types 

Consider the following function:

```haskell
-- old.hs
is_old age has_aches =
    (age > (30::Int)) || (has_aches == True)
```
Notice that it has no explicit type definition. Let's mess around with it a bit in `ghci`.

```console
ghci> :load old
Ok, one module loaded.
ghci> is_old 20 True
True
ghci> :t is_old
is_old :: Int -> Bool -> Bool
ghci> is_old 53.5 False
<interactive>:3:8: error:
• No instance for (Fractional Int) arising from the literal ‘53.5’
```

Some things to notice:
1. We can use `:t` to get the type of a value in Haskell. 
2. In this case, Haskell was able to *infer* the types of all the parameters and the output of the function. Pretty cool! Haskell is able to deduce the types based on the operations used in the function. For example, since we compared `age` to an `Int`, `age` must also be an `Int`! The same logic can be used to determine the type of `has_aches` and the return type.

It's important to understand that just because variable types are not explicitly defined, does NOT mean the variables do not have types. In Haskell, all types are fixed and determined at compile time! Consider the error above. You can't pass a `Double` to a parameter of type `Int`!

So ... why would we even use type declarations? Unlike who we're insulting, it's **good hygiene** (and, best practice)!
- type declarations can provide better compiler warnings and errors
- type declarations make your code **easier to read**, arguably the most important trait in software![^10]
- and, it can help you think about how to write the function itself!

[^10]: The person reading it might be you six months later!

So, we recommend that you annotate your functions - or at least, the user-facing/top-level ones!

### Prefix Notation 

Consider this variation of `poly` from before.

```haskell
-- polynomial.hs
poly :: Double -> Double
poly x  =  (*) 3 x^2
```

It looks strange... what's going on here? In short, Haskell allows us to convert any operator to a prefix operator (i.e. a standard Haskell function) by adding parenthesis around it.[^4] 

Here are some more examples:

| Infix | Prefix |
|-------|--------|
| `6 - 5` | `(-) 6 5`|
| `5 + 6 / 2` | `(+) 5 ((/) 6 2)`|
| `a == b` | `(==) a b`|
| `x || y` | `(||) x y`|

[^4]: Interestingly, the reverse is also true! You can convert any function to an infix operator by adding surrounding backticks.

### Calling Functions

Let's talk a bit more about calling functions in Haskell. Consider the following functions:

```haskell
-- funcs.hs
f :: Int -> Int
f x = x^2

g :: Int -> Int
g x = 3*x
```

What do you think is the result of the following expressions and why?

```haskell
y = f g 2
z = f 5 * 10
```

<details>
<summary><b>Answer</b></summary>
<br><br>

Evaluating y results in a syntax error and z evaluates to 250! This might not be what you initially expected. You might think <code>f g 2</code> computes <code>f(g(2))</code>, but its important to understand that Haskell function calls are left associative. This means that <code>f g 2</code> should be interpreted as passing the function g to f as the first argument (e.g. (f g) 2), which clearly will lead to a type error! To fix this, we can add parenthesis: <code>f (g 2)</code>.
<br> <br>
Similar logic can be applied to understand why z is 250 instead of 2500 (e.g. <code>f(5 * 10))</code>. In Haskell, functions have higher precedence than operators, so they're always evaluated first. f takes 5 as its first argument and returns 25. This is then multiplied by 10 to get 250 (e.g. <code>(f 5) * 10</code>). 
<br> <br>
Carey includes a handy table to translate math to haskell expression on slide 21.
</details>

#### Calling Functions Challenge
{: .no_toc }

Below are defintions for a function f and a partially completed function g. Our goal is to have function g return the following: `f(a, sqrt(a+`))`. Complete the body of g using the minimum number of parentheses.

```haskell
f :: Double -> Double -> Double
f x y = x * y

g :: Double -> Double
g a = __________________
```
<details>
<summary><b>Answer</b></summary>
<pre>
g a = f a (sqrt (a+1))
</pre>
</details>

### If Expressions 

The general syntax of the `if` expression is:

```hs
if <expression> then <expression>
                else <expression>
```

Here's a concrete example:

```haskell
bruin_discount :: Int -> Int 
bruin_discount price = 
    if price > 100 
    then price - 20 
    else price - 10
```

Haskell's if expression is required to have an else clause. Why?

This is because if is *not* a statement like in an imperative language. It's an expression, so it must return a value! As a result every possible case needs to be covered with a return value and an else clause is required!

{: .note}
Think about how `if`-`then`-`else` works with types - if the two branches had different types, we'd break some of Haskell's type rules. This formalizes the above argument - why is that?

### Guards 

Consider the below program that uses another control flow mechanism in Haskell: guards!

```haskell
-- exam.hs
evaluateExam :: Int -> String -> String
evaluateExam    score  prof
  | score < 50 && prof == "Eggert" = 
    "Given the curve, you got a B+."
  | score < 50 = 
    "You got an F."
  | score == 100 = 
     impress prof
  | otherwise = 
     "You passed."


impress :: String -> String
impress prof = prof ++ " is impressed!"
```

Let's run the code:

```console
ghci> :load exam
Ok, one module loaded.
ghci> evaluateExam 30 "Eggert"
"Given the curve, you got a B+"
ghci> evaluateExam 30 "Nachenberg"
"You got an F."
ghci> evaluateExam 100 "Eggert"
"Eggert is impressed!"
```

As you can see, guards provide a cleaner syntax to handle multiple conditions.

Some things to notice:
1. There is no equal sign after the final parameter!
2. Each condition follows the format `| condition = expression`.
3. Conditons are evaluted from top to bottom.
4. The keyword `otherwise` is used as the else condition. [^5]
5. Guards are a form of syntactic sugar for if/else statements. They are functionally identical but are a lot easier to read and write.

[^5]: An interesting note is that the `otherwise` keyword is simply an alias for `True`. That's why guards always return the final condition. You could replace `otherwise` with `True` and it would function exactly the same (but that doesn't look as nice).

#### Guards Challenge
{: .no_toc }

Fill in the blanks to create the factorial function using guards.

```haskell
fact :: Int -> Int
fact n 
  | _________ = _________
  | _________ = ______________
```

<details>
<summary><b>Answer</b></summary>
<pre>
fact :: Int -> Int
fact n 
  | n == 0    = 1
  | otherwise = n * fact (n - 1)
</pre>
</details>

Carey provides a nice animation tracing through `fact` on slide 28!

### Local Bindings

Consider the following which uses a *let expression*:

```haskell
-- A function to get your nerdiness level
get_nerd_status gpa study_hrs = 
  let 
      gpa_part = 1 / (4.01 - gpa)
      study_part = study_hrs * 10     
      nerd_score = gpa_part + study_part
  in  
      if nerd_score > 100 then
           "You are super nerdy!"
      else "You're a nerd poser."
```

`let` allows you to define one or more *bindings*. One way to think of them is as local variables! For example, above you can see the name `gpa_part` is bound to `1 / (4.01 - gpa)`. Once we define our bindings, we can use their names to refer to their values in the function body (following the `in` keyword).

Haskell has another way to assign local bindings through the `where` construct. Here's an example:

```haskell
-- A function to get your nerdiness level
get_nerd_status gpa study_hrs = 
  if nerd_score > 100
     then "You are super nerdy!"
     else "You're a nerd poser."
  where
      gpa_part = 1 / (4.01 - gpa)
      study_part = study_hrs * 10     
      nerd_score = gpa_part + study_part

```
As you can see, with `where` the bindings follow the body of the function. A rule of thumb as to when you should choose `let` vs `where`: when defining a variable for a single expression use `let` and for multiple use `where`.

{: .note }
The second case is mainly for *guards*. You can technically still use `let` with guards but it leads to a lot of duplicated and ugly looking code. Use `where`!

#### Nested Functions
{: .no_toc }

Local bindings are not limited to variables (as one would expect in a functional language). You can define nested functions as follows:

```haskell
-- Function to describe someone's behavior
whats_the_behavior_of name = 
 if name == "Carey"
    then behaves_like name "twelve year-old"
    else behaves_like name "grown-up"
 where
  behaves_like n what = 
      n ++ " behaves like a " ++ what ++ "!"
```

We can actually refine this code further! Nested functions have visibility into all of a functions variables, so we don't need to pass in the name. We can just refer to it directly!

```haskell
-- Function to describe someone's behavior
whats_the_behavior_of name = 
 if name == "Carey"
    then behaves_like "twelve year-old"
    else behaves_like "grown-up"
 where
  behaves_like what = 
      name ++ " behaves like a " ++ what ++ "!"
```

Nested functions are super helpful for breaking down large problems into smaller ones!

#### Where Challenge
{: .no_toc }

Fill in the blanks to create the `isPrime` function.

```haskell
-- returns True if n is prime
isPrime :: Int -> Bool
isPrime n =
  checkDivisors n (n-1)
where
  checkDivisors n divisor
  | divisor == ____         = True
  | n `mod` divisor == ____ = ______
  | otherwise = checkDivisors n ___________

```

<details>
<summary><b>Answer</b></summary>
<pre>
-- returns True if n is prime
isPrime :: Int -> Bool
isPrime n =
  checkDivisors n (n-1)
where
  checkDivisors n divisor
  | divisor == 1         = True
  | n `mod` divisor == 0 = False
  | otherwise            = checkDivisors n (divisor-1)

</pre>
</details>

## Composite Data Types

We'll dive into a few composite types:

- a **`Tuple`** is a **fixed-size** collection of values; each value can be any type.
- a **`List`** is a sequence of values; **each value must be the same type**.

Let's see some examples in action.

### Tuples

You might have been exposed to tuples before (whether it was in another language or a math class), and they're pretty much what you'd expect here. They let us group related data (of possibly varying types) into a single value. We often use tuples for simple temporary groupings of data e.g. to return multiple values from a function.

```console
ghci> school_and_rank = ("UCLA", 1)
ghci> true_wonder = (True, 1, "der")
ghci> fst school_and_rank
"UCLA"
ghci> snd school_and_rank
1
```

You can see that we define tuples by surrounding a comma separated list of values with parentheses. In order to retrieve the first item from a 2-tuple, we use `fst`. We can use `snd` to retrieve the second. 

{: .warning}
Note that `fst` and `second` only work on tuples of length 2! To retrieve values from larger tuples, you'll need to use something called pattern matching, which is a beautiful feature of Haskell that we will get into soon.

Here are some examples of functions that use tuples!

```haskell
-- tuples.hs

-- Halves the value or neg halves it
halveOrNegate :: (Double, String) -> Double
halveOrNegate    t =
    if snd t == "neg"
    then fst t * (-1.0)
    else fst t / 2.0

-- Performs safe division
safeDivide :: Int -> Int -> (Bool, Int)
safeDivide    num    denom
  | denom /= 0 = (True, quotient)
  | otherwise  = (False, 0)
  where
    quotient = num `div` denom

```

Some things to note here:
1. In `halveOrNegate`, we *must* use parenthese to surround negative numbers as in (-1.0).
2. To divide `Ints`, as in `safeDivide`, you use `` `div` ``. You can also use the prefix version of `div` by removing the backticks as in `div 20 5`. The `/` operator only works for floating point values.
3. We use `/=` for not equals!
4. The local binding `quotient` is only evaluated in the case when it is needed i.e. when `denom` is not equal to `0`. This is a result of Haskell's lazy semantics! It comes in handy often in situations like these because it prevents a divide by zero error.
5. We can use a tuple to return multiple values from a function.

One last important thing to note about tuples in general is that their type is defined by three things:
1. the **number** of elements it contains
2. the **types** of each element
3. the **order** of each element

So for example the following are all different types:

```haskell
(True, "hello")
(False, "goodbye", 0)
("hello", 1, True)
```

### Lists

Now moving on to the most important data type in functional programming: lists! Similar to other languages, a list can hold zero or more of the *same* type. Note that **they are not arrays**: they have `O(N)` access and no pre-defined size.

```hs
--- examples.hs
-- A List of primes
primes :: [Int]
primes = [1,2,3,5,7,11,13]

-- A List of jobs
jobs = ["SWE","Chef","Writer"]

-- A List of Lists
lol = [[1,2,3],[4,5],[6,7,8,9]]

-- A list of tuples
lot = [("foo",1),("bar",2),("boo",3)]

-- An empty list
mt = []
```

The most commonly used list functions are `head` and `tail`; `head` returns the first item of the list, and `tail` returns a list of the rest of the items!

```console
ghci> :load examples
[1 of 1] Compiling Main             ( examples.hs, interpreted )
Ok, one module loaded.
ghci> head primes
1
ghci> tail primes
[2,3,5,7,11,13]
```

Here are some other handly simple list functions.

Is a list empty?

```console
ghci> null []
True
```

Get the list's length:

```console
ghci> length primes
7
```

`take` and `drop` are convenient ways to access slices of a list.

```console
ghci> primes
[1,2,3,5,7,11,13]
ghci> take 3 primes -- return the first 3 elements
[1,2,3]
ghci> drop 4 primes -- return all elements except first 4
[7,11,13]
```

`!!` gives you random access by index:

```console
ghci> jobs
["SWE","Chef","Writer"]
ghci> jobs !! 2
"Writer"
```

`elem` tells you if an item is in a list:

```console
ghci> elem "Chef" jobs
True
```

`sum` adds up the entire list; there's also a corresponding `prod`

```console
ghci> sum primes
42
```

`or` performs boolean `or` over the list; there's also a corresponding `and`

```console
ghci> or [True, False, False]
True
ghci> and [True, False, False]
False

```

`zip` creates tuples out of two lists. This is the same as [python's `zip`](https://docs.python.org/3.9/library/functions.html#zip)!

```console
ghci> zip [10,20,30] jobs
[(10,"SWE"),(20,"Chef"),(30,"Writer")]
```

{: .note}
Like the name may imply, Haskell lists are implemented as singly-linked lists. List operations are implemented recursively! This has performance impacts, since there is no linear random access!

#### Constructing Lists: Ranges
{: .no_toc }

*Ranges* are a neat feature in Haskell that let us easily create lists. (You could think of this behaving similarly to when you drag a cell on Excel/Google Sheets to repeat a pattern.)

```hs
-- example.hs
-- All #s between 1 and 10, inclusive
one_to_ten = [1..10]

-- Odd #s between 1 and 10
oddities = [1,3..10]

-- An infinite list from 42 onward!
whole_lotta_numbers = [42..]

-- An infinite cycle of 1,3,5,1,3,5,…
tricycle = cycle [1,3,5]
```

```console
ghci> :load examples
[1 of 1] Compiling Main             ( examples.hs, interpreted )
Ok, one module loaded.
ghci> one_to_ten
[1,2,3,4,5,6,7,8,9,10]
ghci> oddities
[1,3,5,7,9]
ghci> take 5 whole_lotta_numbers
[42,43,44,45,46]
ghci> take 10 tricycle
[1,3,5,1,3,5,1,3,5,1]
```

Woah ... what's going on with infinite lists? That can't possibly work, right?

Well ... it does! In particular, this goes back to Haskell being **lazily-evaluated**. Haskell won't generate list items from a range *until they're needed*: so, if you never ask for list element #1000 out of the infinite list, Haskell won't care! In other words, **none of the list is initialized when it's initially declared**.

You may then ask some questions:

- can you take the `tail` of an infinite list?
  - **yes, you can**! for example, `tail [42..]` is the same as `[43..]`. You can use `tail [42..]` just like `[42..]`.
- you might be wondering, why does `tail [42..] == [43..]` hang, and `tail [42..]` just keep on printing numbers? What's up with that?
  - well, when `ghci` returns a value, it calls a "print function" on it; the print function for lists keeps on printing until the list is empty.
  - so, when you type in `tail [42..]`, Haskell will keep printing until the list is empty, which is ... never. Hence, wall of text!
  - on the contrary, `tail [42..] == [43..]` hangs because of the **implementation** of the `==` operator, which compares heads until the list is empty.
  - since this never happens, the program doesn't terminate!
- what is the `length` of an infinite list?
  - try it :)

{: .warning }
Be careful when trying it! In particular, Haskell (and all programming languages) [cannot always tell if your program goes into an infinite loop](https://en.wikipedia.org/wiki/Halting_problem). Have your `Ctrl+C` ready!

#### Constructing Lists Challenge
{: .no_toc }

The `++` and `:` operators are used to concatenate elements and lists to create new lists. See if you can guess how these operators work!

```console
ghci> strange = ["stalking", "llamas"]
ghci> strange ++ ["in, "pajamas]
["stalking", "llamas", "in, "pajamas]

ghci> "stealthily" : strange
["stealthily", "stalking", "llamas"]

ghci> "dogs" : []
["dogs"]

ghci> "barking" : "dogs" : []
["barking", "dogs"]
```

As you can see, `++` takes two lists of the *same type* and concatenates them together. The cons operator `:` takes a value and prepends it to the beginning of a list. This value must be the same type as the list elements!

In order to append an item onto the end of a list you can do the following:

```console
ghci> value = ["together"]
ghci> strange ++ value
["stalking", "llamas", "together"]
```

### Strings

Consider the following code operating on strings.

```console
ghci> truth = "Del Taco rules"
ghci> head truth
'D'
ghci> tail truth
'el Taco rules"
ghci> truth ++ "!!!"
"Del Taco rules!!!"
ghci> :t truth
truth :: [Char]
ghci> "hey" == ['h', 'e', 'y']
True
```

These are all list operations. Clearly, strings are just lists of characters!

### List Processing Challenges

Consider the following function that counts the number of items in a list of doubles.

Now, using the count function as an example, complete the product function!

```haskell
count :: [Double] -> Int
count    lst
 | null lst  = 0
 | otherwise =  1 + count (tail lst)

 -- computes the product of all values in a list
prod :: [Double] -> _______
prod    lst
 | null lst  = ________
 | otherwise = ________ * prod __________
```

<details>
<summary><b>Answer</b></summary>
<pre>
prod :: [Double] -> Double
prod    lst
 | null lst  = 1.0
 | otherwise = head lst * prod (tail lst)
</pre>
</details>

Here are a few more examples:

#### Squaring a List
{: .no_toc }
```haskell
-- Return [a^2, b^2, c^2] for input [a, b, c]
sqr_it lst 
 | lst == [] = []
 | otherwise = first_sq : (sqr_it rest)
where
  first_sq = (head lst) ^ 2
  rest = tail lst
```

Some things to notice:
- if our list is empty, that means there's nothing left to square
- otherwise we use the cons operator to concatenate the square of the head to result of applying the funtion to the rest of the list
- classic recursive leap and base case!

#### Reversing a List
{: .no_toc }
```haskell
-- Reverse a list
rev lst
 | lst == [] = []
 | otherwise = (rev rest) ++ [first]
where
  first = head lst
  rest  = tail lst
```

Some things to notice:
- once again, an empty list means there is nothing to reverse. this is our base case!
- otherwise we return the result of reversing the rest of the list with the first element concatenated onto the end. this is the recursive step!
- check out slide 45 if you want a nice visualization of this function in action!

#### Smolest Item Challenge
{: .no_toc }

Fill in the blanks to find the smallest value in a list:

```haskell
-- Find sm0l-est value in a list
sm0lest lst 
 | lst == [] = error "empty list"
 | length lst == 1 = _____
 | otherwise = min first ______________
where
  first = head lst
  rest = tail lst
```

<details>
<summary><b>Answer</b></summary>
<pre>
sm0lest lst 
 | lst == [] = error "empty list"
 | length lst == 1 = first
 | otherwise = min first (sm0lest rest)
where
  first = head lst
  rest = tail lst
</pre>
</details>

## Intermediate Haskell

Now that we've covered the basics of Haskell, let's get into some more intermediate topics!

### List Comprehensions

List comprehensions are a compact yet extraordinarily powerful syntax to create lists.

Semiformally, here's a definition:

> A list comprehension is a F.P. construct that lets you easily create a new list based on one or more existing lists.
> With a list comprehension, you specify the following inputs:
> 1. One or more input lists that you want to draw from (these are called "**generators**")
> 2. A set of filters on the input lists (these are called "**guards**")
> 3. A **transformation** applied to the inputs before items are added to the output list.

Carey created a neat animation that maps some Python-like pseudocode into what a Haskell list comprehension looks like on slide 50.

The pseudocode is as follows:
```
process(input_list):
  output_list = []
  for each x in input_list:
       if guard1(x) == True and guard2(x) then
          z = f(x)
          output_list.append(z)
  return output_list

```

Here is the general format of a list comprehension:
```haskell
output_list = [ f(x) | x <- input_list, guard1(x), guard2(x)]
```

Try to map this directly onto the python pseudocode to get a good idea of what's going on (or watch the aforementioned animation on slide 50). We call the `x <- input_list` part a generator.

Comprehensions can also use multiple input lists and guards! 

{: .note}
If you happen to be familiar with [set-builder notation](https://en.wikipedia.org/wiki/Set-builder_notation) from math classes, comprehensions are similar (see section on programming languages for a side-by-side comparison).

#### Simple Examples
{: .no_toc }

Here are some simple examples:

```hs
-- comp.hs
-- Generate squares of 2 thru 7
squares = [x^2 | x <- [2..7]]

-- Generate products of two lists
prods = [x*y | x <- [3,7], y <- [2,4,6]]

-- Generate combinations of strings
nouns = ["party","exam","studying"]
adjs = ["lit","brutal","chill"]
combos = [adj ++ " " ++ noun |
          adj <- adjs, noun <- nouns]

-- Generate combinations of tuples
menu = [(oz,flavor) |
   oz <- [4,6],
  flavor <- ["Choc","Earwax","Booger"]]
```

Now if we load these expressions into ghci:

```console
ghci> :load comp.hs
[1 of 1] Compiling Main             ( comp.hs, interpreted )
Ok, one module loaded.
ghci> squares
[4,9,16,25,36,49]

ghci> prods
[6,12,18,14,28,42]

ghci> combos
["lit party","lit exam","lit studying","brutal party","brutal exam","brutal studying","chill party","chill exam","chill studying"]

ghci> menu
[(4,"Choc"),(4,"Earwax"),(4,"Booger"),(6,"Choc"),(6,"Earwax"),(6,"Booger")]
```

In cases like `prods` where we have multiple input lists, you can think of it as a nested for loop. In other words, we can translate prods to the following pseudocode:

```
process(xs, ys):
  output_list = []
  for each x in xs:
    for each y in ys:
      z = x * y
      output_list.append(z)
  return output_list
```

#### Advanced Examples
{: .no_toc }

Here some more examples, this time with guards.

```haskell
-- comp.hs

-- Generate first 5 #s divisible by 29
-- between 1000 and infinity
div_by_29 = take 5 [x | x <- [1000..],
             (mod x 29) == 0]

-- Extract all non-uppercase chars
lies = "nerds EAT rockS!"
truth = [ c | c <- lies,
          not (elem c ['A'..'Z'])] 

-- Generate right triangles
right_triangle_tuples = [ (a,b,c) | 
     a <- [1..10], b <- [a..10], c <- [b..10],
     a^2+b^2==c^2 ] 
```

Now we load into ghci:

```console
ghci> :load comp.hs
[1 of 1] Compiling Main             ( comp.hs, interpreted )
Ok, one module loaded.
ghci> div_by_29
[1015,1044,1073,1102,1131]
```

Here we get a list of the first five numbers larger than 1000 that are evenly divisible by 29.

```console
ghci> truth
"nerds  rock!"
```

As expected, we filter out all characters that are uppercase.

```console
ghci> right_triangel_tuples
[(3, 4, 5), (6, 8, 10)]
```

Here we create a list of tuples where a is between 1 to 10, b is greater than a and up to 10, and c is greater than b and up to 10 and where the three could form the sides of a right triangle.

{: .note} 
A generator that references a variable from an earlier generator is called a dependent generator.

#### Comprehensions in Action
{: .no_toc }

Think back to CS 32... do you remember quick sort? In short, it's an efficient divide and conquer sorting algorithm. We can very easily implement it using list comprehensions in Haskell!

Quicksort involves:
1. Choosing a pivot value
2. Partitioning the list so that all elements less than the pivot come before it and all elements greater come after
3. Finally, we recursively sort the left and right sides of the pivot until we reach our base case (if the list is empty)

This is a Haskell implementation utilizing list comprehensions:

```haskell
-- Quicksort in eight lines! 
qsort lst 
 | lst == [] = []
 | otherwise = (qsort less_eq) ++ [pivot] ++ (qsort greater)
 where
    pivot = head lst
    rest_lst = tail lst
    less_eq = [a | a <- rest_lst, a <= pivot]    
    greater = [a | a <- rest_lst, a > pivot]
```

You can see that we pick the pivot to be the first element of the list. We then retrieve all elements less than (or equal to) the list and assign it the `less_eq` binding and all elements greater and assign it to `greater`. Finally, in our recursive step we concatenate the result of sorting `less_eq` to the pivot value and to the result of sorting `greater`. Our base case covers the empty list. Pretty neat! 

### Pattern Matching

Now, we'll switch gears to an even more powerful type of syntactic sugar called **pattern matching**. While it's not directly related to functions (or indeed, functional programming), it draws heavy inspiration from functional programming principles. In particular, it'll make list and tuple processing easy!

First, we'll explore parameter-based pattern matching, where:

- we define multiple versions of the same function
- each version of the function must have the same number and types of arguments
- when the function is called, Haskell checks each definition in-order from top to bottom, and calls the *first* function which has matching parameters

#### Pattern Matching with Constants
{: .no_toc }

Let's start with matching constant values: we can place constants instead of the name of an expected argument.

```hs
-- genz.hs
course_critic :: String -> String -> String
course_critic "Carey" "CS131" = 
  "Get ready for pop-tarts and Haskell!"
course_critic "Eggert" course =
  course ++ " is gonna be difficult."
course_critic prof "cs131" =
  "I hope you like coding in 20 languages!"
course_critic prof course = 
  course ++ " with " ++ prof ++ " is fun!"
```

In the function above, the first version has `"Carey"` in the place of the first argument. It only runs when user passes in `"Carey"` for the `name` argument. The same rule applies to other versions.

Here are two straightforward calls:

```console
ghci> :load genz
[1 of 1] Compiling Main             ( genz.hs, interpreted )
Ok, one module loaded.

ghci> course_critic "Smallberg" "CS32"
"CS32 with Smallberg is fun!"

ghci> course_critic "Carey" "CS131"
"Get ready for pop-tarts and Haskell!"
```

Here's a more tricky call, where it *could* match two different versions:

```
ghci> course_critic "Eggert" "CS131"
"CS131 is gonna be difficult!"
```

Here, we use the Eggert version since it's defined *first*. Order matters!

Pattern matching with constants works with other types, like numbers or tuples. There is no limit on how many constant patterns you can have:

```hs
factorial :: Integer -> Integer
factorial 0 = 1
factorial n = n * factorial (n - 1)

list_len :: [a] -> Int
list_len [] = 0
list_len lst = 1 + list_len (tail lst)

exp :: (Int, Int) -> Int
exp (_, 0) = 1
exp (base, exponent) = base ^ exponent
```

In the `exp` example, we use pattern matching to make processing a tuple much easier! Notice that we can use `_`'s as wild cards i.e. if you don't care what a value will be and don't want to bind it to a variable. We could have used the `fst` and `snd` functions, but pattern matching allows for a much cleaner syntax!

{: .note}
Remember last lecture when we said we'd get to extracting values from larger tuples? Well, here it is! You can use pattern matching to extract any element from an n-tuple.

New learners are often tempted to write code like:

```hs
tuple_eq (a, a) = True   -- WRONG!!!
tuple_eq _ = False
```

Haskell will complain about "conflicting definitions for `'a'`" on the first line. To compare the two values obtained from pattern matching, they have to be bound to different names and compared afterwards.

```hs
-- using guards
tuple_eq (a, b)
  | a == b = True
  | otherwise = False

-- or, use (==) directly
tuple_eq (a, b) = a == b
```

This differs from Prolog, a language we'll explore much later in the course.

{: .note }
You might've noticed that `_` is the exception to the above rule. Haskell has some special logic for `_` -- it doesn't perform an actual binding, so you can use multiple `_` in a tuple with no problems!

#### Pattern Matching with Lists
{: .no_toc }

We can apply similar logic to pattern matching with lists. Here, we'll rely on the cons operator (`:`) to form our structure. Something of the form `x:xs` matches the head `x` (first item) and the tail `xs` (last item); we can think of it as matching the entire list of `x : xs`!

```hs
-- lists.hs
-- Extract the first item from a list
get_first_item (first:rest) = 
   "The first item is " ++ (show first)

-- Extract all but the first item of a list
get_rest_items (first:rest) = 
   "The last n-1 items are " ++ (show rest)

-- Extract the second item from a list
get_second_item (first:second:rest) = 
   "The second item is " ++ (show second)
```

`get_second_item` comes naturally from our view of pattern matching: we can think of this as "inverse cons" for `x : (y : xs)`, which means that `x` is the head, and `y` is the next item after the head. Note that it is convention to always use `(x:xs)`, `(x:y:xs)` etc. when pattern matching! 

{: .note}
The `show` function converts built-in Haskell types into a String that can be printed or concatenated with another String.

```console
ghci> :load lists
[1 of 1] Compiling Main             ( lists.hs, interpreted )
ghci> get_first_item [5,10,15,20]
5

ghci> get_rest_items [5,10,15,20]
[10,15,20]

ghci> get_second_item [5,10,15,20]
10

ghci> get_rest_items [5]
[]
```

We can also use the constant `[]` in place of the tail. As you'd expect, this only matches patterns where the tail is *exactly* the empty list.

```hs
-- favs.hs
favorites :: [String] -> String
favorites [] = "You have no favorites."
favorites (x:[]) = "Your favorite is " ++ x
favorites (x:y:[]) = "You have two favorites: "
 ++ x ++ " and " ++ y
favorites ("chocolate":xs) =
  "You have many favs, but chocolate is #1!"
favorites (x:y:_) =
   "You have at least three favorites!"
```


```console
ghci> :load favs
[1 of 1] Compiling Main             ( favs.hs, interpreted )
ghci> favorites []
"You have no favorites."

ghci> favorites ["chocolate"]
"Your favorite is chocolate"

ghci> favorites ["mint","earwax"]
"You have two favorites: mint and earwax"

ghci> favorites ["chocolate","egg","dirt"]
"You have many favs, but chocolate is #1!"

ghci> favorites ["tea","coffee","carrot"]
"You have at least three favorites!"
```

Broadly, pattern matching with lists makes our code more concise and readable! Consider our previous guard-based `sm0lest`:

```hs
sm0lest lst
 | lst == [] = error "empty list"
 | length lst == 1 = first
 | otherwise = min first (sm0lest rest)
 where
   first = head lst
   rest = tail lst
```

With pattern matching, the logic becomes more compact!

```hs
sm0lest [] = error "empty list"
sm0lest [x] = x
sm0lest (x:xs) = min x (sm0lest xs)
```

The same applies to the following reverse list example:

```hs
-- Reverse a list w/o pattern matching
rev lst
 | lst == [] = []
 | otherwise = (rev rest) ++ [first]
 where
  first = head lst
  rest  = tail lst

-- Reverse a list with pattern matching
rev [] = []
rev (x:xs) = (rev xs) ++ [x]
```

{: .note }
This `reverse` algorithm is not optimal; it has `O(N^2)` complexity (in the length of the list). Can you figure out an `O(N)` algorithm?

#### Pattern Matching Challenge
{: .no_toc }

Fill in the blanks for the following function, which takes the first n items from a list!

```haskell
-- Take the first n items from a list
-- take_n 3 [2,4..20] 🡪 [2,4,6]

take_n num  ___  = []
take_n 0 lst =  ___
take_n n (x:xs) = ___ : take_n ___ xs
```

<details>
<summary><b>Answer</b></summary>

<pre>
-- Take the first n items from a list
-- take_n 3 [2,4..20] 🡪 [2,4,6]

take_n num  []  = []
take_n 0 lst =  []  
take_n n (x:xs) = x : take_n (n-1) xs
</pre>
</details>

#### Influence
{: .no_toc }

Pattern matching is mostly used in functional languages, but it's notably been adopted by Python and Rust.

Python added pattern matching *very recently* (in [version `3.10`](https://docs.python.org/3/whatsnew/3.10.html), released in 2021). Python's pattern matching works on tuples, lists, dictionaries, and more! Python is a *dynamically* typed language, so match cases can have different types.

```python
def func(val):  # Python pattern matching example
  match val:
    case (0, x):        # match tuple w/first elem zero
        print(f"Tuple of 0 and {x}")
    case ['a', x, 'c']: # match list w/3 elems: 'a', ??, 'c'
        print(f"List of a, {x}, and c")
    case [1, 2, *rest]: # seq of: 1, 2, ... other elements
        print(f'A list with 1, 2, and *{rest}')
    case {'foo': bar}:  # match dict w/key 'foo'
        print(f"foo maps to {bar} ")
    case _:
        print('no match')
```

Rust has almost always had pattern matching. It's statically typed, so this example looks quite like Haskell. 

```rust
match vtree_type {
  VTreeType::LeftLinear => VTree::left_linear(vars),
  VTreeType::RightLinear => VTree::right_linear(vars),
  VTreeType::EvenSplit(num) => VTree::even_split(vars, num),
  VTreeType::FromDTree => {
    let dtree = DTree::from_cnf(&cnf, &VarOrder::linear_order(cnf.num_vars()));
    VTree::from_dtree(&dtree).unwrap()
  }
}
```

### Type Variables Challenge
{: .no_toc}

Here's a function that returns the last value in a list of *any* type.

```haskell
get_last_item lst = 
  head (reverse lst)
```

Now, here is its type signature:

```haskell
get_last_item :: [t] -> t
```

You might be wondering... *what the heck is a t*?

Turns out this is another neat feature of Haskell called a type variable! Type variables are similar to generics in other langauges; they let you pass in any type that you want. In this case, since we have `t` as a parameter and the return type, we know the function must take a list of some type and return an element of the same type. There are also cases where the opposite is true. Consider the following:

```haskell
flipFlop :: t1 -> t2 -> ________
flipFlop    x     y  =  (y,  x)
```

Try to fill in the blank using your intuition!

<details>
<summary><b>Answer</b></summary>

<pre>
flipFlop :: t1 -> t2 -> (t2, t1)
flipFlop    x     y  =  (y,  x)
</pre>
</details>

{: .note}
Nothing prevents types t1 and t2 from being the same type!  So `flipFlop 10 20` is fine! However, it would be impossible to pass `flipFlopSame 10 "twenty"` if the type signature was `flipFlopSame :: t -> t -> (t, t)`.

So in summary: you can think of a type variable as a generic or a template type.[^1] We'll refine this notion in a later lecture.

What's cool is that we didn't have to do anything special - it just works out of the box! **And, we didn't even need to tell Haskell that it's generic**: the type system figured it out by itself!

[^1]: You may be thinking to yourself: is this polymorphism (or generic programming) at runtime, or is a copy of this function generated for each type? In Haskell, it's the former: there is one version of the language that "just works" for all types. This is similar to how Java does it. In other languages, like C++ or Rust, generics are compile-time: a copy of the function is made for each type that uses it (this is called [monomorphization](https://rustc-dev-guide.rust-lang.org/backend/monomorph.html)). We will be learning more about the different types of generic programming later on in the class!

### Type Classes Challenge

Consider the `bigger` function, which returns the larger of its two arguments:

```console
ghci> bigger x y = if x > y then x else y
ghci> bigger 10 5
10
ghci> bigger "cs" "math"
"math"
```

If we request the type of `bigger`, we get the following:

```console
ghci> :t bigger
bigger :: Ord a => a -> a -> a
```

This looks new! This is another great feature of Haskell: type classes. Type classes provide a way to provide contraints to generic types. In particular in this case, `Ord a => a` requires the type `a` to conform to the `Ord` type class.

`Ord` is short for Ordered, meaning `a` must be of a type that is orderable through comparison e.g. numbers, strings.

Here's another example:

```console
ghci> div5 x = x / 5
ghci> :t div5
div5 :: Fractional a => a -> a
```

Here `a` must conform to the `Fractional` type class, meaning it must be a floating point number. Pretty cool! 

{: .note}
You'll see these used in a lot of built in functions, so best get used to them!

### First-class and Higher-Order Functions

One key characteristic of functional languages is that they enable you to treat a function just like any other variable. In other words, functions are *first-class*!

Formally speaking, a language with **first-class functions** treats functions like any other data. They can be:

- stored in variables or data structures
- passed as arguments to other functions
- returned as values by functions

As we'll see, these features enable more **efficient program decomposition** and allow functions to **generate new functions from scratch**
!

One consequence of first-class functions is something called a **higher-order function**, which is either:

- a function that accepts another function as an argument, and/or
- a function that returns another function as its return value

The following example exercises the property of passing functions as arguments:

```hs
-- first.hs
insult :: String -> String
insult name = name ++ " is so cringe!"

praise :: String -> String
praise name = name ++ " is dank!"

talk_to :: String -> (String -> String) -> String
talk_to name talk_func
  | name == "Carey" = "No comment."
  | otherwise = (talk_func name)
```

`talk_to` is a higher-order function: its second argument, `talk_func`, is itself a function that takes in a `String` and returns a `String`. That's reflected in its type: `(String -> String)`; take note of the parentheses in the type declaration to denote a function-type parameter. 

Let's give it a shot:

```console
ghci> :load first
[1 of 1] Compiling Main             ( first.hs, interpreted )
ghci> talk_to "Devan" praise
"Devan is dank!"
ghci> talk_to "Brendan" insult
"Brendan is so cringe!"
ghci> talk_to "Carey" insult
"No comment."
```

This second example *returns* a function!

```hs
-- first.hs
get_pickup_func :: Int -> (String -> String)
get_pickup_func born
  | born >= 1997 && born <= 2012 = pickup_genz
  | otherwise = pickup_other
 where
   pickup_genz name = name ++ ", you've got steez!"
   pickup_other name = name ++ ", you've got style!"
```

The higher-order function `get_pickup_func` returns different functions according to the argument `born`. The returned function can be stored into variable and invoked later.

```console
ghci> :load first
[1 of 1] Compiling Main             ( first.hs, interpreted )
ghci> pickup_fn = get_pickup_func 2003
ghci> pickup_fn "Jayathi"
"Jayathi, you've got steez!"
ghci> get_pickup_func 1971 "Carey"
"Carey, you've got style!"
```

Although first-class and higher-order functions originate from functional programming, they are a pillar of virtually all modern programming languages.

Their use-cases include:

- providing comparison functions for sorting
- as callbacks when events trigger
- enabling multi-threading

Some examples:

```js
// JavaScript
const sortByDate = (a,b) => {
  if (a.date < b.date) return -1;
  if (a.date > b.date ) return 1;
  return 0;
}

movies.sort(sortByDate);
```

```js
// JavaScript
const handleClick = function (event) {
   alert("A button was clicked!");
};

// Select big button from HTML web page
var button = document.querySelector('#big-button');
// Specify what func should be called when it's clicked
button.addEventListener('click', handleClick);
```

```cpp
// C++
void foo()      { /* does some work & returns */ }
void bar(int x) { /* does some work & returns */ }

int main() {
  std::thread thread1(foo), thread2(bar,42);

  // Run main, foo and bar until they all finish
  thread1.join(); // pause until foo() finishes
  thread2.join(); // pause until bar() finishes
  std::cout << "main, foo and bar completed.\n";
}
```

### Map, Filter, and Reduce

Here we introduce a key set of higher-order utility functions to process lists that are included in virtually all functional programming languages.

These functions fall into three categories: mappers, filters and reducers.

- A **mapper** function performs a one-to-one transformation from one list of values to another list of values using a _transform function_
- A **filter** function filters out items from one list of values using a _predicate function_ to produce another list of values
- A **reducer** function operates on a list of values and collapses them into a single output value

We typically use one or more mappers and filters followed by one or more reducers to crunch data, e.g.: `result = reduce(map(filter(input)))`. 

#### Map 
{: .no_toc }

A mapper function **maps a list of values to another list of values** of the same length. 
Here are a few example mapper functions:

| Map each string in a list to uppercase | Map each name in a list to a tuple of (last,first) |
| `["foo","bar"]` returns `["FOO ","BAR"]` | `["Andy Liu","Tia Tan"]` returns `[("Liu","Andy"),("Tan","Tia")]` |

| Map every prime number in a list to True, every other number to False | Map each number in a list to its absolute value |
| `[2,3,4,5,6]` returns `[True,True,False,True,False]` | `[20,-30,-27,45]` returns `[20,30,27,45]` |


Haskell provides a mapper function called `map` that accepts two parameters:

1. A function to apply to every element of a list
2. A list to operate on

The type signature of map is: `map :: (a -> b) -> [a] -> [b]`

- The first argument is a function that maps an individual item from a value of type `a` to a value of type `b` (`a` and `b` can be the same)
- The second argument is a list of type `a`
- The return value is a list of type `b`

We can define some functions to use with `map`:

```hs
-- mappy.hs
cube :: Double -> Double
cube x = x^3

one_over  :: Double -> Double
one_over x = 1/x

is_even :: Int -> Bool
is_even x = x `mod` 2 == 0
```

```console
ghci> :load mappy
[1 of 1] Compiling Main             ( mappy.hs, interpreted )
ghci> map cube [2,4,10]
[8,64,1000]
ghci> map one_over [2,4,8]
[.5,.25,.125]
ghci> map is_even [1,2,3,4,6]
[False,True,False,True,True]
ghci> map reverse ["mouse","cat","fly"] 
["esuom","tac","ylf"]
```

In the last invocation, we used Haskell's library function `reverse`.

So how does the map function actually work? It's simple. We can use the pattern matching we just learned to implement it.

```hs
-- map.hs
map :: (a -> b) -> [a] -> [b]
map func [] = []      
map func (x:xs) =
  (func x) : map func xs
```

#### Filter
{: .no_toc }

A filter is a function that **filters items from an input list** to produce a new output list. 
Here are a few examples of filter functions:

| Filter all odd numbers from an input list | Filter all spaces from a String |
| `[1,2,3,4,5,6]` returns `[2,4,6]`| `"a man a plan a canal panama"` returns `"amanaplanancanalpanama"` |

Haskell provides a function called `filter` that accepts two parameters:

1. A function that determines if an item in the input list should be included in the output list
2. A list to operate on

The type signature of filter is: `filter :: (a -> Bool) -> [a] -> [a]`

- The first argument is a predicate function that determines if a value from the input list should be included in the output
- The second argument is a list of type `a`
- The returned list include all the items that passed the filter

The filter function can also be easily implemented using guards as follows:

```hs
filter :: (a -> Bool) -> [a] -> [a]
filter predicate [] = []
filter predicate (x:xs)
 | (predicate x) = x : (filter predicate xs)
 | otherwise = filter predicate xs
```

#### Reducers 
{: .no_toc }

The last of the three is called a reducer. A reducer is a function that **combines the values in an input list to produce a single output value**.

Here are some reducer examples;

| Reduce all the items from a list into a sum (or product) | Determine if any value in a list meets some requirement (e.g., is odd) |
| `[10,3,7]` returns `20` | `[2,4,6,8]` returns `False` |
| `[10,3,7]` returns `210`| `[2,4,5,8]` returns `True` |

| Count the # of people with each last name from a list of tuples | Append each word in a list into a single string, adding spaces in between |
| `[("Li","Ari"),("Li","Sam"), ("Bui","Tom"), ("Li","Ann")]` returns `[("Li",3),("Bui",1)]` | `["I","like","candy"]` returns `"I like candy "` |

Each reducer takes three inputs:

1. A function that processes each of the elements
2. An initial "accumulator" value
3. A list of items to operate on

Haskell has two different reducer functions: `foldl` and `foldr`. Let's learn about `foldl`!

Let's first look at the pseudocode for `foldl`

```
foldl(f, initial_accum, lst):
  accum = initial_accum
  for each x in lst:
    accum = f(accum, x)
  return accum
```

The core logic of `foldl` all happens around the accumulator variable `accum`:

- It is first set with the initial value `initial_accum`, the second argument to the function. The initial value can often be 0, 1 or an empty list
- Next, the `accum` is updated as we loop through each item in the list. Each time the function `f` is used to "accumulate" the item to `accum`.
- In the end, the final value of `accum` is returned.

Here is one example `f` function for `foldl`.

```hs
f1 :: Int -> Int -> Int
f1 accum x = accum + x
```

If the function `f1` is used in place of argument `f`, `foldl` computes the sum of the elements in the list `lst`. 

{: .note}
When trying to deciper what a reducer is doing, it's helpful to create a simple example and run through the pseudocode above by hand!

Now let's see the Haskell code for `foldl`:

```hs
foldl f accum [] = accum
foldl f accum (x:xs) =
  foldl f new_accum xs
 where new_accum = (f accum x)
```

As you can see, each new value x is "folded" into the accumulator as it's processed.

- `foldl` is left-associative: `f( ... f(f(accum,x1),x2), ..., xn)`


#### Influencer Alert: map-filter-reduce
{: .no_toc }

Nowadays, virtually all modern languages now implement the map-filter-reduce paradigm. Here are some examples from Python, Java and Javascript

```python
# Python map-reduce example
def adder(a,b):
  return a + b

nums = [1, 4, 9, 16]
roots = map(sqrt, nums)

sum_of_roots = reduce(adder, roots)
```

```java
// Java map-reduce example
List<Double> nums =
  Arrays.asList(1.0, 4.0, 9.0, 16.0);

double sum_of_roots = nums.stream()
        .map((n) -> Math.sqrt(n))
        .reduce(0, Double::sum);
```

```js
// JavaScript map-reduce example
function adder(total, num)
 { return total + num; }

var nums = [1, 4, 9, 16];
var roots = nums.map(Math.sqrt);
var sum_of_roots = roots.reduce(adder, 0);
```

Moreover, these functions form a new paradigm that has radically transformed the Internet, medicine, autonomous vehicles, and virtually every other field.

While mapping, filtering and reducing seem marginally useful when running on a small list of items on a single computer, they can be applied to a huge cluster of servers to process massive amount of data.

All of this is possible because these are pure functions that are stateless, so they can be run in parallel!

{: .note } 
If you're interested in exploring MapReduce in more detail, you should take CS 143!

## Advanced Topics

Now we are in the Functional Programming home stretch, where we are learning some more advanced functional programming. Many concepts covered here are actually the core of FP, so things will get ~interesting~ !

### Lambda Functions

We start from lambda functions. By definition, a lambda function is just like any other function, but it _does not have a function name_. 

For example, the corresponding lambda function of a normal function `cube x = x^3` is `\x -> x^3` in Haskell. The two functions are identical except the second does not have a name. Calling `(\x -> x^3) 3` gives you the same result as `cube 3`.

{: .note }
Having lambda functions is yet another manifestation of first-class functions in functional programming languages. They allow functions to be written as values without giving them a name, just like values of other types (integers, strings, etc.)

People may wonder what practical benefit lambda functions can bring. Sometimes it can make code easier to read. We can use lambdas in higher-order functions when we don't want to bother defining a whole new named function.

For example, with a lambda, we can create the following functions:

```hs
squarer lst = map (\x -> x^2) lst
cuber lst = map (\x -> x^3) lst
```

The `squarer` and `cuber` functions computes the square and cube of each item within the list. We don't need to define separate helper functions and the logic can be understood by looking at the one-liner code.

Ok, let's learn the syntax for defining a lambda function in Haskell. A lambda expression in Haskell is written as `\param_1 ... param_n -> expression`

- The lambda function starts with a backslash (`\`). [^2]
- Then you specify the name of one or more parameters, specified by spaces. These are called _bound_ variables.
- The dash-greater sequence (`->`) follows the list of parameters, separates them with the rest of lambda.
- Finally, we have the function's expression, which is what the function computes and returns.

[^2]: The back slash is supposed to resemble a λ.

Let's see some examples in the Haskell interpreter:

```console
ghci> (\x y -> x^3+y^2) 10 3
1009
```

In the example above, we create a lambda expression and call it with arguments `x=10` and `y=3`. Thus it computes `10^3+3^2` and returns 1009.

Here are two examples with `map`:

```console
ghci> map (\x -> 1/x) [3..5]
[0.3333333333333333,0.25,0.2]
ghci> map (\x -> take 2 x) ["Dog", "Cat"]
["Do","Ca"]
```

We can also assign a lambda function to a variable. After the assignment, we can call lambda just like we would call any other function.

```console
ghci> a_func = \x y -> x++y
ghci> a_func [1,2,3] [4,5]
[1,2,3,4,5]
```

{: .note }
Some may wonder, does assigning a lambda to a variable defeat the purpose of having _anonymous_ functions? In some sense, it does seem so. However, looking from another angle, lambda functions can be regarded as more fundamental than named functions. Defining a function like `a_func x y = x ++ y` can be regarded as syntatic sugar of `a_func = \x y -> x ++ y`.

<!-- 
section removed from slides:
Next, let's look at a fancier usage of lambda expression: using it to generate new functions! Look at the following code:

```hs
-- lambda.hs
wrapFuncWithAbs func = (\x -> abs (func x))

cubed x = x^3
twox = 2*x
```

Our function accepts an input function `func`, it builds a new function that computes `y=abs(func(x))` and returns that new function as output.

```console
ghci> :load lambda
[1 of 1] Compiling Main             ( lambda.hs, interpreted )
ghci> abs2x = wrapFuncWithAbs twox
ghci> abs2x (-42)
84
ghci> abscube = wrapFuncWithAbs (\z -> z^3)
27
```

Both `abs2x` and `abscube` are created from return values of `wrapFuncWithAbs` functions.

- The returned lambda assigned to `abs2x` is `\x -> abs (twox x)`
- Meanwhile, `abscube` is `\x -> abs ((\z -> z^3) x)`

Neat, right? We just generated two new functions from scratch! -->

### Closures

Let's take a closer look at what is happening with functions that generates other functions.

```hs
-- lambda.hs
slopeIntercept m b = (\x -> m*x + b)

twoxPlusOne = slopeIntercept 2 1
fivexPlusThree = slopeIntercept 5 3
```

In the example above, we created another function generator `slopeIntercept`:

- It accepts two parameters: `m` and `b`
- When called, it builds and returns a new function that takes argument `x` and computes `y = m*x + b`

Then we create a function `twoxPlusOne` by calling `slopeIntercept` with `m` equals to 2 and `b` equals to 1. In the process, Haskell "captures" the specific values of `m` and `b` along with the expression `\x -> m*x + b`. This way, when we next call `twoxPlusOne`. Haskell will use `m=2` and `b=1`.

```console
ghci> :load lambda
[1 of 1] Compiling Main             ( lambda.hs, interpreted )
ghci> twoxPlusOne 9
19
```

The combination of a _lambda expression_ with a snapshot of all _"captured" values_ (like `m` and `b`) is called a closure.

When we create another function with the same generator, the other function will have its own generator as well, such as the `fivexPlusThree` above. Its closure has different values of `m` and `b` than the one of `twoxPlusOne`.

How do we decide what variables are captured as part of a closure? In short, any variable that is "free" will be captured.

A **free variable** is any variable that is _not an explicit parameter_ to the lambda. For example, in `\x -> m*x + b`, `m` and `b` are free variables, and `x` is not.

{: .note }
Recall that in the context of a lambda, we call parameters **bound variables**. So if a variable is free it means it is not "bound" to any of the lambda's parameters. Makes sense!

Formally, a closure is a combination of the two things:

1. A function of zero or more arguments that we wish to run at  some point in the future
2. A list of "free" variables and their values that were captured at the time the closure was created

When a closure later runs, it uses the values of the free variables captured at the time it was created.

{: .note }
Note that the exact behavior of free variables may vary by language. In other words, Haskell uses the value as captured upon creation, but this might not be true for other languages.

People may wonder where the free variables are stored, because they are still available for use after the function that created the closure has returned. If they were stored on the stack, that would not be possible.

The answer is that they are often stored on the heap. Unlike C and C++ where local variables are managed on the stack, languages that support lambda expression and closure usually store the relevant information on the heap so that they can be used after the creator function returns. The storage is managed by garbage collectors and will be freed when the runtime decide that the captured variables cannot be referenced by anyone.

### Influencer Alert: Lambda and Closure

Nowadays, even in the languages that are not typically functional, you can often find that they support some form of lambda functions and closures. They are used when we want to pass a simple function to another function.

For example, in C++, we can use lambda to provide comparators for sorting:

```cpp
// C++ code that uses a lambda to sort a bunch of
// Students in alphabetical order.

vector<Student> students;
// ...
sort(students.begin(), students.end(),
    [](const Student & a, const Student & b)
    {
       return a.getName() < b.getName();
    });
```

In C++, we need to explicitly specify the free variables to capture inside the pair of brackets. Then we specify the arguments within the parentheses and the function body in the braces.

{: .note}
Note that C++ handles captures differently from most functional languages, but that's outside the scope of this class.

Javascript also has lambdas! Very often, we use them to create callbacks for event handling. The lambda function is created with the `function` keyword

```js
// javascript: Lambda function is called
// when the user clicks on the "big button"

// Select big button from HTML web page
var button = document.querySelector('#big-button');

// Specify the behavior when the button is clicked
button.addEventListener('click',
    function() { alert("A button was clicked!"); });
```

{: .note}
If you're familiar with Javascript, you're probably familiar with arrow functions. Those are another way to write lambdas in JS!

In Java, you can use lambdas to specify the action in a thread:

```java
// Java thread example
public class LambdaThreadExample {
   public static void main(String args[]) {
    final int count = 50000;
    Thread t = new Thread(() -> {
      for(int i=0; i < count; i++)
         System.out.println("Child Thread: "+ i); 
    });

    t.start(); // Start the background thread
    
    // Main Thread
    for(int j=0; j < 100000; j++)
      System.out.println("Main Thread: "+ j);
   }
}
```

### Partial Function Application

When you call a function with less than the full number of arguments this is called "Partial Function Application" or "Partial Application."

Formally speaking, partial function application is an operation where we define a new function `g` by combining:

- An existing function `f` that takes one or more arguments, with
- Default values for one or more of those arguments

The new function `g` is a specialization of `f`, with hard-coded values for some of `f`'s parameters.

Once defined, we can then call `g` with those arguments that have not yet been hard-coded.

Let's look at some examples:

```console
ghci> product_of x y z = x*y*z 
ghci> product_5 = product_of 5 -- product_5 y z = 5 * y * z
ghci> product_5 2 3
30
```

Here we use PFA to create a specialized function that gets the product of 5 and 2 more values.


```console
ghci> product_5_6 = product_of 5 6 -- product_5_6 z = 5 * 6 * z
ghci> product_5_6 2 
60
```

We can take things a step furthur and fix the second argument to be 6. So now we have a function that finds the product of 5, 6, and some other value.

Here's a similar example with strings:

```console
ghci> yell what name = what ++ " " ++ name ++ "!"
ghci> yell_greeting = yell "hello"
ghci> yell_greeting "arathi"
"hello arathi!"
```

Partial function applications can be useful when combined with pre-existing functions and operators. Let's see the following examples:

```console
ghci> cuber = map (\x -> x^3)
ghci> cuber [2,3,5]
[8,27,125]
```

This creates a new mapper that always cubes items in a list.

```console
ghci> keep_odds = filter (\x -> x `mod` 2 == 1)
ghci> keep_odds [1..10]
[1, 3, 5, 7, 9]
```

This creates a filter function that filters all even values.

```console
ghci> summer = foldl (\acc x -> acc + x) 0
ghci> summer [1..10]
```

Here we create a function that sums all the values in a list.

## Advanced Topics Cont.

### Currying

Here comes currying, a fundamental concept in functional programming. It might be confusing at first, but don't worry! We're here to help you through it. And as a bonus, some of the  quirks of Haskell can be easily understood once you understand currying. [^3] Partial function application, which we discussed last time, is a direct result of functions in Haskell being curried!

[^3]: For example, recall what happened when we called `:t` on a function. That is a result of functions in Haskell being curried!

By definition, currying transforms a function of multiple arguments to a series of functions of a single argument.

We'll first use an example to get a feeling of what currying is like. Let's say we have a following function `f` in a JS-like language:

```js
function f(x, y, z) { return x + y + z; }
```

When we curry the function `f`, it's converted to the following "nightmare":

```js
function f(x) {
  function g(y) {
    function h(z) {
      return x + y + z;
    }
    return h;
  }
  return g;
}
```

As we can see, the original function is converted into a series of nested functions:

- The number of nested level equals to the number of argument of the original function
- Each function takes a single argument in sequence, starting from the outmost one
- The outer functions returns the nested function in the next level
- The inner-most function does the original computation

Let's say we wanted to call our original function like this: `f(10, 20, 30)`

If we want to achieve the same effect with curried version

```js
temp_func1 = f(10);
temp_func2 = temp_func1(20);
final_result = temp_func2(30);
```

According to the closure we have just learned, each call above fills in one variable (`x` and `y` respectively). When we do the final call, only the last parameter is needed and we get the result we want.

Or more concisely, we can do it on a single line: `final_result = f(10)(20)(30)`

As it turns out, every function with two or more parameters can be represented in curried form!

{: .note }
It's called "currying" because our friend, Haskell Curry, came up with the approach. (Well, in fact Moses Schönfinkel first had the idea several years before. It could have been called Schönfinkelisation...)

Formally, Currying is the concept that you can represent any function that takes multiple arguments by another that takes a single argument and returns a new function that takes the next argument, etc.

It turns the function `y = f(a, b, c)` into `y = ((f_c a) b) c`.

Each function takes one argument and returns another function as its result (except for the last function which returns the result).

We can represent currying with the following pseudeocode:

```
Curry(Function f)
  e = The expression/body of function f
  For each parameter p from right to left:
    f_temp = Define a new lambda function with:
      1. p as its only parameter
      2. e as its (expression)
    e = f_temp
return f_temp
```

Let's use the procedure above to curry the function `mult3 x y z = x*y*z`

- At first, `e` is equal to `x*y*z`
- Then we enter the for loop
  - In the first iteration, `z` is the parameter to be handled. `f_temp = \z -> x*y*z`
  - Next, we handle `y` and `f_temp = \y -> (\z -> x*y*z)`
  - Finally, we handle `y` and we get `f_temp = \x -> (\y -> (\z -> x*y*z))`

Thus, the curried version `mult3c = \x -> (\y -> (\z -> x*y*z))`.

Let's see how we call our curried function with `x=2`, `y=3` and `z=5`!

```console
ghci> mult3c = \x -> (\y -> (\z -> x*y*z))
ghci> f1 = mult3c 2
ghci> f2 = f1 3
ghci> result = f2 5
30
```

So, what actually are `f1` and `f2`?

`f1` is obtained by calling the lambda of `mult3c` with `x=2`. Thus we replace the occurrences of `x` inside the lambda function with 2 and remove the `\x ->` at the beginning. `f1` is: `\y -> (\z -> 2*y*z)`.

{: .warning }
When a inner lambda has the same parameter name as the outer one. The inner one takes precedence and _shadows_ the outer one. In this case, the shadowed parameter will not get replaced. For example, when we evaluate `(\x -> ((\x -> x) (x + 1))) 3`, we will get `((\x -> x) (3 + 1))`.

Once again, when we repeat the process, we can get `f2 = \z -> (2*3*z)`, and finally `result = 2*3*5`, thus 30.

In fact, every time you define a function of more than parameter in Haskell, Haskell automatically and invisibly **curries it** for you.

This is evidenced by the fact that even if you replace the `mult3c` above with the "normal" `mult3 x y z = x*y*z`, you will get the same result. You can also legally execute `(((mult3 2) 3) 5)` in Haskell and get 30.

Surprising fact: Our usual way of calling functions `mult3 2 3 5` looks like a _single call_ with three arguments, but it's really _three calls_ to three different functions! What happen's is exactly identical to `(((mult3 2) 3) 5)`, thus we know function invocation is _left associative_.

Also, now we can reveal the fact about Haskell function type signatures:

Remember those ugly (and maybe confusing) Haskell type signatures?

```hs
mult3 :: Double -> Double -> Double -> Double
mult3 x y z = x * y * z
```

Well, here's what they should really look like given that all functions are curried (thus `mult3` is a three-level nested function that returns a function at each outer level):

```hs
mult3 :: Double -> (Double -> (Double -> Double))
mult3 x y z = x * y * z
```

Although the latter form looks more _fundamental_, it is even uglier. That's why we used some more syntactic sugar to remove the parentheses. (We can regard the `->` "operator" in type signatures as **right-associative**)

People may ask, why does Haskell curry functions? We think there are two primary reasons:

- First, it enables partial function application. When we call a function with less than the full number of arguments, the curried function will return closure(s) that incorporate each value! Very cool!
- But mostly it appears to be motivated by theorists! It can facilitate program analysis and other stuff and it is consistent with the _lambda calculus_, the mathematical foundation of functional programming.

### Algebraic Data Types

Algebraic Data Type (ADT) is the term we use in functional programming to describe a user-defined data type that can have multiple fields.

The closest thing to algebraic data types would be C++ structs and enumerated types. We use algebraic data types to create complex data structures like trees.

In Haskell, we use the `data` keyword to define algebraic data types.

The simplest algebraic data types are just like C++ enums.

<!-- ```hs
data Drink = Water | Coke | Sprite | Redbull
data Veggie = Broccoli | Lettuce | Tomato
data Protein = Eggs | Beef | Chicken | Beans
```

For the `Protein` type, its equivalent in C++ would be:

```cpp
enum Protein { Eggs, beef, chicken, Beans };
```

We can also define more complex ADTs where each variant also has one or more fields (like C++ struct).

```hs
data Meal =
 Breakfast Drink Protein |
 Lunch Drink Protein Veggie |
 Dinner Drink Protein Protein Veggie |
 Fasting
```

The definition of first variant `Breakfast` means that it is comprised of `Drink` and `Protein` fields. Its equivalent in C++ would be:

```cpp
struct Breakfast {
  Drink d;
  Protein p;
}
```

The only difference is, the fields in Haskell does not have a name, and they are distuished by the order they appear in the definition.

Note that the variants with and without fields can coexist within a single ADT, the `Fasting` above is just an example.

Here are some examples to define variables with ADT:

```hs
careys_meal = Breakfast Redbull Eggs
pauls_meal = Lunch Water Chicken Broccoli
davids_meal = Fasting
```

#### Syntax

We use another example to introduce the syntax of Algebraic Data Types -->

Here is one way to define an ADT in Haskell:

```hs
data Color = Red | Green | Blue deriving Show

data Shape =
  Circle    { radius :: Float,
              color :: Color } |
  Rectangle { width :: Float, 
              height :: Float, 
              color :: Color } |
  Triangle  { base :: Float, 
              height :: Float, 
              color :: Color } |
  Shapeless
  deriving Show

my_color = Red
my_circ = Circle { radius = 5.0, 
                   color = Blue }
```
In English, a `Shape` can be either a `Circle`, `Rectange`, `Triangle`, or `Shapeless`. A `Circle` has a `radius` and a `color`. A `Rectangle` has a `width`, a `height`, and a `color`. A `Triangle` has a `base`, `height`, and a `color`. Finally, `Shapeless` has no fields.

We can also use a more compact syntax as follows:

```hs
data Color = Red | Green | Blue deriving Show

data Shape =
         -- radius color
  Circle    Float  Color | 
         -- width  height color
  Rectangle Float  Float  Color |
         -- base   height color
  Triangle  Float  Float  Color |
         -- no fields
  Shapeless
  deriving Show

my_color = Red
my_circ = Circle 5.0 Blue
```
You can use the compact syntax for `my_color` and `my_circ` regardless of how you define your ADT.

{: .note}
Add the keywords "deriving Show" at the end of a definition and GHCi will let us print out the field values!

```console
ghci> :load algebraic
[1 of 1] Compiling Main             ( lambda.hs, interpreted )
ghci> c = Circle 5 Red
ghci> c
Circle 5.0 Red
ghci> r = Rectangle 5 6 Blue
ghci> r
Rectangle 5.0 6.0 Blue
ghci> :t r
r :: Shape
ghci> colors = [Red, Red, Blue]
ghci> colors
[Red,Red,Blue]
```
Note that the fields of a variant have no names! That means we must use their order/position to identify them.

#### Pattern Matching with ADT
{: .no_toc }

In functional languages, we use pattern matching to process algebraic variables.

Let's see an example with our shapes. 

```hs
-- shapes.hs

data Color = Red | Green | Blue

data Shape =
         -- radius color
  Circle    Float  Color | 
         -- width  height color
  Rectangle Float  Float  Color |
         -- base   height color
  Triangle  Float  Float  Color |
         -- no fields
  Shapeless

getArea :: Shape -> Double
getArea Shapeless = 0
getArea (Circle r _) = pi * r^2
getArea (Rectangle w h _) = w * h
```

We focus on the `getArea` function:

- The first line is the type signature. As this functions operates on various types of `Shape`, this definition makes sense.
- Then follows the actual pattern matching
  - The first line means: "If the user passes in a Shapeless variable, then always return zero".
  - The second line handles the `Circle` case. It gets the first field (radius), placing it into variable `r` and computes the area.
  - The last line handles `Rectangle`. It extracts the fields from the data structure and computes the area according to the width and height.

<!-- {: .note }
We can combine pattern matching on ADT with lists and tuples to create complex patterns. For example, the following function counts the occurrences of `Circle` within a list of `Shape` without using `map`, `filter` or `foldl`.

```hs
count_circle :: [Shape] -> Integer
count_circle [] = 0
count_circle ((Circle _ _ _):xs) = 1 + count_circle xs
count_circle (_:xs) = count_circle xs
```

Exercise: Implement count_circle with `map`, `filter` and `foldl` respectively. -->

#### Creating Lists with ADTs
{: .no_toc }

Let's examine a more interesting use of ADts--creating our own linked list.

Here's the defintion:

```hs
data List = 
  Nil | 
  Node { 
    val  :: Integer, 
    next :: List 
  } 
  deriving Show
```

A `List` has two possible variants:
- an empty variant called `Nil`
- a `Node` variant that has a value and a reference to the next item in the `List`

Here's how we might use it:

```console
ghci> :load list
Ok, one module loaded.
ghci> list1 = Nil
ghci> list2 = Node 5 list1
ghci> list2
Node {val = 5, next = Nil}
ghci> head = Node 3 ( Node 2 (Node 1 Nil))
ghci> head
Node {val = 3, next = Node {val = 2, next = Node {val = 1, next = Nil}}}
```

Now let's write a function that sums up all the values in a list.

```hs
sumList :: List -> Integer
sumList (Nil) = 0
sumList (Node val next) = 
   val + sumList next
```

Here's how it works:
- we take a list to traverse
- for an empty list (`Nil`) we return 0
- for a non-empty list (`Node`) we return the value of the current node plus the sum of the rest of the list

### Immutable Data Structures

Great! Now consider how you might write a function that adds to the end of the list. Is it possible?

Recall one of the core tenets of functional programming: immutability. Unfortunately, to add even a single item to the end would require us to remake the entire list from scratch. This is because we would have to modify the `next` pointer of the last item in the list to point to the new node, forcing us to create a new node for the last item. This sets off a domino effect resulting in us recreating the entire list from scratch.

Inserting a node at the end of an immutable list is O(n)!

Moral of the story: Immutability impacts how you construct data structures, making some more efficient than others in functional languages!

<!-- #### Creating Trees with ADTs

Here's another interesting use of ADTs – creating a binary search tree.

We'll start by looking at the definition of a tree node ADT.

```hs
-- bst.hs
data Tree =
  Nil |
  Node String Tree Tree
```

A tree data type has two possible variants:

- An empty variant indicating that there is no node, like a `nullptr`
- A node variant representing actual nodes.

In this definition, each Node holds a String value, and two other tree data items that can be either `Nil` or `Node`. The first one is the left child of the node, and the second one is the right child. So this is a binary tree.

```console
ghci> :load bst
[1 of 1] Compiling Main             ( bst.hs, interpreted )
ghci> empty_tree = Nil
ghci> one_node = Node "Lit" Nil Nil
ghci> left_child = Node "Boomer" Nil Nil
ghci> right_child = Node "Zoomer" Nil Nil
ghci> root = Node "Cheugy" left_child right_child
ghci> root
Node "Cheugy" (Node "Boomer" Nil Nil) (Node "Zoomer" Nil Nil)
```

In the end of the code above, we created a tree with three nodes.

With this tree structure, we can implement search algorithm on this BST.

We'll use a classic recursive search function and pattern matching.

```hs
search Nil val = False
search (Node curval left right) val
 | val == curval = True
 | val < curval = search left val
 | otherwise = search right val
``` -->

#### Immutable Trees
{: .no_toc }

Ok, now let's see how to create a function that adds a new node to a tree. We run into similar issues here as with the `List`.

Suppose we create a node N and we want to make it the child of existing node M. We can't just find a node M and change its left or right child to point to the new N due to the immutable nature of variables!

The strategy is, we want to create a new node M', let it point to N and replace the node M we need to change.

But wait... now the new node M is not part of the tree either. But we can just carry it on to create a new node to replace M's parent. Eventually we can do this all the way to the root, and at that point, we can forget about the old tree.

Carey's slides provide a great visualization of this topic.

The following Haskell code implements the logic:

```hs
insert Nil val =
  Node val Nil Nil

insert (Node curval left right) val
  | val == curval =
      Node curval left right
  | val < curval =
      Node curval (insert left val) right
  | val > curval =
      Node curval left (insert right val)
```

This method sounds a little bit wasteful, because we need to create new nodes all the way up. However, if we think carefully about the process, we'll find that to add a new node, we just need to generate replacement nodes for the nodes between our new node and the root. Then if a tree is balanced with `n` nodes, we need to create just `log2(n)` new nodes!

Meanwhile, with the creation of the new node, the old nodes will no longer be used. This is where "Garbage Collection" comes in.

Garbage Collection is a language feature that automatically reclaims unused variables, and all functional languages have built-in Garbage Collection.

We'll learn more about GC later in the course, but the important thing is that GC doesn't impact Big-O!

#### Immutable Data Structures - Pros and Cons
{: .no_toc }

Clearly immutability impacts the efficiency of some data structures. However, there are also big benefits to immutable data structures:

1. **They're thread safe!** There are no race conditions, so there's no need for synchronization.
2. **They're easier to debug!** Since objects cannot change unexpectedly, identifying the origin of issues becomes less difficult.
3. **They're cache friendly!** Caches do not need to be frequently updated or invalidated, improving cache hit rates.

<!-- Although here we see that immutable tree is efficient (in the sense it does not change the Big-O bound of operations). It does not apply to all data structures.

For example, if hash tables are implemented as immutable, the insertion will be inefficient because you must regenerate the full array of n buckets every time you add a new item. -->

#### Influencer Alert: Immutability
{: .no_toc }

Many languages now provide immutable data structures.

Examples include Google's Guava library for Java, and immutable.js for JavaScript.

```java
// Java Guava immutability example
Set<Integer> oldSet = ImmutableSet.of(1,2,3,4);

// Generate a whole new set, with 5 added to it
Set<Integer> newSet = new ImmutableSet.Builder<Integer>().addAll(oldSet)
                               .add(5)
                               .build();
```

```js
// JavaScript immutable.js example

// Create a new immutable set of 1-4
const oldSet = Immutable.Set([1,2,3,4]);

// add returns a new set & doesn't change orig.
const newSet = oldSet.add(5)
```

#### Influencer Alert: Garbage Collection
{: .no_toc }

Most modern languages like Java, JavaScript, C#, and Go are fundamentally based upon garbage collection! 

We'll do a deep dive into garbage collection later on in the course.


### Summary

In summary, here is what we have learned about functional programming

- Every function must take an argument.
- Every function must return a value.
- Functions are "pure" and have no side effects
- Calling the same function with the same input always returns the same result.
- All variables are "immutable" and can never be modified!
- Functions are just like any other data and can be stored in variables and passed as arguments.

And as we have seen along the way, while there are no purely functional languages in mainstream use, the paradigm has influenced every major language.

{: .note }
This concludes our introduction to Haskell and functional programming. Hope you had fun!


------
## Footnotes
{: .no_toc}

<!-- 
## Appendix

Everything that follows are notes for the old version of this lecture from last year! It was originally written by the wonderful founding CS 131 TA Matt Wang.

### The Golden Rule of Haskell Indentation
{: .no_toc }

Code which is part of an expression should be indented further in than the beginning of that expression.

You must align the spacing for all items in a group.

**If you don't follow this rule, you'll get some errors!**

Examples:

```hs
mult x y =
  x * y
```

```hs
let
  x = 5
  y = 6
```

```hs
-- note: this is rather uncommon
let x = 5
    y = 6
```

```hs
where
 x = a
 y = b
```

```hs
where x = a
      y = b
```

```hs
case x of
  42 -> foo
  -1 -> bar
```

```hs
if foo
   then bar
   else boo
```

## Data Types in Haskell
{: .no_toc }

### The Type System
{: .no_toc }

Haskell is a **statically typed language**. That means that the type of *all* variables and *all functions* can be figured out at **compile time**

(this is different from Python - a dynamically typed language - where types can only be determined as the program actually runs)

Haskell also has **type inference**: the compiler can figure out the types of most expressions *without you annotating them*! Neat!

(this started in functional programming, and has made its way into mainstream languages like C++'s `auto` keyword)

### Primitive Types
{: .no_toc }

The built-in data types are:

- `Int`: 64-bit signed integers
- `Integer`: arbitrary-precision signed integers
  - aside: how does this work?
  - in languages like Python, we can treat numbers as arrays of `Int`s of a "Base `2^64`" system [^2]
- `Bool`: booleans (`True` or `False`)
- `Char`: characters (like in other languages)
- `Float`: 32-bit (single-precision) floating point
- `Double`: 64-bit (double-precision) floating point

[^2]: for more information, check out [the Wikipedia article](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic)!

Let's see a live code example of some primitive types:

```hs
-- example.hs
-- Defining an Int
nerds = 150 :: Int

-- Defining an Integer
googleplex = 10^100 :: Integer

-- Defining a Float
rootbeer = 3.14159 :: Float

-- Defining a Bool
carey_has_hair = False :: Bool
```

When we enter just the name of a variable, `ghci` will print its value for us:

```console
ghci> :load examples
[1 of 1] Compiling Main             ( examples.hs, interpreted )
Ok, one module loaded.
ghci> googleplex
10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

`:t` tells us the type of the expression passed to it. This is **really, really useful**: it's a helpful tool to learn more about the language and its type inference rules!

```hs
ghci> :t googleplex
googleplex :: Integer
```

### Operators
{: .no_toc }

Haskell supports operators you're used to: `+`, `-`, `*`, `/`, `^`, etc.

But, there's a few caveats!

This works well:

```console
ghci> :load examples
[1 of 1] Compiling Main             ( examples.hs, interpreted )
Ok, one module loaded.
ghci> 42 + nerds * 10
1542
```

In Haskell, the `/` division operator **does not work on Integers**! (why might that be - there's a good functional reason!)

```console
ghci> nerds / 10
<interactive>:3:7: error:
    • No instance for (Fractional Int) arising from a use of ‘/’
    • In the expression: nerds / 10
      In an equation for ‘it’: it = nerds / 10
```

You must use `` `div` `` instead.

```console
ghci> nerds
150
ghci> nerds `div` 10
15
```

However, `/` works on floats just fine:

```console
ghci> rootbeer
3.14159
ghci> rootbeer / 10
0.314159
```

To make numbers negative, you need to add a parentheses: this makes `-` a *unary operator*:

```console
ghci> nerds
150
ghci> nerds + (-1)
149
```

There is prefix notation for infix operators:

```console
ghci> rootbeer
3.14159
ghci> (+) rootbeer 1.0
4.14159
```

Boolean operators generally work like how you'd expect!

```console
ghci> nerds
150
ghci> carey_has_hair
False
ghci> nerds > 100 && not carey_has_hair
True
```

### Composite Types
{: .no_toc }

We'll dive into three composite types:

- a **`Tuple`** is a **fixed-size** collection of values; each value can be any type.
- a **`List`** is a sequence of values; **each value must be the same type**.
- a **`String`** is a list of characters!

Let's see some examples in action.

#### Tuples
{: .no_toc }

Tuples are great for associating values together, or having a function "return multiple values".

```hs
--- examples.hs
-- A Tuple for a student's score
grade = ("Sergey", 97)

-- A job posting: title, wage, hrs/wk
job = ("Chef", 35.5, 40)

-- A tuple with tuples, lists & ints
whoa = ((1,"two"),['a','b','c'], 17)
```

```console
ghci> :load examples
[1 of 1] Compiling Main             ( examples.hs, interpreted )
Ok, one module loaded.
ghci> grade
("Sergey",97)
ghci> fst grade
"Sergey"
ghci> snd grade
97
```

{: .warning }
`fst` and `snd` are only defined for tuples with two elements! They do not work for tuples with 3 or more elements - you'd use pattern matching instead (coming next lecture)!

What *is* the type of the Tuple? Let's ask ghci:

```console
ghci> :t grade
grade :: (String, Integer)
```

Interesting! Note that the type of the tuple encodes:

- **the number of elements in the tuple**
- **the types of each element in the tuple**

This means, for example, that you can't have a list of any two tuples - they must all be tuples of the same size and types!

{: .note }
And that's all for this lecture! Happy Haskelling :)

## Functions
{: .no_toc}
### Our First Haskell Function
{: .no_toc}
Haskell functions have three core components:

1. (optional): type information about the function's **parameters** and **return value**
2. the **function name** and **parameter** names
3. an **expression** that defines the function's behaviour


Here's a simple function in C++:

```cpp
String insult(String name, String smell) {
  String s = name + " smells like " +
             smell + " and doesn't floss.";
  return s;
}
```

And here's how we'd define the same function in Haskell:

```hs
insult :: String -> String -> String
insult name smell =
  name ++ " smells like " ++ smell ++ " and doesn't floss."
```

Some things to note:

- there's no explicit `return` since the right-hand side of a function *is* what it returns!
- spaces and indentation are used to define code blocks; no braces necessary (think Python)
- `String -> String -> String` may seem strange - it seems like there's no difference between parameters and return types. We'll talk about this later when we discuss **currying**.

{: .note }
Here's an example of how to write multiline functions within `ghci`:

```console
ghci> :{
ghci| insult name smell =
ghci|   name ++
ghci|   " smells like " ++
ghci|   smell ++
ghci|   " and doesn't floss."
ghci| :}
ghci> insult "carey" "cheese"
"carey smells like cheese and doesn't floss."
```

### Optional Type Declarations
{: .no_toc}
Type declaration is optional because of Haskell's type inference! How does it work?

```hs
insult :: String -> String -> String
insult name smell =
  name ++ " smells like " ++ smell ++ " and doesn't floss."
```

- we can only concatenate (`++`) lists, so we know `name` and `smell` are some type of list
- `" smells like "` and `" and doesn't floss."` are lists of characters (recall that `String = [Char]`)
- since we can only concatenate lists of the same type, `name` and `smell` are `String`s!!
- the concatenation of `String`s is `String`, and thus `insult` returns a `String`

So ... why would we even use type declarations? Unlike who we're insulting, it's **good hygiene** (and, best practice)!

- type declarations can provide better compiler warnings and errors
- type declarations make your code **easier to read**
  - arguably the most important trait in software
  - the person reading it may be you six months later!
- and, it can help you think about how to write the function itself!

So, we recommend that you annotate your functions - or at least, the user-facing/top-level ones!

### Main Functions and (avoiding) Monads
{: .no_toc}
{: .note }
This is intentionally hand-wavy. We won't be covering [monads](https://en.wikipedia.org/wiki/Monad_(functional_programming)) or impure FP at all in this class.

What we haven't talked about yet is the *main function* - in other words, how can we have Haskell run a function when we open a file?

(and, while we're at it, how can we get user input - which **is a side effect**?)

To do it, we'll create a function called `main`, and annotate it with the `IO()` "type" (really - [a system and monad](https://www.haskell.org/tutorial/io.html)). We then add this strange `do` block, and write what seems like imperative code:

```hs
-- insult.hs
insult :: String -> String -> String
insult name smell =
  name ++ " smells like " ++ smell ++ " and doesn't floss."

main::IO()
main = do
  putStr "What is your name? "
  name <- getLine
  putStrLn (insult name "unwashed dog")
```

We can then call our `main` function and get some input.

```console
ghci> :load insult
[1 of 1] Compiling Main             ( insult.hs, interpreted )
Ok, one module loaded.
ghci> main
What is your name? Carey
Carey smells like unwashed dog and doesn't floss.
```

Is this hand-wavy? Yes! IO (and monads, a way to deal with side effects) is not a focus for this class. If you're interested, we suggest:

- taking CS 231 (here are some [wonderful notes from Galen Wong](https://galenwong.github.io/blog/2021-08-29-cs231-notes/))
- read the Learn You a Haskell chapters on [IO](http://learnyouahaskell.com/input-and-output) and [Monads](http://learnyouahaskell.com/a-fistful-of-monads)

### More Examples!
{: .no_toc}
Let's see some other examples!

```hs
isBigger :: Double -> Double -> Bool
isBigger a b = a > b
```

```hs
average :: [Double] -> Double
average list =
  (sum list) / fromIntegral (length list)
```

{: .note }
Here, we use `fromIntegral` to cast the value of `length` to a type that we can divide with. In this case it maps from `Integral -> Double`. In general, we can use `fromIntegral` to map any `Integral` type (i.e. `Int` or `Integer`) to any `Num` type.

This last example showcases the power of Haskell's type system:

```hs
get_last_item :: [any_type] -> any_type
get_last_item lst =
  head (reverse lst)
```

This function works for a list of any type! So, *Haskell doesn't care what type it is*. The `any_type` is a **type variable**, which we can think of as a generic or template type. We'll refine this notion in a later lecture.

What's cool is that we didn't have to do anything special - it just works out of the box! **And, we didn't even need to tell Haskell that it's generic**: the type system figured it out by itself!

{: .note }
You may be thinking to yourself: is this polymorphism (or generic programming) at runtime, or is a copy of this function generated for each type? In Haskell, it's the former: there is one version of the language that "just works" for all types. This is similar to how Java does it. In other languages, like C++ or Rust, generics are compile-time: a copy of the function is made for each type that uses it (this is called [monomorphization](https://rustc-dev-guide.rust-lang.org/backend/monomorph.html)). We will be learning more about the different types of generic programming later on in the class!

### Practice Problem
{: .no_toc}
In class, we went over a brain teaser. Consider:

```hs
f :: Int -> Int
f x = x^2

g :: Int -> Int
g x = 3 * x
```

What do the following two lines do?

```hs
y = f g 2
z = f 5 * 10
```

Well, it turns out:

- `y = f g 2` is a **syntax error**!
  - Haskell evaluates functions **from left to right** (keyword: **left-associative**)
  - so, `f g 2` is the same as `((f g) 2)`, which doesn't make sense - since `f` doesn't operate on functions
- `f 5 * 10 = 250`
  - in Haskell, functions have higher precedence than operators, so they're always evaluated first
  - in other words, this is `(f 5) * 10`

```hs
-- f(g(x))
compute_f_of_g x = f (g x)

-- f(x * 10)
compute_f_of_x_times_ten x = f (x * 10)
```

{: .note }
If you're curious, the [Haskell spec defines operator precedence](https://www.haskell.org/onlinereport/decls.html#sect4.4.2) (and, how associativity works).

## Local Bindings
{: .no_toc}
This time, the majority of topics we cover are so called "syntactic sugar" in Haskell. They can make the programming easier. When we write code, sometime it is helpful to create "temporary variables" to simplify the code. In Haskell, it is achieved with `let` and `where` constructs.

### The `let` keyword
{: .no_toc}
Here's an example that demonstrates how to use `let`:

```hs
-- let.hs
get_nerd_status gpa study_hrs =
  let
      gpa_part = 1 / (4.01 - gpa)
      study_part = study_hrs * 10
      nerd_score = gpa_part + study_part
  in
      if nerd_score > 100 then
           "You are super nerdy!"
      else "You're a nerd poser."
```

The `let` construct consists of two parts:

- the first part is between the `let` and the `in`; here, you define one or more "bindings" to associate a name with an expression
  - for example, the third line of the function binds the name `gpa_part` to the expression `1 / (4.01 - gpa)`.
- the second part follows `in`; it contains an expression where the bindings are used.

The bindings are immutable variables - you can't change them!

### The `where` keyword
{: .no_toc}
The `where` keyword is similar; it just changes the syntax. With `where`, you first write the code, then specify the bindings after the `where` keyword. The equivalent example is:

```hs
-- let.hs
get_nerd_status gpa study_hrs =
  if nerd_score > 100
     then "You are super nerdy!"
     else "You're a nerd poser."
  where
      gpa_part = 1 / (4.01 - gpa)
      study_part = study_hrs * 10
      nerd_score = gpa_part + study_part
```

When would you use `let` and when do you use `where`? A nice rule of thumb is:

- when defining bindings (variables) for *a single expression* (e.g. the example here), either one is fine
- when defining bindings for use *across multiple expressions*, use `where`

{: .note }
The second case is mainly for *guards* and *case* statements. We haven't learned them yet but we will see them shortly. It's also more of an opinion - :)

### Nested functions with `let` or `where`
{: .no_toc}
We can also use `let` and `where` to define _nested functions_:

```hs
-- nestfunc.hs
whats_the_behavior_of name =
 if name == "Carey"
    then behaves_like name "twelve year-old"
    else behaves_like name "grown-up"
 where
   behaves_like n what =
      n ++ " behaves like a " ++ what ++ "!"
```

This is very helpful in breaking down bigger problems into small, reusable functions -- and helps us with naming!

Nested functions can use all of their enclosing function's bindings. So, we can make our example more concise by using `name`:

```hs
-- nestfunc.hs
-- Function to describe someone's behavior
whats_the_behavior_of name =
 if name == "Carey"
    then behaves_like "twelve year-old"
    else behaves_like "grown-up"
 where
   behaves_like what =
      name ++ " behaves like a " ++ what ++ "!"
```

### Lazy execution with `let` and `where`
{: .no_toc}
In the previous lecture, we touched on [Haskell's lazy evaluation for lists]({{site.baseurl}}/lectures/02/#creating-lists-ranges). This also applies to `let` and `where`: bindings are **not evaluated when they are defined; only when they are actually used**!

```hs
potentially_slow_func arg =
  let
      val1 = really_slow_function arg
      val2 = very_fast_function arg
      val3 = pretty_fast_function arg
  in
      if val3 > 100 then val1
                    else val2
```

Haskell's behavior when running the code above is:

- in the `let` block, Haskell binds `val1`, `val2`, and `val3` to the expressions - but *doesn't* evaluate them!
- when the function body runs, the `if` condition requires `val3`.
- so, Haskell evaluates the expression associated with `val3`, calling `pretty_fast_function`.
- then, *only one* of `val1` and `val2` will evaluate.
- so, if `pretty_fast_function` returns 20, then we go to the `else` branch. To compute `val2` `very_fast_function` is called. We will _never_ call `really_slow_function` in this case!

Note that Haskell doesn't have to re-evaluate the binding every time it's used - after doing it once, it can just cache it!

{: .note}
Laziness can be confusing with IO, since the sequence of IO actually matters!. Haskell's solution is to disallow IO (and other side-effects) to happen in normal code. Try to think about what will happen if the code below is valid in Haskell.

```hs
-- Which getLine will run first?!?!
laziness_example arg =
  let
      val1 = getLine
      val2 = getLine
  in
      if arg > 100 then val1 + val2
                   else val2 * val1
```

## Control Flow
{: .no_toc}
We've been using some control flow features already, but we haven't formalized them. Let's do that now!

### `if`-`then`-`else`
{: .no_toc}
The general syntax of the `if` statement (or more precisely, expression) is:

```hs
if <expression> then <expression>
                else <expression>
```

For example:

```hs
ageist_greeting age =
  if age > 30 then "Hey boomer!"
              else "Hey fam!"
```

A key insight is that **every `if` statement must have an accompanying `else`**. If there *wasn't* an `else`, and we didn't take the `if` - our function wouldn't return anything. That's not allowed!

{: .note}
Think about how `if`-`then`-`else` works with types - if the two branches had different types, we'd break some of Haskell's type rules. This formalizes the above argument - why is that?

### Guards
{: .no_toc}
Guards are a compact verison of the `if-then` expression (but really, are just syntactic sugar).

It allows us to write

```hs
if <condition> then <expr>
```

as

```hs
  | <condition> = <expr>
```

We can define a function as a series of one or more guards, like

```hs
somefunc param1 param2
  | <if-x-is-true> = <run-this>
  | <if-y-is-true> = <run-that>
  | <if-z-is-true> = <run-the-other>
  | otherwise = <run-this-otherwise>
```

A real example in Haskell:

```hs
major_guesser salary
  | salary > 150000 = "CS"
  | salary > 120000 = "EE"
  | salary < 30000 = "Any major at USC"
  | otherwise = "Probably Poli-sci"
```

Note:

- we don't need the equal sign after defining the function!
- each guard begins with the pipe character: `|`
- after that, the guard contains two parts:
  - the first part is a _boolean expression_; we want to evaluate to see if it is `True`.
  - the second part is the "payload" expression. It will return if the expression is `True`.
- Haskell evaluates all of the guards **from top to bottom**, and returns the payload of the first guard that is `True`.

{: .note}
In guards, `otherwise` acts like a wildcard. In fact, `otherwise` evaluates to `True` (try it in `ghci`)!

Let's take a look at some examples:

```hs
-- Factorial using if/then
fact n =
  if n <= 0 then 1
            else n * fact (n-1)

-- Factorial using guards
fact n
 | n == 0 = 1
 | otherwise = n * fact (n-1)

-- Find sm0l-est value in a list
sm0lest lst
 | lst == [] = error "empty list"
 | length lst == 1 = first
 | otherwise = min first (sm0lest rest)
 where
   first = head lst
   rest = tail lst
```

Note that in the last example, the values defined in the `where` clause are used by multiple guards.

We can use guard to implement quicksort in _eight_ lines of code (or less!). The following implementation uses `where`, guards, and list comprehensions:

```hs
qsort lst
 | lst == [] = []
 | otherwise = less_eq ++ [pivot] ++ greater
 where
    pivot = head lst
    rest_lst = tail lst
    less_eq = qsort [a | a <- rest_lst, a <= pivot]
    greater = qsort [a | a <- rest_lst, a > pivot]
```

Neat! -->

<!-- ## Appendix

Everything that follows are notes for the old version of this lecture from last year! It was originally written by the wonderful founding CS 131 TA Matt Wang.

## Pattern Matching
{: .no_toc}
Now, we'll switch gears to an even more powerful type of syntactic sugar called **pattern matching**. While it's not directly related to functions (or indeed, functional programming), it draws heavy inspiration from functional programming principles. In particular, it'll make list and tuple processing easy!

First, we'll explore parameter-based pattern matching, where:

- we define multiple versions of the same function
- each version of the function must have the same number and types of arguments
- when the function is called, Haskell checks each definition in-order from top to bottom, and calls the *first* function which has matching parameters

### Pattern Matching with Constants
{: .no_toc}
Let's start with matching constant values: we can place constants instead of the name of an expected argument.

```hs
-- genz.hs
genz_critic :: String -> String -> String
genz_critic "Carey" word =
  "Oh Carey... OK boomer!"
genz_critic name "lit" =
  name ++ ", you're a lit genz-er"
genz_critic name "wrekt" =
  name ++ ", git wrekt cheugy"
genz_critic name word =
  name ++ ", " ++ word ++ " is not gucci!"
```

In the function above, the first version has `"Carey"` in the place of the first argument. It only runs when user passes in `"Carey"` for the `name` argument. The same rule applies to other versions.

Here are two straightforward calls:

```console
ghci> :load genz
[1 of 1] Compiling Main             ( genz.hs, interpreted )
Ok, one module loaded.

ghci> genz_critic "Ed" "wrekt"
"Ed, git wrekt cheugy"

ghci> genz_critic "Paul" "unix"
"Paul, unix is not gucci!"
```

Here's a more tricky call, where it *could* match two different versions:

```
ghci> genz_critic "Carey" "lit"
"Oh Carey... OK boomer!
```

Here, we use the Carey version since it's defined *first*. Order matters!

Pattern matching with constants work with other types, like numbers or lists. There is no limit on how many constant patterns you can have:

```hs
factorial :: Integer -> Integer
factorial 0 = 1
factorial n = n * factorial (n - 1)

list_len :: [a] -> Int
list_len [] = 0
list_len lst = 1 + list_len (tail lst)

has_this_many_eyes :: Int -> String
has_this_many_eyes 1 = "Cyclops"
has_this_many_eyes 2 = "Pupper"
has_this_many_eyes 3 = "Sphenodon Punctatus"
has_this_many_eyes 4 = "A nerd with glasses"
has_this_many_eyes n = "A freak!"
```

### Pattern Matching with Tuples
{: .no_toc}
One powerful use-case of pattern matching is with tuples; since we know how many items are in the tuple, we can "match" each item!

```hs
-- tuples.hs

-- Original way
exp :: (Int,Int) -> Int
exp t = (fst t) ^ (snd t)

-- Version #2: With pattern matching
expv2 :: (Int,Int) -> Int
expv2 (a,b) = a ^ b

-- Version #3: Adding a constant parameter
expv3 :: (Int,Int) -> Int
expv3 (a,0) = 1
expv3 (a,b) = a ^ b
```

Here, the advantage of versions 2 and 3 are their conciseness: we didn't need to think about `fst` or `snd`!

Earlier, students asked how to extract values out of tuples that have more than 2 elements. With pattern matching, we can do that now!

```hs
-- tuples.hs
-- Extract the first value of a tuple
extract_1st :: (type1,type2,type3) -> type1
extract_1st (a,_,_) = a

-- Extract the second value of a tuple
extract_2nd :: (type1,type2,type3) -> type2
extract_2nd (_,b,_) = b

-- Extract the third value of a tuple
extract_3rd :: (type1,type2,type3) -> type3
extract_3rd (_,_,c) = c
```

```console
ghci> :load tuples
[1 of 1] Compiling Main             ( tuples.hs, interpreted )
ghci> extract_3rd ("q",5,1.7)
1.7
```

The underscore `_` is a wildcard: anything can match this item. In particular, note that we don't even care about its type!

New learners are often tempted to write code like:

```hs
tuple_eq (a, a) = True   -- WRONG!!!
tuple_eq _ = False
```

Haskell will complain about "conflicting definitions for `'a'`" on the first line. To compare the two values obtained from pattern matching, they have to be bound to different names and compared afterwards.

```hs
-- using guards
tuple_eq (a, b)
  | a == b = True
  | otherwise = False

-- or, use (==) directly
tuple_eq (a, b) = a == b
```

This differs from Prolog, a language we'll explore much later in the course.

{: .note }
You might've noticed that `_` is the exception to the above rule. Haskell has some special logic for `_` -- it doesn't perform an actual binding, so you can use multiple `_` in a tuple with no problems!

### Pattern Matching with Lists
{: .no_toc}
We can apply similar logic to pattern matching with lists. Here, we'll rely on the cons operator (`:`) to form our structure. Something of the form `x:xs` matches the head `x` (first item) and the tail `xs` (last item); we can think of it as matching the entire list of `x : xs`!

```hs
-- lists.hs
-- Extract the first item from a list
get_first :: [a] -> a
get_first (x:xs) = x

-- Extract all but the first item of a list
get_rest :: [a] -> [a]
get_rest (x:xs) = xs

-- Extract the second item from a list
get_second :: [a] -> a
get_second (x:y:xs) = y
```

`get_second` comes naturally from our view of pattern matching: we can think of this as "inverse cons" for `x : (y : xs)`, which means that `x` is the head, and `y` is the next item after the head.

```console
ghci> :load lists
[1 of 1] Compiling Main             ( lists.hs, interpreted )
ghci> get_first [5,10,15,20]
5

ghci> get_rest [5,10,15,20]
[10,15,20]

ghci> get_second [5,10,15,20]
10

ghci> get_rest [5,6]
[6]

ghci> get_rest [5]
[]
```

We can also use the constant `[]` in place of the tail. As you'd expect, this only matches patterns where the tail is *exactly* the empty list.

```hs
-- favs.hs
favorites :: [String] -> String
favorites [] = "You have no favorites."
favorites (x:[]) = "Your favorite is " ++ x
favorites (x:y:[]) = "You have two favorites: "
 ++ x ++ " and " ++ y
favorites ("chocolate":xs) =
  "You have many favs, but chocolate is #1!"
favorites (x:y:_) =
   "You have at least three favorites!"
```


```console
ghci> :load favs
[1 of 1] Compiling Main             ( favs.hs, interpreted )
ghci> favorites []
"You have no favorites."

ghci> favorites ["chocolate"]
"Your favorite is chocolate"

ghci> favorites ["mint","earwax"]
"You have two favorites: mint and earwax"

ghci> favorites ["chocolate","egg","dirt"]
"You have many favs, but chocolate is #1!"

ghci> favorites ["tea","coffee","carrot"]
"You have at least three favorites!"
```

There is another notation that uses brackets (`[]`) instead of parens (`()`). It's not used as commonly as the parenthetic version. The core difference here is that it only matches lists of an exact length; each item in the list is just an item (there are no tails, etc.).

```hs
-- lists.hs\
whats_in_your_list [10] =
    "Your list has just 1 value which is 10."
whats_in_your_list [a,b] = "Your list has 2 values: "
    ++ (show a) ++ " and " ++ (show b)
whats_in_your_list [9,_,c] =
    "Starts with 9 and its third item is:"  ++ (show c)
whats_in_your_list _ =
    "Your list had something else in it."
```

```console
ghci> :load lists
[1 of 1] Compiling Main             ( lists.hs, interpreted )
ghci> whats_in_your_list [10]
"Your list has just 1 value which is 10."

ghci> whats_in_your_list [10,20]
"Your list has 2 values: 10 and 20"

ghci> whats_in_your_list [9,10,20]
"Starts with 9 and its third item is 20."

ghci> whats_in_your_list [9,10,20,30]
"Your list had something else in it."
```

Broadly, pattern matching with lists makes our code more concise and readable! Consider our previous guard-based `sm0lest`:

```hs
sm0lest lst
 | lst == [] = error "empty list"
 | length lst == 1 = first
 | otherwise = min first (sm0lest rest)
 where
   first = head lst
   rest = tail lst
```

With pattern matching, the logic becomes more compact!

```hs
sm0lest [] = error "empty list"
sm0lest [x] = x
sm0lest (x:xs) = min x (sm0lest xs)
```

The same applies to the following reverse list example:

```hs
-- Reverse a list w/o pattern matching
rev lst
 | lst == [] = []
 | otherwise = (rev rest) ++ [first]
 where
  first = head lst
  rest  = tail lst

-- Reverse a list with pattern matching
rev [] = []
rev (x:xs) = (rev xs) ++ [x]
```

{: .note }
This `reverse` algorithm is not optimal; it has `O(N^2)` complexity (in the length of the list). Can you figure out an `O(N)` algorithm?

### Influence
{: .no_toc}
Pattern matching is mostly used in functional languages, but it's notably been adopted by Python and Rust.

Python added pattern matching *very recently* (in [version `3.10`](https://docs.python.org/3/whatsnew/3.10.html), released in 2021). Python's pattern matching works on tuples, lists, dictionaries, and more! Python is a *dynamically* typed language, so match cases can have different types.

```python
def func(val):  # Python pattern matching example
  match val:
    case (0, x):        # match tuple w/first elem zero
        print(f"Tuple of 0 and {x}")
    case ['a', x, 'c']: # match list w/3 elems: 'a', ??, 'c'
        print(f"List of a, {x}, and c")
    case [1, 2, *rest]: # seq of: 1, 2, ... other elements
        print(f'A list with 1, 2, and *{rest}')
    case {'foo': bar}:  # match dict w/key 'foo'
        print(f"foo maps to {bar} ")
    case _:
        print('no match')
```

Rust has almost always had pattern matching. It's statically typed, so this example looks quite like Haskell. 

```rust
match vtree_type {
  VTreeType::LeftLinear => VTree::left_linear(vars),
  VTreeType::RightLinear => VTree::right_linear(vars),
  VTreeType::EvenSplit(num) => VTree::even_split(vars, num),
  VTreeType::FromDTree => {
    let dtree = DTree::from_cnf(&cnf, &VarOrder::linear_order(cnf.num_vars()));
    VTree::from_dtree(&dtree).unwrap()
  }
}
``` -->

<!-- Partial function application is deeply related to currying. In fact, when we used `((mult3 2) 3) 5` earlier, we are implicitly doing partial function application. 

{: .note}
Another way to think of partial function application is as anti-currying, or the inverse operation of currying! -->


<!-- But let's see more examples:

```console
ghci> mult3 x y z = x*y*z
ghci> partial = mult3 2
ghci> partial 3 5
30
ghci> multby10_20 = mult3 10 20
ghci> multby10_20 3
```

Here, we are creating two specialized functions `partial y z = 2*y*z` and `multby10_20 z = 10*20*z` with partial application.

Partial function applications can be useful when combined with pre-existing functions and operators. Let's see the following examples:

```console
ghci> add5 = (+) 5
ghci> add5 100
105
```

All of Haskell's operators are just functions! So the `+` sign is basically a function of two arguments.

{: .note }
Surrounding an infix operator with a pair of parentheses converts it to the prefix form which gives you identical behavior to a normal function.

Haskell also allows you to apply partial application to either operand of an infix operator, as shown below. The function produced by the first line is equivalent to `\x -> x / 10`, while the second line produces `\x -> 10 / x`. You can see the behavior by reading the result returned by the `map` function.

```console
ghci> map (/ 10) [100,200,300]
[10.0,20.0,30.0]
ghci> map (10 /) [100,200,300]
[0.1,5.0e-2,3.333333333333333e-2]
```

Here are more examples:

```console
ghci> filter (>= 6) [2,4,6,8,10,3]
[6,8,10]
ghci> map (++ " is LIT!") ["CS32","CS131"]
["CS32 is LIT!","CS131 is LIT!"]
ghci> filter (`elem` ['A'..'Z']) "Not Every Resistor Drives current"
"NERD"
```

In the last example, the pair of backticks turns a function with two arguments into an infix operator so that you can supply the second argument with partial application.

We can even use partial function application to define a partially-specified mapper function:

```console
ghci> cuber = map (\x -> x^3)
ghci> cuber [2,3,5]
[8,27,125]
```

As you can see, with partial function applications often we don't have to define a full function or lambda to do complex processing! -->

