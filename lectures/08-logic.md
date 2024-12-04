---
title: "08. Logic Palooza "
author: Ashwin Ranade, Einar Balan
layout: lecture
parent: Lecture Notes
---

{: .note}
This lecture note covers the Logic Palooza deck.

## Table of Contents
{: .no_toc }

{:toc}
- dummy item

##  Prolog and Logic Programming

**Logic programming** is a paradigm where we express programs as a set of facts (relationships which are held to be true), and a set of rules (i.e. if A is true, then B is true).

A program/user then issues queries against this these facts and axioms:

Logic programming is declarative – programs specify "what" they want to compute, not "how" to do so.

Examples:

**Facts:**
- Martha is Andrea's parent
- Andrea is Carey's parent

**Rules:**
If X is the parent of Q, and Q is the parent of Y, then X is the grandparent of Y

**Queries:**
- Is Martha the grandparent of Carey?
- Who is the grandparent of Carey?

### Introduction to Prolog

Prolog is used for theorem-proving (type checking is one such use case; it's in the Java compiler!) and has also been used to implement natural language processing and expert systems.

Prolog mostly leans on *declarative* programming: you specify what you want, but now *how* you should get there; Prolog is responsible for figuring that out!

When given a query, Prolog tries to find a chain of connections between the query and the specified facts and rules that lead to an answer.

Every Prolog program is comprised of facts about the world, and a bunch of rules that can be used to discover new facts.

### Prolog Facts

A "fact" is a predicate expression that declares either...
- An attribute about some thing (aka an atom); you can think of this as being "always true"
  - `outgoing(ren)`
- A relationship between two or more atoms or numbers:
  - `parent(alice, bob)`
  - `age(ren, 80)`
  - `teaches(carey, course(cs, 131))`

Implicitly, we're defining some terms:

- an "atom" is a "thing". In the above examples, this includes `ren`, `alice`, and `bob`. They **must be lowercase**.
- a "functor" is like a function (or relationship). In the above examples, this includes `outgoing`, `parent`, and `age`. They **must be lowercase**.

If a fact isn't explicitly stated, it is false. For example, in the above case, since we didn't explicitly state it, `alice` is not outgoing.

### Prolog Rules

A rule lets us define a new fact in terms of one or more existing facts or rules.

Each rule has two parts...
- A "head" which defines what the rule is trying to determine
- A "body" which specifies the conditions (aka subgoals) that allow us to conclude the head is true

Rules can be defined with atoms, numbers, or *variables*. Variables like `X` and `Y` are like placeholders which Prolog will try to fill in as it tries to answer user queries. **Variables must always be capitalized**.


```prolog
% Facts:
outgoing(ren).
silly(ren).
parent(alice, bob).
age(ren, 80).
parent(bob, carol).

% Rules:
comedian(P) :- silly(P), outgoing(P).
grandparent(X, Y) :- parent(X,Q), parent(Q,Y).
old_comedian(C) :- comedian(C),
age(C, A), A > 60.
```

{: .note }

Rules and facts might seem very different, but some would argue that they aren't! You could think of a fact as a rule that has no body. Then, it is [vacuously true](https://en.wikipedia.org/wiki/Vacuous_truth) (and thus, always true).

#### Recursive Rules

Rules can also be recursive and can have multiple parts!

Similar to recursion and pattern matching in Haskell, Prolog processes rules from the top to the bottom. So, the base case(s) should always go first!

```prolog
% Recursive rules:
ancestor(X, Z) :- parent(X,Z).
ancestor(X, Z) :- parent(X,Y), ancestor(Y,Z).
```

Explanation of the above code block:

X is the ancestor of Z if:
- X is Z's parent
- OR if X is some person Y's parent AND Y is an ancestor of person Z

{: .note }
Prolog uses something called the "closed-world assumption" (CWA). Basically, this means that only things that can be proven true by the program are true; *everything else* is false.

#### Negation in Rules

Rules can also use negation – but be careful!

In Prolog, `not(something)` works as follows:

1. Prolog tries to prove `something` is true using all of the program's facts and rules
2. If `something` can't be proven as true, then `not(something)` is found to be true.


```prolog
% Rules with negation:
serious(X) :- not(silly(X)).
```

For example, the query `serious(alice)` is true, since the closed-world assumption says that `silly(alice)` is false.

## Prolog Queries
We can create queries to answer true/false questions:
- A query can match a simple fact...
- Or it can execute a rule.

Given this set of facts and rules (also known as a knowledge base):
```
% Facts:
outgoing(alice).
parent(alice, bob).			 
parent(alice, brenda).
parent(brenda, caitlin).	
parent(caitlin, ned). 		 

% Rules:
grandparent(X, Y) :- parent(X,Q), parent(Q, Y). 
```

We can execute these queries. Prolog will then search through the knowledge base to find the answers.

```prolog
?- outgoing(alice)
true
?- grandparent(brenda, ned)
true
```

The first query essentially asks, "Is Alice outgoing?" One of our facts (the first in our knowledge base) says she is, so the result is `true`. The second query asks, "Is Brenda the grandparent of Ned?" Based on our `grandparent` rule, Prolog determines that she is and the result is true.

This is cool, but Prolog can do more than just answer yes or no questions! We can also retrieve certain information, like "who are Alice's parents?"
- The query will find ALL possible matches.
- The query can specify multiple unknowns.
- And Prolog will find ALL consistent combinations of answers!

```prolog
?- parent(alice, Who)
Who = bob
Who = brenda


?- grandparent(A, B)
A = alice, B = caitlin
A = brenda, B = ned
```

The variables `Who`, `A`, and `B` indicate an unknown that we want Prolog to fill in. Our first query is essentially asking "Who are Alice's parents?" Based on our knowledge base, Prolog deteremines that Bob and Brenda are her parents. Our second query asks to find all grandparents and their grandchildren.

{: .note}
A quick reminder on terminology: `alice` is an atom, `Who` is a variable, and `parent(alice, Who) is a query.



## Resolution: How Prolog Answers Queries

Example facts/rules:

```prolog
parent (nel, ted).
parent (ann, bob).
parent (ann, bri).
parent (bri, cas).
gparent( X ,  Y )  :-
  parent(X,Q), parent(Q,Y).
```

Example query:

```prolog
gparent(ann, W)
```

Algorithm:
1. Add our query to a goal stack
2. Pop the top goal in our goal stack and match it with each item in our database, from top to bottom
3. If we find a match, extract the variable mappings, and create a new map with the old+new mappings
4. Create a new goal stack, copying over existing, unprocessed goals and adding new subgoals
5. Recursively repeat the process on the new goal stack and the new mappings

So initially, our goal stack is simply `gparent(ann,W)`, and our mappings are `{}`.
Then, after matching the last rule, our goal stack becomes:


```
parent(X,Q)
parent(Q,Y)
```
And our mappings are `{X->ann,W<->Y}`

Then, pop the top rule off our goal stack. We have a potential match with `parent (nel, ted)`. So our goal stack becomes:

```
parent(Q,Y)
```
With the mappings: `{X->ann,W<->Y, Q->bob}`

We pop the top rule off the stack again. There's no rule that matches `parent(bob,Y)`, which means the set of mappings we discovered were not valid.

So we'll back-track one level up and continue searching for alternative mappings.

If our goal stack is empty, we output variables from our mapping that were explicitly queried; in this case, our final mapping is:
`{ X ->ann, W <-> Y <-> cas, Q ->bri }`

So we return `cas`, since `W=cas`.

## Unification

As we saw, during the Resolution process, Prolog repeatedly Compares the current goal with each fact/rule to see if they are a match.

If a goal and a fact/rule match, Prolog extracts mappings between variables and atoms (e.g., X->bob).

Together, these two steps (comparison + mapping) between a single goal and a single fact/rule are called **unification**.

Unification pseudocode: 
```
def unify(goal, fact_or_rule, existing_mappings):
   if the goal with the existing mappings matches the current fact/rule:
      mappings = extract variable mappings between goal and fact/rule
      return (True, mappings)	 # We did unify! Return discovered mappings
   otherwise:
      return (False, {})   	# We couldn't unify! So no mappings found
```
- The unification algorithm runs on one goal at a time.
- Unification runs on a single fact or rule at a time.

### Matching a Goal and a Fact/Rule

So how do we determine if a goal with the current mappings matches a specific fact/rule?
1. apply all current mappings to the goal
2. treat both the mapped goal and head of the fact/rule as trees 
3. Compare each node of the goal tree with the corresponding node in the fact/rule tree
  
Use the following comparison rules: 
- If both nodes are functors, then make sure the functors are the same and have the same number of children.
- If both nodes hold atoms, make sure the atoms are the same.
- If a goal node holds an unmapped variable it will match ANY item in the corresponding node of the fact/rule.
- If a fact/rule node holds an unmapped variable it will match ANY item in the corresponding node of the goal.

Note: The animations do a really good job of explaining this -- they are slides 20-21 of the Prolog lecture. I'll do my best, but it's hard to transcribe into text. 

### Unification: Do They Match
Follow the rules above. Many examples are on ~slides 20-30 of the Prolog presentation; they're hard to transcribe into text. 

Here's one example: 

Assuming we have the mapping `{What->ucla}`, and we are trying to match (1) `likes(gene, What)` with (2) `likes(Y,X)`. 
These *DO MATCH*. 
- we replace `What` with `ucla`, so the children of the `likes` node for (1) becomes `gene` and `ucla`. 
- This matches the children nodes for (2), since they both hold unmapped variables [`Y` and `X` respectively]

### Unification: Extracting Variable Mappings
Once we know that a goal matches a fact/rule, we need to extract the variable mappings.

Approach: 
- Iterate through each corresponding pair of nodes in both trees.
- Any time you find an unmapped Variable in either tree that maps to an atom/number in the other tree, create a mapping between the variable and the atom.
- Any time you find an unmapped Variable in either tree that maps to an unmapped Variable in the other tree, create a bidirectional mapping between the variables.

### Unification: Summary
Unification is the process of: 
- comparing a single goal with a single fact/rule, given the current set of mappings
- determining if the goal and the fact/rule match
- If so, extracting all new mappings between variables and atoms on either side.

Unification is used within the broader Resolution algorithm that we traced through.

## Resolution Continued
Here's a simplified version of the Resolution algorithm, showing where Unification fits in.


```
def resolution(database, goals, cur_mappings):
  if there are no goals left:
    tell the user we found a solution and output our discovered mappings!
    return
  for each fact/rule z in the database:
    success, new_mappings = unify(goals[0], fact_or_rule, cur_mappings)
    if success:
      tmp_mappings = cur_mappings + new_mappings
      tmp_goals = sub_goals(z) + goals[1:]
      resolution(database, tmp_goals, tmp_mappings)    # recursion
  # if we get here, we didn't find a match... BACKTRACK and keep trying!
```

The Resolution algorithm is initially called with the user's query as its only goal, and with no initial mappings.
```
# Determine if ann the grandparent of cas?
resolution(database, "gparent(ann, cas)", { })
```

## Prolog Lists
Lists in Prolog are just like lists in Haskell or Python.

Lists can contain numbers, atoms, or other lists.


```
[ ]
[silly, goofy, gleeful]
[1, 2, [dog, cat], 3.5]
```

- Prolog uses a combination of pattern matching (like Haskell) and unification to process lists.
- List processing is also done with facts and rules, just like other inference tasks.

Firstly, here's a Prolog fact with **variables** inside of atoms. 


```
is_the_same(X,X)

is_the_same(lit,lit) --> returns true
is_the_same(ucla,usc) --> returns false
```

Now, time for some Prolog lists!
### Example 1
```
is_head_item(X,[X | XS]).
```
This [X | XS] syntax is Prolog's equivalent of pattern matching, 
just like (x:xs) in Haskell.

X matches the first item of the list.
XS matches the rest of the list.


```
is_head_item(lit, [lit, dank, snack]) --> returns true
is_head_item(drip, [lit,dank,snack]) --> returns false
```

**Explanation:** Prolog is unifying from left-to-right and mapping each variable.
- Once it extracts a mapping, it only "unifies" the query if later uses of the mapping are consistent with the first.

We can also do: 
### Example 2
```
is_second_item(Y, [X, Y | XS]).
```
 [X, Y | XS] equivalent of (x:y:xs) in Haskell.

X matches the first item of the list.
Y matches the second item of the list.
XS matches the rest of the list.

```
is_head_item(dank, [lit, dank, snack]) --> returns true
is_head_item(lit, [lit,dank,snack]) --> returns false
```

### Example 3: Checking if a list contains a value

```
is_member(X,[X|Tail]). 
is_member(Y,[Head|Tail]) :- is_member(Y,Tail). 
```

If we query `is_member(dank,[lit,dank,snack])`: 
- we first try and match the first fact, but it's false
- so we match on the second rule, which turns out to unify, so we return True!

### Example 4: Deleting an atom from a list

The general form will be:
delete(ItemToDelete, ListToDeleteFrom, ResultingList)

A query could look like this:
delete(carey, [paul, carey, david], X)  -->  X = [paul, david] 


```
delete(Item, [Item | Tail], Tail).  
delete(Item_, [Head_ | Tail_], [Head_ | FinalTail])  :-  delete(Item_, Tail_, FinalTail).
```

The first line is like we saw earlier – it handles the base case.
- The base case handles the situation where the first item in the list is the one we want to delete, e.g.:
delete(carey, [carey, david, paul], X)

The second line handles the case where the item we want to delete ISN'T the first item, e.g.:
delete(david, [carey, david, paul], X)
- Our rule uses pattern matching to break up the input list into the Head item and all Tail items.
- the subgoal: "Use delete to remove the Item from amongst the Tail items; FinalTail refers to the resulting tail"
- In this case, we construct our output list by concatenating the Head item from our original list with the tail of the list with the Item removed from it.

## Prolog List Processing: Built-in Facts and Rules

append(X,Y,Z)
```
Determines if list X concatenated with list Y is equal to list Z
append([1,2],[3,4],[1,2,3,4]) yields True
append([1,2],X,[1,2,3,4]) yields X -> [3,4]
```

sort(X,Y)

```
Determines if the elements in Y are the same elements of X, but in sorted order
sort([4,3,1], [1,3,4]) yields True
sort([4,3,1],X) yields X -> [1,3,4]
```

permutation(X,Y)

```
Determines if the elements in Y are the same elements of X, but in a different ordering

permutation([4,3,1], [3,1,4]) yields True
```

reverse(X,Y)

```
Determines if list X is the reverse of list Y

reverse([1,2,3],[3,2,1]) yields True
reverse([1,2,3],X) yields X -> [3,2,1]
```

member(X,Y)

```
Determines if X is a member of the list Y

member(6, [1,6,4]) yields True
member(X,[1,6,4]) yields X -> 1, X -> 6, and X -> 4
```

sum_list(X,Y)

```
Determines if the sum of all elements in X add up to Y
sum_list([4,3,1], 8) yields True
sum_list([4,3,1], Q) yields Q -> 8
```

## Prolog List Syntax is Syntactic Sugar For Functors and Atoms!

Notice that when you remove the syntactic sugar, list processing is implemented just like any other Prolog fact or rule!
- And while we didn't see an example in the slides, the empty list [ ] can similarly be written as the atom nil.

```
The  | operator (as in [X | Tail]) can be replaced with a fact named "cons" that takes two arguments.
- [X|Tail] --> cons(X,Tail)`
```

original facts/rules: 
```
is_member(X, [X | Tail]).
is_member(Y, [Head | Tail]) :- is_member(Y, Tail).
```

rewritten: 
```
is_member(X, cons(X,  Tail)).
is_member(Y, cons(Head, Tail)) :- is_member(Y, Tail).
```
