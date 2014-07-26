---
layout: default
title:  "bruno in a Nutshell"
---

# _bruno_ in a Nutshell

> bruno is a novel likewise classic programming system; in simplified terms it
> could illogical be condensed to _declarative programming with data and extension 
> functions along with concurrent processes and message passing._
> Even though there is a strong influence of functional programming bruno mostly 
> has its own twist and borrows from different paradigms without being a 
> multi-paradigm language. 
> Look at it with a fresh and open mind as the language has several novel ideas.


## Objectives _(Preamble)_
Basic correctness is still too challenging with both classic and modern programming
systems while software simultaneously attempts to solve problems of continually 
increasing size accompanied by an equally increase of complexity to control. 
It is my believe that a programming system must enable *operative modularisation* 
and *local reasoning* to let programmers succeed in dividing and conquering 
large(r) software systems. 

To ease reasoning the possibilities must be restrictable to be effectively simplified.
By expressing what is impossible the programmer limits what needs to be
considered and understood. Impossibility allows to imply logical consequences and 
thereby opens possibilities of simplification and optimisation[^why-restrictions]. 

[^why-restrictions]: E.g. pure functions (the impossibility of site-effects) allow
      to derive, reorder or memorise execution or fuse or inline expressions at 
      compile-time; immutable values (impossibility of mutation) allows to share
      data without the need to copy it.

When composing systems out of restricted specific constructs (rather than 
unrestricted generic ones) application-, compiler- and VM-programmers share a 
more meaningful common language that communicates its possibilities and impossibilities
along the stages. On the contrary the concreteness of the constructs must still
maintain the ability to abstract ideas effectively and allow controling low 
level implementation details.

On these grounds it is an overall theme in the bruno programming system to not 
_extend_ but further and further restrict the more general to the more specific. 

## Data
Values are values, thus immutable. All data is represented as values.

### Simple Values
Two types of simple values, `dimension`s and `unit`s. The distinction between 
`dimension` and `unit` allows to model _type families_ (the type system section 
will go into the details).

		dimension Time [T] :: Int '0..

`Time` is a specialisation of a integral number `>= 0` measured in `T`. While the
dimension `Time` describes a kind of value _time_ as such has no meaningful value 
itself. Therefore some `unit`s are introduced:

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

The presence of a unit system makes it unnecessary for the programmer to declare
or apply conversion between units of the same system.

		Milliseconds two-minutes = '2min + '50sec

The duration of `'2min` and `'50sec`, values of type `Minutes` and `Seconds` can 
directly be used where e.g. `Milliseconds` are needed as the ratio between these 
is known.

### Compound Values
Compound values (often loosely equated with records, structs or tuples) are 
declared as a `data` types.

		data Rect :: (Length width, Length height)

A `Rect` is a tuple with two `Length` fields, `width` and `height`. While all 
types are named they can be used as tuples (structurally typed). A `Rect` is 
also a tuple of type `(Length, Length)`.

Finally there are collection types for arrays/vectors with fixed length:

		data Code :: Char[8]
		data RGB :: Colour[3]

A `Code` is sequence of `8` `Char`acters. A `RGB` colour one of `3` `Colour`s,
where the length is part of the array's type. This length can also be within a
range.

		data UTF8CodePoint :: Byte[1-4]

A UTF8 code point is `1-4` `Byte`s. Length or range furthermore allow the use of
a wildcard `*` to indicate _any unknown_ length.

		data Nonempty :: Int[1-*]

An array is `Nonempty` when it has at least `1` up to any `*` number of elements. [^no-dependent-typing]
In contrast to the fix length of arrays we can declare:

		data Sentence :: [Word]

[^no-dependent-typing]: Note that the language is not dependently typed. Its 
    concept of (type) shapes is related but less open.

A `Sentence` is a _list_ of `Word`s of variable length. Furthermore sets are
supported with special syntax.

		data Team :: {Member}

A `Team` is a _set_ of `Member`s. Maps are nothing else than sets of tuples:

		data Dict :: {(Word, Translation)}

A `Dict` is a set of `(Word, Translation)` twin-tuples (pairs).

### Enumerations
Both simple and compound value types can be initialised with a list or set of
possible values to model enumerations. Classic enumerations are 0-tuples:

		dimension Bool :: () = [ False, True ]

`Bool`eans are derived from unit `()`, allowing for two values: `False` and `True`
where in case of a list `False` will also be associated with index `'0` and
`True` with index `'1`. Dimensions based on unit can also omit the `()` as in:

		dimension Fruits :: = { Apples, Pears }

The possible `Fruits` are `Apples` and `Pears`, this time with _set_ semantics. 
Compound types can be restricted to an enumeration in a similar way:

		data Planet :: (Kilogram weight, Meters radius) = { 
			Mercury ('3.303e+23kg, '2.4397e6m),
			Venus   ('4.869e+24kg, '6.0518e6m),
			Earth   ('5.976e+24kg, '6.37814e6m),
			Mars    ('6.421e+23kg, '3.3972e6m),
			Jupiter ('1.9e+27kg,   '7.1492e7m),
			Saturn  ('5.688e+26kg, '6.0268e7m),
			Uranus  ('8.686e+25kg, '2.5559e7m),
			Neptune ('1.024e+26kg, '2.4746e7m)
		}

All the `Planet`s in our solar system are given as a _set_ of possible values 
with their `weight` and `radius`.

### Constants
A constant is a named `val`ue.

		val Pi :: Float = '3.14159265359

`Pi` is a scalar number of type `Float` having the value `'3.14159265359`. Of 
course constants can be of any type initialised with any statically resolvable
expression using functions and any type of literals.

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
`b` for binary, but also `o` for octal and even `d` for decimal values, what is also
the default and can be left out.

		Int ten = #10

The third type of literals are textual oriented. Compound values are given in a 
way humans are used to write them:

		String msg = "hello world!"
		Point p = "1:2"
		Date imagine = "1971-10-11"

The `Point` and `Date` example illustrates that textual literals can be used for 
any compound type (when defined appropriate, <a href="#formats">formats</a> will
go into the details later). The type of textual literals is inferred from the 
context, such as the variable, parameter or return type.

#### Collection Literals
In accordance with collection types there are three sorts of collection literals:

		Int[] array = [ '1, '2, '3, '5 ]
		[Int] list = [ '1, '2, '3, '5 ]
		{Int} set = { '1, '2, '3, '5 }

Whether a sequence literal `[ ... ]` becomes an `array` or a `list` depends on 
the type is it assigned to. Sets use curly braces `{ ... }`. Like the _map_ type
is a set of pairs the _map_ literals are created as sets of pairs:

		{(Int, Char)} map = { '1 => 'a', '2 => 'b' }

A set literal is used to simulate a _map_ through a set of pairs. Each pair
is the result of the _associate_ `=>` procedure available for any value type. 
It is not a special map syntax but a usual procedure call resulting in a pair.

#### Atoms _("untyped" constant literals)_
Atoms are dimensionless values, conceptually zero-tuples of type `Atom`. Different 
constants are defined by `` ` `` followed by any sequence of characters except 
whitespace.

		[`a `atom `+ `/ `-> ]
		
The above is a list of valid atoms. Any two atoms are equal if their sequence
of characters are. Atoms do not have a value besides being equal to another atom 
or not. They are therefore mostly used as keys or _tags_.
 As such they are important for the declarations of the language's AST.


### Initial Values _(Default Values)_
There is no _null_ pointer/reference/value. All simple values are initialised 
with zero or the lowest possible value should they not be specified otherwise. 
This is somewhat a non-issue as all data starts from literals. 
One either gets a value as an argument or starts from a literal.

When a value is _unknown_ depending on runtime conditions this is made
explicit by using the optional type _variant_ `?` build into the language:

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
parameter (here `a`).

		fn double :: Int a -> Int = a * '2

The `double` of the _instance_ `a` of type `Int` is `'2` times `a`, what is again 
an `Int`. Now when using `double` to compute the `quad`

		fn quad :: Int a -> Int = a double double

`double` is called as if it _extends_ the type `Int` as a _"member function"_.
To calculate 4 times of `a` it is `double`d once and that result, again an `Int`,
is `double`d one more time equal to `(a double) double`. 

While `a` is a value, not an object, functions are syntactically invoked _on_ it 
(similar to methods on an object in OOP). This notation reads more natural and 
often can avoid parentheses. 

### Evaluation and Operators
Expressions are always evaluated **left to right** where 
operators are short hands for function calls. 

		fn plus [+] :: Int a -> Int b -> Int
		fn mul [*] :: Int a -> Int b -> Int
		
The `plus` function is bound to `+` operator, the `mul` function to `*`
(within each namespace this operator _alias_ has to be unambiguous).

		'1 + '2 * '3 == ('1 + '2) * '3
		
If not set in parentheses `'1 + '2 * '3` will be evaluated left to right resulting
in `'9`, not `'7`.[^operator-alias]

[^operator-alias]: Instead of dealing with operator precedence (what is hard and 
     most of all ineffectual to memorise) only parentheses have to be applied correctly 
    (what is easy to apply, check and memorise). What might be mathematically 
    unpleasant does significantly simply reasoning for the programmer.

### Branches and Cases
There is a single control flow construct directly embedded in a function 
declaration syntax. This implies that the construct cannot be nested. Extension
functions and the `where` clause are used to avoid nesting, as shown soon.

		fn min :: Int a -> Int b -> Int 
			\ a < b \= a
			\       \= b

The `min`imum of two values `a` and `b` is `a` in case `a < b` otherwise `b`.
The cases `\ ... \` are checked in order of appearance. The last case has to
be the universal (true'ish) condition. For enumerations alternatively all
values can be covered. 

		fn show :: Bool b -> String
			\ False \= "false"
			\ True  \= "true"

Each case is implemented by an expression (the language has literally no statements).

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
can be chosen for best readability. Also the necessity of computation (order or
at all) of local variables is not the programmer's burden.

### Loops
The primary way to _loop_ is to use recursive functions:

		fn factorial :: Int n -> Int
			\ n == '0 \= '1
			\         \= n * factorial (n - '1)

All tail recursive functions will use tail call elimination, thus not create/use
stack frames. Due to referential transparency and _annotated_ commutativity of 
appropriate functions the multiplication in the above example could also be 
written vice versa `factorial (n - '1) * n` and still be executed without 
consuming stack frames as `*` is such an _annotated_ operation.

Another construct is planed for loops that _do_ something several times or with
several elements of a collection (imperative _for_ loops). The goal is to find a
declarative way that does not require accessible (named) mutable state, as 
counters or the current element in a _foreach_ loop.

### First Class Functions
Functions are also values, they can appear as parameter and return type of a 
function declaration and be assigned to variables (`where`-clause). Such 
functions are _anonymous_ in the sense that their original name is _lost_.

A function that takes an `Int` and computes a `Bool` has a type of `(Int -> Bool)`.

		fn even :: (Int n -> Bool) = n mod '2 == '0

The `even` function has this type and could be passed as an argument to another
function:

		fn is :: Int n -> (Int -> Bool) f -> Bool = f n

`is` might not be very useful in practice but illustrates well how functions can 
be used as arguments, like in the following expression:

		'1 is even

Function types can be seen as a way to _abstract_ over functions. This kind of
abstraction can also be made first class as operations will shortly show.

### Partial Application
Another situation that results in function types is partial application of
arguments of a function. Any number of arguments can be left out, each not 
applied one is indicated by the underscore `_` _wildcard_.

		(Int b -> Int) inc = '1 + _

[^plus]: given `fn plus [+] ::` `Int a -> Int b -> Int`

Instead of passing an actual argument to `+` the function is partially applied
resulting in a function `inc`[^plus] that takes (or _on_) the left out 
argument `b` which itself than results in a value of type `Int`. 

### Unified Arity
(tuple equivalence of parameters)


## Abstractions

### Operations _(abstract over functions)_

### Behaviours _(abstract over sets of functions)_

### Notations _(abstract over cases)_

### Type Schemas _(Generics - abstract over types)_


## Effects _(a.k.a. Side Effects)_

### Transients

### Streams

### Machines _(Processes)_

### Channels _(Message Passing)_


## Types _(Type System)_
(list sorts of types here)

### Specialisation -- Generalisation

### Type Inference
- mandatory type annotations, inference where unambiguous

#### Variance 

### Shapes

### Formats

## Modules _(Artefacts)_

### Namespaces

### Libraries

## Advanced Techniques

### Abstarct Synatx Tree
So far everything has been described in terms of the surface syntax. This syntax 
builds upon the logical constructs of the language and allows to encode them
more readable and concise. The language is however defined in terms of its 
abstract syntax tree[^why-ast], a general `data`-structure. As such the AST is 
vice versa expressed in the surface syntax in the form of usual expressions.
Tagged forms are used to keep type safety as we will shortly see.

[^why-ast]: The AST and why it is the _model_ of the language deserves an article on its own. Digest: It enables better tooling and allows the surface syntax to focus on expressing the usual well as the _unusual_ doesn't need to be possible to express as one always can escape to the AST. 

To encode any expression directly as an AST elment the AST expression is _tagged_ 
with the atom `` `ast ``. The `plus` function could be implemented as:

		fn plus [+] :: Int a -> Int b -> Int 
			= (`ast (`+ ?a ?b))

The `` `ast `` tag instructs the compiler to treat the 2nd element of the outer
form as an AST element. Here this is again a tagged form implementing addition 
as the operation `` `+ `` bound to the variables `?a` and `?b`. 

The tagged form notations gives AST implementations a lisp like appearance 
that also comes with a lot of the flexibility of a lisp like language. The most 
important one might be that it allows to implement almost everything in the 
language itself. Only the transformation to actual machine code or another 
intermediate representation has to deal with _"native"_ (foreign language) code.
This allows to share bruno code very target plattform and VM independent.


#### Tagged Forms _(Runtime Type Annotations)_
By convention any expression from with an [atom](#atoms) as its first tuple 
element is understood as (type) tagged form. The atom _tag_ is similar to an 
explicit type _annotation_ for that form: 

		(`date '2014 '06 '01)

The form is tagged as a `` `date ``, followed by a date value in three components. 
The tag hints what types are expected/checked for the following elements. The 
binding between tag and type is done itself as a tagged form in the namespace.

		(`use `date Date)

A form _annotated_ with `` `date `` is of type `Date`, that e.g. could be:

		data Date :: (Year y, Month m, Day d)
		
Given the declarations above the compiler is aware of the types in the expression
form ``(`date '2014 '06 '01)`` and treats/checks `'2014` as a `Year`, `'06` as 
the `Month` and so forth.

The technique of tagged forms is used to type generic data structures like the 
AST. As all created tagged forms will be type checked it is
valid to expect any tagged form value encountered to be well-typed according to 
the bound type.
				

### Procedures _("Hygienic Macros", Compile-Time Expansion)_
Procedures are compile-time inlined _functions_, hence do not create stack 
frames and are therefore limited to non-recursive implementations. Otherwise a
 procedure `proc` is similar to a function `fn`:

		proc assoc [=>] :: K key -> V value -> (K, V) 
			= (key, value)

Any type `K` has a procedure `assoc` (or `=>`) that, given a `value` produces 
the pair `(K, V)`. When called, for example to add an entry to a map

		{ 'p' => '3, 'i' => '.17 }

the compiler will expand the implementation of the procedure's AST with the 
actual arguments for each usage.

#### Call-Site Inlining
Any function call to a non-recursive function can be inlined at the call-site
using the _inlining_ prefix `~` in front of the called function's name:

		fn starts-equally :: E.. one -> E.. other -> Bool
			= one ~first == other ~first

To receive the `first` element of each of the lists `one` and `other` the call
is inlined using `~`. Instead of a function call the body of `first` is expanded
into the AST of the body of `starts-equally`. This is in particular useful when
a function is called in a _loop_. 

Usually inlining trades better performance for larger artefact code size as code 
is not _reused_ but _duplicated_. In cases of short functions inlining might even 
result in smaller artefacts. Still compile-time inlining always contains the
risk that different callers of the same function run different versions of it.
On the other hand inlining also can remove a runtime dependency as all uses are
inlined.


### Eager Expressions _(Compile-Time Evaluation)_
Statically computable expressions can be evaluated at compile-time. Instead of
normal grouping parentheses the expression is embedded into an evaluation
group `(:= ` _expr_ `)`. For example `Pi` can be calculated through an algorithm:

		val Pi :: Float = (:= '0.000001 pi-gauss-legendre)

The constant `Pi` of type `Float` is computed by the `pi-gauss-legendre` method
using a precision of `'0.000001`. The evaluation of the _eager expression_ 
`(:= ... )` happens at compile-time, the result is inserted as a constant similar
to a declaration like (maybe more or less digits):

		val Pi :: Float = '3.14159265358979323846

Eager expressions can be used for all pure function computations based on
any form of literal(s). In other words, it works as long as no abstractions
or effects are involved.

### Keys _(Typed "Pointers")_
A _key_ is a constant that does not have a (relevant) type on its own but 
_points_ to a value of a specific type. All keys however are of type `Key`, that
type itself is of less importance.

		val ^points-to-Int :: &Int

Key constant start with a `^` followed by any sequence of characters except 
whitespace and `,`. The type `&Int` is a _pointer_ to `Int`. One can think of
the `&` symbolising a keyhole and `^xyz` as the the key(beard) that gives access.

Like atoms keys are used as part of more _dynamic_ concepts. For example think
of a _multityped map_ or _object_ (in e.g a javascript sense) as a behaviour like:

		op get :: O obj -> &T key -> T?

		op put :: O obj -> &T key -> T value -> O

The `obj` is a _map_ where the type of the value is associated with the key used
to look it up. When supplying a `key` of type `&T` one can expect to get a 
value of type `T?` (that is `T` or _nothing_). Conversely a value of type `T`
can only be associated with a key of matching type `&T` in order to transit from 
the origin `obj`ect `O` to a new `O` object as a result.

Keys use a more flexible naming schema to provide a wide range of possible names 
without "polluting" the single overall identifier namespace. 

		[^a ^Key ^+ ^1 ^[other] ]

The list shows kins of different valid _keys_. In contrast to atoms keys are
declared as `val` constants. Two keys are consequently equal if they point to 
the same constant.
