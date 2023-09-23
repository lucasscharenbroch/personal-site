---
title: "Implicit Recursion"
subtitle: "Observing built-in recursive structures in Haskell and C"
date: 2023-09-05T14:21:16-05:00
---

I recently started learning Haskell, and I came across a structure in the language that really made me think: [Currying](https://en.wikipedia.org/wiki/Currying).
Consider a function called "add" that takes two integers and returns an integer.
Here's the function signature in C:

{{< highlight c >}}
int add(int a, int b);
{{< /highlight >}}

And in Haskell:

{{< highlight Haskell >}}
add :: Int -> Int -> Int
{{< /highlight >}}

If you're not familiar with Haskell, the above signature probably doesn't seem very intuitive.
The "->" operator in this context is right-associative; we can add parenthesis to more clearly show what's happening.

{{< highlight Haskell >}}
add :: Int -> (Int -> Int)
{{< /highlight >}}

In plain English, you might describe it like this: "add is a function that receives an Int and returns a function which receives and returns an Int".
So really, under the hood, the "add" function doesn't accept two arguments; it rather accepts a single argument and returns another function which accepts the second and returns the final result. So when we call the function:

{{< highlight Haskell >}}
add 1 2
{{< /highlight >}}

"add 1" is executed ("add" is called *with a single argument*), and returns a nameless (Int -> Int) function[^fn], which is then called with the single Int argument 2, which yields the result of 3.

[^fn]: A function that returns "1 + x" for input x.

This is really cool, because, besides writing the signature, the programmer doesn't need to worry about this inherently recursive nature of functions unless they want to: the implementation of add is completely intuitive:

{{< highlight Haskell >}}
add a b = a + b
{{< /highlight >}}

and, as written above, add can be called as if it received two arguments, even though it actually only receives one, because the syntactical rules of Haskell are laid out in such a way that happens to allow this [^foot].

[^foot]: I'm still very unfamiliar with Haskell, and I'd like to know more about how it works under the hood, but it appears that this functionality comes naturally with consistency: if *all functions and operators* receive a single argument and have a single return value, then any function written with built-in operators will also implicitly work in a similar wary.
For example, consider the above "add" function, which is defined as "add a b = a + b"; we can instead re-write this with the prefix operator (+); I assume that Haskell does this under the hood: "add a b = (+) a b"; from here it's clear why "add" would work the same way that the "+" operator would.

But how is this useful? Functions are ubiquitous in Haskell: they are passed and returned very frequently; such functions are often simple variations or partial applications of built-in functions, so it's natural to simply pass/return a partially applied function instead of making a unique function (or lambda) that encapsulates that behavior. The following are examples[^limited].

[^limited]: Again, my knowledge of Haskell is still very limited, so consider this a high-level subset of use-cases.

{{< highlight Haskell >}}
-- add 5 to each element of `list'
map ((+) 5) list

-- function that adds 5 to each element of `list'
addFive :: [Int] -> [Int]
addFive = map ((+) 5)
-- map is of type `(a -> b) -> [a] -> [b]', where a and b are generic types;
-- passing `((+) 5)' (of type `Int -> Int') to map yields a function of type
-- `[Int] -> [Int]', which is the desired return-type
-- functions can also directly take arguments (see below), but this
-- example shows that returning a function that takes those same arguments
-- has the same effect
alternateAddFive :: [Int] -> [Int]
alternateAddFive l = map ((+) 5) l

-- removes elements from `names' that contain the character 'z'
filter (not . elem 'z') names
-- the '.' operator is "pipe"; it returns the left-hand-side applied on (called with)
-- the result of the function on the right-hand-side

-- sum the odd-indexed elements of each row of 2d `list'
map (sum . map snd . filter (odd . fst) . zip [0..]) list
{{< /highlight >}}


### Sections

While Currying adds lots of flexibility in *when* the arguments are applied, it is inflexible when it comes to the *order in which they are applied* (the first argument must be passed first).
Haskell has a feature called "sections" which allow binary functions to apply their arguments in an arbitrary order.

{{< highlight Haskell >}}
-- divide each element of `list' by 2
map (/2) list

-- convert each element of `list' to its reciprocal
map (1/) list
-- (this is equivalent to)
map ((/) 1) list

-- remove vowels from `chars'
filter (not . (`elem`"aeiou")) chars
-- the backticks convert the function `elem' into an infix operator
{{< /highlight >}}

## C <=> Haskell

This probably sounds like a long-shot, but the implicit nature of Currying struck me as quite similar to a very common syntactical mechanism in a much-less-esoteric language, C, and many of its descendants.
This is probably really obvious to some, but I didn't realize it until a few months ago, and I have yet to come across a teaching resource that points this out.
If you don't already know what it is, I'd encourage you to try to think of it before you scroll past this hilarious Haskell meme.

{{% center-text %}}
<img src="/images/haskell-meme.webp" alt="Picture of dog, with caption 'head' pointing to the head of the dog, and with caption 'tail' spanning the remainder of the dog's body"/>
{{% /center-text %}}

There is no special notion of "else if" in C: "else if" is literally a termination of an else-clause followed by an immediate beginning of another if-statement.
The following code-blocks are *syntactically identical* except for indentation and optional braces.

{{< highlight c >}}
if(a) {
    /* block a */
} else if(b) {
    /* block b */
} else if(c) {
    /* block c */
} else {
    /* block d */
}
{{< /highlight >}}

{{< highlight c >}}
if(a) {
    /* block a */
} else {
    if(b) {
        /* block b */
    } else {
        if(c) {
            /* block c */
        } else {
            /* block d */
        }
    }
}
{{< /highlight >}}

Another way of saying it: an if-else statement accepts exactly two clauses, however, if the second clause is another if-else statement, then the illusion (and effect) of a multi-clause if-statement can be made.

This sounds awfully similar to Currying.

{{< highlight Haskell >}}
f a b c d
{{< /highlight >}}

{{< highlight Haskell >}}
(((f a) b) c) d
{{< /highlight >}}
