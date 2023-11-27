---
title: "APL Tutorial"
subtitle: "Making sense of (⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Nm"
date: 2023-11-26T09:07:11-06:00
draft: true
---

I've been working on writing an APL Interpreter recently.
Understanding the language is a prerequisite to writing an interpreter, and I'm pretty sure I've got it figured out, but I figure it's still a beneficial exercise to transcribe that knowledge into an explanation.

APL was originally a mathematical notation rather than a computer programming language, so there are many different variations of APL. The tutorial focuses on [Dyalog](https://www.dyalog.com)'s implementation, as that is the implementation my interpreter is modeled after.

To add a little motivation, I'll make the goal of this tutorial to explain how the following code works.

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Nm, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Nm
{{< /highlight >}}

While this example isn't close to being representative of all of the functionality of Dyalog APL[^dyalog], I believe it touches on the most difficult parts of the core functionality of the language.

[^dyalog]: Dyalog is really more of an ecosystem than a language: my interpreter only focuses on the core functionality.

## Arrays

The only data type in APL is the (multidimensional) array.

Every array is defined by (1) its shape, and (2) its elements.

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
'a' 2 (3 4) ⍝ shape: [3]
{{< /highlight >}}

## Functions

Functions have two possible types: monadic (taking one argument), or dyadic (taking two arguments)[^mon-dy].
Almost all glyphs that represent functions are overloaded in this sense.
For the example, ***⍳*** ("iota") is the function "index generator" when applied monadically, and "index of" when applied dyadically.

[^mon-dy]: Every glyph or dfn can (syntactically) be applied either monadically or dyadically (Dyalog sometimes returns 'SYNTAX ERROR' when functions are applied with the wrong number of arguments (e.g. ⍲1), but I believe this is a misnomer, as the exception is thrown during execution, not during parsing[^domain-err]), but they may throw errors when they're executed with the 'wrong' number of arguments. This better explains how dfns and operators really work, but I've chosen to leave it out for the sake of conciseness.

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
    ⍵ ≤ ≢⍺⍺ : ⍺⍺[⍵] ⍝ ≢ is tally (length of array)
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
f g => { f ⍺ g ⍵ }           ⍝ (dyadic) overloaded with
       { f g ⍵ }             ⍝ (monadic)

Forks:
X g h => { X g ⍺ h ⍵ }       ⍝ (dyadic) overloaded with
         { X g h ⍵ }         ⍝ (monadic)

f g h => { (⍺ f ⍵)g(⍺ h ⍵) } ⍝ (dyadic) overloaded with
         { (f ⍵)g(h ⍵) }     ⍝ (monadic)
```

Dfn syntax doesn't support overloading in this generalized way, but each pair of functions represents a single function that is overloaded with a monadic and dyadic implementation. The result of forks and atops are *functions*, and they need not be immediately applied.

### Trains

In general, forks and atops only encompass lists ("trains") of 2-3 functions/arrays.
When there is a longer list (4+ functions/arrays in a row), the functions/arrays are grouped into a tree (where each node is either a function [leaf], array [leaf][^arr-in-train], atop [internal node with 2 children], or fork [internal node with three children]) which can be evaluated to a function.

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

### Putting it All Together

Here's an explanation of the initial example of Floyd-Warshall.

{{< highlight apl >}}
⍝ Shortest path length matrix from weighted adjacency matrix Nm, using Floyd-Warshall
⍝ (taken from aplcart.info)
(⊃⍳⍤≢(⊢⌊⌷⍤1∘.+⌷)/⍤,⊂)Nm

⍝ this is a train (in parenthesis) being applied to the matrix Nm
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
(⊢⌊(⌷⍤1)(∘.+)⌷)/ ⍝ derived function from the monadic operator /.
⍝    /
⍝  ┌─┘
⍝ ...

⍝ left-hand-side of /:
(⊢⌊(⌷⍤1)(∘.+)⌷)
(⊢ ⌊ (⌷⍤1) (∘.+) ⌷) ⍝ spaces between the function
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

