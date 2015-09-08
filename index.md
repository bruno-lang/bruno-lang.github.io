---
layout: bw
title:  "bruno programming system"
author: Jan Bernitt
class: nutshell
---

# bruno

## Philosophy

### Types and Values
A type is a set of all the values with certain properties. What is known about
all and any of the values of that type.
Provided the properties in question a value can be a member of many types. 
But it doesn't have to. 
If the program explicitly says so in one case, than a value is a member,
when not in another case, it is not.
Thus, types are not attached to values, they are ways to understand them.


### Type Safety
Types express eternal laws. Truths about all and any of the values of a 
certain quality. 

A program may state that a value of type `A` now should be of type `B` as long as 
`B` has no property that `A` has not. Then, what is known about `A` is sufficient to 
conclude that the value can be seen as a member of type B as well. 
This is always safe. 

A program may also state that a value of type `A` might be of type `B` as long as 
`B` has no contradicting properties compared with `A`. Contradiction means it is
statically clear that a value of type `A` cannot possibly be of type B as well.
So if `A` does not have all properties of `B` but also doesn't have contradicting
properties a program may provide **an alternative** in case the value has all 
properties of type `B`. 
This is always safe too.

But a program can not force a type upon a value as types are only ways to 
understand what is actually true due to the nature of the value.
As a consequence all program that can be expressed are also safe!



## Declarations
Programs are sets of declarations. 
Declarations expresses what is known. Truths. They are types of different kinds.

All programs can be described through a set of the following 10 kinds of 
declarations:

- `data`: describes the _shape_ or _quality_ of a set of values.

- `io`: describes the the sequential form (string form) of a set of values 
in terms of their sequential _shape_.

- `ratio`: declares a _proportional relationship_ between two values.

- `fn`: describes a particular _relationship_ between input and output values. 

- `sub`: like `fn` but must no be recursive

- `op`: describes a _contract_ for an algorithm.
What goes in and what comes out but not how it is done. 
It implies however a certain meaning.  

- `concept`: describes a _contract_ for a set of algorithms that are available 
on the same set of values. 

- `family`: describes all kinds of _properties_ of values. 
A type (what is a set of values) having a family's properties is a member of 
that family. Therefore it can be substituted everywhere the family is used.

- `when`: describes a sequence of steps to take when faced
with a particular situation (a certain state) in order to transit to the next 
state. 

- `on`: asks for an exclusive thread of execution whereas `when` is
satisfied with cooperative task scheduling. 


_to be continued..._

<!--
bla bla

- everything is declarative -> nothing ever changes in one of the declarations


## ???
- errors and other cases here.

-->
