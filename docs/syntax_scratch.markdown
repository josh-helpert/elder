### Reduction

(1)
(a)
(1 + 2)
(1, 2)

### Expand

Different expansions:
* `o.(a, b = 1) = 0`
  * Groups and expands to:
    ```
    (o = 0; o.a = _; o.b = 1)
    ```
* `o.(a, b) = 1, 2`
  * Groups and expands to: `(o.a = 1, o.b = 2)`
    ```
    (o.a = 1, o.b = 2)
    ```
  * Additional parenthesis aren't needed b/c clear w/ parenthesis before expansion?

### Function and Relator usage pattern

### Conflict





Cases
* Precedence
  * `x = (3 + 4) * 5 => x = 35`
  * `x = (1, 2, 3)`
* Code/Block
  * should signatures explicitly look for a block? or should a single expression work?
    * eg
      ```
      if
        x > 0
        y < x
        z > x
      then
        log x, y, z
        z
      else
        log x, y, z
        x
      ```
      translates to shorter:
      ```
      if (x > 0 and y < x and z > x)
      then (log x, y, z; z)
      else (log x, y, z; x)
      ```
      translates to inline:
      ```
      if (x > 0 and y < x and z > x) then (log x, y, z; z) else (log x, y, z; x)
      ```
    * Pattern seems to be a group of children can be parens?
      * relators use this as well:
        ```
        x
          .a
          .b
          .c
        ```
        is also:
        ```
        x .(a, b, c)
        ```

        just like:
        ```
        x
          a
          b
          c
        ```
        is also:
        ```
        x (a, b, c)
        ```
  * eg
    * `if (...) then (...) else (...)`
    * `when (...) (...)`
    * `cond (...) ( (...), (...) )`
* DRY/path
  * `o.(a, b = 1, c) = 0`
  * `o.(a, b, c)`
* other
  * function
    * f(a, b, c, d, e)
    * f(a, (b, c, d), e)
    * f (a, b, c, d, e)
    * f a, b, c, d, e
    * f a, (b, c, d), e
    * f (a, (b, c, d), e)
    * f(a, [b, c, d], e)
    * f [a, [b, c, d], e]
    * f([a, [b, c, d], e])
    * f(a, g(b, c, d), e)
    * f(a, (b + c + d), e)
  * distribute
    * 4 (1, 2, 3)
  * structure
    * same as Code/Block?
      * desirable in any way outside of Block b/c we actually don't want permanent structure from `()` but use `[]`?
  * 

## Brackets and Parenthesis
------------------------------------------------------------------------------------------------------------

Parenthesis Issues:
* What about forms where there may be a mistake and unclear if to eliminate parens?
  * equals
    ```
    x = (1)   => x = 1
    x = ((1)) => x = 1

    a = 1
    x = (a)   => x = 1
    x = ((a)) => x = 1

    x = (1 + 2)   => x = 3
    x = ((1 + 2)) => x = 3

    x = (1, (2, 3), 4)   => x = (1, 2, 3, 4)
    x = ((1, (2, 3), 4)) => x = (1, 2, 3, 4)


    x = f(x)     => 
    x = (f(x))   => 
    x = ((f(x))) => 
    ```
    * Don't want for equals to implicitly evaluate b/c data forms don't evaluate
      * What do they represent then? Just a grouping? the shape of data?
      * or should I add some sugar so that parenthesis are dropped in a similar way so that the data and eval forms match in practice?
* `()` need to be orthogonal to lists `[]` so clear when to use each
  * Since `()` may be dropped not to use them when you want to store the sequence forever?
* `()` reduction rules need to be clear
  * when they'll be dropped
  * what is and isn't dropped or evaluated
  * what is structure
* forms
  * function
    ```
    f a, b, c
    f(a, b, c)
    f (a, b, c)
    f ((a, b, c))
    f((a, b, c))
    ```
  * sibling functions
    * if
      ```
      ```
    * for
      ```
      ```
* Which forms have priority?
  * eg DRY with expression
    ```
    o.(a, b = (1 + 2), c = 3) => ?
    ```
  * 

We have to model tools for both data and code. Consider each aspect:
* aspect/task
  * precedence
    * eg `(3 + 4) * 5`
    * eg `x = (1, 2, 3)`?
    * only useful for infix operators?
  * grouping
    * eg `f(x, y), z`
  * sequence
    * eg `1, 2, 3, 4`
    * eg `1, (2, 3), 4`
    * eg `1, [2, 3], 4`
  * nesting inline sequence
    * eg `1, [a, b, c], 2, 3`
  * DRY, path, common
    * eg `o.(a, b) => o.a, o.b`
  * start/stop
    * eg `o .(a = 1, b = 2, c = 3)`
  * block
    * Used to group into a block so that multiple statements are together
      * Not sure if `()` or `[]` is more appropriate?
        * If we use `()` then it's used for both grouping and blocks while `[]` is reserved for data while `()` is never data.
    * eg inline conditional
      ```
      if (x < 0 and y > 1) then (log x; 0) else (log y; 1)

      if(x < 0 and y > 1 ) then( log x; 0 ) else( log y; 1 )

      if( x < 0 and y > 1 )then( log x; 0 )else( log y; 1 )
      ```
  * code
    * 
  * 
* 




Part of issue may be determining equivalences so that the syntax can flow more cleanly?
* eg conditional
  * multiline
    ```
    if
      x < 0
      y > 1
    then
      log x
      0
    else
      log y
      1
    ```
  * fn-like
    ```
    if(x < 0, y > 1) then(log x; 0) else(log y; 1)
    ```
  * group-like
    ```
    if (x < 0, y > 1) then (log x; 0) else (log y; 1)
    ```
  * keyword-like
    ```
    if x < 0, y > 1 then log x; 0 else log y; 1
    ```
  * block-like
    ```
    if [x < 0, y > 1] then [log x; 0] else [log y; 1]
    ```
* Rebol
  * either
    ```
    either (rebol-is-very...interesting? = true) [
        print "Is Rebol very interesting?"
        print "Signs point to yes"
    ] [
        print "Is Rebol very interesting?"
        print "My sources say no"
    ]
    ```
* Clojure
  * defn
    ```
    (defn greet
      [name]
      (str "Hello, " name))
    ```


Reduction rules
* Determines when a `()` will be removed
* How does outside operations (like `do`, `eval`, `reduce`, `reform`, ...) affect and distribute w/n `()`
  * Diff things distribute based on outside and inside?
    * eg expression inside and integer juxtaposition outside
    * eg sequence of names inside relator outside
* 
* 


Maybe add convenience for parenthesis as they don't represent data so that we can have cleaner syntax?
* add space
  ```
  f a, b, c, d, e   === f(a, b, c, d, e)
  f(a, b, c), d, e  === f(a, b, c), d, e
  f (a, b, c), d, e === f(a, b, c), d, e
  f a, b, c; d, e   === f(a, b, c), d, e
  ```
* chain sibling?
  ```
  f (a, b, c) g (x, y, z) === ???
  ```



Since Elder uses the same syntax for data and code there needs to be a way to group data and group code. Elder differentiates these by using:
* `()` for:
  * Precedence in computations (like most languages): `5 * (3 + 4)`
  * Represents the start `(` and stop `)` of:
    * expression like `(1 + 2)`
    * operator like `+(1, 2)`
    * function like `f(1, 2)`
    * relator like `o.(a, b)`
  * Groups elements together
    * like `x = (1, 2, 3)`? or must use `[]`?
    * like `` `1, (2, 3), 4` `` in code block?
* `[]` for:
  * Represents the start `[` and stop `]` of:
    * sequence literal like `1, [2, 3], 4`
    * multiline anonymous element like:
      ```
      1
      [
        2
        3
      4
      ```
    * 


When one is composing data, there's really only two locations you generally want the sub-data to be located:
* As a sibling to other elements
  * This is referred to as an atom in LISP
  * eg the sequence `1, (2 + 3), 4` should result in the subexpression `(2 + 3)` being inline to it's siblings `1, 4` like `1, 5, 4`
* Isolated w/n a container
  * In LISP, ...
  * In Rebol, ...
  * eg the 

LISP implementations differentiate a expression `(...)` from a sequence literal `` `(...) `` by including a backtick while making brackets `[...]` used as a way to differentiate from parenthesis but semantically the same or sometimes as a `Vector` literal (a la Clojure).






As a reminder, a list of names delineated by commas is a sequence. With our current syntax it's not possible to represent nested sequences inline.

Consider the simple multiline example:
```
a
b, c, d
e, f
```

Notice that this is really a sequence of 3 elements which are themselves sequences. To model this inline we'll have to introduce new syntax like:
```
a, [b, c, d], [e, f]
```

### Parenthesis expanded

* When does expand?
  ```
  f: () -> (1, 2)
  ```
  * Consider using this:
    * Use siblings
      ```
      count(0, f(), 3)
      = count(0, (1, 2), 3)
      = count(0, 1, 2, 3) // Drop () b/c nothing to do?
      = 4
      ```
    * Use immediately
      ```
      count(0, count(f()), 3)
      = count(0, count((1, 2)), 3)
      = count(0, count(1, 2), 3)
      = count(0, 2, 3)
      = 3
      ```
      * Notice subexp are fully reduced before outer expression begins
    * Wrap w/ []
      ```
      count(0, [f()], 3)
      = count(0, [(1, 2)], 3)
      = count(0, [1, 2], 3)
      = 3
      ```
    * Wrap w/ ()
      ```
      count(0, (f()), 3)
      = count(0, ((1, 2)), 3)
      = count(0, (1, 2), 3)
      = count(0, 1, 2, 3)
      = 4
      ```
* This more trouble that it's worth b/c it's like auto-composing w/n a context?
  * the rule can be:
    * whatever the subexp returns it's written in place
      * if it's a singular value then it's written unwrapped there
      * if it's multiple values it's written unwrapped there
    * use `[]` for sequence literals and `()` for grouping, precedence/ordering, evaluation, distribute, start/stop, ...
    * 
  * How to model quoting which changes eval?
    * We use code literals which surround it? Like markdown does?
      * Either use `\` or more backtick than subexpression uses w/ spaces?
    * 
### Brackets vs Parens

### Optional syntax for grouping

Use-cases:
* Common
  * Sequence of single values (eg `x, y, z`)
  * Key-Value is single key and name (eg `age = 26`)
  * Value is a sequence (eg `staff = [ Jile, Bob, Jane ]`)
  * Value is result of evaluation (eg `next = get-next()`)
* Somewhat common
  * A few conjoint values (eg `name = Josh Helpert`)
  * 
* Rare
  * A few conjoint names (eg `first name = Josh`)
  * Complex structure

Common-sense rules:
* words are delineated by spaces. Since the most common these are siblings.
  * eg Elder `x y z` === JS `[ x, y, z ]`
* key-value are delineated by `=`.
  * When inline, the words to the left and right become the name and value
    * eg `x = 1 y = 2 z = 3`
  * When inline name and multiline value, the left becomes the name and the indented children become the value
    * 

* Not ideal to have commas be optional for data? even if slightly noisier?
  * Data are like: Nouns or Direct Objects
    * noun-like:
      * they are what verbs use as their context, choose implementation, configure the function
        * OOP not necessary as often the state is stored in noun-like containers and supplied to functions. This serves a similar conceptual purpose.
      * Having multiple nouns doesn't make sense but all the most rare cases
    * direct object-like
      * what's operated on by the function
      * they are what's acted upon and are often function params
      * Having multiple direct objects is very common
  * Functions are like: Nouns, Verbs, or Direct Objects
    * noun-like
      * this is useful for refinement or meta or composition
    * verb-like
      * they are supplied state, called on a noun-like, or supplied direct object-like
    * direct object-like
      * this is where implementation is selected by context and passed into the calling function
* The difference between using commas and not isn't that it's not possible for each to act in a certain way but rather that they all act in a role.
  * When their role is:
    * noun-like
    * verb-like
    * direct object-like

### Compare to other syntaxes

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

### Levels

* Simple Structure
  * S-Expressions
* Multiple Structures
  * StrictYAML
* Literals
  * YAML
  * JSON
  * 
* Types
  * 
* Schema
  * StrictYAML
  * InternetObject
  * 
* Comptime
  * Codegen
  * Comptime Proofs
  * 
* Decidable
  * Dhall
* Homoiconic
* Next?
  * Metalang
  * Refinement Types
  * Dependent Types
  * 

### Return data constrains of any type
* Extact data values (4, :String)
* Values w/n a range (3..5)
* Type
* Shape of data (*:type = Any)

### Get value vs reference
x = 4   // Def x
y = x   // Ref x
z = x() // Value x

## More Destructuring
------------------------------------------------------------------------------------------------------------

### TODO: matching arity, multiples, skip, ...

## Parenethes and Brackets

* Concat
* A Block which groups statements, often unevaluated, together:
  * inline conditional `if (x > 1 and y < 2) then (log x) else (log y)`
  * inline while `while (x > 0) (log x; x -= 1)`
  * group data together like `((Billy Bob), 35)` as long as it's not evaluated

## Type signature
* arity and types
  * (3 * Bool) === 3 bool expression
  * (Bool, Int, Bool) === sequence of Bool, Int, Bool
  * [3 * Bool] === 3 bool expression w/n brackets

## Parens or blocks determine evaluation
* eg `for ()` vs `for []`
* form, concat, etc. (like Rebol) don't work for `()`
* block is both a list and a type of execution. How to know which acting as?
  * determined by ctx, parent, interpretation, etc.?

### Container and Type meaning
Body for container
  Containers are interpretation contexts

### Example - Covert Parenthesis and Blocks

It's trivial to covert from parentheses (really just a sequence) into a block. You just have to wrap it w/ brackets:
```
x = (1, 2, 3)
y = [ x ] => [ (1, 2, 3) ] => [ 1, 2, 3 ]
```

The opposite, brackets to parentheses, is more complex as we essentially want to get the body of the block. 


Parentheses to Block

Block to Parentheses




* Block relation to indentation
* Anon item
* examples
* edge-cases
  * conditional `if (x > 0 and y < 10) then [log x; x] else [log y; y]`
