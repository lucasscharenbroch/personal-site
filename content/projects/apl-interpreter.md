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
