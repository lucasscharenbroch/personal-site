---
title: "APL Tutorial"
subtitle: "Making sense of (⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Y"
date: 2023-11-27T09:07:11-06:00
tags: ["PL", "Tutorial"]
---

I've been working on writing an APL Interpreter recently.
Understanding the language is a prerequisite to writing an interpreter, and I'm pretty sure I've got it figured out, but I figure it's still a beneficial exercise to transcribe that knowledge into an explanation.

[APL](https://en.wikipedia.org/wiki/APL_(programming_language)) was originally a mathematical notation rather than a computer programming language, so there are many different variations of APL. The tutorial focuses on [Dyalog](https://www.dyalog.com)'s implementation, as that is the implementation my interpreter is modeled after.

To add a little motivation, I'll make the goal of this tutorial to explain how the following code works.

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Y, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Y
{{< /highlight >}}

While this example isn't close to being representative of all of the functionality of Dyalog APL[^dyalog], I believe it touches on the most difficult parts of the core functionality of the language.

[^dyalog]: Dyalog is really more of an ecosystem than a language: my interpreter only focuses on the core functionality.

## Arrays

The only data type in APL is the (multidimensional) array.

Every array is defined by (1) its shape, and (2) its elements.

1. "Shape" is effectively equivalent to "dimensions". A 2x2 array (matrix) has shape [2, 2]. Each element (number) of the shape array is refered to as an "axis".
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
'a' 2 (3 4) ⍝ shape: [3]
{{< /highlight >}}

## Functions

Functions have three possible types: monadic (taking one argument), dyadic (taking two arguments), and ambivalent (both monadic and dyadic)[^mon-dy].
The majority of glyphs are ambivalent.
For the example, ***⍳*** ("iota") is the function "index generator" when applied monadically, and "index of" when applied dyadically.

[^mon-dy]: Every glyph or dfn can (syntactically) be applied either monadically or dyadically (Dyalog sometimes returns 'SYNTAX ERROR' when functions are applied with the wrong number of arguments (e.g. ⍲1), but I believe this is a misnomer, as the exception is thrown during execution, not during parsing[^domain-err]), but they may throw errors when they're executed with the 'wrong' number of arguments.

[^domain-err]: '⍲1⍲3' throws a 'DOMAIN ERROR', not a 'SYNTAX ERROR': unless evaluation is done during parsing, this implies that it is a valid grammar tree (assuming that input is converted into a tree before it is executed).

{{< highlight apl >}}
⍳9                ⍝ returns: 1 2 3 4 5 6 7 8 9 (ints from 1 to 9)
99 98 97 96 95⍳98 ⍝ returns: 2 (index of 98 in 99 98 97 96 95)
{{< /highlight >}}

Arrays are 1-indexed in this case[^io].

Note that monadic functions take arguments on their right.

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
{{< /highlight >}} [^conj]

[^conj]: Monadic + is actually [conjugate](https://help.dyalog.com/latest/Content/Language/Primitive%20Functions/Conjugate.htm), not identity, but they're the same for the purposes of this tutorial (conjugate involves imaginary numbers).

Note the high-minus-sign ('¯') used for numeric *literals* - this prevents overloading '-', which is a function.

The forth line in each of the above examples uses "rank polymorphism": even though the shape of the left and right arguments don't match, because one of the arguments (in this case, the left agument) has shape [1], it is implicitly reshaped to the shape of the right argument (which is [3] in this case, so the left argument effectively becomes 3 3 3).
Rank polymorphism applies to *some* functions - the function itself decides whether or not it supports rank polymorphism.

{{< highlight apl >}}
⍝ ⍴ (monadic: shape, dyadic: reshape)
⍴1
⍝ 1

⍴1 2 3
⍝ 3

2 2⍴1 2 3 4
⍝ 1 2
⍝ 3 4

3 3 3⍴(⍳100)
⍝  1  2  3
⍝  4  5  6
⍝  7  8  9
⍝
⍝ 10 11 12
⍝ 13 14 15
⍝ 16 17 18
⍝
⍝ 19 20 21
⍝ 22 23 24
⍝ 25 26 27

⍴(3 3 3⍴(⍳100))
⍝ 3 3 3
{{< /highlight >}}

## Operators

While functions can only take arrays as arguments, operators can take **arrays or functions** (they might accept one or both, depending on the operator).
Unlike functions, their [valance](https://aplwiki.com/wiki/Operator_valence) (the number of arguments they take) cannot be overloaded[^red-scn], and when used monadically, they take their arguments on the *left* (not the right).
The result of an operator is a function (called a "derived function").

[^red-scn]: Reduce and scan (/⌿\⍀) are exceptions to this: these glyphs are [both monadic operators and dyadic functions](https://aplwiki.com/wiki/Function-operator_overloading).

{{< highlight apl >}}
⍝ / ("reduce") is a monadic operator
+/                 ⍝ the value of this expression (the result of the application of / on +)
                   ⍝ is a monadic function (called the "derived function"), a right-fold
                   ⍝ with the given function (+) as the step-function
(+/) 1 2 3 ⍝ 6

⍝ ⍨ ("selfie") is a monadic operator
1 2 3⍨             ⍝ when given an array as the argument, the derived functions
                   ⍝ (which, for selfie, can be either monadic and dyadic)
                   ⍝ discard their arguments and return the original argument to selfie
(1 2 3⍨)4 5 6      ⍝ 1 2 3
4 5 6(1 2 3⍨)7 8 9 ⍝ 1 2 3
÷⍨                 ⍝ when given a dyadic function as the argument, the derived functions return:
                   ⍝ monadic: the function (selfie's arg) applied with the derived function's
                   ⍝          argument on both sides
(÷⍨)3              ⍝ 1 (the result of 3÷3)
                   ⍝ dyadic: the function (selfie's arg) applied with the derived function's
                   ⍝         arguments, swapped in order (left becomes right, right becomes left)
3(÷⍨)1             ⍝ 0.3333333333 (the result of 1÷3)


⍝ ∘ is a dyadic operator
-∘-                ⍝ when both arguments are functions (let's call them f and g, respectively)
                   ⍝ the derived functions are monadic: f(gY), dyadic: Xf(gY)
                   ⍝ where X and Y are the left and right arguments to the derived function
(-∘÷)2             ⍝ ¯0.5 (the result of -(÷2))
6(÷∘-)2            ⍝ ¯3 (the result of 6÷(-2))
                   ⍝ up to one of the arguments to ∘ can be an array (the other must be a function)
1∘1                ⍝ SYTNAX ERROR (when derived function is applied)
                   ⍝ when an argument is an array, the other must be a dyadic function
                   ⍝ the derived function is monadic: it is the dyadic function
                   ⍝ curried with the array on the respective side
÷∘2                ⍝ "divide by two"
(÷∘2)3             ⍝ 1.5
2∘÷                ⍝ "two divided by..."
(2∘÷)3             ⍝ 0.6666666667
{{< /highlight >}}

## Combining Operators, Functions, and Arrays

Functions are right-associative; operators are left-associative.
Operators have a higher precedence than functions.

## Variables

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

## Dfns

"Direct Functions" ("dfns") are effectively lambda functions. They can be defined inline.
{{< highlight apl >}}
f ← {2 × ⍵}             ⍝ ⍵ is the right argument
g ← {⍺ × ⍵}             ⍝ ⍺ is the left argument

f 3                     ⍝ 6
{2 × ⍵} 3               ⍝ 6
3 g 3                   ⍝ 9
3 {⍺ × ⍵} 3             ⍝ 9
3 {2 × ⍵} 3             ⍝ 6 (all dfns are ambivalent, so, unused arguments (⍺ in this case) are ignored)

fib ← {
    ⍵ ≤ 1 : 0           ⍝ ':' is used to return early - "if ⍵ ≤ 1, then return 0"
    ⍵ = 2 : 1
    (∇ ⍵ - 1) + ∇ ⍵ - 2 ⍝ '∇' refers to the current function ("fib" in this case)
}                       ⍝ (using 'fib' here would have the same effect)

fib 1                   ⍝ 0
fib 2                   ⍝ 1
fib 3                   ⍝ 1
fib 4                   ⍝ 2
fib 5                   ⍝ 3

⍝ redundant wrapper for ÷ (same behavior as div ← ÷)
div ← {
    ⍺ ← 1 ⍝ default left argument (used if no left argument is supplied)
    ⍺ ÷ ⍵
}

3 div 2 ⍝ 1.5
div 2   ⍝ 0.5
{{< /highlight >}}

This same notation can be used to define operators. ⍺, ⍵, and ∇ become the arguments/ recursive call to the *derived function*, and ⍺⍺, ⍵⍵, and ∇∇ refer to the arguments/ recursive call to the operator.

{{< highlight apl >}}
custom_fib ← {      ⍝ arguments are the starting values of the sequence
    ⍵ ≤ 1 : ⍺⍺
    ⍵ = 2 : ⍵⍵
    (∇ ⍵ - 1) + ∇ ⍵ - 2
}

(0 custom_fib 1) 1  ⍝ 0
(0 custom_fib 1) 2  ⍝ 1
(0 custom_fib 1) 3  ⍝ 1
(0 custom_fib 1) 4  ⍝ 2
(0 custom_fib 1) 5  ⍝ 3

(5 custom_fib 9) 1  ⍝ 5
(5 custom_fib 9) 2  ⍝ 9
(5 custom_fib 9) 3  ⍝ 14
(5 custom_fib 9) 4  ⍝ 23
(5 custom_fib 9) 5  ⍝ 37

custom_fib2 ← {     ⍝ left arg (⍺⍺) is initial array, right arg (⍵⍵) is combination function
    ⍵ ≤ ≢⍺⍺ : ⍺⍺[⍵] ⍝ ≢ is tally (first element of the shape of the array (length, in this case))
    (∇ ⍵ - 1) ⍵⍵ ∇ ⍵ - 2
}

(1 2 custom_fib2 ×) 1 ⍝ 1
(1 2 custom_fib2 ×) 2 ⍝ 2
(1 2 custom_fib2 ×) 3 ⍝ 2
(1 2 custom_fib2 ×) 4 ⍝ 4
(1 2 custom_fib2 ×) 5 ⍝ 8

⍝ pow (taken from https://dyalog.com/uploads/documents/Papers/dfns.pdf)
pow ← {                     ⍝ ⍺⍺ is the number of times to apply ⍵⍵ on ⍵
    ⍺⍺ = 0 : ⍵
    (⍺⍺ - 1) ∇∇ ⍵⍵ ⍵⍵ ⍵     ⍝ apply ⍵⍵ once, then recursively apply pow with (⍺⍺ - 1) and ⍵⍵
}

(0 pow (+∘1)) 5             ⍝ 5
(1 pow (+∘1)) 5             ⍝ 6
(2 pow (+∘1)) 5             ⍝ 7
(3 pow (+∘1)) 5             ⍝ 8
{{< /highlight >}}

## Tacit

When functions are placed directly next to each other without being applied to an array, they are combined into a single function through constructions called "forks" and "atops".

[Stefan Kruger's APL Tutorial](https://xpqz.github.io/learnapl/intro.html) has a great [summary of forks and atops](https://xpqz.github.io/learnapl/tacit.html#summary-forks-and-atops), which I've reproduced below.

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
f g => { f ⍺ g ⍵ }           ⍝ (dyadic) overloaded with
       { f g ⍵ }             ⍝ (monadic)

Forks:
X g h => { X g ⍺ h ⍵ }       ⍝ (dyadic) overloaded with
         { X g h ⍵ }         ⍝ (monadic)

f g h => { (⍺ f ⍵)g(⍺ h ⍵) } ⍝ (dyadic) overloaded with
         { (f ⍵)g(h ⍵) }     ⍝ (monadic)
```

Dfn syntax doesn't support ambivalence, but each pair of functions represents a single ambivalent function (a function that is overloaded with a monadic and dyadic implementation). The result of forks and atops are *functions*, and they need not be immediately applied.

### Trains

In general, forks and atops only encompass lists ("trains") of 2-3 functions/arrays.
When there is a longer list (4+ functions/arrays in a row), the functions/arrays are grouped into a tree (where each node is either a function [leaf], array [leaf], atop [internal node with 2 children], or fork [internal node with three children]) which can be evaluated to a function.

Below shows how these groupings are made.
In general, all non-leaf nodes are forks except the root node, which is an atop iff the length of the train is even.

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
Each of these trees represents a single function, which is obtained by recursively applying the above fork/atop equivalences.

```
    +-×÷+-×÷
┌─┴─┐                                                        ⍝ (1) (atop)
+ ┌─┼───┐                                                    ⍝ (2) (fork)
  - × ┌─┼───┐                                                ⍝ (3) (fork)
      ÷ + ┌─┼─┐                                              ⍝ (4) (fork)
          - × ÷

    (+-×÷+-×÷)Y <=> (+ (- × (÷ + (- × ÷))))Y                 ⍝ add parenthesis according to tree
                <=> + (- × (÷ + (- × ÷))) Y                  ⍝ apply (1)
                <=> + (-Y) × ((÷ + (- × ÷)) Y)               ⍝ apply (2)
                <=> + (-Y) × (÷Y) + ((- × ÷) Y)              ⍝ apply (3)
                <=> + (-Y) × (÷Y) + (-Y) × (÷Y)              ⍝ apply (4)

recall:
(f g)Y  <=> f g Y
(f g h)Y <=> (f Y) g (h Y)

    X(+-×÷+-×÷)Y <=> X(+ (- × (÷ + (- × ÷))))Y               ⍝ add parenthesis according to tree
                 <=> + (X(- × (÷ + (- × ÷)))Y)               ⍝ apply (1)
                 <=> + (X - Y) × (X(÷ + (- × ÷))Y)           ⍝ apply (2)
                 <=> + (X - Y) × (X ÷ Y) + ((X(- × ÷))Y)     ⍝ apply (3)
                 <=> + (X - Y) × (X ÷ Y) + (X - Y) × (X ÷ Y) ⍝ apply (4)

recall:
X(f g)Y <=> f X g Y
X(f g h)Y <=> (X f Y)g(X h Y)
```

While, at first glance, it may be seem hard to tell what a given train/tree does, the above examples show that the structure of trains is consistent enough to make them relatively easy to read: in general, every other function is applied on the argument(s), and the other functions are used to combine those results.

### Trains With Arrays


A train can include arrays, but only if it matches the following pattern.
```
[fn] ((arr|fn) fn)* fn
```
Any train that matches this pattern will result in a tree whose array-leaves are the leftmost arguments of forks (and therefore there's a valid reduction).
Expressions that don't match this pattern are not parsed as trains (this usually results in a syntax error).

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

    +1-×
┌─┴─┐
+ ┌─┼─┐
  1 - ×

    1+-2+ ⍝ parsed as: 1+(-(2+ expected_argument))
SYNTAX ERROR: Missing right argument

    1+-2+- ⍝ parsed as: 1+(-(2+ (-expected_argument)))
SYNTAX ERROR: Missing right argument

  1+-+2+-
┌─┼───┐
1 + ┌─┼───┐
    - + ┌─┼─┐
        2 + -
```

## Putting it All Together

Here's an explanation of the initial example of [Floyd-Warshall](https://en.wikipedia.org/wiki/Floyd-Warshall_algorithm).

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Y, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Y

⍝ this is a train (in parenthesis) being applied to the matrix Y
⍝ (⍤), (∘.), and (/) are operators (dyadic, monadic, and monadic, resp.)
⍝ (this may be easy to tell from the syntax highlighting)
⍝ but all other glyphs in the train are functions
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

⍝ use the same process to find the tree for function (3):
((⊢⌊(⌷⍤1)(∘.+)⌷)/)⍤, ⍝ function 3 is a derived function from the dyadic operator ⍤.
⍝    ⍤
⍝  ┌─┴─┐
⍝ ...  ,

⍝ find the tree for the left-hand-side of ⍤:
(⊢⌊(⌷⍤1)(∘.+)⌷)/     ⍝ derived function from the monadic operator /.
⍝    /
⍝  ┌─┘
⍝ ...

⍝ left-hand-side of /:
(⊢⌊(⌷⍤1)(∘.+)⌷)
(⊢ ⌊ (⌷⍤1) (∘.+) ⌷)  ⍝ spaces between the functions
⍝ this is a 5-train; 5 is odd, so the root is a fork, and the resulting tree is:
⍝ ┌─┼─────┐
⍝ ⊢ ⌊ ┌───┼─┐
⍝     ⍤   . ⌷
⍝    ┌┴┐ ┌┴┐
⍝    ⌷ 1 ∘ +
{{< /highlight >}}

The overall tree:

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

{{< highlight apl >}}
⍝ parens to show grouping of forks and atops
( ⊃ ((⍳⍤≢) (((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/)⍤,) ⊂) )Y

⍝ the operator ⍤, when applied to functions, is "atop", and has
⍝ identical behavior to the "atop" constructions used in trains
⍝ the original tree is equivalent to
⍝ ┌──┴──┐
⍝ ⊃ ┌───┼──┐
⍝  ┌┴┐ ┌┴┐ ⊂
⍝  ⍳ ≢ / ,
⍝    ┌─┘
⍝  ┌─┼─────┐
⍝  ⊢ ⌊ ┌───┼─┐
⍝      ⍤   . ⌷
⍝     ┌┴┐ ┌┴┐
⍝     ⌷ 1 ∘ +

⍝ reducing (f⍤g) => (fg) in the expression yields an equivalent result
( ⊃ ((⍳≢) (((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/),) ⊂) )Y

⍝ note that the remaining ⍤ has 1 as a a right argument;
⍝ 1 is not function, so ⍤ is not atop in this case

⍝ walking through the recursive evaluation of the derived function may
⍝ be a little excessive, but we can use the generalization of the behavior from the
⍝ above examples: apply the argument (Y) recursively to the right ascendants of atops
⍝ and the left and right descendants of forks:
⍝ ┌──┴───┐
⍝ ⊃ ┌────┼──┐         (*)
⍝  ┌┴┐  ┌┴┐ ⊂Y
⍝  ⍳ ≢Y / ,
⍝     ┌─┘
⍝   ┌─┼─────┐
⍝   ⊢ ⌊ ┌───┼─┐
⍝       ⍤   . ⌷
⍝      ┌┴┐ ┌┴┐
⍝      ⌷ 1 ∘ +
⊃ ((⍳≢Y) (((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/),) (⊂Y))
⍝         ^^^^^^^^^^^^^^^^^^^^^^^^^

⍝ note that the middle child of the highest fork (*) isn't applied on any arguments
⍝ (it's used as the dyadic function to combine the left and right children)
⍝ let's abstract this away for now; we'll refer to this function as f
f ← ((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/),
⊃ ((⍳≢Y) f (⊂Y))

⍝ ┌──┴──┐
⍝ ⊃ ┌───┼──┐
⍝  ┌┴┐  f  ⊂Y
⍝  ⍳ ≢Y

⍝ monadic ⊃: "first" - returns the first element of the given array, "unboxing" it if it is a "scalar-array"
⍝ monadic ⊂: "enclose" - "boxes" an array (turning it into a "scalar-array", so it can be an element of another array)
⍝ monadic ≢: "tally" - returns the number of "major cells" in an array - this is the first number of the shape
⍝ monadic ⍳: "index generator" - (when given a scalar); returns an array of numbers from 1 to the argument (inclusive)

⍝ it's relatively easy to see what this outer tree does: it returns the first
⍝ element of the array returned by f, when applied on (the numbers from 1 to ≢Y)
⍝ on the left, and (boxed Y) on the right

⍝ here's f:
⍝     ┌┴┐
⍝     / ,
⍝   ┌─┘
⍝ ┌─┼─────┐
⍝ ⊢ ⌊ ┌───┼─┐
⍝     ⍤   . ⌷
⍝    ┌┴┐ ┌┴┐
⍝    ⌷ 1 ∘ +

⍝ we know that f will be applied dyadically, so we can repeat the same process as above

f ← { ⍺(((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/),)⍵ }
⍝     ┌┴─┐
⍝     / ⍺,⍵
⍝   ┌─┘
⍝ ┌─┼─────┐
⍝ ⊢ ⌊ ┌───┼─┐
⍝     ⍤   . ⌷
⍝    ┌┴┐ ┌┴┐
⍝    ⌷ 1 ∘ +

f ← { ((⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))/) ⍺,⍵ }
⍝      ^^^^^^^^^^^^^^^^^^^^^

⍝ again, we have a relatively large function tree on the left that has no arguments.
⍝ let's abstract it below the / operator

g ← { ⍺(⊢ ⌊ ((⌷⍤1) (∘.+) ⌷))⍵ }
f ← { (g/) ⍺,⍵ }

⍝   ┌┴─┐
⍝   / ⍺,⍵
⍝ ┌─┘
⍝ g

⍝ dyadic ,: "catenate": joins two arrays along their "trailing axis" (the last number of their shape)

⍝ f takes its left and right arguments, combines them into an array,
⍝ and returns that array after reducing it with g
⍝ therefore the top-level code catenates (⍳≢Y) and (⊂Y) (appends a boxed Y to the end of
⍝ the array [1...(≢Y)]), reduces that with g, and returns the first element of the result

⍝ here's g:
⍝ ┌─┼─────┐
⍝ ⊢ ⌊ ┌───┼─┐
⍝     ⍤   . ⌷
⍝    ┌┴┐ ┌┴┐
⍝    ⌷ 1 ∘ +

⍝ again, apply recursive reductions

⍝ ┌───┼─────┐
⍝ ⍺⊢⍵ ⌊ ┌───┼──┐
⍝      ⍺⍤⍵  . ⍺⌷⍵
⍝      ┌┴┐ ┌┴┐
⍝      ⌷ 1 ∘ +

g ← { (⍺⊢⍵) ⌊ (⍺(⌷⍤1)⍵) (∘.+) (⍺⌷⍵) }

⍝ dyadic ⊢: "right": returns the right argument
⍝ dyadic ⌊: (element-wise) minimum (of the two arguments)
⍝ dyadic ⌷: "index": the ⍵'th element of ⍺ (when ⍵ is a scalar)

⍝ monadic ∘.: "outer product": the derived function returns a "multiplication-table" between the
⍝                              two array arguments (with the given function instead of multiplication)
{{< /highlight >}}

Contarary to how the tree looks, "∘." is effectively a single (monadic) operator.[^outer-product-op]
[^outer-product-op]: Syntactically, "∘" (which is an operator, *not* a function) is allowed as a left argument to ".". This is a special-case: all other operators and functions are made up of single glyphs. This oddity is also reflected in the function tree.

{{< highlight apl >}}
g ← { (⍺⊢⍵) ⌊ (⍺(⌷⍤1)⍵) (∘.+) (⍺⌷⍵) }
⍝ the above g is equivalent to (remove "⍺⊢")
g ← { ⍵ ⌊ (⍺(⌷⍤1)⍵) (∘.+) (⍺⌷⍵) }

⍝ ⍤ (when applied to a function (on the left) and an array
⍝ (on the right)) is "rank": it is a specialized for-each;
⍝ the left argument is the function to apply to each element (or pair of
⍝ elements, if applied dyadically), and the right argument is *how many trailing axes
⍝ the for-each ignores* during iteration. The derived function is rank-polymorphic,
⍝ so the shapes of the arguments (after ignoring the given number of axes)
⍝ doesn't need to match if one of the two is a scalar

⍝ recall that g is being used to reduce ((⍳≢Y),(⊂Y))
⍝ since / is a right-fold, g is first called with (⍺ ← ≢Y) and (⍵ ← Y)
⍝ thus ⍺ has shape [1], and ⍵ has the same shape as Y (assume Y is 2d)

⍝ if ⍺ is a scalar and ⍵ is a rank-2 (2d) matrix, (⍺(⌷⍤1)⍵)
⍝ applies ⌷ on ⍺ and each *row* of ⍵ (the column axis is ignored)
⍝ (the trailing axis of ⍺ is also ignored, but it remains a scalar)
⍝ this expression yields the ⍺'th column of ⍵

⍝ thus (⍺(⌷⍤1)⍵) (∘.+) (⍺⌷⍵) is an addition-table between the ⍺'th column
⍝ of ⍵ and the ⍺'th row of ⍵, so result[i, j] = ⍵[i, ⍺] + ⍵[⍺, j]
⍝ and, in general (in pseudocode/math notation),
⍝ g maps ⍵[i, j] to min(⍵[i, j], ⍵[i, ⍺] + ⍵[⍺, j])
⍝ this exactly matches the bellman-equation for Floyd-Warshall (with ⍵ instead of dp)

⍝ it follows that ⍵[i, j] = dp[i, j, k] after the k'th application of g
⍝ when the reduction (/g) is complete, then the boxed shortest path-length
⍝ matrix remains; it is unboxed and returned by the left argument of the highest atop (⊃)
{{< /highlight >}}

The above explanation goes pretty far into the weeds, but the bottom line is that g takes in a 2d slice of the 3d dp matrix, and it returns the next 2d slice.
Once this is done N times, once with ⍺ assigned to each value in [1...N], then the dp matrix is the shortest-path-distance matrix.

The following may help clarify exactly how the reduction (right-fold) does this.

```
      Y ← 3 3⍴0 2 99 99 0 3 4 99 0
      Y
 0  2 99
99  0  3
 4 99  0
      g ← { (⍺⊢⍵) ⌊ (⍺(⌷⍤1)⍵) (∘.+) (⍺⌷⍵) }
      ⍳≢Y
1 2 3
      ⊂Y
┌────────┐
│ 0  2 99│
│99  0  3│
│ 4 99  0│
└────────┘
      (⍳≢Y),(⊂Y)
┌─┬─┬─┬────────┐
│1│2│3│ 0  2 99│
│ │ │ │99  0  3│
│ │ │ │ 4 99  0│
└─┴─┴─┴────────┘
      3⌷Y
4 99 0
      3(⌷⍤1)Y
99 3 0
      (3(⌷⍤1)Y) (∘.+) (3⌷Y)
103 198 99
  7 102  3
  4  99  0
      3g(Y)
0  2 99
7  0  3
4 99  0
      2g(3g(Y))
0  2 5
7  0 3
4 99 0
      1g(2g(3g(Y)))     ⍝ (A)
0 2 5
7 0 3
4 6 0
      (g/)((⍳≢Y),(⊂Y))
┌─────┐
│0 2 5│
│7 0 3│
│4 6 0│
└─────┘
      ⊃(g/)((⍳≢Y),(⊂Y)) ⍝ (B)
0 2 5
7 0 3
4 6 0
```

(A) and (B) are identical in applications of g - the applications of g in (A) match the applications of g through the reduction (/) in (B).
The extra enclosure/disclosure (boxing) is only necessary to place Y as an element at the end of the array.

## Everything Else

There are a few syntactical topics I glossed over[^topics] (or left out entirely), but for the most part, all that's left[^all-thats-left] is learning what all the glyphs do.
Dyalog's documentation ([pdf](https://docs.dyalog.com/latest/Dyalog%20APL%20Language%20Reference%20Guide.pdf), [web](https://help.dyalog.com)) is a great resource for this.

[^topics]: Including but not limited to: [array subscripting](https://course.dyalog.com/selecting-from-arrays/), [boxing](https://course.dyalog.com/multidimensional-and-nested-arrays/), [modified assignment](https://help.dyalog.com/latest/Content/Language/Primitive%20Operators/Assignment%20Modified.htm), [selective assignment](https://help.dyalog.com/18.0/Content/Language/Primitive%20Functions/Assignment%20Selective.htm), [array shape and rank](https://course.dyalog.com/cells-and-axes/), [function valence](https://aplwiki.com/wiki/Function), and (of course) the vast majority of Dyalog's libraries and workspace functionality.

For a much more complete overview, I'd recommend [Stefan Kruger's APL Tutorial](https://xpqz.github.io/learnapl/intro.html), or Dyalog's own [Mastering Dyalog APL](https://mastering.dyalog.com/README.html).

[^all-thats-left]: For the "core-functionality" (according to me) of Dyalog APL.
