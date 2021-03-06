---
layout: default
title:  "in a Nutshell"
author: Jan Bernitt
class: nutshell
---

# _bruno_ in a Nutshell

<author><i>by</i> Jan Bernitt</author>

bruno is a declarative high level programming language designed for correctness
on the basis of data and pure functions. 
Robust, inherently parallel systems are composed out of job-oriented state 
machines, each being an isolated lightweight process that communicates with 
other processes through message passing.

## Prologue
Basic correctness is still too challenging with both classic and modern
[programming systems](/glossary/#programming-system) while software 
simultaneously attempts to solve problems of continually increasing scale 
accompanied by an equally increase of complexity. 

Mastering complexity however is an utopian fallacy -- it has to be avoided in the
first place. To let programmers succeed in dividing and conquering large(r) 
software systems those have to be decomposed into many simple parts
(the principle of [*operative modularisation*](/glossary/#operative-modularisation)) 
the programmer can reason about independently 
(the principle of [*local reasoning*](/glossary/#local-reasoning)). 

To ease reasoning the possibilities must be restrictable to be effectively simplified.
By expressing what is impossible the programmer limits what needs to be
considered and understood. Impossibility bears logical consequences 
that enable possibilities of simplification and optimisation[^why-restrictions]. 

[^why-restrictions]: For example, pure functions (the impossibility of site-effects) allow
      to derive, reorder or memoize execution or to fuse or inline expressions at 
      compile-time; immutable values (the impossibility of mutation) 
      allow to share data without copying it.

By composing systems out of a few specific and restricted constructs (rather than 
unrestricted generic ones) application-, compiler- and VM-programmers share a 
meaningful common language that carries its possibilities and impossibilities
along the stages so that they can be utilised. 
On the contrary the limitations of the constructs still must maintain the 
ability to abstract ideas intuitively and allow to control implementation details.

On these grounds it is an overall theme in the bruno programming system to not 
_extend_ but further and further restrict the more general to the more specific. 


## Introduction
This guide book introduces the reader into the bruno programming language 
through (briefly) explained examples. Sections build on one another. Later 
section will assume the reader has a sketchy idea of earlier sections. 
Readers already familiar with the language may very well use this book as
a reference text.

The 1st section explains how to declare data types -- the basis of programming 
in bruno. The other central aspect - (pure) functions - is presented
directly afterwards in the 2nd section. Ways to abstract follow in the 3rd
section before the 4th section explains error handling. 
The 5th section discusses how (side) effects occur and systems are composed 
using processes and channels. 
In this section the concepts learned about in preceding sections are applied 
and composed.
The 6th section continues with the modularisation strategy of the language
before the 7th gives a in depth description of the type system. 
In the 8th section a handful of quirks are touched very briefly. 
The final 9th section closes with thoughts on systems design and the qualities
of the approach taken by the bruno programming system.

#### Notes on Examples
Note that the code examples given are exemplary and use mostly fictional 
types or functions to be more descriptive and familiar to the reader.
The section on [types](#types) introduces the actual build-in types.
Often the example code isn't even idiomatic in that it often uses very general
types (like `Int` would be) to not overwhelm the reader with details that are
not important to exemplify a concept. In practice general types like `Int`
would only be used for generic code at a higher levels of abstraction. 
As types are not burdened with a runtime cost and stay applicable it is 
idiomatic to use very precise types as long it is beneficial to understanding 
and usage.

#### Notes on Syntax
The language uses sets of more or less independent declarations of the form:

		sort name :: declaration

* `sort` : what kind of thing is declared?
* `name` : what is the identity of the declared thing?
* `::` (_is declared as_-operator)
* `declaration` : a truth or how does the thing relate to other things?

The syntax has no statements or ending marks (like a semicolon) but predefined
sets of character used for operators, literal borders and expressions structure.
There are no reserved words in the classical sense. 

A single `=` usually stands for: _has the value_ (where value could also refer to
the expression that implements a function).



## Data
Data is always represented by values, thus it is immutable with value equality 
and no notion of identity.

### Ordinals
Ordinal `data` type are simple values with a finite value range and an absolute 
order like numbers or characters. Mostly this are 1-tuples. Some are 0-tuples,
what means they have no value besides their ordinal.

		data Time :: Int{0..}

`Time` is a _specialisation_ of a integral number larger than or equal to `0`
up to the maximum of `Int`.
While the dimension `Time` describes a kind of value, _time_ is more of a concept 
than a measurable thing itself. Therefore some time units are introduced:

		data Minutes :: Time
		data Seconds :: Time

Both `Minutes` and `Seconds` are _units_ within the `Time` _dimension_.
Often simple value are extended with a literal notation supplement:

		data Seconds   :: Time
		with "Seconds" :: (Digits "sec")

A `Second` literal consists of a sequence of `Digits` and the literal `"sec"`.

#### Value Ratios
Units of the same _dimension_ can be related using `ratio`s:

		ratio SI :: 1min = 60sec
		ratio SI ::	1sec = 1000ms 

A `ratio` within the unit system `SI` declares a minute `1min` as 
`60sec`, a second `1sec` as `1000ms` (and so forth). 
`SI` here refers to a unit system that is used as a fix point to group ratios
within the same system.

To use a the `ratio`s of a unit system it is added to the name-space. 
This makes it unnecessary to declare or explicitly apply conversions between 
related values:

		Milliseconds three-minutes = 2min + 60sec

The duration of `2min` and `60sec`, values of type `Minutes` and `Seconds` can 
directly be used where e.g. `Milliseconds` are needed as the ratio between these 
is known.

### Tuples _(Product Types)_
Composite values (also called records or structs) are declared as n-tuple 
`data` types:

		data Rect :: (width: Length, height: Length)

A `Rect` is a tuple with two `Length` fields, here labelled as `width` and 
`height`. While `data` types are nominal typed they can be used as 
(structurally typed) tuples. So a `Rect` can be _generalised_ to a tuple of 
type `(Length, Length)`. The labels are not part of a tuple type definition
an can be chosen differently in each context or left out altogether.

Any simple data type is equivalent to the corresponding 1-tuple notation.
`Int` is the same type as `(Int)` and so forth.

### Arrays
Arrays are sequences of fixed length with elements of identical type:

		data Code :: Char[8]
		data RGB  :: Colour[3]

A `Code` is an array of `8` `Char`acters. A `RGB` colour one of `3` `Colour`s.
The length is part of the array's type and not encoded in the value as some sort
of array _header_ so that a `Char[3]` requires exactly 3 times the memory of a
`Char`. Nevertheless, a length can also be within a span.

		data UTF8CodePoint :: Byte[1-4]

The exact length of a UTF-8 code point is statically indeterminable but will be 
in the range of `1-4` `Byte`s. Different cases are either handled by a 
specialised function or the actual length is passed as an extra argument.

Length or span furthermore allow the use of a wild-card `*` to indicate 
_any unknown_ length.

		data Nonempty :: Int[1-*]

An array is `Nonempty` when it has at least `1` up to any `*` number of elements.
Wild-card ranges allow to describe code on a abstract level. When it is invoked
with actual values spans and wild-cards are substituted with a known actual 
length.

### Views
Slicing an array of type `T[E]` creates a slice of type `T[<E>]`. 

		Char[<2>] he = "hello world" slice 0 2 

By `slice`ing a section of the string `"hello world"` starting at position
`0` with a length of `2` the variable `he`, that is a slice of two
characters `Char[<2>]`, refers to the first two characters of that string,
namely `"he"`.

A slice is a view upon a section of an immutable underlying array extended with 
information about the position of the slice within the originating array and 
the ability derive slices upon the same underlying array that are moved or 
changed in length.

		Char[<2>] ld = he shift-to-end

For example `he` could be moved to the end of the underlying array resulting in
the slice `ld` referring to the sequence `"ld"`.

### Enumerations
Classic enumerations are 0-tuples `()` listing 2 or more constants:

		data Bool :: () [ False | True ]

`Bool`eans specialise unit `()` allowing for two values: `True` and `False`.
As the constants are declared as a list `[ | ]` `False` will also be associated 
with index `0` and `True` with index `1`.

		data Fruits :: () { Apples | Pears }

The possible `Fruits` are `Apples` and `Pears`, this time with _set_ semantics.

For an enumeration of a simple type (1-tuple) a value can be assigned to each
constant.

		data Sign :: Char 
			[  Plus  = '+'
			|  Minus = '-' ]

A `Sign` is either `Plus` (what is the `Char` value `'+'`) or `Minus`.
This, however, is often better modelled by specialising the type for each 
enumeration constant instead:

		data Sign :: 
			[  Plus  : Char{'+'}
			|  Minus : Char{'-'} ]

`Plus` is a specialisation shape of `Sign` and restricted to the `Char` in range
of only the `'+'` value. Similar `Minus` is a specialisation shape restricted to
the value `'-'`. If constants in an enumeration use type specialisation their
types must be mutual exclusive but have a common generalisation type. In case
of range types the ranges are exclusive but the base type is the same.

Composite types can be restricted to an enumeration in a similar way:

		data Planet :: (weight: Kilograms, radius: Meters) 
			{  Mercury = (3.303e+23kg, 2.4397e6m)
			|  Venus   = (4.869e+24kg, 6.0518e6m)
			|  Earth   = (5.976e+24kg, 6.37814e6m)
			|  Mars    = (6.421e+23kg, 3.3972e6m)
			|  Jupiter = (1.9e+27kg,   7.1492e7m)
			|  Saturn  = (5.688e+26kg, 6.0268e7m)
			|  Uranus  = (8.686e+25kg, 2.5559e7m)
			|  Neptune = (1.024e+26kg, 2.4746e7m) }

All the `Planet`s in our solar system are given as a _set_ of possible values 
with their `weight` and `radius`.

Ordered enumerations can also be used as dimension type of arrays.

		data Menu :: Meal[Weekday]

Assuming `Weekday` is a enumeration having `Monday`, `Tuesday` and so on `Menu`
is an array type of fixed length 7 that is index accessed using the enumeration
constants: `Meal on-monday = menu at Monday`. Index access has no special syntax
and uses a function (`at`) like all other operations.

### Constants
Constants are `data` "enumerations" with a single value.

		data Pi :: Float = 3.14159265359

`Pi` is a scalar number based on `Float` having the value `3.14159265359`. 
The _constant_ `Pi` itself has a shape that is more special than `Float`.
As a consequence a function can expect or result in a particular constant.

Of course constants can be of any type initialised with any statically 
resolvable expression using functions and any type of literals.

### Collections
Two collection types have special syntax support. While the collection types
itself are abstract [`concept`s](#concepts) they can just be implemented using 
`data` types.

In contrast to the fix length of arrays collection are of a variable length 
(wherefore the length is not reflected on type level).

		data Sentence :: [Word]

A `Sentence` is a variable length _list_ of `Word`s. 
Furthermore _sets_ are supported with special syntax.

		data Team :: {Member}

A `Team` is a _set_ of `Member`s. Maps are nothing else than sets of tuples:

		data Dict :: {(Word, Translation)}

A `Dict` is a set of `(Word, Translation)` twin-tuples (pairs).


### Literals

#### Ordinal Literals
There are three types of value literals; the mostly used conceptual value 
literals for simple values as scalars, characters and user defined units:

		Int scalar = 42
		Float real = -0.42
		Char letter = 'a'
		Mass weight = 10kg
		Teaspoon salt = 1/2tsp

Conceptual literals start with a `'` or a number and end with next white-space 
or a couple of symbols like bracket and punctuation symbols. This includes 
potential unit of measure, like `kg` or `tsp`. 
`Char`acters have `'` as their _unit_. Scalar numbers have no unit of measure. 
When a unit of measure is present the type of a literal is derived form it as 
it has to be unambiguous in any scope.

#### Binary Literals
Secondly binary oriented literals typically used for simple numerical values:

		Colour red = #xFF0000
		Int mask = #b10101010

Binary oriented literals start with a `#` followed by the type, `x` for hexadecimal, 
`b` for binary, but also `o` for octal and even `d` for decimal values.

#### Textual Literals
The third type of literals are textual oriented. Composite values are given in a 
way humans are used to write them:

		String msg = "hello world!"
		Point p = "1:2"
		Date imagine = "1971-10-11"

The `Point` and `Date` example illustrate that textual literals can be used for 
any composite type (when these are declared [polyvalent](#type-polyvalence)). 
The type of textual literals is inferred from the context, such as the variable, 
parameter or return type.

#### Collection Literals
In accordance with collection types there are three sorts of collection literals:

		Int[] array = [ 1, 2, 3, 5 ]
		[Int] list = [ 1, 2, 3, 5 ]
		{Int} set = { 1, 2, 3, 5 }

Whether a sequence literal `[ ... ]` becomes an `array` or a `list` depends on 
the type is it assigned to. Sets use curly braces `{ ... }`. Like the _map_ type
is a set of pairs the _map_ literals are created as sets of pairs:

		{(Int, Char)} map = { 1 => 'a', 2 => 'b' }

A set literal is used to simulate a _map_ through a set of pairs. Each pair
is the result of the _associate_ `=>` procedure available for any value type. 
It is not a special map syntax but a usual procedure call resulting in a pair.

#### Atoms _("type-less" constant literals)_
Atoms are dimensionless values, conceptually zero-tuples of type `Atom`. 
Different constants are defined by `` ` `` followed by any sequence of 
characters except white-space.

		[ `a `atom `+ `/ `-> `0 ]
		
The above is a list of valid atoms. Any two atoms are logically equal if their 
sequence of characters are. 
Atoms do not have a value besides being equal to another atom or not. 
They are therefore mostly used as keys or _tags_.
As such they are important for the declarations of the language's AST.


### Initial Values _(Default Values)_
There is no _null_ pointer/reference/value.
This is somewhat a non-issue as all data starts from literals. 
One either gets a value as an argument or starts from a literal.

When a value is _unknown_ depending on runtime conditions this is made
explicit by using the optional type _variant_ `?` build into the formalism:

		Line a = "1:2-4:5"
		Line b = "3:2-6:2"
		Point? maybe-point = a point-of-intersection-with b

A _optional_ `Point?` is either a `Point` or _nothing_.




## Functions
[^purity]: Strictly speaking, there are _types_ that have effects and functions upon 
      those consequently have as well - but as this is explicitly in the types 
      the compiler can and does tell them apart and applies different rules to them.

Functions are functions, thus pure, statically resolved and referentially transparent[^purity]. 
All functions are understood as _extension functions_ on the type of the 1st 
parameter.

		fn double :: Int a -> Int = a * 2

The `double` of the _instance_ `a` of type `Int` is `2` times `a`, what is again 
an `Int`. Now when using `double` to compute the `quad`

		fn quad :: Int a -> Int = a double double

`double` is called as if it _extends_ the type `Int` as a _"member function"_.
To calculate 4 times of `a` it is `double`d once and that result, again an `Int`,
is `double`d one more time equal to `(a double) double`. 

While `a` is a value and not an object, functions are syntactically invoked _on_ 
it (similar to methods on an object in OOP). This notation reads more natural and 
often can avoid parentheses. 

### Evaluation and Operators
Expressions are always evaluated **left to right** where 
operators are short hands for function calls. 

		fn plus `+` :: Int a -> Int b -> Int
		fn mul  `*` :: Int a -> Int b -> Int
		
The `plus` function is bound to `+` operator, the `mul` function to `*`
(within each module this operator _alias_ has to be unambiguous).

		1 + 2 * 3 == (1 + 2) * 3
		
If not set in parentheses `1 + 2 * 3` will be evaluated left to right resulting
in `9`, not `7`.[^operator-alias]

[^operator-alias]: Instead of dealing with operator precedence (what is hard and 
     most of all ineffectual to memorise) only parentheses have to be applied correctly 
    (what is easy to apply, check and memorise). What might be mathematically 
    unpleasant does significantly simply reasoning for the programmer.

### Branches and Cases
There is a single control flow construct directly embedded in a function 
declaration syntax. This implies that the construct cannot be nested. Extension
functions and the `where` clause are used to avoid nesting, as shown soon.

		fn min :: Int a -> Int b -> Int =
			a | a < b
			b

The `min`imum of two values `a` and `b` is `a` in case `a < b` otherwise `b`.
The cases `<expr> | <condition>` are checked in order of appearance. 
The last case has to be the universal (true'ish) condition. 
For enumerations all values can be covered alternatively. 

		fn show :: Bool v -> String =
			a. "false" | v == False
			b. "true"  | v == True

Each case is implemented by an expression (the formalism has literally no 
statements). 
Note also that each case can be labelled (here `a.` and `b.`) with a lower case 
letter followed by a dot. This has no role other than the possibility to 
improve readability and explicability.

##### Case-Matching
Cases can also be used to describe patterns using wild-cards similar to pattern 
matching. Case-matching however is a straight value comparison that 
helps to avoid reoccurring expressions in the conditions of cases.

		fn name :: Point p -> String
			       =    | p x == _ | p y == _
			a. "ORIGIN" | 0        | 0
			b. "ONE"    | 1        | 1
			c. p show

As usual a condition follows the `|`. This can be split into more than one 
_column_ for multiple conditions. Here the first row uses wild-cards `_` that 
are substituted in the following rows labelled `a.`, `b.` and `c.`.
A row might also use a horizontal bar made from dashes to emphasis the table
layout. This has no special meaning though. Note also that the `=` can be put
on the next line so it appears as heading to the result _column_. Syntactically
this is though the same position as it always has. 

### Local Variables _(Where-Clause)_
A function body is one expression, sometimes split into cases. As there are no
statements local variables are declared in a function's context, the `where`
clause:

		data Point :: (x: Int, y: Int)
		fn distance :: Point a -> Point b -> Float
			= (dx2 + dy2) sqrt
		where 
			dx2 = dx * dx
			dy2 = dy * dy
			dx  = b x - a x
			dy  = b y - a y

The function `distance` calculates the distance between 2 points `a` and `b`.
Local variables like `dx`, `dy`, `dx2` and `dy2` can  be 
declared based on the function's parameters and any of the other variables. 
Fields of `data` types like `Point` are accessed similar to calling a equally 
named function `x` or `y`. If a value is computed or not is truly transparent
from the callers point of view.
Due to referential transparency the order of declaration is irrelevant and
can be chosen for best readability. Also the necessity of computation (order or
at all) of local variables is not the programmer's burden. 

### Loops
The primary way to _loop_ is to use build in operators that work on arrays in
the fashion of
[APL](http://en.wikipedia.org/wiki/APL_%28programming_language%29) like 
languages like [K](http://en.wikipedia.org/wiki/K_%28programming_language%29)
(the exact operators and syntax haven't been decided yet). 

The secondary way to _loop_ is to use recursive functions:

		fn factorial :: Int n -> Int =
			1 | n == 0
			n * factorial (n - 1)

All tail recursive functions will use tail call elimination, thus not create/use
stack frames. Due to referential transparency and _annotated_ commutativity of 
appropriate functions the multiplication in the above example could also be 
written vice versa `factorial (n - 1) * n` and still be executed without 
consuming stack frames as `*` is such an _annotated_ operation.

<!-- 
TODO example that shows how to annotate
-->

### First Class Functions
Functions are also values, they can appear as parameter- or return type of a 
function declaration and be assigned to variables (`where`-clause). Such 
functions are _anonymous_ in the sense that their original name is _lost_.

For example, a function that takes an `Int` and computes a `Bool` has a type 
of `(Int -> Bool)`.

		fn even :: (Int n -> Bool) = n mod 2 == 0

The `even` function has this type and could be passed as an argument to another
function `is`:

		fn is :: Int n -> (Int -> Bool) f -> Bool = f n

`is` might not be very useful in practice but illustrates well how functions can 
be used as arguments, like in the following expression:

		1 is even

Function types can be seen as a way to _abstract_ over functions and see it as
a member of a _function type family_. This kind of abstraction can also be made 
first class as operations will shortly show.

### Partial Application
Another situation that results in function types is partial application of
function arguments. 
Each not applied parameter is indicated by the underscore `_` _wild-card_.

		(Int -> Int) inc = 1 + _

[^plus]: given `` fn plus `+` :: `` `` Int a -> Int b -> Int ``

Instead of passing an actual argument to `+` the function is partially applied
resulting in a function `inc`[^plus] that takes (or _on_) the left out 
argument `b` which itself than results in a value of type `Int`. 

When a function should be passed as argument without calling or applying it at
all simply no arguments are given. Depending on context it might be necessary 
to wrap the function in a group to dissolve ambiguity.

		(plus)

The expression `(plus)` makes sure the function `plus` is understood as a value.
If the context isn't ambiguous the parentheses aren't necessary. 

### Equivalence of Arity
Excursion: Any simple value is identical to the 1-tuple of that simple value. 
For example a value of type `Int` is identical to the type `(Int)`.
Similarly a `[Int]` is `([Int])` or a `{Int}` is also `({Int})`.

A function on the other hand can have any number of parameters. However, another
way to look at it is to assume that all functions just have one parameter being 
a n-tuple of the multiple elements. 

		fn multi :: Int a -> Int b -> Int c    -> Int = ...
		fn uni   :: (Int,    Int,     Int) abc -> Int = ...

Both `multi` and `uni` compute an `Int` from three inputs of type `Int`.
They have a functionally identical signature. This does not mean that everything
is normalised to one (multiple simple parameters) or the other (one tuple 
parameter) but that an variable amount of parameters can be expressed through
its tuple equivalent. 

### "Not Yet Implemented" Functions
When fleshing out an implementation a function's body can be left out marked as
_not yet implemented_ by a `?` as body:

		fn not-yet-implemented :: (Foo f -> Bar) = ?

The `?` is just a hint for the compiler that the body is missing but will be
added later one so the signature can be used to implement other functions on
top of that before returning and implementing this function. 

<!-- 
TODO functions as alias ala : fn foo :: = bar
-->



## Abstractions

> The purpose of abstraction is *not* to be vague, but to create a new semantic level in which one can be absolutely precise.

> -- Edsger Dijkstra[^EWD340]

[^EWD340]: [EWD340 - The Humble Programmer](https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD340.html)

<!-- (unclear if Notations are really needed)
### Notations _(abstract over cases)_
-->

### Type Families _(abstract over types)_
A type `family` is mostly a _type variable_ bound to certain constraints in
terms of the qualities of member types. 
Any actual type is a member of a vide variety of families bases on its properties. 
Membership is not declared by an actual type but implicitly given or not, 
depending on its sort, shape, value range or suchlike qualities. 

		family T :: Int

The family for type variable `T` contains all types that can be generalised to
`Int`. Type variables generally have names consisting of a single upper-case 
letter (that might be followed by a single digit), what distinguishes them from 
concrete types that cannot have such names. 

		fn plus `+` :: T a -> T b -> T = (`+ ?a ?b)

The `plus` function is in effect declared for all types that are members of
the type family `T`. Most notable this also results in a value of type `T` 
without having to specify any particular type or requiring value _factories_.

When used a type variable is conceptually substituted with the actual type 
that satisfies the constraints of the family.

		data Mass :: Int{0..} with "Mass" :: (Digits "kg")
		Mass m = 1kg + 2kg 

As `Mass` is a specialisation of `Int` -- its values can be added using the `plus`
function, resulting in a value of type `Mass`. If `plus` would have been 
declared in terms of `Int`s, like

		fn plus `+` :: Int a -> Int b -> Int = (`+ ?a ?b)

the function would still be usable for `Mass` values (as they would be
generalised to `Int`) but it would result in an `Int` instead of a `Mass`:

		Int mass = 1kg + 2kg 

That way using functions would generalise the type of the return value what 
often isn't intended for calculations on values of one type. 
This behaviour however is expected for functions that are type conversions, so
these are simply declared like that.

#### Equality of Type Families
Type families are local to the declaring module as they should make code become 
independent of a particular type (that needs to be referenced; being 
_abstract_ or not). Instead a family describes the qualities of _some_ type that 
are required to be suitable. Yet, this does not prevent the use of code based 
on type families in other modules as two type families or variables from two 
different modules are considered equal if they have the exact same qualification. 

Within one module two separate type families are used whenever two independent
type variables are required. When used as such classical _type variables_ both 
type families are often not constrained at all.

		family A :: _
		family B :: _

Both `A` and `B` can be of any type (described by the `_` type wild-card).
This is often used for general functions so that `A` and `B` can be substituted
with different actual types.

		fn map :: [A] list -> (A -> B) f -> [B]

To `map` a `list` of any type `A` to a list of another type `B` two type 
variables are used so that `A` and `B` can be of different actual type.

#### Types of Qualification
The examples of type families so far used simple base type qualification. 
Other properties of types can be used just as well. 

##### Qualifying Range 
In addition to base type a value range is given that a type must satisfy at
the minimum. 

		family N :: Int{0..}

All types based on `Int` that have a value range that covers `0` and higher
is a member of the family `N`.

		family M :: Int{0..1023}

Similar types are members of the family `M` if their range covers zero to `1023`.
Note that most numbers would satisfy this qualification. Just types with a 
smaller range or a range starting higher than `0` would not satisfy it.

##### Qualifying Shape
In this context the shape of a type refers to the structure of tuples or
functions. 

A tuple type can be described in terms of variable component types.

		family T1 :: (_, Int)

Any `data` type that has two components, the first being of any type (`_`) and
the second being generalisable to `Int` is a member of family `T1`. 

		data Coordinate :: Int
		data Point      :: (x: Coordinate, y: Coordinate)

`Point` is a member of `T1` as `y` is generalisable to `Int` (and `x` to `_`).

Besides abstracting the type of a tuple's element a family can also abstract 
from the element quantity and position (with some limitation).

		family T2 :: (String, ..)

Any tuple having a `String` as its first element is a member of `T2`; this 
even includes the type `String` itself as `String` is identical to `(String)`.
Similar reasoning can be applied to far more sophisticated shapes.

		family T3 :: ([T2], .., Bool)

`T3` includes all types of tuples with a list of any type that satisfies `T2` 
and a last element of `Bool`. The simplest case would be `([String], Bool)` as
the `..` can also be substituted to no extra element(s).

When dealing with recursive types the _self_ type `~` can be used to describe 
the self referencing shape of a type.

		family T4 :: (E, ~)

`T4` could describe a typical element in a linked list with an value type `E`
and another field referring to the tail of the list that is of the same type
as `T4` will be substituted for.

Similar to tuples the shape of functions can be qualified with type families.

		family F1 :: (_ -> _)

Any function with one parameter is a member of `F1`. 
Further or different qualification of functions works like it did for tuples.

		family T  :: _
		family F2 :: (T -> T -> Bool)

To give one more example: all functions with two identical parameter types 
resulting in a `Bool` are a member of the family `F2`.

		fn in-range :: Int32 a -> Int64 b -> Bool

Assuming `Int32` is a 32-bit number and `Int64` a 64-bit number both derived 
from a common base type then `in-range` is a member of `F2` as a type `T` exists
that is a generalisation of both `a` and `b`. 
If on the other hand the two types do not share a common base type `in-range` 
is not a member of `F2`.
In such cases it is often easier to consider the tuple equivalent:
`(Int32, Int64)` would satisfy `(Int, Int)` (if `Int` is the common base type).

##### Qualifying Kind
Even more general than qualifying a shape is to just qualify the kind of type
expected.

		family T :: (,)
		family F :: (->)
		family A :: _[]
		family V :: _[<>]
		family L :: [_]
		family S :: {_}
		family D :: *

`T` is any tuple regardless of its shape (including _one-tuples_, thus simple 
value types), `F` any function regardless of its arity, 
`A` is any array regardless of its element type or dimension range,
`V` is any view or slice of such an array, `L` is any list and `S` is any set 
regardless of their element types. Finally `D` is any (array dimension) 
length type.

There are three more qualifications for kinds that haven't been mentioned so far.
Their semantics don't need to be understood to abstract over them by kind.

		family M :: _[@*]
		family R :: _[<]
		family W :: _[>]

`M` is manual managed memory, `R` is any input- and `W` any output-channel type 
regardless of their element types.

The _any_ type wild-card `_` used within kinds concerns the type of the 
_elements_ of a composite or function. 
It can likewise be substituted with any _base type_ the composite should be
further constraint to. Again, this would be the type to be satisfied by an
actual type and not the only actual type possible. This means all types that
are generalisable to the stated type could be used as well. 

In addition to the basic kinds above variants of any of these are possible as 
kinds on their own. Each variant is build by appending the variants symbol to 
the base type. To emphasise the variant the following examples use the 
_any_ type as basis of the variants. 

		family O :: _?
		family E :: _!
		family P :: _*

`O` is any _maybe_ or _option_ type variant (where the value is either of the 
base type or _nothing_), `E` is any fault or _error_ type (where the value is
either of the base type or an _error_) and `P` is any _transient_ type variant
(where the value is of the base type but can be mutated _in place_ under 
certain conditions). 

Lastly there are different indirections to base types that can be combined with 
all of the above mentioned kinds to build other kinds. 
A indirection is build by prepending a indirection symbol to the base type.

		family T :: $_
		family L :: >_
		family K :: @_
		family I :: #_

`T` is any _type of_ a type, `L` is any _lazy_ type (what is syntactic sugar for
`(() -> _)`), `K` is any _key of_ a type and `I` is indexed ordinal type.
The section on the type system will
explain in more detail what those are. Key take-away here is that those exist 
so it is possible to express code on this level of abstraction. 

##### Qualifying the Existence of Functions or Concepts
In addition to the qualities of the type itself a family can also be qualified
by the kind of function(s) that should exist (in the scope) for that type.
These constraints are given in the form of operations (abstract functions) 
and/or concepts (sets of abstract functions) for which there should exist
a known implementation (actual functions) for the actual type. 

This is further illustrated in detail shortly in connection with operations and 
concepts but the principle is this:

		family A :: _ with plus

Any type (indicated by the wild-card `_` as usual) `with` an actual 
implementation for the `plus`-operation is a member of family `A`.
Equally multiple existential qualifications can be given:

		family B :: _ with plus, minus

To be a member of family `B` there need to be implementation for both `plus`
and `minus`. Of course this can be combined with qualifications of the type
itself.

		family C :: Int with plus, minus

Likewise the existence of one or more concepts can be added as a qualification.

		family D :: _ as Stack

Any type that can act `as` a `Stack` (that means an implementation for a 
particular set of operations exists) is a member of the family `D`. Again,
this can be extended to multiple concepts.

		family E :: _ as Stack, Collection

Only types that have an actual implementation for the operations of both the 
`Stack` and the `Collection` concept are members of the family `E`. 

Lastly a type can be qualified by the existence of both operations and 
concepts (and if required by the type itself).

		family F :: _ as Stack with plus


#### Qualification &amp; Runtime
Type families are a powerful way to abstract over types in terms of their
qualities that compose well and don't clutter generic functions with excessive
type parameterisation. 

Type families allow to be precise on different levels of abstraction that (mostly) 
only exist conceptually for the programmer and the compiler to reason and 
understand how everything fits together. At runtime many of these aspects and
abstractions are not important any longer as a physical machine makes no 
difference between most kinds of values. All that matter to the machine is
how wide values are in terms of bits or bytes. This are the limitations
that type families cannot cross. 


### Operations _(abstract over functions)_
Operations are used to abstract over the meaning of a group of congeneric 
functions. This meaning is expressed through the operation's name.

An operation `op` is an abstract or virtual nominal typed function. 
Most often operations are used in combination with a type variable as no 
particular type is targeted but a [type family](#type-families) with certain qualities. 

		op eq :: (T one -> T other -> Bool)

The `eq` operation (`op`) is a virtual function of type `(T -> T -> Bool)` where
`one` and `other` are of the same actual type substituted for the 
type variable `T`. 

#### Polymorphism à la carte
Operations are implemented by functions of matching type through a explicitly 
declared specialisation for a particular library or module context.

		(`auto equal-ints eq [Int])

The [variant](#variants) `` `auto `` declares the automatic (implicit)
specialisation of the function `equal-ints` to the operation `eq` where the 
_type variable_ `T` is of actual type `Int`. 
In this example `equal-ints` has to be similar to:

		fn equal-ints :: Int a -> Int b -> Bool = a == b

Operations can also be bound in the `where`-clause of a function.

		where
			equal-ints +> eq Int

The function `equal-ints` is specialised `+>` to the operation `eq` for type 
`Int` within the body of the function, that belongs to the `where`-clause.

On the use-site operations are not declared as explicit parameters but passed 
implicitly as existential type constraints expressed as part of a type family.

		family E :: _ with eq

Any type (`_`) for which an implementation of the `eq` operation exists 
in usage context is a member of the type family `E`. 

		fn index-of :: E[*] arr -> E sample -> Int pos -> Int? =
			()  | pos >= arr length    
			pos | arr at pos eq sample
			arr index-of (e, pos + 1)

The function `index-of` is implemented using the operation `eq` to compare the
`sample` with the actual element `e`. Because `E` is constraint to types for
which the `eq` operation exists it can be used in connection with `E`
like a usual function of `E`. When using `index-of` the actual comparison
can be chosen by the function bound to the operation in the caller's context.

		fn contains :: Char[*] arr -> Char sample -> Bool 
			= arr index-of (sample, 0) exists?
		where
			equals-ignore-case +> eq Char

`contains` compares the characters on the basis of the `equals-ignore-case`
functions that takes the role of `eq` operations for `Char` values so that it is
used as implicit argument when `index-of` is called.

### Concepts _(abstract over data)_
Like operations give meaning to functions a `concept` gives meaning to a group
of functions that have particular interaction semantics. Thus a concept 
describes an abstract data type (ADT) by the set of supported operations.
Actual implementations will consist of persistent data structures so `concept`s
are not used to model mutable state. Their API:s are liable to the 
principles that immutability brings about.

Given the three operations `push`, `pop` and `top`, 

		op push :: S stack -> E e -> S
		op pop  :: S stack -> S
		op top  :: S stack -> E?

where both the _stack_ `S` as well as its _elements_ `E` in principle can be 
of any (`_`) type,

		family S :: _
		family E :: _

a `Stack` is declared as a `concept` having the three operations:

		concept Stack E :: = { push, pop, top }
		
The `Stack` captures the type variable of the stack _elements_ `E` so that it 
can be parametrised and thereby further specialised when used. 

With the behavioural _contract_ in place a `Stack` is used through type families. 
Assuming code in another module declares two fresh type variables 
(here named `S1` and `E1` to avoid confusion with `S` and `E`):

		family E1 :: _
		family S1 :: _ as Stack E1

Any type (`_`) that in known to _behave_ `as` a `Stack` is a member of the
type family `S1`; thus `S1` can be used as `Stack` of elements of type `E1`. 
The fresh type variable `E1` is substituted for the earlier captured variable `E`.
Here `E1` must be sufficient to satisfy `E` but can be more specialised.

On the basis of `S1` and `E1` functions can be declared using the `Stack` 
operations.

		fn init :: S1 stack -> E1 e -> S1 =
			stack push e | stack top is-nothing?
			stack

The `init` function only pushes the element `e` to the `stack` if the stack is 
empty, otherwise the stack is left unchanged. `S1` is known to follow the `Stack`
`concept` so `push` and `top` can be used as if they were functions declared
on type `S1`.

Practically any type that has matching functions can implement the concept of a 
`Stack`. This is thou (as always) declared explicitly for a particular context:

		(`auto Stack { 
			array-push => push, 
			array-top  => top,
			array-pop  => pop })

The `` `auto`` -matic specialisation to `Stack` is declared by mapping the
actual function (here `array-*`) to the matching operation from the `Stack` 
concept.

The actual types that can be specialised to a `Stack` depend on the type(s)
stated in the mapped functions. If multiple functions of same name are in the
name-space the function is qualified with the containing module or library name.

A concept's implementation consists of usual functions. 
Here `push` is implemented on basis of an array:

		family E :: _
		fn array-push :: E[] stack -> E e -> E[]
			= stack prepend e

The `array-push` function has a compatible signature to the operation `push`.
`S` becomes the actual type `E[]`, the actual `E` is still any (`_`) type.
As `array-push` is equivalent to `prepend` the mapping could very well also 
used `prepend` directly. 

		(`auto Stack { 
			prepend => push, ... }
		)

## Errors _(Error-Handling)_
TODO

### Faults

### Exceptions

<!--
are a VM thing, these occur loosely related to the operation. Like disk full, out of heap space, etc. 
its not the the operation was wrong but physical limitations were reached at that point - the idealised machine illusion cannot be hold up any longer.
-->


## Effects _(a.k.a. Side Effects)_

### Transients
Transient types are _mutable_ specialisations of the in general _immutable_ 
data types. This does not say that transient values can be mutated freely. 
A transient value has only one owner at any point in time so that a modification
_in place_ is conceptually indistinguishable from creating a new value as there
in no other reference to that place that would expect to see the old value.

> Transient types disallow more than one possible reference to a value 
> at a time but in return allow to mutate that value in place.

To become the owner of a transient value the value either needs to be created 
within the function or passed to the function as a transient value.
In the latter case the ownership is transferred from the calling owner into
the called function. Once a function became the owner of a transient value it 
can use that value once. 

Return types are not explicit labelled as transient types as all return values
always have to be owned (or usual immutable) values.

A transient type (variant) is indicated by a `*` after the data type.
For example `Int*` is the transient variant of `Int`.

		data Matrix :: Int[N][N]
		
		fn rotate90 :: Matrix* m -> Matrix

A `Matrix` is a 2-dimensional array. When `rotate90` is called `m` has to be
owned within the calling context; the ownership is transferred into `rotate90`.

		fn rotate180 :: Matrix* m -> Matrix 
			= m rotate90 rotate90

To rotate a matrix by 180 degrees `rotate90` is used twice. The first call 
`(m rotate90)` transfers the ownership of `m` from within `rotate180` into 
`rotate90`. Its return value is again owned by `rotate180` before the 
ownership is transfered into `rotate90` for the second rotation. 
Effectively the input matrix `m` has been mutated in place twice.

A disallowed use is illustrated by the following function:

		fn rotate-and-add :: Matrix* m -> Matrix 
			= (m rotate90) + (m rotate180)

Surely `m` is owned by `rotate-and-add` initially but it is used in two places.
Even if the second usage would not require ownership it still must be disallowed
to use `m` again as in effect the value `m` does not exist any longer after 
execution of `rotate90` as it has been mutated _in place_ to some other value.

### Streams _(IO)_

### Channels _(Message Passing)_
A channel is a typed queue of fix or variable length. Each channel transports
values of a particular simple or composite data type. Implementation-wise a
channel is a concept that can be implemented in various ways. 

		op send!    `>>` :: C channel -> E element -> C
		op receive! `<<` :: C channel -> E
		concept Channel  :: { send!, receive! }

The usual way is use a (fixed length) array so that the overall system
features reasonable [back pressure](http://en.wikipedia.org/wiki/Backpressure_routing). 

While each channel has an input and an output end a channel type is either
an input `[<]` end or an output `[>]` end.

		Message[<] input
		Message[>] output

Usually channels are of fixed length and block on send or receive if the channel
is full or respectively empty. The second type of channel is non-blocking 
indicated by square brackets that are open to the outside.

		Message]<[ non-blocking-input
		Message]>[ non-blocking-output

Finally a channel type can also express that the blocking behaviour of the 
channel is unknown or unimportant indicated by the square bracket being open
in line with the arrow.

		Message[<[ may-block-input
		Message]>] may-block-output

The different types of channels allow to write functions that work just for
blocking (`M[>]`,`M[<]`) or non-blocking (`M]>[`,`M]<[`) channels or both of 
them (`M]>]`,`M[<[`) and with particular value types or a wide range 
(possibly any) type using a type variable like `M` would be one.

#### Cross-Channel-Synchronisation
As channels could block a situation naturally arises where a value should be
send to the first of a couple of channels that accepts the value or where
a value should be received from the first of a couple of channels that has
a value available with the guarantee that when it happens only one channel is 
affected. This is called a _select_ operation; it is used in the `where`-clause 
of a function.

To receive (`<<`) the first available value from a list of channels the alternatives
are listed with the _select_ operator `|=` (instead of usual _let_ `=`):

		T value
			|= channel-a <<
			|= channel-b <<
			|= channel-c <<

_Select_ tries to receive a `value` of type `T` in the order given, top to bottom.
If non of the channels offers a value (they all blocked waiting) `value` becomes 
the first value available for any of the channels with no particular priority 
should multiple channels offer a value _simultaneously_.

Similarly to send (`>>`) to the first channel that accepts a value the _select_ 
operator `|=` is used as well:

		T[>] channel 
			|= value >> channel-a
			|= value >> channel-b
			|= value >> channel-c

The _select_ tries to send a `value` in the order given, top to bottom.
If non of the channels directly accepts the value (they all blocked waiting)
the `value` is send to the first channel accepting it with no particular priority
should multiple channels accept a value _simultaneously_.
In any case the operation returns the channel that accepted the value.
The value could also be different for each alternative. 

#### Time-outs
When a blocking send or receive should time out a _select_ is used together with
a type of channel that accepts/offers a value after a certain time. 

		Int value
			|= channel <<
			|= 7 after 5ms

The `after` procedure creates an ad hoc channel that offers `7` after `5ms`.
To synchronise time-outs for multiple parallel processes a common _time-out_
channel is used as alternative for all of them. 

#### Receive with Defaults
When a blocking receive should use a computed or constant default value in
case the channel cannot offer a value the last alternative simply does not
involve a channel.

		fn receive-or! :: T[<] chan -> T default -> T = value
		where
			T value
				|= chan <<
				|= default

The _select_ operation will try to receive a `value` in order of given
alternatives and returns the result of the first path that offers a value.
As the last alternative does not involve potentially blocking constructs the
`default` value will be the result in case the `chan`nel did not offer a value
directly. 

#### (A)synchronous Interconnections
In general channels decouple processes. In principle the connection between a 
consuming and a producing process can be seen as an asynchronous connection 
where the producer can send a message without waiting for the receiver or where
the receiver can receive a message without waiting for the sender. 
Depending on the kind of channel used this might change to a synchronous 
connection for sending messages if the queue is full or for receiving messages
if the queue is empty. This means, to emulate a direct synchronous _call_ 
between processes as caller and callee those are connected using 
a blocking queue with a fixed length of 1.

#### Order of Messages
While it is obvious for the most channel implementations that they will 
keep the order of messages the general contract of channels does not implicate 
an order-guarantee as channels are also used to communicate with remote systems.

Secondly any process should be as self-contained as possible what includes 
not to expect a certain behaviour from the process surrounding in which it is
embedded through channels. The order of messages is such an expectation that 
might be necessary in some cases but should not be in general.

In the future the language might even distinguish channels that guarantee to
keep order as a specialisation type of channels that do not.

### Processes
A process is a isolated lightweight thread of execution exclusively connected 
to the _outside world_ by channels (and indirectly streams). 
It is the only construct to model behaviour around enduring state. 
The state of a process is always local to that process. 
Each process is a state machine that decides when to receive from or send 
messages to channels and change its state.

A formal `Process` is declared in terms of the possible state transitions 
between states that are declared as an enumeration:

		data Server :: Process { ..
			| Ready = [ Ready ] 
			| _     = [ Ready ]
			| Out-Of-Heap-Space! = [] }

A `Server` process has one state `Ready` that only can transit to itself.
Any other (error) state (`_`) transits to `Ready` as well. 
Should, however, an `Out-Of-Heap-Space!` exception occur the process transits 
to no other state, what indicates process termination. All possible 
exception states are _inherited_ from `Process` enumeration the `Server` extends 
upon.

So a process declaration encapsulate a behavioural pattern that can be used 
for many concrete automata. 
This contract includes even the error behaviour.
The compiler makes sure all transitions of actual implementations follow
this specification. 

The state `data` of a concrete process could look like the structure `HttpServer`.
It holds channels to communicate with the _outside world_.

		data HttpServer :: (
			requests:  HttpRequest[<],
			responses: HttpResponse[>],
			app:       (HttpRequest -> HttpResponse))

The concrete process is given as a set of `when`-_then_ state transitions that 
run a sequence of effects before the successor state is computed (both in terms
of the step and the data). 

		when Ready :: HttpServer server -> HttpServer
		1. server responses >> 
			   (server app (server requests <<))
		.. Ready: server

The body of a process function enumerates the steps of the transition. 
The `1.` step of the server is to send (`>>`) a message to `server responses` 
channel after it has been computed by the `server app` function for the request 
that is received (`<<`) from the `server requests` channel. 
The process continues (`..`) in state `Ready` with same `server` state as before.

#### Spawning Processes
To spawn a process the `spawn!` function is called on the data structure that
is the concrete process, here `server`.

		fn start! :: HttpServer server -> PID = server spawn!

`spawn!` is called to `start!` a `HttpServer` value as process. 
The `spawn!` procedure itself is implemented by the virtual machine instruction 
`` `spawn! `` that returns the ID of the created process (`PID`).

		family P :: (,)
		proc spawn! :: P a-process -> PID = (`spawn! ?process)

Any value type `(,)` can be a process `P`. The relevant `Process` declaration 
is identified via `when` transitions in scope that use states of process type 
`P` in question.

#### Error Handlers
Faults or Exceptions that occur during a state transition automatically 
continue with the matching error state transition. 
These use the type of fault instead of a state constant. For example:

		when Out-Of-Heap-Space! :: HttpServer server -> ()
		..

The `server` does not continue (`..`) with any state `when` a 
`Out-Of-Heap-Space!` error occurs. This indicates the termination of the process. 
The return type `()` also indicates that the server does not continue.

Error handlers make it easy to model robust enduring and self-repairing 
processes or fragile daemons with task specific tear-down behaviour.

For a generic error handling the actual `error` could also be passed to the 
state transition by adding it as an extra argument.

		family E :: _!
		when E! :: HttpServer server -> E! error -> HttpServer?

Any error `_!` is handled by the `when` transition that might result in a
successor state `HttpServer?` or _nothing_ depending on the actual `error`.
On type level the optional type `HttpServer?` models the possibility of 
process termination.

#### System Ports and Interfaces
In a system composed out of processes and channels input devices like mouse and
keyboard are _driver_ processes that feed data into a particular channel. 
Sockets are channels, the file system a process that starts up helper
processes for actual IO and so forth. 
For a program that is composed out of processes there are just channels and 
streams to affect the _outside world_. 

This achieves uniformity and technical decoupling from high levels of expression 
down to low level facilities. 
It becomes natural and trivial to emulate hardware as no actual coupling to the 
hardware exists (except within a _driver_ process that might be on the other 
end of a channel). 

#### Cooperative Concurrency
Processes in bruno use a form of cooperative concurrency closest to 
[coroutines](http://en.wikipedia.org/wiki/Coroutine) but with the quality of
[subroutines](http://en.wikipedia.org/wiki/Subroutine) that only one stack 
(per physical CPU) is favourable. 

In contrast to an [actor](http://en.wikipedia.org/wiki/Actor_model) a process 
does not respond to messages but represents a state machine that decides when 
to pull input from channels or push output to channels. 
The state transitions it undergoes start as control is transferred from the 
scheduler to the process and end or pause whenever a process interacts with 
channels or changes over to another state. 
Then control is transferred back to the scheduler carrying along a resume state. 
At this point the stack is empty so that the scheduler can continue any other 
process until that again returns control back to the scheduler and so forth.
So processes never actually interact with channels themselves but express their
intent to do so to the scheduler that is carrying out the operations when it is
possible and the process is chosen to be resumed. 
This is guaranteed to not block during a state transition as just pure functions
can be executed in-between context switches.

As a model of thought a process is analogous to a biological cell that is 
somehow connected to its surrounding. While it is affected by environmental 
changes in a wider sense there is not necessarily a direct
[reaction](http://en.wikipedia.org/wiki/Reaction_%28physics%29) - although it
very well might have a deterministic behaviour on a larger scale. 
A program or system is likewise analogous to networks of cells that behave
as an organism. 



## Modules _(Artefacts)_

### Source Code
Source code is contained in files with the ending `.bruno`. 
Each file is said to be a module. 
It contains a sequence of declarations with no requirements regarding order 
or limitations to the types of declarations as each declaration states
a fact that should universally hold regardless where it appears. 

There are no _headers_ or other accessory parts in source files. A file
could literally just contain a single declaration like this:

		data Hash :: Byte[*]

Beside source code files there are library files that are concerned with the
code organisation and interconnection as described shortly.

### Name-spaces
The language has no distinct name-spaces for types, function and so forth.
Everything lives within the same naming space. Yet the schemata for legal names
of different constructs are chosen in way so that they mostly cannot clash. 
Types and named constants start with upper-case letters, functions and fields
start with lower-case letters, type variables consist of a single upper-case
letter.

There is no notion of fully qualified names of types or functions and alike. 
Names are always just _simple_ as they have been specified in their declaration.
To be able to use a type or functions these have to be imported into the 
name-space of a module, what is done in libraries. 
Within the name-space of a module all names have to be unambiguous. In some cases
this does not mean that there is just one thing with a certain name but that
all with the same name are mutual exclusive regarding the involved types so that
just one of them will fit in any possible usage situation. 

### Source Code Organisation
How declarations are best split into modules is up to the programmer.
Usually code that belongs together is defined together. Depending on size or
usage it might be split into several modules so that each of them is coherent
and manageable. 

#### Libraries
The organisation of modules in directory structures is used as a form of
dependency and name-space management. 
Each directory is considered to be a library. The code within a library must
only depend on other libraries that are parent directories or have the same
direct parent as the library. As a consequence the core runtime is located at
the _root_ folder. 

		@runtime
			library @a
				library @a1
			library @b
				library @b1

The `@runtime` depends on nothing as it has no parent or parallel library.
library `@a` might depend on library `@b` or vice versa -- but libraries must not
be cross-dependent or have dependency cycles. Further library `@a1` must not
depend on `@b` or `@b1` but can depend on `@a` or the `@runtime`.
A library may however refer to any other library that is considered as 
_external_, that is already compiled third-party code. 

A library in this regards is identical with a physical location. It is not
an abstract path or canonical name in such a way that code that is located in
physically different locations can be part of the same library. 

#### Benefits of a Constraint Layout
The layout and the equivalence between directories and libraries has been 
chosen because it provides very important characteristics. Code can always be
compiled from the _root_ directory downwards. With each step down in the 
directory tree the compilation can be parallelised as all dependencies must 
have been compiled already. Partial compilation is simpler and more efficient 
as all modules that might be affected by a certain change must be in a 
sub-directory or a directory parallel to the direct parent. 
At runtime the layout largely simplifies code loading as references to libraries
and the modules within them are identical to the physical location of the code.
Furthermore the organisation allows for introspection of libraries and modules
as it is known where to look for them. 
On a final note, the layout and the 1:1 mapping to physical storage also
make it very clear that it is not possible for a third-party to add modules to
a library. 

TODO everything that happens in libraries...




## Types _(Type System)_

The fundamental idea of the type system and types in bruno is that types are
not attached to values at runtime. 
Types are statically defined or decidable properties of values in a certain 
static context. 
In case of type variables actual types are passed as additional, implicit value 
arguments. 
Consequently types are never _inspected_ - they are either known statically or
reconstituted by a language level dispatch selecting the case where the value 
representation matches with the types.

> A type is the sum of valid assumptions that can be made about the qualities
> of runtime values.

Values are represented to be efficient and convenient for the machine to compute 
without any meta information attached. 
All values of the same machine _form_ are technically compatible -- here the 
compiler uses a type relation called generalisation--specialisation to decide if 
compatibility is wanted by the programmer or should result in a type-error.

The purpose of types is to ease reasoning, aid formal correctness checking and
allow to derive the optimal machine representation based on the qualities of 
a type. Through type polyvalence and shapes more such properties can be encoded 
within types than possible in other languages. 

In some sense types _carry_ static properties of values. A value that has been 
proven to conform to the qualities of a certain type can be treated as a value
of that type. 
Yet, type conversion is only allowed between values of related types that 
are structurally compatible. This is to say that the static knowledge about 
the value may not conflict with the target type. 
It follows that a _universal_ top type does not and cannot exist as it would 
have to be structurally compatible with all other (structurally different) types.

Simply speaking types give meaning to bits and bytes of data structures, 
structures that can be meaningful in different ways what is reflected through 
different (often related) types. 

### Specialisation -- Generalisation _(Type Polymorphism)_
Specialisation--generalisation is a type relation with static type enforcement 
that models a type polymorphism different from textbook subtyping 
(interface inheritance) or OO-like inheritance (implementation inheritance). 

The fundamental difference is well illustrated by the observation that types are 
related in a graph-like hierarchy (more precisely a 
[DAG](http://en.wikipedia.org/wiki/Directed_acyclic_graph)) while values
(actual instances) do not have such a relation. A value of formal type `A` is 
always of actual type `A`. This stands to reason considering that a type is only 
associated with a variable in a static context but never attached to the value 
at runtime. A value treated as `A` _is_ an `A`. Consequently formal and 
actual value types are always identical, a distinction in practice unnecessary. 

> A type that is derived from a base type is said to be a specialisation of 
> the more general type -- and vice versa -- the base type is said to be the 
> generalisation of more special derived type. 

In contrast to implementation inheritance a specialisation is a strictly more 
_limited_ type than its generalisation. It can neither add fields to it nor can 
it widen the range or change the type of possible values. 
The set of all possible values of a specialisation `B` (of `A`) is a strict 
subset of the set of possible values of the generalisation type `A`. 
Yet, any value in the set the of more special type `B` can appear (at least) in 
two flavours, that of `A` and that of `B`. The data of such a value however 
is indistinguishable for `A` and `B` using inspection -- by declaration it is 
typed as `A` in one context and as `B` in another. 

#### Type Conversion
When `B` is generalised to `A` its value (instance) is simply passed along as 
`A`. It effectively _became_ an `A` and will be treated as such from then on. 
Whereas for a specialisation from `A` to `B` the value needs to be checked for 
the more limited specification of `B` before it can be passed along as a `B`. 
Actual allocation or instantiation of a new value is not necessary as values do
not have a reference to their type.
So while any `B` can become an `A` it is unclear if an `A` can become a `B`. 
Therefore a generalisation from `B` to `A` is a implicit _noop_ but 
a specialisation from `A` to `B` has to be _demanded_ explicitly as it could 
fail at runtime depending on the actual value. 

However, that types aren't attached to values does not mean that 
type conformance cannot or is not checked at runtime when necessary. 
The type (value) to use is simply known statically or, in case of type variables,
passed as implicit additional argument to the scope of a function.

Remarkably enough it is, strictly speaking, wrong to say that a value of one 
type can be _assigned_ to another type as only values of the exact same type 
are _assignable_; but, values of other types might be possible to convert to 
the required type before they are assigned.

#### Type Reconstitution
In contrast to most programming languages a value cannot be _cast_ to any
arbitrary type. As type information is lost there is less and less that is
possible to do with a certain value as it would require to be of an 
appropriate type. On the flip-side of this it becomes trivial to see that
all programs are type-safe as there is no way to subvert type correctness. 

A loss of partial type information as experienced for generalisation though is
not as problematic as in other type systems as it is perfectly valid and safe 
to specialise a more general type to any compatible specialisation. This does
not contradict type safety. In the same way functions are specialised to
operations or data types to concepts a type can be specialised to another
one using the specialisation operator `+>` or a library `` `auto` `` declaration.

		(`auto A B)

The specialisation from `A` to `B` is, however, just allowed if `B` is knowingly 
a specialisation of `A` or if they share a common generalisation. 

The second possibility to regain partially lost type information is to provide
a set of functions with the same name but each with a different type for the
first parameter that all have a common generalisation type. A special function
invocation does a dispatch by reading the actual value property that would 
otherwise be statically known through the type.

A typical example is the loss of array length type information. Given a naive 
`UTF8` representation as 1-4 `Byte`s the `code-point` is computed depending on
the actual length. 

		data UTF8 :: Byte[1-4]
		
		fn code-point :: Byte[1] -> Int = ...
		fn code-point :: Byte[2] -> Int = ...
		fn code-point :: Byte[3] -> Int = ...
		fn code-point :: Byte[4] -> Int = ...

To invoke such an overloaded function a `?` suffix is appended to indicate the
dispatch done by the VM. 

		UTF8 char = ...
		Int cp = char code-point?

As `UTF8` is known to have 1-4 `Byte`s and a version for each possibility has
been specified it is statically established that one of the functions will work.
In case the `code-point` should be computed from a `Byte[]` of unknown length
another case would be demanded to cover all other possibilities.

		fn code-point :: Byte[*] -> Int = -1

The above case acts as a fall-back in case non of the more specifically typed
ones matches the value. Naturally all present cases must be mutual exclusive. 
Alternatively a optional type variant could be used for the fall-back case
(even though the optional variant does not occur in the other cases):

		fn code-point :: Byte[*] -> Int? = ()

As a result when calling `code-point?` the dispatch will return a value of type
`Int?`. Similarly the fall-back behaviour can use fault types to handle the
default case.

		fault Not-A-Code-Point! :: Int
		fn code-point :: Byte[*] -> Int! = Not-A-Code-Point!

Now when calling `code-point?` the dispatch would return a value of type `Int!`
that becomes the `Not-A-Code-Point!` fault in the default case.


#### Type Variance
The generalisation--specialisation type relation mostly doesn't require the
programmer to think of variance as formal an actual types are always identical.
Consequently there is no distinction between _the same_ type and
a _subtype_ or a _supertype_ of it. A instance declared as `A` is an `A`.
So the most confusing part of type variance is literally simplified: Values
are of the type declared.
However, variance does occur in the type system for tuple- and function-types 
even though this might not be obvious as the behaviour is fairly intuitive. 

Given `B` is a specialisation of `A` a tuple of `(B, B)`, `(B, A)` or `(A, B)` 
is generalisable to `(A, A)` as all fields can be generalised to the 
corresponding target fields. Therefore it can be said that the components types
of a tuple are covariant when it comes to generalisation of a tuple type. 

For a function the situation sounds more difficult than it actually is. 
A function `(A -> A -> B)` or `(B -> A -> B)` can be generalised to 
`(B -> B -> B)` but not to `(B -> B -> A)`
as all parameters can be more general but the return type must be more special 
(or identical) to the corresponding target type. So it can be said that parameter
types are contravariant and the return type is covariant. 
Broadly speaking a function can take the place of another function if it accepts 
parameters that are less specific and returns a value that is more specific
than the target function. 

#### Structural Generalisation
Similar to nominal subtyping type relations are declared explicitly. 
In addition a named `data` type can implicitly be generalised to an _anonymous_ 
_structural_ tuple type. This is nothing else than a type conversion for 
a tuple type where the target has no name for the composite but each field
is converted as usual.


### Type Polyvalence
A polyvalent type has a number of different _cooperating_ forms, purposes or 
meanings,thus types. This is based on the observation that a type is a choice
of perspective. It reflects how the programmer thinks about something, 
how something is formally written down or how a machine represents this thoughts. 
Classically a choice has to be made in favour of the perspective that matters 
most for a particular use case. In order to switch perspective type conversions 
were required for a lack of expressiveness. 

Type polyvalence is to declare a type with or from multiple perspectives each 
being expressed in terms of other types. 
This continues the perception of types presented by the 
generalisation--specialisation relation and is equally grounded in and made
possible by the static conception of types that detaches types from values.

A type can have several generalisation types as long as these do not conflict 
with each other. This means the static properties that can 
be derived from the perspectives may not disagree with each other.

To give an example: A `Point` type might be defined as:

		data Coordinate :: Int 
		data Point :: (x: Coordinate, y: Coordinate)

In order to effect e.g. the memory layout (how the machine represents the 
conceptual types) perspectives are added using `&` operator that has been
chosen as logically all stated types have to be satisfied.
		
		data Coordinate :: Int & Bit[32]
		data Point :: Word & (x: Coordinate, y: Coordinate) 

`Coordinate` now is a 32-`Bit`s wide `Int`. The `Point` made a `Word`, what
means the raw value is a _primitve_ with maximum of 64-`Bit`s. In case of 
`Point` these are split into `x` stored in the upper 32 bits and `y` in the 
lower ones. 
Both `Coordinate` and `Point` now have two types each that aren't in conflict
with each other but take a different perspectives on the underlying value. 
This can also lead to changes in the underlying raw values and thereby the
other types a type is in principle compatible with.

TODO describe \ instead of , => \ = do not align but directly connect data in
memory.

		data BCD :: Int{0..9} & Nibble & Bit[4]

`BCD` encodes numbers from zero to nine using 4 `Bit`s, what is also called a
`Nibble`. 

#### Use-Site Mixins
Like a type can define multiple perspectives on declaration-site further 
perspectives can be added on a use-site what could also be seen as a _mixin_
defined on the use-site and therewith in a certain context.


Assuming the `BCD` type had not been declared with the `Nibble` type,

		data BCD :: Int{0..9} & Bit[4]

but a library should be used that provides the `Nibble` type together with some
utilities for it.

		data Nibble :: Bit[4]

`Nibble` can be mixed-in the `BCD` type for a particular module or library, by:

		(`mixin BCD Nibble)

With the implicit specialisation from `BCD` to `Nibble` any `BCD` value now can
be used as if it had been declared as a `Nibble` from the beginning. 


### Type Supplements

#### Textual Supplement
To allow the use of `Point` literals (as suggested earlier) a textual 
perspective is added to the type definition (note the last `&` part). 

		data Point   :: (x: Coordinate, y: Coordinate) 
		with "Point" :: (x: Digits ':'  y: Digits)

A `Point` literal consists of the sequence `x`-`Digits`, `':'` and
`y`-`Digits`. Strikingly literals can be used in the definition
of types ([shapes](#shapes) explain this soon). Nevertheless, now it is possible 
to write:

		Point p = 2:3

Values of type `Point` are now also printed and parsed as described by the 
textual perspective. Naturally it is hard or impossible to have two textual
perspectives that do not conflict with each other. 

Most often when adding textual representations to n-tuples the literal would
conflict with the syntax of the language. Depending on the extra symbols this
can also happen for simple data types. In both cases such literals are simply
double quoted instead. If a `Point` would have been declared differently...

		data Point   :: (x: Coordinate, y: Coordinate) 
		with "Point" :: ('(' Digits ':' Digits ')')

with parentheses the point literal is written as:

		Point p = "(2:3)"

Nevertheless are such literals checked at compile time. They are not just 
_strings_ of characters, except when their type is just that, namely `[Char]`
or `Char[]` or likewise.

#### Array Supplement
A special binary perspective is concerned with the machine representation of
arrays of a type. Values can be represented differently in an array using
an array perspective (note the `with Point[]` part).

		data Point   :: (x: Coordinate, y: Coordinate) 
		with "Point" :: (Digits ':' Digits)
		with Point[] :: (xs: Coordinate[*], ys: Coordinate[*])

The last perspective describes a `Point[*]` as a tuple of two `Coordinate[*]` 
fields, one for the `x` and one for the `y` values. 		


### Shapes

TODO

<!--
Types and values are not two independent universes. 
There are several circumstances where types and values blend into each other; 
this should not be confused with dependent typing.

(a form of latent typing but only for constants or more general statically decidable properties)
-->


### Primitives

##### Uninterpreted

`()` _(unit)_
: used as base type for enumerations (0-tuples). The unit value also represents 
  _nothing_ for optional types.

		data Unit  :: ~ with "Unit" :: "()"

`Bit`
: a bit: `#0` or `#1`, the special sign-bit is `Positive` or `Negative` in 
  contrast to magnitude `M-Bit`s.

		data Bit   :: () { ^0 | ^1 }
		data S-Bit :: Bit { ^+ | ^- }
		data M-Bit :: Bit

`Byte` 
: the smallest addressable block is a unsigned 8-`Bit` number (range 0-255)

		data Byte  :: M-Bit[8]

`Word`
: basis for a primitive values with up to 64 `Bit`s (interpreted as unsigned or 
  signed with [two's complement](http://en.wikipedia.org/wiki/Two%27s_complement)
	dependent on the presence of a `S-Bit`).

		data Word  :: Bit[1-64]

##### Logic

`Bool` _(ean)_
: logic or truth values `True` and `False`.

		data Bool :: () { False | True }

##### Numeric

`Int` _(eger)_
: a singed or unsigned 32-`Bit` integral number.

		data Int   :: Word & Bit[1-32]

`Dec` _(imal)_
: 64-bit decimal floating point number ([dec64](http://dec64.org/)).

		data Coefficient :: Int  & Bit[56] & (S-Bit M-Bit[55])
		data Exponent    :: Int  & Byte
		data Dec         :: Word & (Coefficient Exponent)

`Frac` _(tion)_
: fraction number with 32-bit numerator and denominator ([frac64](http://frac64.github.io/)).
		
		data Numerator   :: Int  & Bit[32] : (S-Bit M-Bit[31])
		data Denominator :: Int  & M-Bit[32]
		data Frac        :: Word & (Numerator Denominator)

##### Text

`Char` _(acter)_
: a none character code

		data Char     :: Int{0..0xFFFF} & M-Bit[32]

### System Types

`Text`
: a [none](http://non-encoding.github.io/) encoded character sequence. 

		data Null     :: Byte & (^0 MBit[7])
		data Next     :: Byte & (^1 MBit[7])
		data CharCode :: Byte[1-4] & (Next[0-3] Null)
		data Text     :: Byte[0-*] & CharCode[0-*]

`Process` 
: base enumeration for all processes describing all possible system exceptions.

		data Process :: [~] 
			{ Out-Of-Heap-Space! => [],  
			| Out-Of-Disk-Space! => [] }

`Ptr`
: a _pointer_ used just within VM code.

		data Ptr :: Word & M-Bit[64]

### Type Constructors
Many qualities of values can be expressed through types. The first group (in 
the below table) are named types. 
These are naturally nominal typed and act as the basis of more complex type 
constructs. <i>X</i> stands for a name starting with an upper case letter,
<i>x</i> for a name starting with a lower case letter.
A `data` structure is either a specialisation of the generalisation type 
<i>T<sub>g</sub></i> or expressed in terms of a <i>Tuple</i> with fields. 
Similar an operation `op` is a named virtual function described by a 
<i>Function</i> type. 
A `concept` is a set of operation types <i>T<sub>f<sub>i</sub></sub></i> referenced by name. 

All but the first group build complex type variants that are structurally typed.
That means as long as the nominal typed parts in two similar, independently
defined types are identical the two types are identical as well.

| Type Constructor      | Type Components      | Syntax                   |
|-----------------------|----------------------|--------------------------|
| `data` <i>X</i>       | <i>T<sub>g</sub></i> | `data` <i>X</i> `::` <i>T<sub>g</sub></i> |
| `data` <i>X</i>       | <i>&lt;Tuple&gt;</i>         | `data` <i>X</i> `::` <i>&lt;Tuple&gt;</i> |
| `op` <i>x</i>         | <i>&lt;Function&gt;</i>      | `op` <i>x</i> `::` <i>&lt;Function&gt;</i> |
| `concept` <i>X</i>  | <i>T<sub>f<sub>0</sub></sub></i>&#8230;<i>T<sub>f<sub>n</sub></sub></i> | `concept` <i>X</i> `::` `=` `{` <i>T</i> (`,` <i>T</i>)* `}` |
| `process` <i>X</i>    |                      | `process` <i>X</i> |
|-----------------------|----------------------|--------------------------|
| Tuple                 | &#8853;<i>T<sub>0</sub></i>&#8230;&#8853;<i>T<sub>n</sub></i> | `(` [ <i>T</i>(`,`<i>T</i>) ] `)` |
| Function              | &#8854;<i>T<sub>p<sub>0</sub></sub></i>&#8230;&#8854;<i>T<sub>p<sub>n</sub></sub></i> &#8853;<i>T<sub>r</sub></i> | `(` <i>T</i> (`->` <i>T</i>)* `->` <i>T</i> `)` |
|-----------------------|----------------------|--------------------------|
| Length                |                      | (`0`&#8230;`9`)+         |
| Span                  | <i>T<sub>l</sub></i> <i>T<sub>l</sub></i> | <i>T</i>[`-`<i>T</i>]    |
| Array                 | <i>T<sub>e</sub></i> | <i>T</i>`[` <i>&lt;Span&gt;</i> `]` |
| Slice                 | <i>T<sub>e</sub></i> | <i>T</i>`[<` <i>&lt;Span&gt;</i> `>]` |
|-----------------------|----------------------|--------------------------|
| List                  | <i>T<sub>e</sub></i> | `[` <i>T</i> `]`         |
| Set                   | <i>T<sub>e</sub></i> | `{` <i>T</i> `}`         |
|-----------------------|----------------------|--------------------------|
| Range                 | <i>T<sub>s</sub></i> | <i>T</i>`{`&lt;Min&gt;`..`&lt;Max&gt;`}` |
|-----------------------|----------------------|--------------------------|
| Optional              | <i>T<sub>v</sub></i> | <i>T</i>`?`              |
| Faulty                | <i>T<sub>v</sub></i> | <i>T</i>`!`              |
| Transient             | <i>T<sub>v</sub></i> | <i>T</i>`*`              |
|-----------------------|----------------------|--------------------------|
| Type Of               | <i>T</i>             | `$`<i>T</i>              |
| Key Of                | <i>T</i>             | `@`<i>T</i>              |
| Lazy Of               | <i>T</i>             | `>`<i>T</i>              |

A structural <i>Tuple</i> type has _covariant_ &#8853; fields 
<i>T<sub>0</sub></i>&#8230;<i>T<sub>n</sub></i>. 
That means a tuple-type can be generalised to another tuple-type when each of 
its fields has a more special (or identical) type.

A <i>Function</i> type is _contravariant_ &#8854; in all its parameters 
<i>T<sub>p<sub>0</sub></sub></i>&#8230;<i>T<sub>p<sub>n</sub></sub></i> 
and _covariant_ &#8853; in its return-type <i>T<sub>r</sub></i>. 
Thus a function can be generalised to another function type when all its 
parameter types are more general (or identical) and its return type is more 
special (or identical).

A <i>Span</i> type is composed of two <i>Length</i> types <i>T<sub>l</sub></i>.
Both are used to build range specific <i>Array</i> types. The element
type <i>T<sub>e</sub></i> of an <i>Array</i>, <i>Slice</i>, <i>List</i> or 
<i>Set</i> type can be any other simple or complex type. A 2-dimensional array 
has type <i>T<sub>e</sub></i>`[][]`, a list of lists would be 
`[[`<i>T<sub>e</sub></i>`]]`.

A <i>Range</i> is used to narrow the possible value range of a simple data type
<i>T<sub>s</sub></i> (1-tuple). <i>Min</i> and <i>Max</i> values are specified by a literal 
or <i>Length</i> type.

To build an <i>Optional</i>, <i>Faulty</i> or <i>Transient</i> variant of a
value-type <i>T<sub>v</sub></i> the corresponding suffix is appended. 
The <i>Type Of</i> a type is build with `$` prefix, the <i>Key Of</i> a type
with a `@` prefix. A <i>Lazy</i> type gets the `>` prefix.
In contrast to array, list or set type variants a complex type may only 
incorporate one prefix and/or suffix variant.

<!-- where?
Fundamentally four kinds of types are to be distinguished:

* Value Types
* Function Types
* Effect Types
* Virtual Types

For simplicity the following explanations should be associated with value types
that are the basis for everything else. 
The semantics of function types are a consequence of value type semantics.
Effect- and abstraction types have mostly slightly modified value semantics.
-->


## Quirks

### Abstract Syntax Tree
So far everything has been described in terms of the surface syntax. This syntax 
builds upon the logical constructs of the language and allows to encode them
more readable and concise. The language is however defined in terms of its 
abstract syntax tree[^why-ast], a general `data`-structure. As such the AST is 
vice versa expressed in a subset of the surface syntax in the form of usual 
expressions. Tagged unions are used to keep type safety as we will shortly see.

[^why-ast]: The AST and why it is the _model_ of the language deserves an article on its own. Digest: It enables better tooling and allows the surface syntax to focus on expressing the usual well as the _unusual_ doesn't need to be possible to express as one always can escape to the AST. 

To encode any expression directly as an AST element the AST expression is _tagged_ 
with the atom `` `ast ``. The `plus` function could be implemented as:

		fn plus `+` :: Int a -> Int b -> Int 
			= (`ast (`+ ?a ?b))

The `` `ast `` tag instructs the compiler to treat the 2nd element of the outer
form as an AST element. Here this is again a tagged union implementing addition 
as the operation `` `+ `` bound to the variables `?a` and `?b`. 

The tagged union notations gives AST implementations a lisp like appearance 
that also comes with a lot of the flexibility of a lisp like language. The most 
important one might be that it allows to implement almost everything in the 
language itself. Only the transformation to actual machine code or another 
intermediate representation has to deal with _"native"_ (foreign language) code.
This allows to share bruno code mostly target platform and VM independent.


#### Variants  _(Tagged Union)_
By convention any expression form with an [atom](#atoms) as its first tuple 
element is understood as (type) tagged form called variant or tagged union. 
The atom _tag_ is similar to an explicit type _annotation_ for that form: 

		(`date 2014 06 01)

The form is tagged as a `` `date ``, followed by a date value in three components. 
The tag hints what types are expected/checked for the following elements. The 
binding between tag and type is done itself as a variant in the library: 

		(`use `date Date)

A variant _annotated_ with `` `date `` is of type `Date`, that e.g. could be:

		data Date :: (Year Month Day)
		
Given the declarations above the compiler is aware of the types in the expression
form ``(`date 2014 06 01)`` and treats/checks `2014` as a `Year`, `06` as 
the `Month` and so forth.

The technique of tagged unions is used for typed generic data structures like 
the AST. As all created variants will be type checked it is valid to expect 
any variant value encountered to be well-typed according to the bound type.
				

### Procedures _("Hygienic Macros", Compile-Time Expansion)_
Procedures are compile-time inlined _functions_, hence do not create stack 
frames and are therefore limited to non-recursive implementations. Otherwise a
 procedure `proc` is similar to a function `fn`:

		proc assoc `=>` :: K key -> V value -> (K, V) 
			= (key, value)

Any type `K` has a procedure `assoc` (or `=>`) that, given a `value` produces 
the pair `(K, V)`. When called, for example to add an entry to a map

		{ 'p' => 3, 'i' => 0.17 }

the compiler will expand the implementation of the procedure's AST with the 
actual arguments for each usage.
Note that functions passed to a procedure will not come at the expense of a
function call as the actual function is substituted by the compiler.

#### Call-Site Inlining
Any function call to a non-recursive function can be inlined at the call-site
using the _inlining_ prefix `\` in front of the called function's name:

		fn starts-equally :: E[1-*] one -> E[1-*] other -> Bool
			= one \first == other \first

To receive the `first` element of each of the lists `one` and `other` the call
is inlined using `\`. Instead of a function call the body of `first` is expanded
into the AST of the body of `starts-equally`. This is in particular useful when
a function is called in a _loop_. 

Usually inlining trades better performance for larger artefact code size as code 
is not _shared_ but _duplicated_. In cases of short functions inlining might even 
result in smaller artefacts. Still compile-time inlining always contains the
risk that different callers of the same function run different versions of it.
On the other hand inlining also can remove a runtime dependency when all 
dependent usages are inlined.


### Eager Expressions _(Compile-Time Evaluation)_
Statically computable expressions can be evaluated at compile-time. Instead of
normal grouping parentheses the expression is embedded into an evaluation
group `(:= ` _expr_ `)`. For example `Pi` can be calculated through an algorithm:

		data Pi :: Float = (:= 0.000001 pi-gauss-legendre)

The constant `Pi` of type `Float` is computed by the `pi-gauss-legendre` method
using a precision of `0.000001`. The evaluation of the _eager expression_ 
`(:= ... )` happens at compile-time, the result is inserted as a constant similar
to a declaration like (maybe more or less digits):

		data Pi :: Float = 3.14159265358979323846

Eager expressions can be used for all pure function computations based on
any form of literal(s). In other words, it works as long as no abstractions
or effects are involved.


### Keys _(Typed "Paths")_
A _key_ is a constant that _carries_ the type of values that are accessed using
that key. The keys themselves are of a uniform type `Key`, that is of little 
importance.
Each key is an abstract path to values rather than a reference or pointer to 
a specific location or value. 

		data @refers-to-Int :: Int

Key constants start with a `@` followed by any sequence of characters except 
white-space and `,`. Yet to avoid confusion with a key type the key constant name
cannot start with a upper case letter. 
The type `@Int` is the type of a key being a _path_ to `Int` values 
while `@int` is a key named `int`.

Like atoms keys are used as part of more _generic_ concepts. 
For example, a _object_ (in say the javascript sense) could be modelled like:

		op get :: O obj -> @T key -> T?

		op put :: O obj -> @T key -> T value -> O

The `obj` is a _container_ where the type of the value is brought along by a key. 
When supplying a `key` of type `@T` one can expect to get a value of type `T?` 
(that is `T` or _nothing_). 
Conversely a value of type `T` can only be associated with a key of matching 
type `@T` in order to transit from the origin `obj`ect `O` to a new `O` object 
as a result.

Keys use a more flexible naming schema to provide a wide range of possible names 
without "polluting" the single overall identifier names-pace. 

		[ @a @key @+ @1 @[other] ]

The list shows kinds of different valid _keys_. In contrast to atoms keys are
declared as `data` constants where the declared type is the type of the value 
the key leads to. 
Two keys are consequently equal if they are the same constant.


### Checks

TODO



## The Ghost in the Machine

<!-- A System of Systems -->

### Modular Software
<!--
- where can we declare code? (where/who does declare -> use site vs declaration site)
- programming with and upon properties!
- type families => generic programming without abstract code
- sandboxing (a process just has access to channels it got as long as it cannot aquire channels from keys)
-->

### The Self-Aware System

### Bootstrapping Scripts
Imagine a VM that does not run on top of an OS but that also takes over
responsibility of the OS and BIOS layers providing the bare minimum needed to 
program everything but the VM in terms of the virtual machine instructions that 
the VM will directly translate into machine code. Starting the (computer) system 
means to start the VM that has a _hard coded_ initialisation logic to get ready
up to the point where it can provide scheduling for processes and channels. 

To _boot_ into the user configured system the VM would run a bootstrapping 
_script_ detected by convention. Such a script is a sequence of messages (data)
together with the channel it should be send to identified through a key.

		@keyboard.config =>
			(`layout `QWERTZ)

		@monitor.config =>
			(`frequency 75Hz) 

For example, the keyboard should use `` `QWERTZ `` layout, the monitor run at `75Hz`. 
_Scripts_ are written in the same language with same type safety guarantees and
tools. They don't directly _do_ something but assemble data structures that 
describe what should be done. Booting becomes sending messages to channels, 
something that is not different from a running system.
This ensures a _working_ system at any point in time. The _booting_ sequence
just tries to bring the system to a particular configuration by playing-back 
messages that take the role of commands. 
Should some of them fail the system is still usable without the failed adaptation. 

This makes is easy to control the fundamental configuration of a system that
essentially cannot _crash_ when playing around with settings. A user can
tweak the system by sending messages from a console. When a satisfactory
change is found it is added to the bootstrapping script so that the system
gets into that same state on next start up as well.

### Sandboxing


## Epilogue
<!--
- a computer system that
	- does not crash, 
	- can tell why value on what function caused an error
	- makes it easy to collect all such errors, and 
	- to find and fix known errors through local reasoning

- programming paradigms do not matter - just the properties of the system we work with

-->
