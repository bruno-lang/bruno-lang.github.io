---
layout: bw
title:  "bruno programming system"
author: Jan Bernitt
class: nutshell
---

# bruno

## Philosophy

### Types and Values
Types express eternal laws. Truths about all and any of the values of a 
certain quality. The type is just a name for the set of all values having
certain properties.
Provided the properties in question a value can be a member of many types. 
But it doesn't have to. 
If the program explicitly says so, it is, if not, than it isn't.
And this can differ from place to place.
Thus, types are not attached to values, they are ways to understand them.
As such a program always has to communicate its understanding. 
The assumptions it is gonna rely upon.

### Type Safety


A program may state that a value of type `X` now should be of type `Y` as long 
as `X` has all the qualities of `Y`. 
This is clearly safe. 
No assumption is made that ins't know to be true by means of `X`.

A program may also state a path **alternative** taken in case a value provably 
has all qualities of a type `Z` as long as `Z` has no properties that are 
knowingly in conflict with the source type `X`.
This is safe too. Only values that qualify for the assumptions made by a path
will take that path. 

But a program can not force a type upon a value as types are only ways to 
understand what is actually true due to the nature of the value.
As a consequence all program that can be expressed are also safe.



## Declarations 
Declarations, like types, expresses what is known. Truths.
A programs is a sets of declarations of the sorts:

`data`
: describes the _shape_ and _quality_ of member values.

`io`
: relates a serialized _shape_ with another `data` type.

`ratio`
: declares a _proportional relationship_ between two values.

`fn`
: describes a particular _relation_ between input and output values. 

`sub`
: like `fn`, but the _relation_ must no be recursive

`op`
: captures the semantics of a input-output- _transtion_.

`concept`
: captures the semantics of a set of related _transitions_.

`family`
: describes _properties_ of values. Any type having a family's properties is a 
member of that family and can be substituted for it.

`when`
: describes a sequence of steps to take when faced with a particular situation 
(a certain state) in order to transit to the next state.

`on`
: describes the (immediate) reaction to a situation (triggered externally) and
the caused next state.


### Declarative Programming
It is the primary quality of of declarions that they allow to describe any 
program purely based on what is known. Universal truths. 
Correctness of a declaration never depends on changing or invisible parts.
We can solely reason about the correctness of each declaration on a purely 
logical level. This support high modularization and sharing.
All in all the declarations foster efficient and understandable programs. 

The second most important quality its their concise, flexible but very natural
syntax that allows to express ideas close to the problem domain. Yet there is
no burden to learn multiple languages.


_to be continued..._

<!--
bla bla

- everything is declarative -> nothing ever changes in one of the declarations


## ???
- errors and other cases here.

-->
