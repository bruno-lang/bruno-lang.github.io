---
layout: default
title:  "in a Nutshell"
---

# bruno in a Nutshell

bruno is a declarative high level programming language designed for correctness
on the basis of data and pure functions. 
State is restrained in isolated lightweight processes that communicate only by
message passing.

bruno is influenced by ideas found in Erlang, Haskell and Clojure and appears 
to have a philosophical basis similar to that of Virgil III but there is no 
direct predecessor language.

<noindex/>
## Prologue
Basic correctness is still too challenging with both classic and modern
[programming systems](/glossary/#programming-system) while software 
simultaneously attempts to solve problems of continually increasing scale 
accompanied by an equally increase of complexity. 

Mastering complexity however is an utopian fallacy, it has to be avoided in the
first place. A programming system that allows for 
[*operative modularisation*](/glossary/#operative-modularisation)
and [*local reasoning*](/glossary/#local-reasoning) lets programmers succeed 
in dividing and conquering large(r) software systems by composing them out of
many simple components.

To ease reasoning the possibilities must be restrictable to be effectively simplified.
By expressing what is impossible the programmer limits what needs to be
considered and understood. Impossibility is coupled with logical consequences 
that enable possibilities of simplification and optimisation[^why-restrictions]. 

[^why-restrictions]: E.g. pure functions (the impossibility of site-effects) allow
      to derive, reorder or memoize execution, fuse or inline expressions at 
      compile-time; immutable values (the impossibility of mutation) allows to 
      share data without copying it.

By composing systems out of a handful restricted specific constructs (rather than 
unrestricted generic ones) application-, compiler- and VM-programmers share a 
meaningful common language that carries its possibilities and impossibilities
along the stages so that they can be utilised. 
On the contrary the concreteness of the constructs must still maintain the 
ability to abstract ideas intuitively and allow to control low level details.

On these grounds it is an overall theme in the bruno programming system to not 
_extend_ but further and further restrict the more general to the more specific. 


## Format
The language uses sets of more or less independent declarations of the form:

		keyword name [unit] :: declaration-body

* `keyword` what kind of thing is declared?
* `name` what is the identity of the declared thing?
* `unit` how to identify values of this particular thing?
* `::` (is the _is declared as_ operator)
* `declaration-body` how does the thing relate to other things?

There are a dozen types of declarations each using a distinctive _keyword_.
In the declaration body however any valid identifier (including these words) 
can be used. There are no reserved words.

The syntax has no statements or ending marks (like a semicolon) but predefined
sets of character used for operators, literals and forms of expressions. 
While white-space is not significant in general a line-break might mark the
end of some types of expressions.

A single `=` usually stands for: _has the value_ (where value could also refer to
the expression that implements a function).

**Note** that the code examples given are exemplary and use mostly fictional 
types or functions to be more descriptive and familiar to the reader.
The section on [types](#types) goes into the details of the actual build-in 
language base.

## Data
Data is always represented by values, thus is immutable with value equality and
no concept of identity.

### Simple Values
There are two types of simple values, independent `dimension`s and dependent `unit`s. 

		dimension Time [T] :: Int '0..

`Time` is a specialisation of a integral number `>= 0` abbr. as `T`. While the
dimension `Time` describes a kind of value _time_ is more of a concept than a
measurable value. Therefore some _time_ `unit`s are introduced:

		unit Minutes [min] :: Time
		unit Seconds [sec] :: Time

Both `Minutes` and `Seconds` are units within `Time` dimension measured in `min`s
and `sec`s. Units of the same dimension can be related using `ratio`s:

		ratio Time :: SI = {
			'1min = '60sec,
			'1sec = '1000ms 
		}

		dimension SI :: ()

A `Time` `ratio` within the unit system `SI` declares a  minute `'1min` as 
`'60sec`, a second `'1sec` as `'1000ms` (and so forth). 
The `SI` dimension models a unit system used as a fix point to group ratios
within the same system.
Literals use a `'` prefix and are typed through their unit of measure.

The presence of a unit system with `ratio`s makes it unnecessary to declare or 
explicitly apply conversion between values within the same system.

		Milliseconds three-minutes = '2min + '60sec

The duration of `'2min` and `'60sec`, values of type `Minutes` and `Seconds` can 
directly be used where e.g. `Milliseconds` are needed as the ratio between these 
is known.

### Composite Values
Composite values (often loosely equated with records or structs) are declared as 
`data` types.

		data Rect :: (Length width, Length height)

A `Rect` is a tuple with two `Length` fields, `width` and `height`. While all 
types are nominal they can be used as (structurally typed) tuples. A `Rect` is 
also a tuple of type `(Length, Length)`.

Finally there are collection types for arrays/vectors of elements of identical 
type with fixed length:

		data Code :: Char[8]
		data RGB :: Colour[3]

A `Code` is vector of `8` `Char`acters. A `RGB` colour one of `3` `Colour`s,
where the length is part of the array's type. A length can also be within a
range.

		data UTF8CodePoint :: Byte[1-4]

A UTF-8 code point is `1-4` `Byte`s. Length or range furthermore allow the use of
a wildcard `*` to indicate _any unknown_ length.

		data Nonempty :: Int[1-*]

An array is `Nonempty` when it has at least `1` up to any `*` number of elements. [^no-dependent-typing]

#### Collections
Two collection types are supported with special syntax although conceptually 
they aren't data types but [behaviours](#behaviours).
In contrast to the fix length of arrays collection are of a variable length 
that is not reflected on type level.

A `Sentence` is a _list_ of `Word`s of variable length. 

		data Sentence :: [Word]

[^no-dependent-typing]: Note that the [formalism](/glossary/#formalism) is not dependently typed. Its 
    concept of (type) shapes is related but less open.

Furthermore sets are supported with special syntax.

		data Team :: {Member}

A `Team` is a _set_ of `Member`s. Maps are nothing else than sets of tuples:

		data Dict :: {(Word, Translation)}

A `Dict` is a set of `(Word, Translation)` twin-tuples (pairs).

### Views
Slicing arrays `[*]` creates a slice `[:*]`. 

		Char[:2] he = "hello world" slice '0 '2 

By `slice`ing a section of the string `"hello world"` starting at positon
`'0` with a length of `'2` the varibale `he`, that is a slice of two
characters `Char[:2]`, refers to the first two characters of that string,
namely `"he"`.

A slice is a view upon a section of a already immutable underlying array
extended with the information about the position of the slice within the 
originating array and the ability derive slices upon the same underlying array
that are moved or changed in length.

### Enumerations
Both simple and composite value types can be initialised with a list or set of
possible values to model enumerations. Classic enumerations are 0-tuples:

		dimension Bool :: () = [ False, True ]

`Bool`eans are derived from unit `()`, allowing for two values: `False` and `True`
where in case of a list `False` will also be associated with index `'0` and
`True` with index `'1`. Dimensions based on unit can also omit the `()` as in:

		dimension Fruits :: = { Apples, Pears }

The possible `Fruits` are `Apples` and `Pears`, this time with _set_ semantics. 
Composite types can be restricted to an enumeration in a similar way:

		data Planet :: (Kilograms weight, Meters radius) = { 
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

Enumerations can also be used as dimension type of arrays.

		data Menu :: Meal[Weekday]

Assuming `Weekday` is a enumeration having `Monday`, `Tuesday` and so on `Menu`
is an array type of fixed length 7 that is index accessed using the enumeration
constants: `Meal on-monday = menu at Monday`. Index access has no special syntax
and uses a function (`at`) like all other operations.

### Constants
A constant is a named `val`ue.

		val Pi :: Float = '3.14159265359

`Pi` is a scalar number of type `Float` having the value `'3.14159265359`. Of 
course constants can be of any type initialised with any statically resolvable
expression using functions and any type of literals.


### Literals

##### Conceptual Literals
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

##### Binary Literals
Secondly binary oriented literals typically used for simple numerical values:

		Colour red = #xFF0000
		Int mask = #b10101010

Binary oriented literals start with a `#` followed by the type, `x` for hexadecimal, 
`b` for binary, but also `o` for octal and even `d` for decimal values, what is also
the default and can be left out.

		Int ten = #10

##### Textual Literals
The third type of literals are textual oriented. Composite values are given in a 
way humans are used to write them:

		String msg = "hello world!"
		Point p = "1:2"
		Date imagine = "1971-10-11"

The `Point` and `Date` example illustrates that textual literals can be used for 
any composite type (when these declare a text-[format](#formats)). 
The type of textual literals is inferred from the 
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

#### Atoms _("type-less" constant literals)_
Atoms are dimensionless values, conceptually zero-tuples of type `Atom`. 
Different constants are defined by `` ` `` followed by any sequence of 
characters except white-space.

		[`a `atom `+ `/ `-> ]
		
The above is a list of valid atoms. Any two atoms are logically equal if their 
sequence of characters are. 
Atoms do not have a value besides being equal to another atom or not. 
They are therefore mostly used as keys or _tags_.
As such they are important for the declarations of the language's AST.


### Initial Values _(Default Values)_
There is no _null_ pointer/reference/value. All simple values are initialised 
with zero or the lowest possible value should they not be specified otherwise. 
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
(within each module this operator _alias_ has to be unambiguous).

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

		fn min :: Int a -> Int b -> Int =
			a < b : a
			      : b

The `min`imum of two values `a` and `b` is `a` in case `a < b` otherwise `b`.
The cases `condition : expr` are checked in order of appearance. The last case has to
be the universal (true'ish) condition. For enumerations alternatively all
values can be covered. 

		fn show :: Bool b -> String =
			False : "false"
			True  : "true"

Each case is implemented by an expression (the formalism has literally no statements).

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
The primary way to _loop_ is to use build in operators that work on arrays in
the fashion of
[APL](http://en.wikipedia.org/wiki/APL_%28programming_language%29) like 
languages like [K](http://en.wikipedia.org/wiki/K_%28programming_language%29). 
The exact operators and syntax haven't been decided yet. 

The secondary way to _loop_ is to use recursive functions:

		fn factorial :: Int n -> Int =
			n == '0 : '1
			        : n * factorial (n - '1)

All tail recursive functions will use tail call elimination, thus not create/use
stack frames. Due to referential transparency and _annotated_ commutativity of 
appropriate functions the multiplication in the above example could also be 
written vice versa `factorial (n - '1) * n` and still be executed without 
consuming stack frames as `*` is such an _annotated_ operation.

### First Class Functions
Functions are also values, they can appear as parameter and return type of a 
function declaration and be assigned to variables (`where`-clause). Such 
functions are _anonymous_ in the sense that their original name is _lost_.

For example, a function that takes an `Int` and computes a `Bool` has a type 
of `(Int -> Bool)`.

		fn even :: (Int n -> Bool) = n mod '2 == '0

The `even` function has this type and could be passed as an argument to another
function:

		fn is :: Int n -> (Int -> Bool) f -> Bool = f n

`is` might not be very useful in practice but illustrates well how functions can 
be used as arguments, like in the following expression:

		'1 is even

Function types can be seen as a way to _abstract_ over a functions and see it as
a member of a _function familiy_. This kind of abstraction can also be made 
first class as operations will shortly show.

### Partial Application
Another situation that results in function types is partial application of
arguments of a function. Any number of arguments can be left out, each not 
applied parameter is indicated by the underscore `_` _wildcard_.

		(Int -> Int) inc = '1 + _

[^plus]: given `fn plus [+] ::` `Int a -> Int b -> Int`

Instead of passing an actual argument to `+` the function is partially applied
resulting in a function `inc`[^plus] that takes (or _on_) the left out 
argument `b` which itself than results in a value of type `Int`. 

### Arity Equivalence 
Excursion: Any simple value is identical to the 1-tuple of that simple value. 
For example a value of type `Int` is identical to the type `(Int)`.
Similarly a `[Int]` is `([Int])` or a `{Int}` is also `({Int})`.

A function on the other hand can have any number of parameters. However, another
way to look at it is to assume that all functions just have one parameter being 
a n-tuple of the multiple elements. 

		fn multi :: Int a -> Int b -> Int c -> Int
		fn uni :: (Int, Int, Int) abc -> Int 

Both `multi` and `uni` compute an `Int` from three inputs of type `Int`.
They have a functionally identical signature. This does not mean that everything
is normalised to one (multiple simple parameters) or the other (one tuple 
parameter) but that an variable amount of parameters can be expressed through
its tuple equivalent. 



## Abstractions
Abstractions are forms of indirections that allow to formulate code on more
abstract levels. Code duplication is avoided at the costs of a minimal runtime
overhead.

### Notations _(abstract over cases)_

### Type Families _(abstract over types)_

### Operations _(abstract over functions)_
Operations are used to abstract over the meaning of a group of congeneric 
functions. This meaning is expressed through the operation's name.

An operation `op` is an abstract or virtual nominal typed function. 
Most often operations are used in conbination with a type variable as the 
target type is abstract as well. 

		op eq :: (T one -> T other -> Bool)

The `eq` operation (`op`) is a virtual function of type `(T -> T -> Bool)` where
`one` and `other` are of the same actual type substituted for the 
type variable `T`. 

#### Polymorphism à la carte
Operations are implemented by functions of matching type through a explicitly 
declared specialisation binding validly within the declaring context of a 
library or module.

		(`auto equal-ints eq [Int])

The [tagged form](#tagged-forms) `` `auto `` declares the automatic (implicit)
specialisation of the function `equal-ints` to the operation `eq` where the 
_type variable_ `T` is of actual type `Int`. 
So in this example `equal-ints` has to be similar to:

		fn equal-ints :: Int a -> Int b -> Bool = a == b

Operations can also be bound in the `where`-clause of a function.

		where
			equal-ints ~> eq Int

The function `equal-ints` is specialised to the operation `eq` for type `Int`
within the body of the `where`-clause's function.

On the use-site operations are not declared as explicit parameters but passed 
implicitly as existential type constraints expressed as part of a type family.

		family E :: _ with eq

Any type (`_`) for which an implementation of the `eq` operation exists 
in the usage context is a member of the type family `E`. 

		fn index-of :: E[*] arr -> E sample -> Int pos -> Int? =
			pos >= arr length    : ()
			arr at pos eq sample : pos
			                     : arr index-of (e, pos + '1)

The function `index-of` is implemented using the operation `eq` to compare the
`sample` with the actual element `e`. Because `E` is constraint to types for
which the `eq` operation exists it can be used in connection with `E`
like a usual function on `E`. When using `index-of` the actual comparison
can be chosen by the function bound to the operation in the caller's context.

		fn contains :: Char[*] arr -> Char sample -> Bool 
			= arr index-of (sample, '0) is-something?
		where
			equals-ignore-case ~> eq Char

`contains` compares the characters on the basis of the `equals-ignore-case`
functions that takes the role of `eq` operations for `Char` values so that it is
used as implicit argument when `index-of` is called.

### Behaviours _(abstract over data)_
Like operations give meaning to functions a `behaviour` gives meaning to a group
of functions that have particular interaction semantics. The behaviour describes
an abstract data type (ADT) by the set of supported operations.
While this abstracts their behaviour the actual data types still are persistent 
data structures.

Given the three operations `push`, `peek` and `pop`, 

		op push :: S stack -> E e -> S
		op peek :: S stack -> E?
		op pop  :: S stack -> S

where both the _stack_ `S` as well as its _elements_ `E` can be of any type 
(`_`),

		family S :: _
		family E :: _

a `Stack` is declared as a `behaviour` having the three operations.

		behaviour Stack E :: = { push, peek, pop }
		
The `Stack` captures the type variable of the _elements_ `E` so that it can
be parametrised and thereby further specialised when used. 

With the behavioural _contract_ in place a `Stack` is used through type families. 
Assuming code in another module declares two fresh type variables 
(here named `S1` and `E1` to avoid confusion with `S` and `E`):

		family E1 :: _
		family S1 :: _ as Stack E1

Any type (`_`) that in known to _behave_ `as` a `Stack` is a member of the
type family `S1`; thus `S1` can be used as `Stack` of elements of type `E1`. 
The fresh type variable `E1` is substituted for the earlier captured variable `E`.
That mean `E1` must be sufficient to satify `E` but can be more specialised.

On the basis of `S1` and `E1` functions can be declared using the `Stack` 
operations.

		fn push? :: S1 stack -> E1 e -> S1 =
			stack peek is-nothing? : stack push e
			                       : stack

The function `push?` just pushes the element `e` to the `stack` if the stack is 
empty, otherwise the stack is left unchanged. `S1` is known to have the `Stack`
`behaviour` so `push` and `peek` can be used as if they were function declared
on type `S1`.

Practically any type that has matching functions can get the behaviour of a 
`Stack`. This is thou (as always) declared explicitly for a module or library:

		(`auto Stack { 
			array-push => push, 
			array-peek => peek,
			array-pop  => pop })

The `` `auto`` -matic specialisation to `Stack` is declared by mapping the
actual function (here `array-*`) to the operation from the `Stack` behaviour.

The actual types that can be specialised to a `Stack` depend on the type(s)
stated in the mapped functions. If multiple function of same name are in the
name-space the function is qualified with the containing module or library name.

A behaviour's actual implementation consists of functions. 
Here `push` on basis of an array:

		family E :: _
		fn array-push :: E[] stack -> E e -> E[]
			= stack prepand e

The `array-push` function has a compatible signature to the operation `push`.
`S` becomes the actual type `E[]`, the actual `E` is still any (`_`) type.
As `array-push` is equivalent to `prepand` the mapping could very well also 
used `prepand` directly. 




## Effects _(a.k.a. Side Effects)_

### Transients

### Streams _(IO)_

### Channels _(Message Passing)_
A channel is a typed queue of fix or variable length. Each channel transports
values of a particular simple or composite data type. 

		TODO how are channels created...

While each channel has an input and an output end a channel type is either
an input `[<]` end or an output `[>]` end.

		Message[<] input
		Message[>] output

Usually channels are of fixed length and block on send or receive if the channel
if full or respectively empty. The second type of channel is non-blocking 
indicate by square brackets that are open to the outside.

		Message]<[ non-blocking-input
		Message]>[ non-blocking-output

Finally a channel type can also express that the blocking behaviour of the 
channel is unknown or unimportant indicated by the square bracket being open
in line with the arrow.

		Message[<[ may-block-input
		Message]>] may-block-output

The different types of channels allow to write functions that work just for
blocking (`M[>]`,`M[<]`) or non-blocking (`M]>[`,`M]<[`) channels or both of 
them (`M]>]`,`M[<[`) and with particular value types or wide range (possibly any)
type using a type variable like `M` would be one.

#### Cross-Channel-Synchronisation
As channels could block a situation naturally arises where a value should be
send to the first of a couple of channels that accepts the value or where
a value should be received from the first of a couple of channels that has
a value available with the guarantee that when it happens only one channel is 
affected. This is called a _select_ operation.

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
In non of the channels directly accepts the value (they all blocked waiting)
the `value` is send to the first channel accepting it with no particular priority
should multiple channels accept a value _simultaneously_.
In any case the operation returns the channel that accepted the value.
The value could also be different for each alternative. 

#### Time-outs
When a blocking send or receive should time out a _select_ is used together with
a type of channel that accepts/offers a value after a certain time. 

		Int value
			|= channel <<
			|= '7 after '5ms

The `after` procedure creates an ad hoc channel that offers `'7` after `'5ms`.
To synchronise time-outs for multiple parallel processes a common _time-out_
channel is used as alternative for all of them. 


### Processes
A process is a isolated lightweight thread of execution exclusively connected 
to the _outside world_ by channels (and indirectly streams). 
It is the only construct to model behaviour around enduring state. 
The state of a process is always local to that process. 
Each process is a state machine that decides when to receive from or send 
messages to channels. 

A `process` is declared in terms of the possible state transitions. 

		process Server :: = { 
			Ready => [ Ready ], 
			    _ => [ Ready ] 
		}

A `Server` process has one state `Ready` that only can transit to itself.
Any other (error) state (`_`) transits to `Ready` as well. A `process`
declaration encapsulate a behavioural pattern that can be used for many concrete
automata. 

The state `data` of a concrete process could look like the structure `HttpServer`.
It holds channels to communicate with the _outside world_.

		data HttpServer :: (
			HttpRequest[<] requests,
			HttpResponse[>] responses,
			(HttpRequest -> HttpResponse) app
		)

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

#### Error Handling
Faults or Exceptions that occur during a state transition automatically 
continue with the matching error state transition. 
These use the type of fault instead of a state constant. For example:

		when Out-Of-Memory! :: P p -> P?
		..

Should a `Out-Of-Memory!` error occur the process `p` does not continue (`..`)
with any state what indicates the termination of the process. 
The optional type `P?` indicates on type level that this might be the outcome 
of the transition. 

Error handlers make it easy to model robust enduring and self-repairing 
processes or fragile daemons with task specific tear-down behaviour.

For a generic error handling the actual `error` could also be passed to the 
state transition by adding it as an extra argument.

		when _! :: P p -> E! error -> P?

Any error `_!` is handled by the `when` transition that might result in a
successor state `P?` or _nothing_ depending on the actual `error`.

#### IO-Devices
In a system build out of processes and channels input devices like a mouse or a
keyboard become endpoints (processes) that feed data into a particular channel. 
Sockets become types of channels and so forth. 
From the point of view of a program composed out of processes there are just 
channels and streams to affect the _outside world_.


## Modules _(Artefacts)_

### Libraries

## Types _(Type System)_
(list sorts of types here - type constructors)

### Specialisation -- Generalisation _(Type Polymorphism)_
The bruno type system uses a type polymorphism that is crucially different
from textbook subtyping (interface inheritance) or OO like inheritance 
(implementation inheritance). 
Therefore it is important not to think in these familiar terms/semantics but to 
develop an new, independent understanding. 

Fundamentally four kinds of types are to be distinguished:

* Value Types
* Function Types
* Effect Types
* Abstraction Types

For simplicity the following explanations should be associated with value types
that are the basis for everything else. 
The semantics of function types are a consequence of value type semantics.
Effect- and abstraction types have mostly slightly modified value semantics.

Two (value) types can be related in such a way that 
a value of a _subtype_ is also usable as a value of the _supertype_.
When a value of a _subtype_ is used as a value of its _supertype_ it is 
conceptually substituted with an _equivalent_ value of the _supertype_. 
This means a value instance never _acts_ as a value of its _supertype_ but 
literally _becomes_ such a value.

As this is different from the semantics usually associated with the terms 
_subtype_ and _supertype_ this novel kind of relation is called 
_specialisation_ / _generalisation_. A _derived_ type is said to be a 
specialisation of a more general type - or vice versa - any type another type
is derived from is said to be the generalisation of of that type. 

The fundamental difference is well illustrated by the observation that types are 
related in a _hierarchy_ (more precisely in a 
[DAG](http://en.wikipedia.org/wiki/Directed_acyclic_graph)) while values
(actual instances) do not have such relations. A value of formal type `A` is 
always an instance of actual type `A`. Consequently formal and actual value 
types are always identical, a distinction is unnecessary.

In contrast to implementation inheritance a specialisation is a strictly more 
_limited_ type than its generalisation. It can neither add state nor can it 
widen the range or change the nature of possible values. 
The set of all possible values of a specialisation `B` is a strict subset of 
the set of possible values of a generalisation `A`. 
Yet any value in the set the of more special type `B` can appear (at least) in 
two flavours, that of `A` and that of `B`. The data of such a value however 
is indistinguishable for `A` and `B` but it might be typed as `A` in one context 
and as `B` in another as given by a program. 

When `B` is generalised to `A` its value (instance) is simply passed along as 
`A`. It effectively _became_ an `A` and will be treated as such from then on. 
Whereas a specialisation from `A` to `B` the value needs to be checked for the 
more limited specification of `B` before it can be passed along as a `B`. 
Actual allocation or instantiation of a new value is not necessary as values do
not need or have a reference to their type since it is statically known from 
the formal type declaration.
So while any `B` can become a `A` it is unclear if a `A` can become a `B`. 
Therefore a generalisation from `B` to `A` is a implicit _noop_ but 
a specialisation from `A` to `B` has to be _demanded_ explicitly as it could 
fail at runtime. 

It could be said that types _carry_ static properties of values. A value that 
has been proven to conform to the properties of a type can be specialised to
that type. Type conversion however is only allowed between values of related
types that are structurally compatible. That means a specialisation can be
tried for any two types that have a common generalisation type and are not
known to have mutual exclusive value sets.

Similar to nominal subtyping a type relation is explicitly declared. 
While declared types are always named types a _abstract_ generalisation of 
any named type is its _anonymous_ structure that can be expressed as a 
structural type as well.

To put it yet another way, types give meaning to bits and bytes of data
structures. Data structures of the same form can be meaningful in different
ways what is reflected through different, often related types. 

A _universal_ top type does not and cannot exist as it would have to be
structurally compatible with all other, structurally different types.

- strictly speaking it is wrong to say that one type can be assigned to another
  as one values of the exact same type are compatible; but other types might be
  possible to be converted into the demanded type.

#### Variance

#### Runtime Type Information

### Type Inference
- mandatory type annotations, inference where unambiguous

### Shapes
Types and values are not two independent universes. 
There are several aspects where types and values blend into each other; 
this should not be confused with dependent typing.

(a form of latent typing but only for constants or more general statically decidable properties)

### Formats


## Advanced Techniques

### Abstarct Synatx Tree
So far everything has been described in terms of the surface syntax. This syntax 
builds upon the logical constructs of the language and allows to encode them
more readable and concise. The language is however defined in terms of its 
abstract syntax tree[^why-ast], a general `data`-structure. As such the AST is 
vice versa expressed in the surface syntax in the form of usual expressions.
Tagged forms are used to keep type safety as we will shortly see.

[^why-ast]: The AST and why it is the _model_ of the language deserves an article on its own. Digest: It enables better tooling and allows the surface syntax to focus on expressing the usual well as the _unusual_ doesn't need to be possible to express as one always can escape to the AST. 

To encode any expression directly as an AST element the AST expression is _tagged_ 
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
This allows to share bruno code very target platform and VM independent.


#### Tagged Forms _(Runtime Type Annotations)_
By convention any expression from with an [atom](#atoms) as its first tuple 
element is understood as (type) tagged form. The atom _tag_ is similar to an 
explicit type _annotation_ for that form: 

		(`date '2014 '06 '01)

The form is tagged as a `` `date ``, followed by a date value in three components. 
The tag hints what types are expected/checked for the following elements. The 
binding between tag and type is done itself as a tagged form in the library or 

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

		{ 'p' => '3, 'i' => '0.17 }

the compiler will expand the implementation of the procedure's AST with the 
actual arguments for each usage.
Note that functions passed to a procedure will not come at the expense of a
virtual function call as the actual function is substituted by the compiler.

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
is not _shared_ but _duplicated_. In cases of short functions inlining might even 
result in smaller artefacts. Still compile-time inlining always contains the
risk that different callers of the same function run different versions of it.
On the other hand inlining also can remove a runtime dependency when all 
dependent usages are inlined.


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


### Keys _(Typed "Paths")_
A _key_ is a constant that _carries_ the type of values that are accessed using
that key. The keys themselves are of a uniform type `Key`, that is of little 
importance.
Each key is an abstract path to values rather than a reference or pointer to 
a specific location or value. 

		val @refers-to-Int :: Int

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

		[@a @key @+ @1 @[other] ]

The list shows kinds of different valid _keys_. In contrast to atoms keys are
declared as `val` constants where the declared type is the type of the value the
key leads to. 
Two keys are consequently equal if they are the same constant.


## A System of Systems

### Modular Software
- where can we declare code? (where/who does declare -> use site vs declaration site)

## Epilogue

- a computer system that
	- does not crash, 
	- can tell why value on what function caused an error
	- makes it easy to collect all such errors, and 
	- to find and fix known errors through local reasoning

- programming paradigms do not matter - just the properties of the system we work with
