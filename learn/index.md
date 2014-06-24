---
layout: default
title:  "A cursory view on the Language"
---

# A cursory view on the Language

### Abstract
In a rather oversimplified digest `bruno` as a language could be described as 

> _declarative programming with data and extension functions_

While there is a strong influence of FP apparent, thinking in language paradigms
is quite misleading as the language often has its own twist on properties without 
being a multi-paradigm language. It is the best to look at it with a fresh and 
open mind.

## Design Goals

## Data
Values are values, hence immutable.

### Simple Values
Two types of simple values, `dimension`s and `unit`s. 

		dimension Time [T] :: Int '0..

`Time` is a specialisation of a integral number `>= 0` measured in `T`. While the
dimension `Time` describes a kind of value it is no meaningful value in itself.
Therefore some `unit`s are introduced:

		unit Minutes [min] :: Time
		unit Seconds [sec] :: Time

Both `Minutes` and `Seconds` are units for the dimension `Time` measured in `min`s
and `sec`s. Units of the same dimension can be related using a `unit system` and
`ratio`s:

		unit system SI :: =
			ratio Time :: [
				'1min = '60sec,
				'1sec = '1000ms 
			]

A minute `'1min` is `'60sec`, a second `'1sec` is e.g. `'1000ms`. Hence literals
use a `'` prefix and are typed through their unit of measure.

### Compound Values
Compound values aka records, structs or tuples are declared as a `data` type.

		data Rect :: (Length width, Length height)

A `Rect` is a tuple with two `Length` fields, `width` and `height`. While all 
types are named they can be used as tuples. A `Rect` is also a tuple of type 
`(Length, Length)`.

Finally there are collection types for arrays/vectors with fixed length:

		data Code :: Char[8]
		data RGB :: Colour[3]

A `Code` is sequence of `8` `Char`acters. A `RGB` colour one of `3` `Colour`s.
Whereas:

		data Sentence :: [Word]

a `Sentence` is a _list_ of `Word`s of variable length. Furthermore sets are
supported with special syntax.

		data Team :: {Member}

A `Team` is a _set_ of `Member`s. Maps are nothing else than sets of tuples:

		data Dict :: {(Word, Translation)}

A `Dict` is a set of `(Word, Translation)` tuples.

### Enumerations
Both simple and compound value types can be initialised with a list or set of
possible values to model enumerations. Classic enumerations are 0-tuples:

		dimension Bool :: () = [ False, True ]

`Bool`eans are derived from unit `()`, allowing for two values: `False` and `True`
where in case of a list `False` will also be associated with index `'0` and
`True` with index `'1`. Dimensions based on unit can also ommit the `()` as in:

		dimension Fruits :: = { Apples, Pears }

The possible `Fruits` are `Apples` and `Pears`. Compound types can be restricted 
to a enumeration in a similar way:

		data Planet :: (Mass, Radius) = { 
			Mercury ('3.303e+23kg, '2.4397e6m),
			Venus   ('4.869e+24kg, '6.0518e6m),
			Earth   ('5.976e+24kg, '6.37814e6m),
			Mars    ('6.421e+23kg, '3.3972e6m),
			Jupiter ('1.9e+27kg,   '7.1492e7m),
			Saturn  ('5.688e+26kg, '6.0268e7m),
			Uranus  ('8.686e+25kg, '2.5559e7m),
			Neptune ('1.024e+26kg, '2.4746e7m);
		}


## Functions

### Pure Functions
Functions are functions, hence pure and statically resolved, they provide
referential transparency. All functions are understood as _extension functions_.

		fn double :: Int a -> Int = a * '2

The `double` of instance `a` of type `Int` is `'2` times `a`, what is again an 
`Int`. Now when using `double`

		fn quad :: Int a -> Int = a double double

it is called as if it _extends_ the type `Int` and is a _"member function"_.
So code reads similar to OOP but is declared similar to FP. 

## Literals

## Specialisation and Generalisation


## Modularity

## Specialities 


