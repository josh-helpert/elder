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

To understand this concept better consider other languages which are often hard coded:
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

Although we strive for tersness, parenthesis are essential. They are a tool for a developer to specify information to the compiler but are not within the grammar.

Let's start with a simple example where we want to relate `x` and the sequence `a, b, c` you may expect something like this to work:
```
x = a, b, c
```
This doesn't work like you expect because the precedence of `=` is higher than `,`.

This means it is interpreted as:
```
x = a
b
c
```

To get what we want we have to add parenthesis to alter the precedence:
```
x = (a, b, c)
```
An important point is that the `()` don't make `a, b, c` a sequence, the `,` does. Instead, the `()` alters the precedence my making `a, b, c` into a group.

Think of parenthesis as a way to create a temporary group to provide information to the software (and human) reader. Parenthesis group it's contents together but will often be dropped when the reader encounters them.

Parenthesis perform multiple duties depending on how they're used:
* Like in most languages they're the highest precedence:
  * expression  `(1 + 2) * 3`
  * infix operator `x = (1, 2, 3)`
* Is the start `(` and stop `)` of an operator, relator, function, etc.:
  * expression `(1 + 2)`
  * operator `+(1, 2)`
  * function `f(1, 2)`
  * relator data definition `o.(a, b = 1, c)` or `o .(a, b = 1, c)`
  * relator selection `o.(a, b, c)`
* A Block which groups statements, often unevaluated, together:
  * inline conditional `if (x > 1 and y < 2) then (log x) else (log y)`
  * inline while `while (x > 0) (log x; x -= 1)`

Parenthesis we're designed to work in a compatible way between the multiple ways they're used. Often the developer is forced to use parenthesis in cases where it's ambiguous, rare, or to be consistent with other syntax. For example, `x = (1, 2, 3)` is required to describe defining a sequence to also make it compatible with other literals (we'll go in depth later):
```
a = (1, 2, 3)             // Sequence
b = [4, 5, 6]             // List
c = {x = 7, y = 8, z = 9} // Map
```

### Parenthesis are not a container

Since parenthesis are not a container, they do have a effect that's novel. Consider the following psuedo code:
```
x = (1, 2, 3)
y = (4, 5, 6)
z = (x, y)
```

The value of `z` won't be 2 sequences like one may expect. Instead here's the process which starts by substitution in place:
```
z = ((1, 2, 3), (4, 5, 6))
```

Then the inner expression is reduced:
```
z = (1, 2, 3, 4, 5, 6)
```
Since there's nothing left to do with the inner expression the parenthesis are dropped.

This may seem weird but it's a natural extension to do what we do in other languages where the options are usually:
* return a single item like `4`, `8.7`, `'c'`, `"Hello"`
* return 0 to many items w/n some type of container like `[1, 2, 3]`, `{x, y, z}`

What parenthesis allow us to do is add another category which act like multiple, single items w/o a container. This allows us to avoid extra wrapping and unwrapping of data which leads to more visual noise.

For example, consider this psuedo code where we use an external function to construct the properties of a HTML-like element:
```
f = Fn () -> Seq
  (width = 100%, height = 20px)

header .(f(), background-color = FFF)
footer .(f(), background-color = 000)
```

This results in what you'd hope w/o having to unwrap the function result:
```
header .(width = 100%, height = 20px, background-color = FFF)
footer .(width = 100%, height = 20px, background-color = 000)
```

### Example - Common Structure

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

## Common Patterns
------------------------------------------------------------------------------------------------------------

At this point we've introduced all the basic syntax needed to describe the common syntactical patterns.

For each pattern, there's a few effects to consider:
* When a pattern in used next to it's siblings; what name(s) are visible to other siblings.
  * For example, below `x` and `y` are siblings but `z` isn't:
    ```
    a
      x
      y
    b
      z
    ```
  * For convenience, we'll consider a "scope" as what siblings and their decendents can access. For Elder syntax, these are often just names and paths to those names.
* When used as a node in a tree what is returned by the subtree.
  * For example, the middle element is a sequence which is both a subtree and subexpression. It is the sequence `(b, c)`:
    ```
    (a, (b, c == 1, 2), c)
    ```
    * TODO: Is the result `(a, b, c, d)` or `(a, (b, c), d)`? How would it be differentiated as a subexpression/subsequence?
      * Like other cases, it's about the parent vs the children? the container vs the contents? ...

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
  * This can only be composed within other multiline literal patterns as it's the only pattern which uses multiple lines.
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
* `(a, (b, c, d), e)` is?
  ```
  - a
  - b
    c
    d
  - e
  ```
  or
  ```
  a
  b
  c
  d
  e
  ```
  * Need different syntax when expression vs subsequence? Need to differentiate container vs contents?
    * eg can emit literal using syntax like `=(b, c, d` which forces `(a, =(b, c, d), e)` to be `(a, b, c, d, e)`
  * Different contexts mean different results
    * replaced with result
      * functions
      * operator
      * expression
    * returns sequence
      * literal
      * selection
        * x.(a, b):(c, d)
      * destructure
        * (a, b == 1, 2)
      * relator selection
        * examples
          * a vs (a)?
          * x.a vs x.(a)?
          * a, b, c vs (a, b, c) vs x.(a, b, c)
          * ((a, b, c), (d, e, f)) vs (a, b, c, d, e, f)
* How is `3 + 4` in Elder syntax?
  * Not clearly a sequence?
  * A sugar for infix operator? desugars to `+(3, 4)`?
* Be clear `x.(a, b, c)` is names `(x.a, x.b, x.c)` not `(a, b, c)`
* Invalid syntaxes
  * x, y, z, y == 0, 1, 2, 3
    * Cannot repeat same name in selection?
  * Redef structure?
