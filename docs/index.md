---
layout: default
title:  "Bruno Programming System"
---

# 0. The **Bruno** Programming System

## Data
The foundation is data. There a simple values (primitives) given as `dimension`s 
and `unit`s. While dimensions represent conceptually different kinds of values
a family of units of the same dimension model conceptual similar values of 
different representation.

Each dimension (or unit) has a finite value range. Newly defined dimensions are 
derived from existing ones all originating from language level dimensions `Num`
, `Char` and `Bit`.

		dimension Length :: Int '0 ..

A `Length` value is a numeric value based on `Int` having a range that starts 
at zero and ends at the same limit as the `Int` type.

Different representations of a `Lenth` could be `Meters` or `Miles`.

		unit Meters [m] :: Length
		unit Miles [mil] :: Length

Units often have a unit symbol that is used to distinguish value literals.

		'1m
		'1mil

The type `Meters` is different from `Miles` in the same way one meter is 
different from one mile. They are both measures of `Length`; about the same 
thing but in different actual representations. 


## Specialisation and Generalisation
White types are derived from more general ones they do not form a hierarchy of 
subtypes and respectively supertypes as found in OOP. A derived type is said to 
be a _specialisation_ of the more general type it is derived from. 
As such it a constraint and known property that it is safe to generalise any
more special type to one of its generalisations. This is a type _conversion_
where the value becomes of another type. The type of a value is not attached to
a instance but given through the statics of the declarations.


		dimension Count :: Int '0..

Too count `Apples` and `Pears` without confusion them with each other those are
declared as specialised `dimension`s of a `Count`.

		dimension Apples [A] :: Count
		dimension Pears [P] :: Count

A arbitrary function working with `Count`s like the average is given as following:

		fn avg :: (Count one -> Count other -> Count) = one + other / '2

As both `Apples` and `Pears` are generalisable to a `Count` the `avg` function
is available for the 10 `Apples` and can be called with 20 `Pears` (both constructed 
from constants) - the result is a `Count`.

		Count avg = '10A avg '20P
		
Once again: before calling `avg` the `Apples` are converted to the `Count` value
`one`, the `Pears` to the `Count` value `other`, the result is of course also a
`Count` as given from the function declaration. This _conversion_ is almost
always a no-op as the more special and the more general type share the same
actual runtime representation of the value. This is known at compile time so the
no-op doesn't even have to appear in a compilation target.

While specialisation-generalisation allows to share common functionality 
(declared on more general types) it does not confuse with the consideration of 
polymorphic types inevitably dealt with in OOP.



