---
layout: default
title: Elder Core
description: Core language
permalink: core
---

Core is a shorthand for the core operation of the Elder language and is a implementation of the Elder syntax.



I was inspired to begin real work as I began using Red-lang and it's ability to work across the entire stack but felt it need not be constrained to the functional programming pardigm. It seemed at least plausable to create a language w/ flexible syntax and semantics which could more approach full-stack development w/ less visual noise and more consistency than most languages are capable of.

The syntax of Elder is primary inspired by:
* the clarity and elegance of Python
* homoiconicity of LISP and Rebol and the various formulations of the syntax from S-exp, M-exp, Sweet-exp, O-exp, ...
* operator definition and extension of Fortress
* terseness of tree modeling (whether HTML, CSS, etc.) of Pug, SASS
* data serialization and literal representations of StrictYAML and SDLang
* TODO: more?

## Equals Syntax
---
`=` captures the idea of relating a ***L-hand*** to a ***R-Hand***. The type of relationship varies with the context it's defined in.

TODO: Disjoint contexts used in?

`=` is defined as you expect in most languages but is more flexible and widely used:
* Only defined as a `binary operator` and `infix fixity`
* Used in many locations to describe assignment, mapping between names and values, key-value entry, etc.
* Semantic meaning depends on what context it's defined

TODO: Relate this to examples in other languages!!!

## Relator

Assign and destructure multiple values
```
// Use fully qualified
id, class = html/body/header.id, html/body/header.class

// or, select only the values you need
id, class = html/body/header.(id, class)

// or, assign w/n
obj/my-new-header.(id, class) = html/body/header.(id, class)
```

### Assigment Ambiguities
`=` is used in multiple contexts where it means different things. For example:
* In a Block, it is assignment
* In a Map, it is a mapping between a key and a value
* In a Function Argument List, it is a default value
* In a Condition, it is equality test

Here's an example to demonstrate:
```
x = 4
```

By reusing this code in multiple contexts we can see that it's meaning varies based on context. Here's some psuedo-code examples:

In a ***Block*** the example is interpreted as a assignment of `x` to value `4`.
```
{
  x = 4
}
```

In a ***Map*** the example is interpreted as mapping between a key `x` to a value `4`.
```
Map
  x = 4
```

In a ***Function Argument List*** the example is interpreted as a sequence of inputs to a `Fn` with argument `x` with default value `4`.
```
my-fn = Fn (x = 4) -> Int
```

In a ***Condition*** the example is interpreted as `test if x equals 2`.
```
if (x = 4) ...
```

No one likes ambiguities but the alternatives are worse. Ultimately it was decided that ambiguities are fine at the level of syntax as disambiguation is enforced by semantics not syntax. Further discussion about this decision and alternatives considered can be found here (TODO: link to blog article).

## Shadowing
---
Shadowing can cause issues when it's done w/o expecting it. This is especially true in languages like CoffeeScript and Python where adding names to an outer scope will affect inner scope.

There's been several approaches by different languages to make this clear:
* JavaScript, Java, etc. use keywords like `var`, `const`, `let` for declaration and just allow to redeclare w/n a block
* Go is much like JavaScript-ilk but additionally has a shorthand for declaration using `:=`
* Python will always use names from an outer scope using `nonlocal` and `global` to disambiguate when to shadow
* Zig doesn't allow it at all (this is probably the safest option and b/c it's goal is system programming it makes sense to prioritize safety but not w/o it's downsides)
* Almost like Zig, Coffeescript generally doesn't allow shadowing but is less consistent b/c it allows function parameters to shadow

From these and other solutions in other languages, here's the problems that arise:
* Declaring a name in a locally, then later, someone uses the same name in outer scope which changes meaning of local name to assignment
* Declaring a name in a outer scope, then later, one doesn't see the outer name (eg a long file or deeply nested) and resuses the name assuming it's not used

Additional to these, we have a few more goals generally:
* Shadowing is significant enough should be an explicit choice by the developer, not implicit
* The most common, least complex, and desirable use-case should have the most minimal, terse syntax. The less common, more complex, and less desirable should have more explicit syntax.

With all this, here's what we've chosen:
* In order to declare a name the least amount of syntax is using `=` (eg `x = 4`).
  * This remains as a way to declare a name as well as assign. Most use-cases follow this flow.
* If one choses to shadow, you must explicitly declare it using `:=` (eg `x := 4`).
  * This is read as "declare new name x and define as 4".
  * If `x` was not previously defined it will allow it as the intent is to isolate `x` to the current scope.
* Elements that implicitly declare a new scope (eg `Functions` and `Blocks`) either must not shadow (ie must declare unique names) or must declare it's intent to shadow (using `:=`)
* The same name under the condition that it's knowable at compile-time that they won't collide like:
  ```
  Block
    x = 1

  Block
    x = 2
  ```

## Juxtaposition
---
In order to have terse and less noisy syntax, we chose to support juxtaposition (meaning next to one another).

Like Fortress, juxtaposition has to be defined for specific cases (types, rules, etc.)
* for functions, this is **always** application
* for numbers, this is multiplication but can be overridden in contexts (explained later)

This makes representing complex mathematical notation w/ less noise
```
2 x sin 3 y + 4 tan 8 z 2
```

which is equivalent to
```
2 * x * sin(3 * y) + 4 * tan(8 * z * 2)
```
TODO: Must differentiate precedence between `sin` and `3 y` in `sin 3 y` and precedence in general.

## Evaluation
---
Evaluation is left mostly to the developer to specify (much like languages like Haskell, Rebol, and Red). This means for many cases that a developer would need to implement a macro it's not necessary.

There are only a few cases where evaluation will occur:
* within a Block on the r-hand of `=`
* within Function parameters the r-hand of `=` if there's no supplied value in order to get the default value
* the following `Block` after a `do` or `#do`

Otherwise evaluation is left for the developer to specify.

## Precedence
---
There are 3 levels of precedence: global, local, and custom:
* Global rules define the high-level patterns to assure that all syntax generally acts like one would expect
* Local rules refine (but can't break) the global rules and allow enough flexibility to implement custom operators, functions, and some macro-like operations
* Custom rules are used for everything else like domain-specific changes, new programming paradigms, etc. The custom rules are the most flexible and most dangerous; generally they should be avoided.

The global rules are:
* Precedence follows
  * `()` are top-level precedence
    * eg expressions like `(3 + 4) * 5`
    * eg explicit calls like `f(3 + 4, 5, 6) + 8`
  * Selection (TODO: use better term like traversal or browse?)
    * eg `x.y.z!` is `((x.y).z)!` not `x.y.(z!)`
  * Attached operators (ie no spaces) regardless of fixity
    * eg `-3 + 4` is `(-3) + 4` not `-(3 + 4)`
    * eg `3 + 4!` is `3 + (4!)` not `(3 + 4)!`
    * TODO: `-3!`? and why not `!-3!` (must have rule for not stacked or force users to `()` it?)
  * Infix w/o parenthesis
    * `f 3, 2 + 3, 5` is ???
    * `2 + f 3, 4, 5` is ??
    * `f 2 + g 3, 4, 5` is ???
  * Juxtaposition prefix w/o parenthesis
    * These associate right until the end of the line and explained more below
    * eg `f x, h y, g z` is `f(x, h(y, g(z)))`
  * ...TODO rest...
  * Juxtaposition sibling
  * Juxtaposition keyword function
  * Unary keyword function
  * UFCS
  * Equals
  * Comma
  * Semicolon

Local rules are specific to a domain. When mixing domains, developers must either rely on global rules or use parenthesis. The local rules are isolated to domains like:
* Arithmetic (`+ - * / ^`)
* Logic (`and or not for-all at-least-one`)
  * Propositional
  * Predicate
* Bitwise (`>> << ~ & |`)
* Comparison (`> < >= <= != ==`)
* 

TODO: What about concepts which span multiple domains? eg equality is local to many domains (like comparison, identity) and a general concept. Although implemented in each, this still shouldn't allow them to relate to one another and that should be clear.

### Example - Nested functions
Goal is to simplify:
```
2 * x * sin(3 * y + 4) + cos(3 / z)
```
this can be simplified using precedence rules.

since juxtaposition is often multiplication we can remove most of the `*`:
```
2 x sin(3 y + 4) + cos(3 / z)
```

using 2 rules:
* function application using juxtaposition groupes until end of line
* parenthesis are terminated at end of line w/o indentation
```
2 x sin(3 y + 4) + cos 3 / z
```

can use `;` to terminate a keyword function input:
```
2 x sin 3 y + 4; + cos 3 / z
```
the `;` informs `sin` when to stop reading for input.

### Operators
Consider using `,` `;` and `.` to control precedence (and thus sequencing) as operators?





## Functions
---
// Functions, operators (eg `+ - !`), and relation-spaces (eg `x.y, x/y, x:y`)

Operators are functions with a few additional rules:
* Operators are only allowed to use specific character (like `+ - !`) which are only punctuation and don't conflict with relation spaces
* Operators and relation-spaces are allowed to be attached to an identifier (eg prefix `!x` or postfix `x!`) as an identifier isn't allowed to start or end with punctuation
* TODO: Operators have different fixity and must be coered to act differently?
* TODO: Repeated operator is new operator or applied twice? eg `!!x` vs `--x`; notice these aren't consistent in most languages

TODO: Rules for relation-space vs operator vs function vs variables vs literals
* also need syntax to group together (eg date or name w/ whitespace)
  * ideas
    * use escape like `staff = /billy\ bob = 27, /jane\ johnson = 46`
    * use code literal ``staff = /`billy bob` = 27, /`jane johnson` = 46``
* punctuation: ``! @ # % ^ & * - _ + = ` ~ ( ) [ ] { } | \ : ; " ' < > , . ? /``
  * Remember whitespace terminates a identifier and longest matches first (ie `##` matches before `#`)
  * Operators and relation spaces share these punctuation characters
  * Some are reserved (remember reserve exactly as operators don't stack like `!!x != !(!x)`)
  * Operator and relator both reserve some characters to avoid common confusion:
    * syntactically reserved ``
    * operator reserves `+ - * > < >= <=`
    * relator reserves `. :`
    * they share `/`
    * available ``
* 

TODO: Ways to convert between forms
* eg binary can be used for UFCS, chaining, infix, pipe/chainable, juxtaposition

TODO: Function parts
* How to handle optional `config` and `args`? degrades to one always? must be explicit?

TODO: Issues w/ diff call syntax implies different semantics and generally not consistent
* eg `1 /(2, 3, 4)` can mean:
  * 1 / (2 * 3 * 4)` or `1 /2 /3 /4` (child literal) or `1 / 2 / 3 / 4` (distribute `/`)
  * How does one know?
* Similar issues for reused syntax which is common and used in different contexts like `-`, `/`
* Cases:
  * unary attached operator (`x!`, `-x`, `!x`)
  * unary prefix (`not x`)
  * infix (`x - 4`, `a and b or c`)
  * prefix juxtaposition (`f x, y, z`)
  * prefix () (`f(x, y, z)`)
  * relation space literal (`obj /(a, b, c) .(x, y, z) :(1, 2 ,3)`)
  * relation space selection (`x/a.b:(c, d) = 1, 2`)
* Order of match
  * function
  * operator
  * relation space

Nim UFCS:
```
type Vector = tuple[x, y: int]
 
proc add(a, b: Vector): Vector =
  (a.x + b.x, a.y + b.y)
 
let
  v1 = (x: -1, y: 4)
  v2 = (x: 5, y: -2)
 
  v3 = add(v1, v2)
  v4 = v1.add(v2)
  v5 = v1.add(v2).add(v1)
```

in Elder
```
// TODO: Need to clarify derived class?
Vector = Tuple:type(x:Int, y:Int)
Vector = is-a(Tuple)(x:Int, y:Int)
Vector = Type is-a Tuple of x:Int, y:int

add = Fn (a:Vector, b:Vector) -> Vector
  a.x + b.x, a.y + b.y

v1 = (x = -1, y =  4)
v2 = (x = 5,  y = -2)

v3 = add(v1, v2)
v4 = v1.add(v2)
v5 = v1.add(v2).add(v1)
v7 = v1 |> add v2 |> add v3         // Pipe w/ partial evaluation
v6 = v1 |> add(_, v2) |> add(_, v3) // Pipe w/ placeholder '_'? How do know into first place? should be able to target? Look at lodash (default) and then more syntax to customize?
```

* Different types of call syntax
  * Multiline
    ```
    f
      x
      y
      g
        a
        b
        c
      z
    ```
  * Prefix, juxtaposition
    ```
    f x, y ,z                       === f(x, y, z)
    f g h x                         === f(g(h(x)))
    f x, g y, h z                   === f(x, g(y, h(z)))
    f a, b, g i, j, h x, y, z, k, c === f(a, b, g(i, j, h(x, y, z, k, c)))
    f a, b, g i, j, h x, y, z; k; c === f(a, b, g(i, j, h(x, y, z) k) c)
    ```
  * UFCS
    ```
    obj/child.f(x, y, z) === obj/child.f x, y, z
    obj/child.+(1, 2, 3) === obj/child + 1 + 2 + 3
    ```
  * open-paren
    ```
    f( x, y
    f(( x, y
    ```
  * infix
    ```
    1 + 2 + 3
    ```
  * attached operator
    * prefix: `!x`
    * suffix: `x!`, `x++`
  * Relation space select
    ```
    obj/child.x:(y, z) = 1, 2
    ```
  * Relation space literal
    ```
    p .class.type.(x, y) = 1, 2

    html/body/header/content.style.
        width  = 100%
        height = 20px
    ```
  * Derived, bind, on-action, etc.?
* Arity
  * nullary
  * unary
  * binary
  * varargs
  * curry?
* Not all forms always defined?
  * or precedence (eg `//` inline explicit before body)
* Partial apply
* Bind
* Compound (eg `for-in-by`)
* Input pattern matching
  * Can be useful for comptime optimizations?
  * More minimal syntax than explicit match syntax?
* Parts
  * Input
    * Before/After
    * Config
      * When not present will either be inferred or will assume is args?
    * Args
      * Inline or Multiline
  * Output
    * Before/After
  * Definition
  * Call
  * Partial Apply
* 

### Custom Function Identifier
The function identifier isn't limited to alpha-numeric as it's often useful to allow for syntax to be customized.

For example, it'd be nice to do customize a comment:
```
//(mode = markdown)
  ### Comment Header
  * Point 1
  * Point 2
  * Point 3
```

### Juxtaposition Application
Since juxtaposition is supported it can also be used with functions. Unlike most juxtaposition which is used for multiplication, juxtaposition is used to apply a function.

Multiple solutions have been considered:
* Always make explicit using `()`. Most languages take this approach as it's the most explicit and unambiguous.
* 



## TODO
---

## Distribution
* Need a way to describe how everything distributes across `()`
* Much like juxtaposition it depends on the type and context
* Types
  * `function(Seq)` is call function on Seq
    * eg `sum(1, 2, 3) = do sum; 1, 2, 3`
  * `Int(Subexp -> Int)` is multiplication
    * eg `4(1 + 2 + 3) == 4(6) == 24`
      * Notice this isn't `4 (1 + 2 + 3)` which is juxtaposition
  * `Int(Seq)` is ???
    * eg `4(1, 2, 3) == (4 * 1, 4 * 2, 4 * 3) == (4, 8, 12)`
  * `lambda(Seq)` is ???
  * `infix-operator(Seq)` is ???
    * eg `+(1, 2, 3) == (1 + 2 + 3) == 6`
    * eg `-(1, 2, 3) == (1 - 2 - 3) == -4`
    * eg `>>(1, 2, 3) == (1 >> 2 >> 3)`
    * eg `10 + +(1, 2, 3) == 10 + (1 + 2 + 3) == 10 + (6) == 16`
    * eg `10 - -(1, 2, 3) == 10 - (1 - 2 - 3) == 10 - (-4) == 14`
    * eg `10 * >>(1, 2, 3) == 10 * (1 >> 2 >> 3)`
  * `attached-operator(Seq)` is ???
    * eg `!(x, y, z)`
    * eg `!(x or y and z)`
  * `relator(Seq)` is distribute path
    * relator selection `a/b.c:(d, e) = 1, 2 === a/b.c:d = 1; a/b.c:e = 2`
    * relator literal `x /a/b.c:(d, e) == x /a/b.c:d /a/b.c:e`
  * `Int(String)` is ???

## Terse data representation tools
* Compare to SDLang, StrictYAML, Pug, Rebol, SASS, O-Exp, etc.
  * eg use identifier to represent semantics a la Pug, SASS
  * Selector and define values a la SASS
* Give some examples
* SASS
  * mixin + include
  * selector inheritance
     * multiple inheritance
* StrictYAML
* SDLang
* Pug
  * semantic meaning to selectors for `.` and `#`
* O-Exp
* Rebol
* InternetObject
* Smalltalk
* GraphQL
* 

## Fixity
Fixity means the position of a operator relative to it's arguments.

For simplicity, there are only a fixities: prefix (operator before it's arguments), infix (operator between it's arguments), and aggregate fixity (something custom).

infix is for:
* Arithmetic: `+ - / *`
* Boolean: `is isnt not`
* Predicate: `and or xor`
* Comparison: `> >= < <=`
* Shift: `>> <<`
* Power: `^`
* Modulus: `% %%`
* Binary: `& |`
* 

prefix is for:
* Negation: `-2` `-(1, -3, 5)`
* Not: `!true` `!(true, false, my-bool)`
* Function: `my-fun(x, y, z)`
* Relation Space: `obj /my-child` `i :type = Int` `div .width = 100%`
* Anonymous Head (must be at start of line after whitespace): `- my-elem` `* my-elem`
* 

aggregate fixity is for:
* TODO: `if, then, else-if, else`
* 

Differences between: `4 - (2, 3, 4)` vs `4 -(2, 3, 4)` vs `4 -( 2, 3, 4` vs `4 -(( 2, 3, 4` vs `4 -( 2, 3, 4 )`
* How infix and prefix distributes
* How to change fixity and grouping using `( )`

Difference between: relation-space, operator, and function
* relation-space is only certain punctuation like: `: . /`
* operator is only certain punctuation: `+ - * / % ^`
* function is identifier

## Application
```
x - 1 - 2 - 3
x - 1, 2, 3
x - (1, 2, 3)
x -(1, 2, 3)
x -( 1, 2, 3
x -(( 1, 2, 3
minus x, 1, 2, 3
minus(x, 1, 2, 3)
minus (x, 1, 2, 3)
```

## Identifiers
* Consider Rebol literals?
  * Email
  * File
  * Hash
  * Image
  * Issue (eg serial #)
  * Block
  * List
  * Paren
  * Path
  * Tag
  * Url
  * Character
  * Date
  * Logic
  * Money
  * None (non-existence)
  * Pair (spacial coordinates, sizes, etc.)
  * Refinement (like adjectives in English) like `append/only` where `append = Function` and `only = Refinement`
  * Time
  * Tuple (eg IP address)
* TODO: Explain why is important to be expressive w/ identifier
  * Should express many things like: form, mem layout, units, etc.
    * May need ways to make these meaningful as well like adding meaning to `_` w/n identifier?
  * Identifiers should impart meaning? eg Pug or Units or ...?

## Identifier
---
Generally what an identifier starts with determines what it's interpreted as. This becomes useful to accurately describe the meaning of a identifier, include units, etc.

TODO rules:
* can't start or end with punctuation (b/c could be an operator or relator)
* 

There are special identifiers which can't be used:
* Type
  * First character is uppercase alpha (ASCII 65 - 90)
  * eg types like `Int`, `U32`, `List`
* Literals
  * Numeric
    * Start with a Numeric
    * eg `2` `3.14` `2e10` `2e-10`, `0xFF` `0b1010` `0o123` `100,000,000` `1+3i`
    * TODO: Arbitrary base? eg 064x123 (base64)?
  * Date
  * Currenty
  * Boolean
    * eg `True`, `False`
  * String
    * Inline `"` or Block `"""`
    * eg inline string literal `"Jill Jacques"`
  * Code
    * Inline `` ` `` or Block ```` ``` ````
    * eg inline code literal `` sum-a-b = `a + b` ``
* Arithmetic
  * TODO: Can be undefined or not included?
  * Matches arithmetic characters
    * `+ - * /`
    * `** //`
    * `% %%`
* Relator
  * Includes only `: . / $ TODO!`
* Unary
  * eg `-1` `!true`
* `\` escape
  * eg escape character `\n`, `\t`
  * eg escape in string literal `"Hello \(name)"`
  * eg escape in code literal `` `a + b / \(denominator)` ``
* Equals
  * eg `x = 4`
  * and shadow `x := 5`

Whitespace is very significant. In identifiers this means if an identifier doesn't contain whitespace it's considered the same identifier. This also means we can include otherwise special characters as long as it follows a few rules:
* must not have whitespace within identifier (or must be explicitly grouped using codegen or code literal)
* must not exactly match an existing reserved literal, operator, reserved (eg ``#, `, "``)
* it doesn't start with a character that would make it be interpreted as an existing type (eg starts with `#` means it's comptime)

Everything else can be used as a identifier. Multi-word identifiers are interspered using hyphen (`-`) b/c:
* It's simpler to write than ``_`` (underscore)
* It's arguably just as clear, if not more, than other options (camel case, snake case, etc.)
* Whitespace is significant and it's not allowed that `x-y` (identifier) and `x - y` (subtraction) are the same

There are a few categories of special identifiers:
* Literals
  * Numeric: `2`, `3.14`
* Types
  * Only rule is that the first character is alphabetic and upper-case (ASCII 65 - 90)
  * eg `Int`, `U32`, `List(Int)`
* Reserved
  * eg `, ; = : . /`

Everything else can be used as a identifier. Multi-word identifiers are interspered using `-` b/c:
* It's simpler to write than `_` (underscore)
* It's arguably just as clear, if not more, than other options
* Whitespace is significant and it's not allowed that `x-y` (identifier) and `x - y` (subtraction) are the same

Here's a few examples:
```
x
my-staff
x-0
x-1
x-2
x+y
x+y-2z
x'
x''
```


## Infererence rules

## Rules
* How traits, interface, types, etc. are a specialization of rules
* Positive rule (by construction), negative rule
* 

## Consistency
Developers recognize that consistency is important if a language is going to be reliable, enjoyable, and predictable (ie it should do what you expect it to). However, like most things, there are costs:
* LISPs abundant visual noise and parethensis pairing are mostly consistent but harm the ability to grok the code w/o help from IDEs, formatting tools, etc.
* PHP uses different operators that **conceptually** do the same thing but have slight differences
* C++ operator overloading causes issues when does something clever that doesn't **conceptually** match what it does in other contexts. To be clear, the issue isn't that the same operator is implemented differently (as it must be for different types) but rather that the conceptual model of the operator does things in it's implementation that don't match it's conceptual model or are not consistent w/ other implementations.

To minimize those cost here's the heuristic we use when making a design choice: what is most common (ie use most often) and difficult to reason should be consist while what is rare, easy to understand, or simple to memorize can be less consistent.

### Offside rule
### Only 2 spaces, nothing else
### Newlines
### Concepts of syntax
### Significant Whitespace
By design, whitespace is very significant and used throughout the syntactical structure. The intent is to make the actual layout of the code mimic the AST and to minimize visual noise.

Here are the types of significant whitespace:
* 2 space indent and dedent which signify a child
* Juxtaposition of function with data is invocation (eg `f x, y, z === f(x, y, z)`)
* `\n` indicates a sibling and terminates any inline sequences

Due to characteristics of whitespace it's often not required to close literals when it reaches the end of it's scope. Consider the psuedo-code showing what this may look like:
```
new-staff = [
  [
    "Janet Johnson"
    26
  [
    "James Jackson"
    18
  [
    "Josh Jameson"
    57
```

and is equivalent to:
```
new-staff = [
  [ "Janet Johnson", 26
  [ "James Jackson", 18
  [ "Josh Jameson",  57
```

and:
```
new-staff = [ [ "Janet Johnson", 26 ], [ "James Jackson", 18], [ "Josh Jameson", 57 ] ]
```

and (b/c the `\n` terminates the ***Sequence***):
```
new-staff = [ [ "Janet Johnson", 26 ], [ "James Jackson", 18], [ "Josh Jameson", 57
```
this isn't idiomatic and should be avoided but can be useful in cases like nested function invocation.

### What isn't consistent
`=` is ambiguous in diff contexts
* Concept is consistent (relate LH to RH)
* Precedence is ambiguous
  * But can be disambiguated by context or manually
* Only a few variants
  * Should do what's obvious and expects in each case: Block, Map, Condition, Function Arg

#### Automattic closing at `\n`
#### Automattic closing at `DEDENT`
#### Different forms (multiline to inline)
* Based on use-case and thus be flexible as syntax matches what is most appropriate
  * Expanded makes hard to understand b/c too hard to grok as many lines make even trivial programs quite long
  * Inline makes hard to clear see structure, heirarchy, and flow
* Mimics shape of data and use-case
* All transformed to multiline internally

### Definition
### Application (as a specific example of juxtaposition?)

#### Prefix
#### Infix
#### Explicit inline
#### Multiline
#### Example

### Varargs
#### Declare (eg multiple arrays)
#### Start
#### End
#### Application
#### Example

## Mix

## Compare to: S-exp, M-exp, other LISP syntaxes, Red/Rebol syntax, SDLang, StrictYAML, Pug, SASS, ...

## Sugar
* Anonymous tree head w/ `-` and `*`
* Say anything about const-ness? propogate constness? or foreign convern?
* Codegen when prefix? or foreign concern? ie this only defines it as infix?
* Reserved
* Ambiguities
* Warts
* Ideas and Proposals
* IDs
  * Standards
  * After `=`? With all types of identifiers?
* Priorities and Goals
* Very significant whitespace
  * Indent and Dedent
  * Juxtaposition
  * Spacing and invocation
  * Significant \n
  * Significant \n \n
  * Within literals
    * Explicit, Start of next line, etc.? Copy from Pug and similar?
  * Spaces w/ operators (eg `=`) b/c otherwise identifier?
* Block delimiters?
* Complex examples
* Break assignment into a new block?
  ```
  x =
    a
    b
    c
  ```
* When ***relative space*** and dive into have explicit children
  * Difference between these type of children and `/`? How to make clear? Defaulting?
* Literals
* Comptime?
* Unit suffix? or handled w/ metadata? or type?
  * Is underscore significant to mean units?
* Tree anonymous head `-` and `*`?
* Dive into
* Limitations
* Syntax operators (eg `//`)
* Multiple forms
  * eg differentiate comment syntax `//` modes from input
* Parts of a fn-like thing
* Precedence
* Ways to call a function and how it affects implementation choice?
* Keywords
  * Nested? Sibling?
* differences between:
  ```
  a.b.c.f()                              // Invoke 'f'
  a.b.c.f() = 0                          // 
  a.b.c.f(x = 0)                         // 
  a.b.c.f(x = 0, y = 1, z = 2)           // 
  a.b.c.f(x, y, z) = 0, 1, 2             // select 'x, y, z' and assign to '0, 1, 2'
  a.b.c.f(x = 0, y = 1, z = 2) = 3, 4, 5 // 
  ```
* Interperet as? and how this relates to interpretation and evaluation?
* 

### Highest Precedence
Like many languages, parenthesis also represent grouping and are the highest precedence.

Consider the difference in arithmetic:
```
4 + 6 / 2 == 7

(4 + 6) / 2 == 5
```

Much like a relator, parenthesis also used to represent the start and end of an operator, function, keyword, and others:
```
my-fn(1, 2, 3)   // A prefix function
+(1, 2, 3)       // A operator
Map(String, Int) // A type
```
Notice that the `(` must be connected to the operator, function, keyword, etc. in order to be the start and end.

Since parenthesis are the highest precedence it's also a tool to alter the grouping of arguments:
```
// Let f, h be functions
f 1, h 2, 3 == f(1, h(2, 3))

// but can alter using parenthesis
f 1, h(2), 3 == f(1, h(2), 3)
```
There are other tools which do the same and may be more convenient explained in ***Precedence***.

### Grouping
Parenthesis also group elements together.

Consider a conditional:
```
if
  x > 0
  y > x
  z > y
then
  log "passed:", x, y, z
else
  log "failed:", x, y, z
```

this could be made inline:
```
if (x > 0, y > x, z > y) then (log "passed:", x, y, z) else (log "failed:", x, y, z)
```

## Types, Traits, and Interface
