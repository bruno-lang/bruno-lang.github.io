---
layout: default
title:  "bruno in a Nutshell"
---

# **bruno** in a Nutshell

<div class='abstract'>
At a first glance <b>bruno</b> appears like

<b>declarative programming with data and extension functions</b>.

While there is a strong influence of FP bruno often has its own twist and picks 
from different paradigms without being a multi-paradigm language. 
It will be the best to look at it with a fresh and open mind as the language
also has several novel ideas.
</div>

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

		data Planet :: (Mass weight, Length radius) = { 
			Mercury ('3.303e+23kg, '2.4397e6m),
			Venus   ('4.869e+24kg, '6.0518e6m),
			Earth   ('5.976e+24kg, '6.37814e6m),
			Mars    ('6.421e+23kg, '3.3972e6m),
			Jupiter ('1.9e+27kg,   '7.1492e7m),
			Saturn  ('5.688e+26kg, '6.0268e7m),
			Uranus  ('8.686e+25kg, '2.5559e7m),
			Neptune ('1.024e+26kg, '2.4746e7m);
		}

### Literals
There are three types of value literals; the mostly used conceptual value 
literals for simple values as scalars, characters and user defined units and 
dimensions:

		Int scalar = '42
		Float real = '-0.42
		Char letter = 'a'
		Mass weight = '10kg

Conceptual literals start with a `'` followed by a number or symbol and the unit
of measure, like `kg`. `Char`acters have `'` as unit and scalars the empty string;
(numbers without a leading `'` are a length type). When a unit of measure is
present the type of a literal is derived form it as it has to be unambiguous in 
any scope.

Secondly binary oriented literals typically used for simple numerical values:

		Colour red = #xFF0000
		Int mask = #b10101010

Binary oriented literals start with a `#` followed by the type, `x` for hexadecimal, 
`b` for binary, but also `o` for octal and even `d` for decimal values.

The third type of literals are textual oriented. Compound values are given in a 
way humans are used to write them:

		String msg = "hello world!"
		Point p = "1:2"
		Date 2k = "2000-00-00"

The `Point` and `Date` example illustates that textual literals can be used for 
any compound type (when defined properly, <a href="#formats">formats</a> will
go into the details later). The type of textual literals is inferred from the 
context, such as the variable, parameter or return type.

## Functions
Functions are functions, hence pure and statically resolved and are
referentially transparent. All functions are understood as _extension functions_
on the type of the 1st parameter.

		fn double :: Int a -> Int = a * '2

The `double` of instance `a` of type `Int` is `'2` times `a`, what is again an 
`Int`. Now when using `double`

		fn quad :: Int a -> Int = a double double

it is called as if it _extends_ the type `Int` and is a _"member function"_.
To calculate 4 times of `a` it is `double`d once and that result, again an `Int`,
is `double`d one more time. While `a` is a value, not an object, functions are
syntactically invoked _on_ it similar to methods on an object in OOP. This 
notation reads more natural and often can avoid parentheses. 

### Branches and Cases
There is a single control flow construct directly embedded in a function 
declaration. This implies that the construct cannot be nested; extension
functions and the `where` clause are used to avoid nesting, as shown soon.

		fn min :: Int a -> Int b -> Int 
			\ a < b \= a
			\       \= b

The `min`imum of two values `a` and `b` is `a` in case `a < b` otherwise `b`.
The cases `\ ... \` are checked in order of appearance. The last case has to
be the universal (true'ish) condition. For enumeration alternatively all
values can be covered. Each case is implemented by an expression.

		fn show :: Bool b -> String
			\ False \= "false"
			\ True  \= "true"

### Local Variables _(Where-Clause)_
A function body is one expression, sometimes split into cases. As there are no
statements local variables are declared in a function's context, the `where`
clause:

		data Point :: (Int a, Int b)
		fn distance :: Point a -> Point b -> Float
			= (dx-square + dy-square) sqrt
		where 
			Int dx-square = dx * dx
			Int dy-square = dy * dy
			Int dx = b x - a x
			Int dy = b y - a y

The function `distance` calculates the distance between 2 points `a` and `b`.
Local variables like `dx`, `dy`, `dx-square` and `dy-square` can  be 
declared based on the function's parameters and any of the other variables. 
Thanks to referential transparency order of declaration is irrelevant and
can be done for best readability. 

### Loops
- say something about how loop constructs are done

### First Class Functions
Functions are values as well, they can appear as parameter and return type of
a function declaration and be assigned to variables.

Show function types `(A -> B)`, 

### Partial Application
- mention tuple equivalence of parameters

## Abstractions
### Operations _(abstract over functions)_

### Protocols _(abstract over sets of functions)_

### Notations _(abstract over cases)_

### Type Schemas _(Generics - abstract over types)_

## Type System
- mandatory type annotations, inference where unambiguous

### Specialisation - Generalisation

### Formats

## Modularity


## Side Effects
### Machines


