---
layout: default
title:  "Glossary"
---
# Glossary

## Terms

### Programming System
In contrast to the term _programming language_ that emphasises the language as
the essential aspect of programming the (almost antique) point of view of a 
_programming system_ emphasises an integrated understanding of the overall 
process of making software that depends on and is influence by many aspects 
one of which is the language. 

In a programming system other aspects of the overall programming activity 
influence the design of the language in such a way that it supports these 
aspects well. The language is designed with a system in mind and the system
vice versa to fit the language.

In the bruno programming system the [formalism](#formalism) has been designed
so that its properties allow to develop software in a particular way. This 
applies from program design up to deployment, maintenance, testing. The
concerns span...

### Local Reasoning
_Local reasoning_ should be understood as a form of 
[formal](http://en.wikibooks.org/wiki/Effective_Reasoning/Informal_and_Formal_Reasoning) 
or [deductive reasoning](http://en.wikipedia.org/wiki/Deductive_reasoning) 
that is possible to do in a narrow local context. 
This implies that everything that has to be considered is directly (locally) 
present or affected.
Thereby correct and complete reasoning becomes possible without taking the
_outside world_ into account. This doesn't imply that the local context is
not conneced to the outside world but that it is sufficient to consider 
the outside worlds only through the connection ends visible in the local 
context.

The process of reasoning is simplified by making all aspects of a context 
explicit and in its effects locally confined. This often also leads to a 
reduction of the sheer number of significant aspects in consideration.

### Operative Modularisation
All programming systems provide a concept of modularisation of some kind.
These are by far not equally powerful. The only common understanding of
_modularisation_ or _modular software_ is such that software can be 
devided into modules and re-combined from such into another program.
As this is basically true for all programming systems it is of little 
significance as a property of a system. It does not express the degree
of independence of the program modules.

The term _operative modularisation_ in contrast explicitly demands a
degree of module independence to the extent that modules can be _operated_
independently. Expandability has to be given in such a way that extending
modules can effectivly extend the concepts of an extended module without
the need of re-building it. The more concepts are extendable by pure 
addition external to the extended module the more modular is such a
modularisation. It becomes operative when any additon (intended to be 
possible) that  could be made within a module can likewise be done on 
the outside.
