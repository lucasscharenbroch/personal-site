---
title: "APL Interpreter"
subtitle: "An implementation of Iverson's programming language (Haskell)"
date: 2023-11-22T09:59:40-05:00
draft: true
---

## Why APL?

[APL](https://en.wikipedia.org/wiki/APL_(programming_language)) is an *array* programming language[^name].
Its **only data type** is the (multidimensional) *array*.
This fact alone makes APL infesible for many typical applications of programming.
However, its lack of the constructions of conventional languages allows for a syntax that is incredibly compact and expressive, which forces the programmer to approach problems at a higher level.

[^name]: APL actually stands for "A Programming Language" (not "array programming language").

I was first drawn to APL by the nature of its syntax: with the exception of user-defined variables, all built-in functions and operators are single[^single] unicode symbols.
As a result, code looks like this.

[^single]: Outer-product (∘.) (two symbols) is an exception

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Nm, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Nm
{{< /highlight >}}

As you might expect, learning to write programs like this[^trad] requires a totally different mind-set.
Array programming is similar to functional programming -- the primary way to control execution involves composition of functions -- but this is true to a greater degree for array programming, because chaining functions is so primitive and natural, and the ways in which function calls can be combined are much more numerous and streamlined.

[^trad]: APL has a "traditional function" syntax that is more C-like, but this is not implemented in my interpreter.

I'd like to think that learning to program this way provides greater insights for programming in general.

## Why Haskell?

I've been learning Haskell recently, and I made this project as an excuse to write a reasonably-sized program with it.

There are some conveniences to using Haskell, especially when it comes to working with compositions of functions (during execution, composition of functions in APL can be implemented by passing around composed functions in Haskell).

The cons of using Haskell include speed, clumsiness of storing arrays, and inconvenience of working with mutable global variables in purely-functional code.

## Brief Explanation of APL Syntax

I don't intend to include a full APL tutorial in this post, but I don't want to get into the weeds of implementation without some syntactical explanation.

### Arrays

Every array in APL is defined by (1) its shape, and (2) its elements.

1. "Shape" is effectively equivalent to "dimensions". A 2x2 array (matrix) has shape [2, 2].
2. Each element of an array is either a number, character, or array[^rank-ne-depth].

[^rank-ne-depth]: Though nesting arrays is permitted (and often necessary), adding depth is **not** equivalent to changing the shape. (1 2) (3 4) ≢ 2 2⍴⍳4.

Array literals are always scalars (shape [1]) or vectors (shape [n]).

{{< highlight apl >}}
1           ⍝ shape: [1]
1 2 3 4 5 6 ⍝ shape: [6]
1 (2 3 4) 5 ⍝ shape: [3]
(1 2) (3 4) ⍝ shape: [2]
'a' 'b' 'c' ⍝ shape: [3]
'abc'       ⍝ shape: [3]
{{< /highlight >}}

### Functions

Functions have two possible types: monadic (taking one argument), or dyadic (taking two arguments)[^mon-dy].
Almost all glyphs that represent functions are overloaded in this sense.
For the example, ***⍳*** ("iota") is the function "index generator" when applied monadically, and "index of" when applied dyadically.

[^mon-dy]: Every glyph or dfn can (syntactically) be applied either monadically or dyadically (Dyalog sometimes returns 'SYNTAX ERROR' when functions are applied with the wrong number of arguments (e.g. ⍲1), but I believe this is a misnomer, as the exception is thrown during execution, not during parsing[^domain-err]), but they may throw errors when they're executed with the 'wrong' number of arguments. This better explains how dfns and operators really work, but I've chosen to leave it out for the sake of conciseness.

[^domain-err]: '⍲1⍲3' throws a 'DOMAIN ERROR', not a 'SYNTAX ERROR': unless evaluation is done during parsing, this implies that it is a valid grammar tree (assuming that input is converted into a tree before it is executed).

{{< highlight apl >}}
⍳9 ⍝ returns: 1 2 3 4 5 6 7 8 9 (ints from 1 to 9)
5 3 2 4 1⍳3 ⍝ returns: 2 (index of 3 in 5 3 2 4 1)
{{< /highlight >}}

Arrays are 1-indexed in this case[^io].

Note that monadic functions take their arguments on their right.

[^io]: Dyalog APL has a notion of "index origin" (⎕IO), so the way arrays are indexed can be changed dyamically. For the remainder of this post, assume ⎕IO←1 (arrays are 1-indexed).

{{< highlight apl >}}
⍝ more functions

⍝ + (monadic: identity, dyadic: plus)
+3          ⍝ 3
+3 4 5      ⍝ 3 4 5
3+4         ⍝ 7
3+4 5 6     ⍝ 7 8 9
3 4 5+6 7 8 ⍝ 9 11 13

⍝ - (monadic: negate, dyadic: minus)
-3          ⍝ ¯3
-3 4 5      ⍝ ¯3 ¯4 ¯5
3-4         ⍝ ¯1
3-4 5 6     ⍝ ¯1 ¯2 ¯3
3 4 5-6 7 8 ⍝ ¯3 ¯3 ¯3

⍝ × (monadic: direction, dyadic: multiply)
×3          ⍝ 1
×3 ¯4 ¯5    ⍝ 1 ¯1 ¯1
3×4         ⍝ 12
3×4 5 6     ⍝ 12 15 18
3 4 5×6 7 8 ⍝ 18 28 40

⍝ ÷ (monadic: inverse, dyadic: divide)
÷3          ⍝ 0.3333333333
÷3 4 5      ⍝ 0.3333333333 0.25 0.2
3÷4         ⍝ 0.75
3÷4 5 6     ⍝ 0.75 0.6 0.5
3 4 5÷6 7 8 ⍝ 0.5 0.5714285714 0.625
{{< /highlight >}}

The forth line in each of the above examples uses "rank polymorphism": even though the shape of the left and right arguments don't match, because one of the arguments (in this case, the left agument) has shape [1], it is implicitly reshaped to the shape of the right argument (which is [3] in this case, so the left argument effectively becomes 3 3 3).

### Operators

While functions can only take arrays as arguments, operators can take **arrays or functions** (they might accept one or both, depending on the operator).
Unlike functions, their [valance](https://aplwiki.com/wiki/Operator_valence) (the number of arguments they take) cannot be overloaded[^red-scn], and when used monadically, they take their arguments on the *left* (not the right).
The result of an operator is a function (called a "derived function").

[^red-scn]: Reduce and scan (/⌿\⍀) are exceptions to this: these glyphs are both monadic operators and dyadic functions.

{{< highlight apl >}}
⍝ / ("reduce") is a monadic operator
+/         ⍝ the value of this expression (the result of the application of / on +)
           ⍝ is a monadic function (called the "derived function"), a right-fold
           ⍝ with the given function (+) as the step-function
(+/) 1 2 3 ⍝ 6

⍝ ⍨ ("selfie") is a monadic operator
1 2 3⍨              ⍝ when given an array as the argument, the derived functions
                    ⍝ (which, for selfie, can be either monadic and dyadic)
                    ⍝ discard their arguments and return the original argument to selfie
(1 2 3⍨)4 5 6       ⍝ 1 2 3
4 5 6(1 2 3⍨) 7 8 9 ⍝ 1 2 3
÷⍨                  ⍝ when given a dyadic function as the argument, the derived functions return:
                    ⍝ monadic: the function (selfie's arg) applied with the derived function's
                    ⍝          argument on both sides
(÷⍨)3               ⍝ 1 (the result of 3÷3)
                    ⍝ dyadic: the function (selfie's arg) applied with the derived function's
                    ⍝         arguments, swapped in order (left becomes right, right becomes left)
3(÷⍨)1              ⍝ 0.3333333333 (the result of 1÷3)


⍝ ∘ is a dyadic operator
-∘-     ⍝ when both arguments are functions (let's call them f and g, respectively)
        ⍝ the derived functions are monadic: f(gY), dyadic: Xf(gY)
        ⍝ where X and Y are the left and right arguments to the derived function
(-∘÷)2  ⍝ ¯0.5 (the result of -(÷2))
6(÷∘-)2 ⍝ ¯3 (the result of 6÷(-2))
        ⍝ up to one of the arguments to ∘ can be an array (the other must be a function)
1∘1     ⍝ SYTNAX ERROR (when derived function is applied)
        ⍝ when an argument is an array, the other must be a dyadic function
        ⍝ the derived function is monadic: it is the dyadic function
        ⍝ curried with the array on the respective side
÷∘2     ⍝ "divide by two"
(÷∘2)3  ⍝ 1.5
2∘÷     ⍝ "two divided by..."
(2∘÷)3  ⍝ 0.6666666667
{{< /highlight >}}

### Combining Operators, Functions, and Arrays

Functions are right-associative; operators are left-associative.
Operators have a higher precedence than functions.

### Variables

Variables can be arrays, functions, or operators.

{{< highlight apl >}}
plus ← +
reduce ← /
iota ← ⍳
six ← 6
iota_six ← iota six

plus reduce iota six ⍝ 21
plus reduce iota_six ⍝ 21
{{< /highlight >}}

### Dfns

"Direct Functions" ("dfns") are effectively lambda functions. They can be defined inline.
{{< highlight apl >}}
f ← {2 × ⍵} ⍝ ⍵ is the right argument
g ← {⍺ × ⍵} ⍝ ⍺ is the left argument

f 3         ⍝ 6
{2 × ⍵} 3   ⍝ 6
3 g 3       ⍝ 9
3 {⍺ × ⍵} 3 ⍝ 9
{{< /highlight >}}

### Tacit

When functions are placed directly next to each other without being applied to an array, they are combined into a single function through constructions called "forks" and "atops".

[Stefan Kruger's APL tutorial](https://xpqz.github.io/learnapl/intro.html) has a great [summary of forks and atops](https://xpqz.github.io/learnapl/tacit.html#summary-forks-and-atops), which I've reproduced below.

```
Assume X, Y, and Z are arrays, and f, g, and h are functions.

Atops:
X(f g)Y <=> f X g Y
(f g)Y  <=> f g Y

Forks:
X(Z g h)Y <=> Z g X h Y
X(f g h)Y <=> (X f Y)g(X h Y)
(X g h)Y <=> X g h Y
(f g h)Y <=> (f Y) g (h Y)
```

The outer arguments (the X and Y outside of the parenthesis) show how the resulting function is formed (the arguments themselves aren't necessary for the construction).
The below is another way of writing them that hopefully makes this more clear.

```
Atops:
f g => { f ⍺ g ⍵ }           ⍝ when applied dyadically
    => { f g ⍵ }             ⍝ when applied monadically

Forks:
X g h => { X g ⍺ h ⍵ }       ⍝ when applied dyadically
      => { X g h ⍵ }         ⍝ when applied monadically

f g h => { (⍺ f ⍵)g(⍺ h ⍵) } ⍝ when applied dyadically
      => { (f ⍵)g(h ⍵) }     ⍝ when applied monadically
```

In general, forks and atops only encompass lists ("trains") of 2-3 functions (with an optional array replacing the first function).
When there is a longer list, the functions are grouped into a tree (where each node is either a function [leaf], atop [internal node with 2 children], or fork [internal node with three children]).

Below shows how these groupings are made.
In general, all non-leaf nodes are forks except the root node, which is an atop iff the number of functions in the list is even.

```
    +-    ⍝ two functions (atop)
┌┴┐
+ -

    +-×   ⍝ three functions (fork)
┌─┼─┐
+ - ×

    +-×÷  ⍝ four functions
┌─┴─┐
+ ┌─┼─┐
  - × ÷

    +-×÷+ ⍝ ...
┌─┼───┐
+ - ┌─┼─┐
    × ÷ +

    +-×÷+-
┌─┴─┐
+ ┌─┼───┐
  - × ┌─┼─┐
      ÷ + -

    +-×÷+-×
┌─┼───┐
+ - ┌─┼───┐
    × ÷ ┌─┼─┐
        + - ×

    +-×÷+-×÷
┌─┴─┐
+ ┌─┼───┐
  - × ┌─┼───┐
      ÷ + ┌─┼─┐
          - × ÷
```

A similar pattern occurs when the train starts with an arrays, except for even-numbered trains with leading arrays, which are not valid trains.

```
    1+-    ⍝ fork
┌─┼─┐
1 + -
    1+-×   ⍝ parsed as: 1+(-× expected_argument)
SYNTAX ERROR: Missing right argument
    1+-×÷
┌─┼───┐
1 + ┌─┼─┐
    - × ÷
    1+-×÷+ ⍝ parsed as: 1+(-×÷+ expected_argument)
SYNTAX ERROR: Missing right argument
    1+-×÷+-
┌─┼───┐
1 + ┌─┼───┐
    - × ┌─┼─┐
        ÷ + -
```

Each of these trees represents a single function, which is obtained by recursively applying the above fork/atop equivalences.

### Putting it All Together

Here's an explanation of the initial example of Floyd-Warshall.

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Nm, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Nm

⍝ this is a train (in parenthesis) being applied to the matrix Nm
⍝ (⍤), (/), and (∘.) are operators, but all other glyphs in the train are functions
⍝ add parenthesis to show the evaluation of the operators:
(⊃(⍳⍤≢)(((⊢⌊(⌷⍤1)(∘.+)⌷)/)⍤,)⊂)
⍝ add spaces between functions:
( ⊃ (⍳⍤≢) (((⊢⌊(⌷⍤1)(∘.+)⌷)/)⍤,) ⊂)
⍝ ^ ^^^^^ ^^^^^^^^^^^^^^^^^^^^^  ^
⍝ 1   2             3            4
⍝ we have four functions, so this is a 4-train
⍝ 4 is even, so the root is an atop
⍝ so the top of the tree looks like this
⍝ ┌──┴──┐
⍝ ⊃ ┌───┼─┐
⍝   ⍤   ⍤ ⊂
⍝  ┌┴┐  |
⍝  ⍳ ≢ ...
{{< /highlight >}}

Doing this recursively yields the final tree:

```
┌──┴──┐
⊃ ┌───┼─┐
  ⍤   ⍤ ⊂
 ┌┴┐ ┌┴┐
 ⍳ ≢ / ,
   ┌─┘
 ┌─┼─────┐
 ⊢ ⌊ ┌───┼─┐
     ⍤   . ⌷
    ┌┴┐ ┌┴┐
    ⌷ 1 ∘ +
```










