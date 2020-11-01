---
layout: default
title: Syntax
description: A minimal, data-oriented, and expressive syntax
permalink: syntax
---

Elder syntax's goal is to unify homoiconic and serialization data-oriented syntax structure as tersely as possible in order to create a general purpose syntax. Elder core language uses the Elder syntax as it's choice of syntax much like most LISP implementation uses S-expressions.

Elder syntax is a general purpose syntax which competes with a variety of syntax use-cases:
* homoiconic syntaxes like s-expressions, sweet-expressions, o-expressions, rebol
* serialization and data-modeling
  * external schema and types like StrictYAML
  * syntax and literal types like JSON, YAML, SDLang, OGDL
  * schema like InternetObject, OGDL
  * custom DSL like Pug, SASS, Markdown

Implementors decide the level of complexity of the syntax by opting into dialects that suite their needs.

This document only describes the ***Elder syntax*** itself. It doesn't tell you about the ***semantics*** (meaning) of the syntax. The semantics are added once the syntax is used w/n a specific context. For example, a sequence within the context of a HTML document are HTML nodes. Clarifying the semantics and other language uses are part of the core language and not described here as they're beyond the scope of the syntax. However, to help, there are some examples with psuedo-code at the end.

## Sequence
------------------------------------------------------------------------------------------------------------

A ***Sequence*** (much like LISP, Rebol) is a core syntactical structure. A sequence is:
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
* For consistency and clarity, you can't mix inline and multiline syntax `,` for the same sequence

Using this, we can compact the sequence to:
```
x, y, z
```

## Tree
------------------------------------------------------------------------------------------------------------

A natural extension of a ***Sequence*** are ***Nested Sequences***. This is a ***Tree***.

A ***Tree*** is a primitive structure b/c:
* It's how code is naturally structured
* AST are modeled as trees which makes it easier for our code layout to match the expected syntax tree which make it more guessable and predictable

The multiline form (2 space `INDENT` represents a child):
```
x
  a
  b
  c
```
This code tells us that:
* `a, b, c` are children of `x` b/c they're nested under x w/ a 2 space `INDENT`
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

## Equals
------------------------------------------------------------------------------------------------------------

Equals (`=`) is a syntax which captures the pattern of relating ***L-hand*** to a ***R-hand***.

The precise relationship between ***L-hand*** and ***R-hand*** varies with the context it's defined in (eg assignment, mapping between names and values, key-value entry)

We can model definition or assignment:
```
x = 1
```
This is read as:
* name `x` relates to value `1`

We can model a map:
```
obj
  a = 1
  b = 2
  c = 3
```
This is read as:
* name `obj` has 3 children
* the 1st child is `a = 1`
  * name `a` relates to value `1`
* the 2nd child is `b = 2`
  * name `b` relates to value `2`
* the 3rd child is `c = 3`
  * name `c` relates to value `3`

### Equals Newline
Sometimes it makes sense to not have the value on the same line as the `=`.

Essentially the child or children become the value of the R-hand of the expression.

There's a few cases where the values is:
* A single value
  ```
  x =
    a
  ```
  * This is equivalent to `x = a` since `a` is the only immediate child of `x`
  * Often useful when the value is a self-executing expression
* A single, complex value
  ```
  x =
    a
      b
      c
  ```
  * This associates `x` name to the structure starting with `a` since `a` is the only immediate child of `x`
  * Often useful when the value is a complex data-structure
* Multiple values
  ```
  x =
    a
    b
    c
  ```
  * This is equivalent to `x = (a, b, c)`. This is b/c the sequence `a, b, c` are all immediate children of `x`
  * Often useful when you want to store a literal, multi-valued, container-like (eg `List`, `Sequence`, `Map`)

## Relator
------------------------------------------------------------------------------------------------------------

A ***Relator*** is a syntactical tool which models the ***relation*** between a ***target*** and a ***descriptor***. Customizing the syntax allows developers to add structure to the syntax without the need of macros (although more constrained than what macros provide).

This concept is usually hard coded within most languages such as:
* `C++`
  * accessing a `Namespace` uses `::` and is used like `my_namespace::my_val`
  * accessing a `Struct` field uses `.` and is used like `my_stuct.field` or `my_struct.field = 1`
* `Clojure`
  * accessing a `Namespace` uses `/` and is used like `my_namespace/my_val`
  * interop with `Java` or `JavaScript` uses `.` and is used like `(.toUpperCase "jane")`

[//]: # (TODO: choose punctuation: ``! @ # % ^ & * - _ + = ` ~ ( ) [ ] { } | \ : ; " ' < > , . ? /``)
[//]: # (TODO: Describe how relator is like mix of namespace, relation, and operator?)

Functionally it works much like a **namespace** but:
* character are only specific punctuation
  * Unicode will eventually be supported, but for now only ASCII printable characters (32 - 126) minus whatever is reserved
* identifier characters and relator characters must not overlap b/c relators are used as delimiters between identifiers (like `a/b/c` where `/` is a relator)
* is more general concept than a namespace and can be used to model more patterns
* can be syntactically used in more ways
* it is a tree node
* If multiple relators have the same syntax, the longest match wins
  * For example, if both `:` and `::` are defined `::` will always win
  * This also means multiple characters can be combined to make new relators.
    * For example, if `:`, `.`, `/` are all relators then `:./` would form a new relator distinct from the others.

Although developers can define their own, there are 3 common relators that are commonly used:
* `/` the child relator for parent-child relation
  * Often used to represent child inline
  * Replaces indentation
* `.` the dot relator for object-property relation
  * Often used to represent properties or attributes of an object
* `:` the meta relator for data-metadata relation
  * used to represent metadata of any data (and since it's homoiconic it can target almost anything)
  * does the work of describing conceptsa like types, constraints, relative types, rules, etc.

For demonstration, let's use the object-property relation `.` syntax.

### Literal
In order to model data without additional syntactical noise, data can be written as literal trees. For relators, there's a few rules:
* When multiline, it must match the layout of a tree as described above.
* When inline, there must be a space between the target and the first relator.

With a example the rules become obvious:
```
obj
  .a = 0
  .b
  .c = 2
```

This is read as:
* declare tree literal with name `obj`
* add child to `obj` of `.` relation with name `a` then define as `0`
* add child to `obj` of `.` relation with name `b`
* add child to `obj` of `.` relation with name `c` then define as `2`

is equivalent to when grouped under a relator:
```
obj
  .
    a = 0
    b
    c = 2
```

This is read as:
* declare tree literal with name `obj`
* add child to `obj` of `.`
* add child to `.` with name `a` and define as `0`
* add child to `.` with name `b`
* add child to `.` with name `c` and define as `2`

is equivalent to inline:
```
obj .a = 0, .b, .c = 2
```

### Path
One aspect that's unique to relators is that they're used to create paths in order to:
* define data
* abstacts over concepts like file and network paths
* traverse data structure

Consider a slightly more complex example which uses both the `.` (object-property) and `:` (data-metadata) relators:
```
o
  .a = 0
  .b
    :x = 1
    :y = 2
  .c = 3
```

Using paths we can access and define this data in multiple ways. Syntactically paths acts more like other languages and uses the relator as a delimiter.

For example, to select `x` in `o` we write:
```
o.b:x
```
which is read as:
* access `o` name
* access `.` relation in `o`
* access `b` name in `.` relation
* access `:` relation in `b`
* access `x` name in `:` relation
* select name `b`

By default, selecting a value will return it's result.

It's also useful in when paired with equals syntax:
```
o.b:x = 3
```
This is read the same as the previous example however the final step will assign `3` to `x`.

Using paths we can also change how we define data. Consider an alternative way to define the above example:
```
o.
  a = 0
  b:
    x = 1
    y = 2
  c = 3
```

This is slightly more terse than the previous, structural representation as each of the common paths (`o.` and `b:`) are used. Although there's not much difference in this case; the usefulness is more pronounced in data which have deeply nested structure of which you only use a small amount like in the ***Deeply Nested Element*** example.

### Example - Deeply Nested Element
A more realistic example is to browse to HTML element we're concerned with and work w/n that scope:
```
html
  body
    header
      .
        style
        .
          css
            .
              width  = 100%
              height = 20px
```
Notice that there's lots of extra structure to get to the real work of setting `width` and `height` which is the purpose of this block.

Using paths we can do better and remove most of the syntactical noise:
```
html/body/header.style.css.
  width  = 100%
  height = 20px
```
We use paths to collapse the heirarchy from above to what we really are concerned about.

## Parenthesis
------------------------------------------------------------------------------------------------------------

Although we strive for terseness, parenthesis are essential. Like much of the syntax they perform multiple duties depending on how they're used:
* Is highest precedence (like in most languages)
* Represent the start `(` and stop `)` of a expression, operator, relator, function, etc.
* Groups elements together
* Is a Sequence literal

Consider an example which has common structure:
```
o
  .
    a = 1
    b = 2
    c = 3
  :
    x = 4
    y = 5
    z = 6
```

Using our existing syntax we can represent this inline as a literal but it's rather noisey:
```
o .a = 1, .b = 2, .c = 3, :x = 4, :y = 5, :z = 6
```

Using parenthesis we can express this in multiple forms which more clearly expresses the natural structure of the data. As a multiline literal:
```
o
  .(a = 1, b = 2, c = 3)
  :(x = 4, y = 5, z = 6)
```

as a inline literal:
```
o .(a = 1, b = 2, c = 3), :(x = 4, y = 5, z = 6)
```

### Example - HTML Generation
Parenthesis especially help when combining multiple relators to describe more complex structure. Consider the template psuedo code:
```
div .(id = "intro", class = "spashscreen", style.css.(width, height) = 100%, 20px), :visible-if = "is-first-view"

div .(id = "intro", class = "spashscreen", style.css.(width = 100%, height = 20px)), :visible-if = "is-first-view"
  h1 .class = "title"
  p .class = "description"
```

using a theoretical HTML generator this would translate to:
```
<div id = "intro" class = "spashscreen" data-visible-if = "is-first-view">
  <h1 class = "title"></h1>
  <p class = "description"></p>
</div>
```
Notice that `:visible-if` is an example of a field used for logic in a template. It could either be rendered w/n the HTML (as above) or erased at comptime depending on the use-case.

This syntax is much closer to what we see in languages that model trees directly such as: SDLang, YAML, Pug, SASS. To compact the syntax further syntax sugar and macros should be used although they outside the scope of this document.

### Example - Terse HTML inline CSS
Consider a use-case where we want to model a HTML element w/ inline CSS.

Implicitly create `.style.css` structure as they're necessary steps for valid HTML:
```
// As a multiline literal
my-element
  .style.css.
    width  = 100%
    height = 20px

// or, as a multiline literal w/ '.' ending
my-element
  .style.css
    .width  = 100%
    .height = 20px

// or, as a multiline literal w/ fully collapsed context
my-element.style.css.
  width  = 100%
  height = 20px

// or, as inline literal (notice space between head and relator '.')
my-element .style.css.width = 100%, .style.css.height = 20px

// or, as assignment (no space)
my-element.style.css.width  = 100%
my-element.style.css.height = 20px

// or, as assignment w/ selection
my-element.style.css.(width, height) = 100%, 20px
```
These variations are made available to fit various use-cases. The most clear and terse should be preferred.

## Simple Destructure
------------------------------------------------------------------------------------------------------------

Destructuring is useful as it allows us to deal directly with the structure of data and is generally more terse.

With our current syntax there's a problem. `=` is a higher precedence than `,`. This means code like:
```
x, y = 1, z
```

is parsed as:
```
x
y = 1
z
```

not:
```
x = 1
y = z
```

Since destructuring is very common we introduce a new syntax `==` which makes it easier to express with minimal visual noise. This is nearly identical to `=` except that:
* It's relative precedence is less than `,`
  * Note: We don't support a global precedence table like many C-style languages, instead all precedence is either relative or not-defined and you must use `()`
* It groups the L-hand and R-hand into `()`. This means `a, b == x, y` is basically `(a, b) = (x, y)`

Using the new syntax, there's a few ways to make this example parse as a destructure:
* Use `()` to group children together
  ```
  (x, y) = (1, z)
  ```
  It's really only essential to group the L-hand is the R-hand can be inferred as a sequence. So, in this case, this would work as well:
  ```
  (x, y) = 1, z
  ```
  It's clear now that we mean we can to assign both values `x, y`.
* Use `==` syntax
  ```
  x, y == 1, z
  ```

## Forms
------------------------------------------------------------------------------------------------------------

At this point we've introduced all the basic syntax needed to construct the common syntactical patterns.

For each pattern, there's a few effects to consider:
* When the pattern in used in a specific scope what names will be in scope.
  * In general all top-level names of a pattern will be added to scope.
* When used in a subexpression what is returned by the subexpression.
* 

These are the common patterns along with the effects they will produce:
* Multiline Literal
  ```
  x = 0
    .a = 1
    .b = 2
    .c
    :d = 3
  y
  z
    .e = 4
  ```
  * Useful to model complex data as the structure is most explicit and verbose.
  * This can only be composed within other multiline literal patterns as it's the only multiline pattern.
  * Effects
    * The sequence of names `(x, y ,z)` will added to scope.
    * When used in a subexpression `(x, y, z)` will be returned by the expression.
* Inline Literal
  ```
  x .a = 1, .b = 2, .c, :d = 3
  ```
  * Useful to model moderately complex data which is often structured line-by-line like HTML, CSS, etc.
  * It's possible to have multiple literals but the syntax becomes harder to understand.
  * Effects
    * The name `x` will be added to scope.
    * When used in a subexpression `x` will be returned by the expression.
* Chain
  ```
  x.(a = 1, b = 2, c):(d = 3) = 0
  ```
  * Useful to model nested structure as this more easily composes within other expressions.
  * Often used to mimic syntax like `x:Int = 4` which we approximate w/ our current syntax as `x:(type = Int) = 4`
  * Effects
    * The name `x` will be added to scope.
    * When used in a subexpression `x` will be returned by the expression.
* Selection
  ```
  x.(a, b, c):(d), z.e
  ```
  * Useful to traverse (or define) the structure to a name.
  * Selection multiple names results in a sequence of names. The above example expression becomes `(x.a, x.b, x.c, x:d, z.e)`
    * Since selection only returned the deepest names of a structure. If you want to pass higher level names (like `x` in the above example) they need to be listed as well. For example `x, x.(a, b, c):(d), z.e` would expand to `(x, x.a, x.b, x.c, x:d, z.e)`.
  * Effects
    * This pattern is a bit different. Remember only top-level names will added to scope. For this example, only the sequence of names `(x, z)` are top level while the rest describe structure internal to those names.
    * When used in a subexpression `(x.a, x.b, x.c, x:d, z.e)` is returned by the expression.
* Destructure
  ```
  x.(a, b):(d), z.e == 0, 1, 2, 3
  ```
  * Destructure is a combination of L-hand selection and R-hand value. The pattern is names of the L-hand and values on the R-hand.
  * Useful to model multiple return values, build data in a specific structure, define multiple names within scope.
  * Effects
    * This pattern is a bit different like selection above. For the same reason, only the sequence of names `(x, z)` are top level while the rest describe structure internal to those names.
    * When used in a subexpression `(x, x.a, x.b, x.c, x:d, z.e)` is returned by the expression.
* Sequence of Expressions
  ```
  x = 0, y, z.a = 2
  ```
  * Useful to model internals of a relator. Generally not preferred as a way to introduce names into a scope as can be confusing compared to multiline literal or destructure.
  * Since `,` is lower precedence than `=` this is a series of 3 expressions.
  * Effects
    * The sequence of names `(x, y, z)` will be added to scope.
    * When used in a subexpression `(x, y, z.a)` is returned by the expression.

Although each of these are generally equivalent to one another, some patterns have different effects. The next sections discuss what they are and the reasoning behind their design decisions.

### Orthogonal Syntax

Syntax patterns generally fall into 2 categories of focus:
* Describing internal structure of a name (multiline literal, inline literal, chain)
* Describing multiple names regardless if they belong to a common parent (selection, destructure, sequence of expression)

Each of these categories are designed so that it's easier to compose data without worrying about the internal structure of a name. To keep them mostly orthogonal, we introduce a rule which we'll explore with examples below. The rule is: if a syntax is defining the internal structure of name, only the name itself will be be returned by the expression.

Here's a simple example:
```
x.(a = 1) = 0
```
If the L-hand expression `x.(a = 0)` returned `x.a` this would result in `a` being overwritten on the R-hand value. This is rarely, if ever, what is desired. Often we want to ignore the internal structure of `x`.

Instead this expands to:
```
x = 0
  .a = 1
```

If we instead wanted to model this using destructuring:
```
(x, x.a) = 0, 1
```

Although the intent may be clear w/ a simple syntax, it becomes harder to track as the syntax becomes more complex:
```
x, y.(a, b:(t = 1), c == 1, 2, 3), z == 4, 5, 6
```

Although not recommended, this syntax is valid. Since we don't allow for inner structure to be passed (unless we do so manually using selection) we can read this as 2 expressions:
* Ignore the internal structure of `b`
  ```
  a, b:(...), c == 1, 2, 3
  ```
* Ignore the internal structure of `y`
  ```
  x, y.(...), z == 4, 5, 6
  ```
Althought not perfect, ignoring the internal structure makes it easier to read which names are mapped to which values.

If we instead wanted to model using destructuring it is more manual:
```
x, y, y.a, y.b, y.b:t, y.c, z == 4, 5, 1, 2, 1, 3, 6
```

or equivelently a bit more tersely (and a bit more structure preserving):
```
x, y, y.(a, b, b:t, c), z = 4, 5, 1, 2, 1, 3, 6
```

For clarity, here is the same example but as a multiline literal (which should preferred for complex syntax):
```
x = 4
y = 5
  .a = 1
  .b = 2
    :t = 1
  .c = 3
z = 6
```

### Headless selection

When selection syntax is used, notice that only the names at the edge are passed. Consider the example:
```
x.(a, b:(t, u, v), c)
```

this will expand to:
```
(x.a, x.b:t, x.b:u, x.b:v, x.c)
```

The structure describes that path to each name instead of being passed themselves.

You can pass the names which are used for structure they have to be passed as well. Let's edit this example to pass `x` as well:
```
(x, x.(a, b:(t, u, v), c))
```

this will expand to:
```
(x, x.a, x.b:t, x.b:u, x.b:v, x.c)
```

To make this example ignore the internal structure simply make any name within the structure assign it a value:
```
x.(a, b:(t, u = 1, v), c)
```

this will make the result of the expression as `x` instead of each name.

### Top level names in scope

The names which are added to the current scope can be different than what is returned by an expression. The rule for this is rather simple: only the top-level names will be brought to scope.

Here's a few examples with the names brought into scope:
* Multiline Literal
  ```
  x
    .a = 1
    .b = 1
  y
    :c = 2
  z
  ```
  `(x, y, z)` are brought into scope. Their internal structure is internally scoped. For example `a` and `b` share the same scope of `x.`.
* Inline Literal
  ```
  x .a, .b = 1, :c
  ```
  `x` is brought into scope. `a, b` share scope under `x.` and `c` under `x:`.
* Chain
  ```
  x.(a, b = 1):(c)
  ```
  Scope rules are just like the previous example.
* Selection
  ```
  x.(a, b:(t, u), c)
  ```
  `x` is brought into scope. `a, b, c` share scope under `x.` and `t, u` under `x.b:`
* Destructure
  ```
  x.a, x.b, y.c.d, z == 1, 2, 3, 4
  ```
  Since only the top-level names are brought into scope; `x, y, z` are brought into scope.

  To better visualize the structure view it as a multiline literal:
  ```
  x
    .a = 1
    .b = 2
  y
    .c
      .d = 3
  z = 4
  ```
  It's clear which scopes are shared viewing it this way.
* Series of Expressions
  ```
  x, y = 1, z.a.b = 2
  ```

  This is read as a series of expressions. Rewriting it makes it even more clear:
  ```
  x
  y = 1
  z.a.b = 2
  ```
  It's clear now that `x, y, z` are brought into scope.


## Composing (In Progress)
------------------------------------------------------------------------------------------------------------

* VIA
  * Parent-child
  * Siblings
  * Subexp

## Composing Destructure
------------------------------------------------------------------------------------------------------------

### TODO: matching arity, multiples, skip, ...

This composes as well like you'd expect. Consider a slightly more complex example with nested destructuring:
```
a, (b, c == 1, 2) == (x, y == 3, 4), z
```

which after 1 step of evaluation is:
```
a, (b = 1, c = 2) == (x = 3, y = 4), z
```

after another step:
```
(a, b = 1, c = 2) = (x = 3, y = 4, z)
```

finally evaluates to:
```
(a, b = 1, c = 2) = (x = 3, y = 4, z)
```

## Comments
------------------------------------------------------------------------------------------------------------

There is only one syntax for comments `//`.
Like most choices, this syntax serves multiple purposes and is done for consistency across codebases to avoid inconsequential variants and minimize holy wars.

Comments function in multiple ways:
* Comments a line
  ```
  // My header
  div .class = header main
  ```
* Inline comment
  ```
  div
    .class = header //( My header ) main
  ```
* Comments until the end of a line
  ```
  div 
    .class       = header main // My main header
    .style.width = 100%        // Full width
  ```
* Block comments
  ```
  // My Notes
    Notes both ignore the end of a line and it's children.
    So these are part of the comment as well.
  ```

Comments are data just like the rest of the syntax and makes them machine parsable.

## Escape
------------------------------------------------------------------------------------------------------------

The only escape character is `\`. It can be used in multiple ways:
* Escapes a single character
  ```
  Hello\tWorld\n
  ```
* Escapes whitespace
  ```
  my\ complex\ name = 1
  ```
* Escapes inline
  ```
  name = Jane

  "Hello \(name)!"
  ```
* Escapes expression
  ```
  x = 1
  y = 2

  "x + y is \(x + y)"
  ```

## Identifier (in progress)
------------------------------------------------------------------------------------------------------------

In order to model concepts like operators, prefix/suffix fixity, relators, literals, etc. it's important that:
* it's very clear what are allowed identifiers for each type
* concepts which combine into single words use orthogonal characters
* deliniation?
* 

* cases
  * relator
  * relator name
  * identifier
  * reserved
  * canonical
  * upper case
* 




## Reserved (in progress)
------------------------------------------------------------------------------------------------------------

We reserve some identifiers for future use. TODO...

* comptime `#`
  * `#do`
* `do`
* 


## Codegen (in progress)
------------------------------------------------------------------------------------------------------------

### Comptime
### Codegen emit where in subexp
### Deterministic Patterns

## Sugar (in progress)
------------------------------------------------------------------------------------------------------------

### Operators act-like expanded tree node
Now that we know how relators are modeled; we can now describe an additional syntactical tool. Most operators can become a tree node when in the expanded syntax form.

Since we've already discussed it; let's use `=` which is usually a binary, infix operator.

Equals can also act-like a relator in it's expanded form. It works the same as a `= \n` as discussed above and does what you'd expect.
```
x
  =
    a
    b
    c
  .
    x
    y
    z
```

is equivalent to:
```
x = (a, b, c)
  .
    x
    y
    z
```

This works for most syntactical operators since they all support a tree structure.

### Defaulting (or space relator?)

because, by default (explained in more depth at `Defaulting`), a `INDENT` signifies a parent-child relation, this is equivalent to:
```
x
  a
  b
  c
```

### Anonymous Item
There are cases where we want to represent a series of elements without a meaningful name.

There are a few notes:
* The only valid identifiers are `-` and `*`
* Each sibling (items on the same indentation level) must consistently use the same syntax
* The identifier for a sibling must only be at the start of a line in a multiline form
* The identifier acts like a tag which means it can be added just to make items more clear

Consider the psuedo code:
```
my-elems = [
  { a, b, c = 3 },
  { t, u, v = 6 },
  { x, y, x = 9 }
]
```

Using our current syntax we'd have  ...

With our current syntax there isn't a way to represent this without weird syntax and visual noise. To avoid issues like this we'll introduce new syntax using `-` and `*` which:
* work like most markup languages use them as list-item identifiers at the start of a line. If they're used immediately after the leading whitespace then it doesn't indicate an item.
* only mean 

Rules:
* must use consistently for siblings
  * can't mix identifier or add identifier or not
* 

## Literals
------------------------------------------------------------------------------------------------------------

[//]: # ( These are optional; when not present the values are interpretable as that type regardless. They are a means of adding constraints (like a Schema) )

* Containers
  * Sequence `()`
  * List `[]`
  * Map `{}`
  * String
    * inline `""`
    * block `"""`
  * Code
    * inline `` ` ``
    * block ```` ``` ````
* Primitives
  * Integral
  * Floating Point
  * Fixed Point
  * Constants
    * PI
    * 
  * 
* 

## Types
------------------------------------------------------------------------------------------------------------

Types are useful when modeling data as it constraints the possible values the variable may contain.

When creating a type, there are a few rules:
* Must start w/ an upper-case alpha character (really necessary? can't we derive if it starts w/ a numeric?)
* 

### Sugar
How to model `x:Int = 4` which expands to `x:(type = Int) = 4`?

## Schema
------------------------------------------------------------------------------------------------------------

TODO
* Look at other solutions: StrictYAML, OGDL, OpenInternet, GraphQL, other schema definition systems?
* 

## Precedence
------------------------------------------------------------------------------------------------------------

### Global Precedence
### Relative Precedence

## Issues
------------------------------------------------------------------------------------------------------------

* Mixing destructuring (`==`) w/ cases which wouldn't normally destructure (but should chain or something else).
  * There may be other interactions as well.
  * Consider:
    ```
    o.(a, b, c) =  1, 2, 3
    o.(a, b, c) == 1, 2, 3

    o.(a, b = 1, c) =  1, 2, 3
    o.(a, b = 1, c) == 1, 2, 3

    o.(a = 1, b = 2, c = 3) =  4, 5, 6
    o.(a = 1, b = 2, c = 3) == 4, 5, 6

    o.(a, b), t.(x, y) =  1, 2, 3, 4
    o.(a, b), t.(x, y) == 1, 2, 3, 4

    o.(a = 1, b), t.(x = 3, y) =  2, 4
    o.(a = 1, b), t.(x = 3, y) == 2, 4

    o.(a = 1, b = 2), t.(x = 3, y = 4) =  5, 6
    o.(a = 1, b = 2), t.(x = 3, y = 4) == 5, 6
    ```
  * since `x == y` is actually `(x) = (y)` doesn't that mean there shouldn't be any partial forms?
* Should destructuring require all names be matched to values of R-hand?
  * Cases:
    * L-hand == R-hand
    * L-hand <  R-hand
    * L-hand >  R-hand
    * L-hand or R-hand arity 0
    * L-hand or R-hand arity undetermined (eg stream, infinite sequence, ...)
  * eg `x:(a, b, c) = 0` should this be allowed? what would be the effect?
* How does one define structural names but also assign the parent a value?
  * That is case where children have name w/o value but want the parent to have a value
  * eg `x:(a, b, c) = 0` is interpreted as a destructure and `a` is assigned `0`. What if we wanted the result to be like `(x = 0):(a, b, c)`?
    * Is it worth it as structural names are less common than destructuring is?
* Too many ways to describe data each w/ their own quirks and nuances. Is there a more optimal, consistent, predicable solution?
  * Main factors: form, emit names, composition method, count (1, seq, ..), , correspondance/equivalence, ???
  * When to use each? what are the use-cases?
  * Forms
    * Multiline literal
      ```
      o = 0
        .a = 1
      ```
      * Emit `o` name
    * Multiple Statements
      ```
      o   = 0
      o.a = 1
      ```
      * Emit
        * 1st emit `o` name
        * 2nd statement emit `o.a` name
    * Literal
      ```
      o .a = 1 == 0
      ```
      * Emit `o` name
    * Chain
      ```
      o.(a = 1) = 0
      ```
      * Emit `o` name
    * Inline Exp
      ```
      (o = 0).(a = 1)
      ```
      * Emit `o` name
    * Destructure
      ```
      o, o.a  == 0, 1   // Using == operator
      (o, o.a) = 0, 1   // Using L-hand ()
      (o, o.a) = (0, 1) // Using both ()
      ```
      * Emit `o`, `o.a` sequence of names
    * Inline Exp Relator Literal
      ```
      (o = 0).a = 1
      ```
      * Emit `o` name?
  * Compositions of Forms
    * Methods
      * by sequence-element
      * by parent-relator
      * by exp-subexp?
  * Correspondance/Equivalence
    ```
    o = (1, 2, 3)
      .
        a
        b = 4
        c
      :
        d = 5
        e

    o = (1, 2, 3)
      .a
      .b = 4
      .c
      :d = 5
      :e

    o = (1, 2, 3)
      .(a, b = 4, c)
      :(d = 5, e)

    o = (1, 2, 3)
      .(a, b, c) = _, 4, _
      :(d, e)    = 5, _

    o = (1, 2, 3)
      .a, .b = 4, .c
      :d = 5, :e

    (o = (1, 2, 3)) .a, .b = 4, .c, :d = 5, :e

    (o = (1, 2, 3)).(a, b = 4, c):(d = 5, e)

    o .a, .b = 4, .c, :d = 5, :e  == (1, 2, 3)
    o .(a, b = 4, c), :(d = 5, e) == (1, 2, 3)

    o.(a, b = 4, c):(d = 5, e) = (1, 2, 3)

    o, o.a, o.b, o.c, o:d, o:e = (1, 2, 3), _, 4, _, 5, _
    o, o.(a, b, c), o:(d, e)   = (1, 2, 3), _, 4, _, 5, _
    ```
    * All not equivalent in all aspects b/c:
      * which names are emitted change based on which form
      * Not all forms are composable
        * Multiple lines/statements aren't w/n a inline form
  * Questions
    * Is the common pattern what names an expression will emit?
      * Different forms but same form:
        ```
        o .a, .b = 1, .c =  0
        o .a, .b = 1, .c == 0

        o .a, .b = 1, .c = 2 == 0

        o .(a, b = 1, c) =  0
        o .(a, b = 1, c) == 0

        o, o.a, o.b, o.c == 0, _, 1, _
        o, o.(a, b, c)   == 0, _, 1, _

        (o = 0).(a, b = 1, c)
        (o = 0) .(a, b = 1, c)
        (o = 0).(a, b, c == _, 1, _)
        (o = 0) .a, .b = 1, .c

        o = 0
          .(a, b = 1, c)

        o = 0
          .(a, b, c) = _, 1, _

        o = 0
          .a
          .b = 1
          .c
        ```
        * Think of expand rules?
* Use equal precedence for grouping?
  * eg `a + b == c / d == e * f` equivalent to `(a + b) = (c / d) = (e * f)` b/c `==` repeats and groups?
    * May not work b/c there can be multiple repeating operator-like thing `==`, `||`, ...
  * This may need to be paired w/ precedence, grouping, and associativity
* Not associative types to assure there's only 1 per expression
  * This forces developer to specify using `()`
  * eg `x = 0 = 1` isn't valid b/c `=` doesn't associate except `(x = 0) = 1` is valid b/c associativity is clear
  * and/or have a default associativity like a function? where associate to R and till EOL?
* If destructure vs not is determined by assigment on L-hand of `=` which level of `=` expression is poisoned?
  * Just the parent? The entire L-hand to the top-level expression?
  * consider:
    ```
    o.(x, y, z) = 1, 2, 3

    o.(x = 1, y = 2, z = 3)
    ```
    
    then, when composed:
    ```
    t.( o.(x, y, z) = 1, 2, 3 ) = 0

    ( o.(x, y, z) = 1, 2, 3 ), t.(a, b, c) = ...

    
    ```
* Edge cases
  * Inner destructure without parent?
    ```
    a, (b, c == 1, 2) == (x, y == 3, 4), z
    ```
  * Assignment on L-hand and destructure b/c no parent?
    ```
    (x, y = 1, z) = 0, 2
    ```
* Emit names into scope
  * There's only 2 cases for what to emit:
    * Unambiguous describe a single name
      * Forms
        * Literal
          * x .a             // emit x
          * x .a, .b         // emit x
          * x .(a, b)        // emit x
          * x .a = 1, .b = 2 // emit x
        * Multiline
          * x = 0            // emit x
              .a = 1
              .b
              .c = 2
        * Chain
          * x.(a = 1)         // emit x
          * x.(a = 1).(b = 2) // emit x
          * x.(a = 1, b)      // emit x
    * Describe one to many names
      * Cases: 0, 1, finite, unbounded
      * Forms
        * Sequence of Expressions
          * x = 0                           // emit x
          * x.a = 1                         // emit x.a
          * x, y = 0, z                     // emit (x, y ,z)
          * a, b = 0, c, d.(x, y) = 1, 2, e // emit (a, b, c, d.x, d.y, e)
        * Destructure
          * (x, y) = 1, 2                 // emit (x, y)
          * x.(a, b) = 1, 2               // emit (x.a, x.b)
          * (x, y.(a, b), z) = 0, 1, 2, 3 // emit (x, y.a, y.b, z)
  * Mix
    * (x, y.(a = 1, b), z) = 1, 2, 3 // 
      (x, y, z)            = 1, 2, 3 // After emit y
    * a, (b, c == 1, 2) == (x, y == 3, 4), z // 
      a, b, c           == x, y, z           // After inner eval will emit (b, c), (x, y)
      (a, b, c)         =  x, y, z           // After eval ==
* Compose methods
  * With current syntax can only compose in a few ways
    * Sequence - Sequence Element
      * chain
        ```
        (x, y.(a = 1):(b = 2), z) = 1, 2, 3
        (x, y, z) = 1, 2, 3
        ```
      * series of expressions
        ```
        (a, b, (x, y = 0, z), c) = 1, 2, 3, 4, 5, 6
        (a, b, (x, y, z), c)     = 1, 2, 3, 4, 5, 6
        (a, b, x, y, z, c)       = 1, 2, 3, 4, 5, 6
        ```
      * destructure
        ```
        (a, b, (x, y, z == 1, 2, 3), c) = 1, 2, 3, 4, 5, 6
        (a, b, (x, y, z), c)            = 1, 2, 3, 4, 5, 6
        (a, b, x, y, z, c)              = 1, 2, 3, 4, 5, 6
        ```
      * get
        ```
        (a, b, (x, y, z), c) = 1, 2, 3, 4, 5, 6
        (a, b, x, y, z, c)   = 1, 2, 3, 4, 5, 6
        ```
      * literal
        ```
        (a, b, (x .(t, u) :v), c) = 1, 2, 3, 4
        (a, b, (x), c)            = 1, 2, 3, 4
        (a, b, x, c)              = 1, 2, 3, 4
        ```
      * multiline
        ```
        a
        b
        x
          .t
          .u
          :v
        c
        ```
      * eg
      ```
      x, y = 1, z
      (x, y = 1, z) = 1, 2, 3
      (x, y.a = 1, z) = 1, 2, 3
      (x, y.(a = 1), z) = 1, 2, 3
      (x, (y .a = 1), z) = 1, 2, 3
      (x, y.(a, b), z) = 1, 2, 3, 4
      ```
      
    * Parent - Relator Element
      * eg
      ```
      x.(y.(a = 1)) = 0
      ```

    * Expression - Subexpression

* Is the real issue dealing w/ parent-child relationship vs sibling?
  * All that emit the parent name is parent-child while get/destructure are sibling?
  * eg consider:
    ```
    x.(y.(a = 1)) = 0     // results in x = 0
    
    vs

    x.(y .a = 1) = 0      // results x = 0
    
    vs

    x.(y.a = 1) = 0       // results in x = 0

    vs

    x.(a, b, c) = 0       // resutls in x.a = 0 or error?

    vs

    x.(a, b = 1, c) = 0   // results in x = 0

    vs

    x.(y, z == 1, 2) = 0  // results x = 0
    ```











## TODO
------------------------------------------------------------------------------------------------------------

## Compare to other syntaxes

### Paths and selectors

* From CSS
  * Basic
    * universal: `*`
    * type: `input`, `button`
    * class: `.my-header`
    * id: `#my-id`
    * attribute:
      * Syntax: [attr] [attr=value] [attr~=value] [attr|=value] [attr^=value] [attr$=value] [attr*=value]
      * `[autoplay]` will match all elements w/ `autoplay` to any value
  * Grouping
    * selector list: `div, span`
  * Combinators
    * descendent: `div span` match all `span` that are inside a `div`
    * child: `ul li` match all `li` nested directly under a `li`
    * general sibling: `p ~ span` match all `span` that follow a `p` immediately or not (not necessarily adjacent)
    * adjacent sibling: `h2 + p` match all `p` that directly follow an `h2`
    * column combinator: `col || td` match all `td` that belong to the scope of `col`
  * Psuedo
    * class:
      * state info that is not in the document tree
      * `a:visited` match all `a` visited by the user
    * element:
      * elements not included in the document tree
      * `p:first-line` match the first line of all `p` elements
* From SASS
  * 

### Issue: Inconsistent syntax
* Selecting uses the ends of a tree while chaining returns the target value
  * They have very similar syntax while the semantics are different
* Nesting forces them to assign values w/n a `()` like `x.(y, z = 4) = 3`
* Maybe rule is ???
  * Rules
    * In exp, if using selection only then will not pass head
      * eg `x:(a, b, b.(c, d))` will not pass `x` when an expression
                   vs
           `x:(a, b = 1, b.(c, d))` will pass `x` b/c b is using an assignment?
    * Otherwise, head is passed as first element to binding
  * Issues w/ composing this?
    * Does it cause side-effects one wouldn't expect?
      * What about when mixed w/ the multiple ways to select values?
* In general, when should we constrain the methods of assignments
  * Types:
    * selection/destructure (L-hand name and R-hand values)
      * eg `o.(x:(a, b), y/c, z), t = 1, 2, 3, 4, 5`
    * path
      * eg `o.x:a, o.x:b, o.y/c, o.z, t = 1, 2, 3, 4, 5`
    * w/n parens/subexp
      * eg `o.(x:(a = 1, b = 2), y/c = 3, z = 4), t = 5`
    * literal
      * eg `o .(x:(a = 1, b = 2), y/c = 3, z = 4); t = 5`
        * Should we terminate `o` using `;`? and to indicate `t` as a sibling of `o`?
    * multiline
      * eg 
        ```
        o.
          x:
            a = 1
            b = 2
          y
            c = 3
          z = 4
        t = 5
        ```
    * mixed
      * eg 

```
// This is inconsistent b/c inner sub-exp doesn't act like inner exp
// - x:(a, b) = 1, 2 === x:a = 1; x:b = 2
// - o.(...) === o = 3 .(...)
o.(x:(a, b) = 1, 2) = 3

o.(x:(a = 1, b = 2)) = 3

o.(x:a = 1, x:b = 2) = 3

o = 3, o.x:a = 1, o.x:b = 2

o, o.x:(a, b) = 3, 1, 2

o = 3
  .x:(a, b) = 1, 2

o .(x:(a, b) = 1, 2) = 3
```

```
o.(x, y, z) = 1, 2, 3

vs

o.(x, y = a, z) = 1, 2, 3
```

```
o.(x.y.(a = 1, b) = 2) = 3
```

### Same task, diff ways (consider what readable, composable, useful for multiple contexts, etc.)

* Allowed
  ```
  // Multiline
  o = 1
    .x = 2
      :a = 3
      :b = 4
    .y = 5

  // Build
  o          = 1
  o.(x, y)   = 2, 5
  o.x:(a, b) = 3, 4

  // Destucture w/ paths
  o, o.x, o.x:a, o.x:b, o.y = 1, 2, 3, 4, 5

  //

  ```
* Maybe
  ```
  // Destucture w/ parens
  o, o.(x, x:(a, b)), y) = 1, 2, 3, 4, 5

  // Literal w/ parens w/ R-hand value
  o .(x :(a = 3, b = 4) = 2, y = 5) = 1

  // Literal w/ parens w/ L-hand value
  o = 1 .(x = 2 :(a = 3, b = 4), y = 5)

  // Use chain implicit return to assign to R-hand value
  o.(x:(a = 3, b = 4) = 2, y = 5) = 1

  // Literal w/ terminal
  o = 1 .x = 2 :a = 3, :b = 4; .y = 5

  // Chaining implicit return of 1st value
  o.(x:(a = 3, b = 4) = 2, y = 5) = 1

  // Chaining
  o.(x:(a = 3):(b = 4) = 2).(y = 5) = 1

  // Implicit return name in subexp
  (o = 1).((x = 2):(a = 3, b = 4), y = 5)

  //

  ```
* Not Allowed
  ```
  ```

### Complex, nested example
```
// Multiline
o = 1
  .a = 2
  .b = 3
    :x = 4
    :y = 5
  .c = 6
  :d = 7
  :e = 8

// Incremental build syntax TODO!

// Not possible? when and what is possible for this form?
o = 1 .a = 2, .b = 3 :x = 4, :y = 5; .c = 6, :d = 7, :e = 8

// possible using `;` terminal? reads terribly...
o .a = 2, .b :x = 4, :y = 5; = 3 .c = 6, :d = 7, :e = 8; = 1

// literal has similar purpose of connected w/ paren?
o .(a = 2, b :(x = 4, y = 5) = 3, c = 6), :(d = 7, :e = 8) = 1

// chaining has similar purpose to spaced w/ paren?
o.(a = 2, b:(x = 4, y = 5) = 3, c = 6):(d = 7, :e = 8) = 1

// paths to destructure
o, o.(a, b, b:(x, y), c) = 1, 2, 3, 4, 5, 6
```

```
f 1, 2, g 3; 4, h 5, 6; 7

f(1, 2, g(3), 4, h(5, 6), 7)
```

### Orthogonal syntaxes
* Purposes
  * get using paths
  * assign: path on L-hand, values on R-hand
    * This may affect destructure and chaining?
  * 

### Nesting
* almost looks like default values of a function?
* 

o
  .x = 1
  .y = 2
    :a = 4
    :b = 5
    .t = 6
    .u = 7
  .z = 3

o .(x = 1, y :(a = 3, b = 4), .(t = 6, u = 7) = 2, z = 5)

o .(x = 1, y:(a = 3, b = 4).(t = 6, u = 7) = 2, z = 5)

or don't allow it? or only allow in multiple lines when nesting?

### Mixed
* Problems
  * path and inline assignment syntax too close where it becomes hard to tell
    * need to be clear about when selecting, when assigning, when def literal, etc. and should ORTHOGONAL task
      * select must use L-hand for paths to names and R-hand for value
      * literal w/ space
      * define via chaining
      * define via

o .x = 1, .b, .c = 1, 2 // Does this mean b, c = 1, 2 or o?






## Invalid, Dissallowed, Issues

### object literal vs equals?

### Determine when we have assignment and literal what name is exposed?
```
obj 1, 2, 3
```
vs 

```
o = obj 1, 2, 3
```

```
obj 1, 2, 3

o = obj
```
* is `o` or `obj` mutable?
* which name is exposed? `o` or `obj`?
* any copying? or only referencing?
* issue of data literal vs reference?
* issue is the `o` is a variable while `obj` is a tree literal
  * could make a rule that only variables are mutable while tree literals are not?


### Determine if this is allowed or not (consider w/ nested functions)
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
This is close but we specifically disallow having nested inline relators. Notice that `/1, /2, /3` are children of `b` and are nested under `x`. This is disallowed to make the code readable and allows us to make the assumption that a inline relator always refers to the first element. We'll be able to better model this w/ syntax introduced later.

### Make it clear what's reserved

### Nested assignment
Meaning of:
```
x
  a = 1
  b = 2
    y = 3
  z = 4
```
Ideas:
* Just a series of entries w/ a mapping
  * Depends on interpretation if valid semantically or not?
* 

### Top level
* literals
* adjacent lines
* names/variables?

### Inline Sequence of Sequence (& top level?)
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
  * `'( (a, b, c), (x, y, z), (1, 2, 3) )` in LISPy languages
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

### Access
Accessing relator names syntactically acts more like other languages.

to access:
```
obj.b
```
which is read as:
* access `obj` name
* access `.` relation in `obj`
* access `b` name in `.` relation
* return value of `b`

to assign:
```
obj.b = 3
```
which is read as:
* access `obj` name
* access `.` relation in `obj`
* access `b` name in `.` relation
* name `b` equals 3

[//]: # (TODO: obj.* get and set tuple?)

### Parent-Child Relator

Here's an example:
```
x
  /a
  /b
  /c
```

this is equivalent to:
```
x
  /
    a
    b
    c
```

and can be inline:
```
x /a, /b, /c
```

to access:
```
x/c
```

to assign:
```
x/c = 1
```

### Object-Property Relator

Here's an example:
```
obj
  .a = 0
  .b
  .c = 2
```

this is equivalent to:
```
obj
  .
    a = 0
    b
    c = 2
```

and can be inline:
```
obj .a = 0, .b, .c = 2
```

to access:
```
x.c
```

to assign:
```
x.c = 3
```

### Data-Metadata Relator

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

and can be inline:
```
Age :type = Int, :min = 0
```

to access:
```
x:min
```

to assign:
```
x.min = 1
```

Elder levels:
* Syntax (s-exp, StrictYAML)
  * with implicits (JSON)
  * with schema (InternetObject, StrictYAML w/ Schema)
  * with types
  * with variables
  * with functions

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

### Destructure
* Complex?
  * Incomplete forms?
* Wildcards
* Skip `_`
  * Skip multiple? `_(2)`
* Will destructure automatically if:
  * No values are assigned?
  * Is totally ambiguous L-hand
    * eg `x = 4`
    * eg `(x, y) = 1, 2`
      * L-hand values are a sequence literal `(...) = ...` (eg `o.(a, b, c) = 1, 2, 3`)
* `==` forces destructuring
  * Desugars to `) = (`
* Issues
  * data vs execution context
    * Ambiguous between execution context or data context where precedence of `,` and `=` are swapped
      * in data, precedence order: `=` > `,` (for multiple elems)
      * in exec, precedence order: `,` > `=` (for destructure)
    * How to know if data or exec context? may not be worth it...
  * Compare:
    ```
    x = a
    x = a, b, c
    x = (a, b, c)
    x == a
    x == a, b, c
    x == (a, b, c)
    ```

* Syntax
  * Consider using `==` for destructure which acts just like `=` but a lower precedence than `,`
    * This means we can then easily mix `=`, `,`, and `==` universally?
    * Precence order will be like:
      * `=` top
      * `,` middle
      * `==` bottom
    * Then can also compose other aspects we have to consider like:
      * Shadowing using `:=` or `:==`
      * Constness using `-=` or `-==`
        * Likely constness needs to be it's own thing b/c it's rather nuanced?
      * Mixed liked `-:=` or `-:==`?

or as destructure:
```
o.a, o.b, o.c, o:x, o:y, o:z = 1, 2, 3, 4, 5, 6
```

or as destructure:
```
o.(a, b, c), o:(x, y, z) = 1, 2, 3, 4, 5, 6
```

or even as a inline literal with destructure subexp (TODO: precedence an issue here between `,` and `=` or can be dissambiguated? possible to dissambiguate or must choose?)):
```
o .(a, b, c = 1, 2, 3), :(x, y, z = 4, 5, 6)
```

### Juxtaposition

`o .x .y .z`

`o .x, .y, .z`

### Distribute

* based on type
  * For relators, paths are distributed
  * For numbers, do what?
  * For symbol, do what?
  * For fn, do what?
* syntax pattern
  * options
    * `el (...)`
    * `el(...)`
    * `(...) el`
    * `(...)el`
    * `(...) (...)`
    * `(...)(...)`

## Relator
### Nested Inline? like subexp or nested fn? eg `o .x .y .z`
### Declaration Sugar

## Parenthesis
// Then nest them? and show how need better syntax

```
o
  .
    a = 1
    b = 2
      :
        x = 4
        y = 5
        z = 6
    c = 3
```




  ```
  o
    .x
      :a = 1
      :b = 2
    .y
      c = 3
    .z = 4
  t = 5
  ```

  ```
  o = 1
    .x = 2
      :a = 3
      :b = 4
    .y = 5
  ```

  ```
  o.
    a = 1
    b = 2
    c.d.
      x = 3
      y = 4
  ```




### TODO: selection

### TODO: selection and assignment (w/n and not)
o.
  a = 1
  b = 2
  c.d.
    x = 3
    y = 4

o .(a = 1, b = 2, c.d.(x, y) = 3, 4)
o .(a = 1, b = 2, c.d.(x = 3, y = 4))
o.(a, b, c.d.(x, y)) = 1, 2, 3, 4


o.(x, y) = 1, 2
o.(x = 1, y = 2)
o.(x, y = 1, 2)
o.(x = 1, y = 2) = 3, 4
o.(x, y = 1, 2) = 3, 4

### Advanced Destructure

o.(x, y, y:a, y:(t, u)) = 1, 2, 4, 6, 7

o.(x, y).y(:a, .(t, u)) = 1, 2, 4, 6, 7

o.(x, y), o.y(:a, .(t, u)) = 1, 2, 4, 6, 7

### Chaining
------------------------------------------------------------------------------------------------------------

It useful (and common) to have syntax like `x:Int = 0`. We can't represent this exactly with our current syntax but we can get close using a pattern named chaining.

Chaining is meant to be a very natural extension of the the canonical usage of `=`. Here's a simple logical progression of the syntax:
* Assign a name to a value
  ```
  x = 4
  ```
* Add some metadata to a name as well as assign it to a value
  ```
  x:(type = Int) = 4
  ```
* Add some additional metadata
  ```
  x:(type = Int, min = 0, max = 1024) = 4
  ```
* Add some properties
  ```
  x:(type = Int, min = 0, max = 1024).(a = 0, b = 1) = 4
  ```
* For clarity, here's the expanded form:
  ```
  x = 4
    :type = Int
    :min = 0
    :max = 1024
    .a = 0
    .b = 1
  ```

The effect of chaining is to pass along the name on the L-hand of `=` so it can be related to the R-hand value. This isn't always desirable so there are rules when chaining will occur:
* Within an expression, if all names aren't assigned values it will not chain. Conversly, if any name is assigned it will.
  * This is b/c when destructuring we don't set values on the L-hand, just reference them. We need the syntax to be clearly different between destructuring and chaining.
* It must be inline
* Generally chaining should be avoided if there's too much information and the expanded form should be used instead but this isn't enforced

TODO:
The effect of these rules is a simple syntax rule:

TODO:
The effect of this rule is that it's not possible to both define data on the L and R hand of a `=` in a arbitrary way. This segments the type of `=` syntaxes into 2 tasks:
* Destructuring where the L-hand is references to one or many names
* Data definition where the L-hand is describing data and values. Depending on the syntax, this may chain.


Here's a few valid examples which will chain:
* Simple inline
  ```
  x:(type = Int) = 0
  ```
* Some values set, but not all
  ```
  x:(a, b = 1, c) = 0
  ```
  which is equivalent to the more expanded form:
  ```
  x = 0
    .a
    .b = 1
    .c
  ```
  which chains although not all the names on the L-hand are set, some of them are.

  This works b/c when values are set on the L-hand side this generally only occurs when we're defining data. Generally we only want chaining when defining data.

  When values aren't set (here `a` and `c`) they are still present and assumed to be structural names which may be used later even w/o a value.
* Inner subexpression destructure
  ```
  x:(a, b, c == 1, 2, 3) = 0
  ```
  which chains b/c in inner subexpression is evaluated before the outer expression.
* Multiple chain relators
  ```
  x:(type = Int).(a, b = 1, c) = 0
  ```
  this is equivalent to the more expanded form:
  ```
  x = 0
    :type = Int
    .a
    .b = 1
    .c
  ```
  which chains b/c some names are set on the L-hand.

  What's unique about this form is that it's allowable to have multiple relators on the L-hand. This is why we used the word chaining b/c it's possible to chain multiple relators and each will emit the starting name.

and a few counter examples which won't chain:
* Path
  ```
  x:a.b = 0
  ```
  this is read as a path `x:a.` to the name `b`. This means the name `b` is emitted from the L-hand which relates to value `0`.
* Destructure
  ```
  x:(a, b, c) = 0
  ```
  This is read as a destructure b/c no names are assigned on the L-hand side. This means the names `a, b, c` are emitted from the L-hand. Since there's only a single value on the R-hand it's only related to `a`.
* ???
  ```
  (x, y = 1, z) = 0, 2
  ```
  There's nothing to chain here but there's also assignment on L-hand...
