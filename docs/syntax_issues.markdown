
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
