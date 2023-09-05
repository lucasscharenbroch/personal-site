---
title: "Implicit Recursion"
subtitle: "Observing built-in recursive structures in Haskell and C"
date: 2023-09-05T14:21:16-05:00
---

I recently started learning Haskell, and I came across a structure in the language that really made me think.
Consider a function called "add" that takes as input two integers and returns an integer.
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

"add 1" is executed ("add" is called *with a single argument*), and returns a nameless (Int -> Int) function, which is then called with the single Int argument 2, which yields the result of 3.

This is really cool, because, besides writing the signature, the programmer doesn't need to worry about this inherently recursive nature of functions unless they want to: the implementation of add is completely intuitive:

{{< highlight Haskell >}}
add a b = a + b
{{< /highlight >}}

and, as written above, add can be called as if it received two arguments, even though it actually only receives one, because the syntactical rules of Haskell are laid out in such a way that happens to allow this [^foot].

[^foot]: I'm still very unfamiliar with Haskell, and I'd like to know more about how it works under the hood, but it appears that this functionality comes naturally with consistency: if *all functions and operators* receive a single argument and have a single return value, then any function written with built-in operators will also implicitly work in a similar wary.
For example, consider the above "add" function, which is defined as "add a b = a + b"; we can instead re-write this with the prefix operator (+); I assume that Haskell does this under the hood: "add a b = (+) a b"; from here it's clear why "add" would work the same way that the "+" operator would.

This quality is strikingly similar to a very common syntactical mechanism in a much-less-esoteric language, C, and many of its descendants.
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

This sounds awfully similar to Haskell.
