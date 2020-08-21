---
layout: default
title: Syntax
permalink: /syntax/
---

The Elder syntax arose out of desire to unify data literals with homoiconic programming.

I was inspired to begin real work as I began using Red-lang and it's ability to work across the entire stack but felt it need not be constrained to the functional programming pardigm. It seemed at least plausable to create a language w/ flexible syntax and semantics which could more approach full-stack development w/ less visual noise and more consistency than most languages are capable of.

The syntax of Elder is primary inspired by:
* the clarity and elegance of Python
* homoiconicity of LISP and Rebol and the various formulations of the syntax from S-exp, M-exp, Sweet-exp, O-exp, ...
* operator definition and extension of Fortress
* terseness of tree modeling (whether HTML, CSS, etc.) of Pug, SASS
* data serialization and literal representations of StrictYAML and SDLang
* TODO: more?

This document only describes the ***Elder syntax*** itself. It doesn't tell you about the ***semantics*** (meaning) of the syntax. The semantics are added once the syntax is used w/n a specific context. For example, a sequence within the context of a HTML document are HTML nodes. Clarifying the semantics and other language uses are part of the core language and not described here as they're beyond the scope of the syntax. However, to help, there are some examples with psuedo-code at the end.

## Sequence
A ***Sequence*** (much like LISP, Rebol, Red) is a core syntactical structure. A sequence is:
* A ordered, series of elements
* A syntactical concept
* Not a List, Array, Series, Set, et al. b/c these all make more and different guarantees

There are multiple forms (syntactical layout) that are used and we will slowly build them up over this document.

The simplest is the multiline form:
```
x
y
z
```

### Inline Siblings
To write more compact code we have to introduce new syntax `,`
* Represents a sibling w/n a sequence
* Internally `,` translates into a `\n` so the two forms are equivalent
* For consistency, you can't mix inline and multiline syntax `,` for the same sequence

Using this, we can compact the sequence to:
```
x, y, z
```

### Inline Sequence of Sequence
If we wanted to represent a sequence of seqences using the current syntax we'd be stuck with something like this:
```
a, b, c
x, y, z
1, 2, 3
```
Notice that we know:
* this isn't one long list b/c it's not allowed that we mix expanding and inline siblings
* each sub-sequence belong to the same parent sequence b/c they are visually on each line (ie no gaps or `\n \n`)
* is equivalent to:
  * `` `( (a, b, c), (x, y, z), (1, 2, 3) )`` in LISPy languages
  * `[ [a, b, c], [x, y, z], [1, 2, 3] ]` is the closest JavaScript can represent

We can futher compact the code using another new syntax `;`
* Represents the end of sequence
* Often useful for a sequence of functions and arguments and some data-structures
* For consistency, you can't mix inline and multiline syntax `;` for the same sequence
* At the end of a line if there's a `DEDENT` it implicitly terminates a sequence

Using this, we can compact the previous sequence of sequences to:
```
a, b, c; x, y, z; 1, 2, 3;
```

## Tree
A natural extension of a ***Sequence*** are ***Nested Sequences***. This is a ***Tree***.

A ***Tree*** is a primitive structure b/c:
* It's how code is naturally structured.
* AST are modeled as trees which makes it easier for our code layout to match the expected syntax tree which make it more guessable and predictable.

The multiline form (2 space `INDENT` represents a child):
```
x
  a
  b
  c
```
This code tells us that:
* `a, b, c` are children of `x` b/c their indentation level = `x` indentation level + 1
* `a, b, c` are siblings b/c they're at the same indendation level under the same parent

2 spaces are used b/c it's the minimal amount to make it clear there's indentation. No tabs, no variable spaces, nothing but 2 spaces per indentation. Later in this document there are ways to compact the syntax when it makes sense to do so.

This can be nested arbitrarily and the structure matches what you expect as the layout matches the code layout:
```
x
  a
  b
    1
    2
    3
  c
```

## Equals Syntax
`=` captures the idea of relating a ***L-hand*** to a ***R-Hand***. The type of relationship varies with the context it's defined in.

TODO: Disjoint contexts used in?

`=` is defined as you expect in most languages but is more flexible and widely used:
* Only defined as a `binary operator` and `infix fixity`
* Used in many locations to describe assignment, mapping between names and values, key-value entry, etc.
* Semantic meaning depends on what context it's defined

TODO: Relate this to examples in other languages!!!

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

## Relation Space
A ***Relation Space*** is a syntactical tool which models the ***relation*** between a ***target*** and a ***descriptor***.

A relation space has the qualities:
* Abstracts over syntax which is hard-coded, are reserved, or not present in other languages
* It acts like the child elements of a tree and itself is the parent of multiple children (ie it's a tree node)
* It characters are only made of punctuation
  * Unicode will eventually be supported, but for now only ASCII printable characters (32 - 126) minus whatever is reserved
* It is defined and accessed in a unique way
* If multiple relation spaces have the same syntax, the longest match wins
  * For example, if both `:` and `::` are defined `::` will always win

It differs in several key ways to concepts like namespaces which justified introducing a new term in order to distance it from concepts like namespaces.

Although users can define their own, there are 3 common relation spaces that are commonly used:
* `/` the child relation space for parent-child relation
* `.` the dot relation space for object-property relation
* `:` the meta relation space for data-metadata relation

### Parent-Child Relation Space
The `/` relation space is the parent-child relation:
* Often used to represent child inline
* Replaces indentation

Using this, the ***Tree*** from above is equivalent to:
```
x
  /
    a
    b
    c
```
Notice that the `/` is redundant in this case b/c an `INDENT` signifies a parent-child relation but this example makes it explicit. This will be explained later in the document in `Defaulting`.

and can be compacted to:
```
x /a, /b, /c
```

The complex tree can be compacted as well:
```
x
  a
  b /1, /2, /3
  c
```

but not fully inline with our current syntax:
```
x /a, /b /1, /2, /3; /c
```
This is close but we specifically disallow having nested inline relation spaces. Notice that `/1, /2, /3` are children of `b` and are nested under `x`. This is disallowed to make the code readable and allows us to make the assumption that a inline relation space always refers to the first element. We'll be able to better model this w/ syntax introduced later.

### Object-Property Relation Space
The `.` relation space is the object-property relation:
* Often used to represent properties or attributes of an object
* In some cases will replaces the indentation of a multiline definition (eg a `Struct`)

Here's an example:
```
obj
  .a = 0
  .b = 1
  .c = 2
```

this is equivalent to:
```
obj
  .
    a = 0
    b = 1
    c = 2
```

and can be compacted to:
```
obj .a = 0, .b = 1, .c = 2
```

### Data-Metadata Relation Space
The `:` relation space is the data-metadata relation:
* Often used to represent metadata of any data (and since it's homoiconic it can target almost anything)
* Often does the work of describing conceptsa like types, constraints, relative types, rules, etc.
* It will almost never replace the indentation of a multiline definition

Here's an example which models `Age` as positive, integral number:
```
Age
  :type = Int
  :min  = 0
```

this is equivalent to:
```
Age
  :
    type = Int
    min  = 0
```

and can be compacted to:
```
Age :type = Int, :min = 0
```

### Collapse Relation Space
Just like referencing data within most languages, you can do the same but it's used in multiple ways:
* To browse to a location and work w/n that scope
* Implicitly creating a heirarchy of relations
* Pattern matching

Here are a few examples.

Browse to HTML element we're concerned with and work w/n that scope:
```
html/body/header/
  .id = my-header

  logo
    .class = my-logo
```
Here `html/boy/header/` would create syntactic noise as we only really care to do work w/n the `header` so we need to have syntax that allows us to do this naturally.

Implicitly create `.style.css.width` as they're necessary steps for valid HTML:
```
// As a literal (notice space)
my-element .style.css.width = 100%

// or, as assignment (no space)
my-element.style.css.width = 100%
```

Assign and destructure multiple values
```
// Use fully qualified
id, class = html/body/header.id, html/body/header.class

// or, select only the values you need
id, class = html/body/header.(id, class)
```

## Parenthesis
Although we avoid excessive parenthesis, they're often essential. Like much of the syntax they perform multiple duties depending on how they're used:
* Represent grouping to describe precedence (like in most languages)
* Represent the start `(` and stop `)` of a expression, operator, relation space, function, etc.

Using this, we can further compact our previous examples from:
```
x /a, /b, /c
obj .a = 0, .b = 1, .c = 2
Age :type = Int, :min = 0
```

to:
```
x /(a, b, c)
obj .(a = 0, b = 1, c = 2)
Age :(type = Int, min = 0)
```

Parenthesis especially help when combining multiple relation spaces to describe more complex structure. Consider the template psuedo code:
```
div .(id = "spashscreen", class = "spashscreen") :visible-if = "is-first-view"
  h1 .class = "title"
  p .class = "description"
```

using a theoretical HTML generator this would translate to:
```
<div id = "spashscreen" class = "spashscreen" data-visible-if = "is-first-view">
  <h1 class = "title"></h1>
  <p class = "description"></p>
</div>
```
Notice that `:visible-if` is an example of what could be used to add logic to the template.

This syntax is much closer to what we see in languages that model trees directly such as: SDLang, YAML, Pug, SASS. To compact the syntax further syntax sugar and macros should be used although they outside the scope of this document.

### Grouping
Parenthesis also represent grouping and are the highest precedence.

TODO w/ precedence?
* Non-transitive? or Smalltalk-like?
* 

## Consistency
Developers recognize that consistency is important if a language is going to be reliable, enjoyable, and predictable (ie it should do what you expect it to). However, like most things, there are costs:
* LISPs abundant visual noise and parethensis pairing are mostly consistent but harm the ability to grok the code w/o help from IDEs, formatting tools, etc.
* PHP uses different operators that **conceptually** do the same thing but have slight differences
* C++ operator overloading causes issues when does something clever that doesn't **conceptually** match what it does in other contexts. To be clear, the issue isn't that the same operator is implemented differently (as it must be for different types) but rather that the conceptual model of the operator does things in it's implementation that don't match it's conceptual model or are not consistent w/ other implementations.

To minimize those cost here's the heuristic we use when making a design choice: what is most common (ie use most often) and difficult to reason should be consist while what is rare, easy to understand, or simple to memorize can be less consistent.

### Offside rule
### Only 2 spaces, nothing else
### Juxtaposition
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

## Identifier
Generally what an identifier starts with determines what it's interpreted as. This becomes useful to accurately describe the meaning of a identifier, include units, etc.

TODO: Explain why is important to be expressive w/ identifier
- Should express many things like: form, mem layout, units, etc.
  - May need ways to make these meaningful as well like adding meaning to `_` w/n identifier?

There are special identifiers which can't be used
* Type
  * First character is uppercase alpha (ASCII 65 - 90)
  * eg types like `Int`, `U32`, `List`
* Literals
  * Numeric
    * Start with a Numeric
    * eg `2`, `3.14` `2e10`, `2e-10`, `0xFF` `0b1010`, `0o123`, `100,000,000`, `1+3i`
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
* Relation Space
  * Includes only `: . / $ TODO!`
* Unary
  * eg `-1`, 
* `\` escape
  * eg escape character `\n`, `\t`
  * eg escape in string literal `"Hello \(name)"` or code literal `` `a + b / \(denominator)` ``
* Equals
  * TODO
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
  * 

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
  * eg `,`, `;`, `=`, `:`, `.`, `/`

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
### What isn't consistent
#### `=` is ambiguous in diff contexts
* Concept is consistent (relate LH to RH)
* Precedence is ambiguous
  * But can be disambiguated by context or manually
* Only a few variants
  * Should do what's obvious and expects in each case: Block, Map, Condition, Function Arg, 
#### Automattic closing at `\n`
#### Automattic closing at `DEDENT`
#### Different forms (multiline to inline)
* Based on use-case and thus be flexible as syntax matches what is most appropriate
  * Expanded makes hard to understand b/c too hard to grok as many lines make even trivial programs quite long
  * Inline makes hard to clear see structure, heirarchy, and flow
* Mimics shape of data and use-case
* All transformed to multiline internally

## Functions
* TODO
  * Different types of call syntax
    * f(x, y, z)
    * f x, y ,z
    * f g h x === f(g(h(x)))
    * f
        x
        y
        z
    * UFCS
      * obj.f(x, y, z) === obj.f x, y, z
    * open-paren
      * f( x, y
      * f(( x, y
    * fixity
      * infix: 1 + 2 + 3
      * prefix: !x
      * suffix: x++
  * Arity
    * nullary
    * unary
    * binary
    * varargs
    * curry?
  * Not all forms always defined?
    * or precedence (eg `//` inline explicit before body)
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

## Juxtaposition

## Comment



### Definition
### Application (as a specific example of juxtaposition?)

### Precedence
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

## Examples

## Sugar
TODO
* Anonymous tree head w/ `-` and `*`

## TODO
---
* Say anything about const-ness? propogate constness? or foreign convern?
* Codegen when prefix? or foreign concern? ie this only defines it as infix?
* First character/operator is special b/c defines how rest is interpreted?
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
* Limitations
* Comments
* Syntax operators
* Multiple forms
  * eg differentiate comment syntax `//` modes from input
* Parts of a fn-like thing
* Precedence
* `=` vs `:=`
  * For shadow and not?
* Ways to call a function and how it affects implementation choice?
* Keywords
  * Nested? Sibling?
* 
