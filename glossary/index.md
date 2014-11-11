---
layout: default
title:  "Glossary"
---
# Glossary

## Terms

### Programming System
In contrast to the more modern term _programming language_ that emphasises the 
language - in particular its syntax - as the essential aspect of programming 
the unfashionable point of view of a _programming system_ emphasises an 
integrated understanding of the overall process of making software that 
depends on and is influence by many aspects one of which is the language. 

In a programming system other aspects of the overall programming activity 
influence the design of the language in such a way that it supports these 
aspects well in the intended way of working. The language is designed with the 
system in mind and vice versa the system to fit the language.

The considerations of a programming system are diverse but typically include
all stages of software production process. For example how software can be 
designed, reasoned about, modularised, edited, analysed, tested, deployed or 
maintained.

Another understanding of a programming system (that is not meant in this context) 
is focused on the capabilities of a language to be used as a basis to build a 
customised language used to write a particular program. 
While related to the idea of 
[domain specific languages](http://en.wikipedia.org/wiki/Domain-specific_language)
this understanding is more focused on the generality and simplicity of the
core language (such as Lisps) to be able to use it to build a problem specific
solution including its _language_ (the set of _primitives_ and their 
semantics) the program than will build upon. Its syntax often is a subset 
of the originating language (especially for Lisps).

### Programming Formalism
The term of a _programming formalism_ is proposed for programming languages or 
systems that embody an understanding of programming and of a program as a formal, 
almost mathematical way of expressing intent. These are to be distinguished from 
those _programming languages_ that embody programming as expressing intent 
through (ideally) natural language.

A formal nature should not be confused with mathematical notations or a syntax
that is hard to read. A formalism aims for readability like languages do. 


### Local Reasoning
_Local reasoning_ should be understood as a form of 
[formal](http://en.wikibooks.org/wiki/Effective_Reasoning/Informal_and_Formal_Reasoning) 
or [deductive reasoning](http://en.wikipedia.org/wiki/Deductive_reasoning) 
that is possible to do in a small, self-contained local context. 
This implies that everything that has to be considered is directly 
(locally) represented.
Thereby correct and complete reasoning becomes possible without taking the
_outside world_ into account. This does not imply that the local context isn't
connected to the outside world but that it is sufficient to consider 
the outside world only through the connection ends visible in the local 
context.

The process of reasoning is simplified by making all aspects of a context 
explicit and in its effects locally confined. Often this also leads to a 
reduction of the sheer number of significant aspects to reason about.

As such local reasoning is related to the idea of 
[referential transparency](http://en.wikipedia.org/wiki/Referential_transparency_(computer_science))
that allows to deduce the absence of side effects on the outside world.
Whatever happens within the referential transparent component can be 
simplified to the resulting value it produces.


### Operative Modularisation
All programming systems provide a concept of modularisation of some kind.
These are by far not equally powerful. The only common understanding of
_modularisation_ or _modular software_ is such that software can be 
divided into modules and re-combined from such into another program.
As this is basically true for all programming systems it is of little 
significance as a property of a system. It does not express the degree
of independence of the program modules.

The term _operative modularisation_ in contrast explicitly demands a
degree of modularisation independence to the extent that a program is 
_operational_ independent of its module structure. 
Expandability has to be given in such a way that extending modules can 
effectively extend the concepts of an extended module without the need 
to re-build (compile) it. 
The more concepts are extensible by pure addition done external to the 
extended module the more modular is such a modularisation. 
It becomes operative when any addition (intended to be possible in the system) 
that could be made within a module can likewise be done on the outside. 

In other words: Program semantics and the expressiveness of the 
language are independent of the way a program is split into code units. 

To give an example: Classes are not an operative modularisation concept 
as it is -- among other things -- impossible to add methods outside the 
file (code unit) that contains the class. 
Class based inheritance is also not an operative modularisation concept 
as the reference to the extended class can only be defined by the extending 
class and therefore within the defining _module_. 
In fact almost all programming systems - be it OOP or FP systems - have none or
just very few concepts of operative modularisation what is e.g. reflected by 
the complications they undergo to solve the 
[expression problem](http://en.wikipedia.org/wiki/Expression_problem). 
