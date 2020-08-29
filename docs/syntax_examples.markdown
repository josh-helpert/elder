---
layout: default
title: Elder Syntax by Example
description: Learn by valid, invalid, and other language examples
permalink: /elder/syntax-examples
---

This document overviews syntax examples without much discussion and is meant as a quick overview.

Notice this doesn't describe the semantics of a language which uses this syntax, only the syntax itself.

## Sequence
---
* Ordered sequence of elements
* 

Multiline
```
x
y
z
```

Inline
```
x, y, z
```

## Tree
---
* Sequences of Sequences
* Mimics natural layout of formatted code
* Indent implies parent-child relationship
  * Only 2 spaces allowed (no tabs, no variable spaces) to assure consistent formatting across codebases
* 

Multiline
```
x
  a
  b
    1
    2
    3
  c
```

compact nested children
```
x
  a
  b /(1, 2, 3)
  c
```

compact even more to inline using parenthesis
```
x /(a, b /(1, 2, 3), c)
```

or using sequence terminal `;`
```
x /a, /b /1, /2, /3; /c
```

### Anonymous head
* Necessary for many structures
* Helps ability to emphasize heirarchy

We use syntax to mimic bullet `*` and dash `-`. Both are:
* Interpreted as head if at the start of a line after indentation
* Only used in multiline form
* `*` and `-` both allowed to make complex structures level clear and are interchangeable
* When infix means multiplication and subtraction as expected (eg `2 * 4`, `2 - 4`)

Consider a staff listing structured by departments then members
```
staff
  *
    - Billy Bob
      38
    - Jane Jill
      55
  *
    - Hillary Hill
      27
    - Thomas Tom
      20
```

equivalent to
```
staff
  * - Billy Bob
      38
    - Jane Jill
      55
  * - Hillary Hill
      27
    - Thomas Tom
      20
```

equivalent to
```
staff
  * - Billy Bob,    38
    - Jane Jill,    55
  * - Hillary Hill, 27
    - Thomas Tom,   20
```

also equivalent to (b/c `*` and `-` are interchagable)
```
staff
  - - Billy Bob,    38
    - Jane Jill,    55
  - - Hillary Hill, 27
    - Thomas Tom,   20
```

is equal to
```
  * * Billy Bob,    38
    * Jane Jill,    55
  * * Hillary Hill, 27
    * Thomas Tom,   20
```

which would encode to JSON like
```
{
  "staff" = [
    [
      [ "Billy Bob", 38 ],
      [ "Jane Jill", 55 ]
    ],
    [
      [ "Hillary Hill", 27 ],
      [ "Thomas Tom",   20 ]
    ]
  ]
}
```

## Equals
---
* Relates name on left hand to natural value on right hand
  * Natural value is result of executing the right hand
* Right hand is **always** executed **if** statement is evaluated
* Only defined as infix and binary
* Precedence is determined by the context

Used in Block context means assignment
```
x = 1
y = 2
z = 3
```

used in Map context means key-value
```
Map
  x = 1
  y = 2
  z = 3
```

Used in function input arguments context means default value
```
my-fn = Fn (x = 1, y = 2, z = 3) -> Int
```

## Relation space
---
* Abstracts over syntax which is hard-coded, are reserved, or not present in other languages
* Describes relation between a target and a descriptor
* Only made of punctuation (and not reserved like: ``+ - , ; " ' ` # .. ...``)
* If reused syntax, longest match found wins (eg `::` is matched before `:`)
* Each context defines their own but commonly used:
  * `/` the child relation space for parent-child relation
  * `.` the dot relation space for object-property relation
  * `:` the meta relation space for data-metadata relation

### Parent-Child relation
* 

Multiline per element
```
x
  /a
  /b
  /c
```

equivalent to
```
x
  /
    a
    b
    c
```

compact to inline
```
x /a, /b, /c
```

### Object-Property relation
*

Multiline
```
obj
  .a = 1
  .b = 2
  .c = 3
```

equivalent to
```
obj
  .
    a = 1
    b = 2
    c = 3
```

compact to inline
```
obj .a = 1, .b = 2, .c = 3
```

### Data-Metadata relation
*

Multiline
```
Age
  :type = Int
  :min  = 0
```

equivalent to
```
Age
  :
    type = Int
    min  = 0
```

compact to inline
```
Age :type = Int, :min = 0
```

### Browse
* Browse to a relation space and do work w/n that scope
* When data literal, will implictly create heirarchy
* When using existing variable, will reference

Browse to HTML element we're concerned with and work w/n that scope
```
html/body/header/
  .id = my-header

  logo
    .class = my-logo
```
Fully representing the `html/boy/header/` heirarchy would create syntactic noise without much value as we only really care to do work w/n the `header`. To avoid added noise, this syntax allows us to do this naturally. If we needed to add additional relations making the heirarchy explicit would be necessary.

Implicitly create `.style.css.width` as they're necessary steps for valid HTML
```
// As a literal (notice space)
div .style.css.width = 100%

// or, as assignment (no space between tag head and relation space)
div.style.css.width = 100%
```

Since `header` already exists we can reference it
```
id, class = html/body/header.id, html/body/header.class
```

which can be simplified further by using a selection syntax (explained in ***Parenthesis*** section)
```
id, class = html/body/header.(id, class)
```
the R-hand should be read as: within `html/body/header` relation space `.` get `id, class`

## Parenthesis
---
* Used for grouping to describe precedence (explained in ***Precedence***)
* Used for the start `(` and stop `)` of a expression, operator, relation space, function, etc.

Consider some data literals in their current inline form
```
x /a, /b, /c
obj .a = 0, .b = 1, .c = 2
Age :type = Int, :min = 0
```

can be compacted to
```
x /(a, b, c)
obj .(a = 0, b = 1, c = 2)
Age :(type = Int, min = 0)
```

## Identifier
---
TODO: Break into categories of what dev will use: variable name, type/trait/interface name, comptime name

Valid variable and function names
```
// Names
x
my-staff
my-really-long-descriptive-name

// Coordinates
x-0,3
x-1,2
x-(2,4)-y-(5,3)

// Computation
x+y
x+y-2z

// Approximate Pug for HTML generation
// - Notice
//   - Using '#' is fine here as long as it's not at start of name
//   - We have to use '..' to not conflict w/ '.' relation space or redefine '.'
p#my-id..my-class..my-other-class

// New value of x (as in mathematics)
x'  
x''
```
comptime names are just the same but prefixed w/ `#`

Valid type, trait, interface name
```
X
My-Type

Matrix-3x3

Point-[3,3,3]
```

Invalid universally b/c reserved
```
TODO
```

## Precedence
---
TODO
* General overview
* Using parenthesis
* How they interact w/ functions, relation spaces, etc.?

## TODO
---
* Codegen
* Operator/Function
* Comptime
  * `#` w/n expression or start of line will invoke at comptime
* data literal vs reference vs assign
  * gap between head and rest?
  * eg 
    ```
    x, y = 4, 5             // select x, y then assign 4, 5
    (x = 4, y = 5)          // 
    (x, y = 4, 5)           // 
    x:y = 4                 // select meta 'y' in 'x' then assign 4
    x :y = 4, :z = 5        // data literal
    x :(y = 4, z = 5)       // data literal
    x :(y, z) = 4, 5        // 
    x:(y, z) = 4, 5         // select meta `y`, `z` in `x` then assign to 4, 5
    x:(y = 4, z = 5)        // 
    x:(y = 4, z = 5) = a, b // 
    ```
* mode vs input vs body vs output
  * eg `Map(String, Int) vs Map String, Int vs Map String, Int x, y, z vs Map(String, Int) (x, y, z)`
* call
  ```
  sum 1, 2, 3   // call prefix 'sum' on sequence '1, 2, 3'
  x - y         // call infix '-' on 'x, y'
  -(x, y)       // 
  x + -(y)      // 
  x + -y        // 
  x + - 1, 2, 3 // 
  x -(1, 2, 3)  // 
  x -( 1, 2, 3  // 
  x -(( 1, 2, 3 // 
  ```
* difference of:
  ```
  f(x, y, z)

  f x, y, z

  f
    x
    y
    z

  f(
    x
    y
    z

  f
    (
      x
      y
      z
  ```
* Literals
  * String
  * Code
  * []
  * {}
  * ()
  * 
* Fibonacci
  ```
  fib = Fn (n:Int) -> Int
    if n <= 1
    then
      n
    else
      fib(n - 1) + fib(n - 2) 

  fib 30
  ```
* LISO operator aggregation
  * Syntax concat
    ```
    culprit <in> room <with> weapon =
       format[msg, culprit, room, weapon] $
          msg = "Mr. Boddy was killed by ~a in the ~a with the ~a"

    "Colonel Mustard" <in> "ball room" <with> "candlestick"
    ```

  * For
    ```
    @define-syntax for:
       @syntax-rules [list, begin]:

          for[{}, body, ...] =>
             {body, ...}

          for[{spec, rest, ...}, body, ...] =>
             for[spec, for[{rest, ...}, body, ...]]

          for[var = start <to> end <by> increment, body, ...] =>
             @vars loop[var = start]:
                @when var <= end:
                   body
                   ...
                   loop[var + increment]

          for[var = start <to> end, body, ...] =>
             for[var = start <to> end <by> 1, body, ...]
        
    @for x = 1 <to> 3
         y = 10 <to> 30 <by> 10:
       display[[x, y]]
    ```
