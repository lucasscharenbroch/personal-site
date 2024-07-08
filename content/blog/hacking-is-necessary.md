---
title: "Hacking Is Necessary"
subtitle: "The Sobering Futility of Idealistic Programming"
date: 2024-06-26T10:06:46-05:00
notability: 6
tags: ["Workflow"]
---

Note: this isn't a security post: "hacking" here means "unclean coding", i.e. "I quickly hacked together a shell script as a temporary fix to the issue". Sorry, cybersecurity enthusiasts.

---

## Obsession Over Details

The better we get at coding, the more we become aware of the fine-grained structural details of our programs.
As programmers (whose nature it is to put [everything in its right place](https://en.wikipedia.org/wiki/Everything_in_Its_Right_Place)), we are inclined to focus quite heavily on these details, often to the point of obsession[^obsession].

[^obsession]: Obsession and polarizing opinionation.
This blog is guilty of this: I've made many rash statements about the merits of functional idioms (and the detriment of some of their alternatives).

Obsessing over these details can be quite fruitful: restructuring code almost always leads to some sort of tangible result[^utility] (even if it is partially subjective or imagined).
This is what makes it so alluring (and addicting).

[^utility]: The utility of which is highly variable.

## Impossible Ideals

Our ultimate goal in restructuring is usually to obtain (or become closer to) some sort of ideal, e.g.

- Clarity
- Extensibility
- Safety

The (obvious) problem is that it's impossible to obtain these ideals: you must approach them asymptotically.
Thus, the closer you get to an ideal, the more difficult it is to make progress towards it.

This becomes paramount during development, where you must constantly make trade-offs.
You must decide the point at which you stop pursuing the ideal (and accept imperfection).

## Hacking

**Hacking** is when you sacrifice an ideal for timeliness/convenience.

Since ideals are impossible, technically all development is hacking.
We don't decide **whether or not** to hack; instead we decide **to what degree** we hack.
In this way, "hackiness" is a spectrum, and it's the inverse of idealism.

## Types and "Safety"

A quintessential example of the benefits and drawbacks of "hackiness" is the spectrum of type strength[^strength].

[^strength]: "Strength" here doesn't necessarily imply "strength of enforcement" (though that's required to some degree for types to even make sense, but as a whole it's another topic) but rather "strength of assumption", i.e. "how much can you guarantee about the data you're handed?".

```

int/i32    size_t/usize    Integer    Natural    NonZeroNatural    (x:Natural | 1 <= x <= 1e100)
<---------------------------------------------------------------------------------------------->

"Hack-y"                                                                                 "Ideal"
"Dangerous"                                                                               "Safe"
```

Consider a function that takes an integer as an argument.
```rs
type SomeIntType = ???;
fn foo(arg: SomeIntType) {
    // use `arg`
}
```

To use <b>`arg`</b> in any non-trivial capacity, we must make some assumptions about it.
That is, we must categorize it into one of the above integer types (or something similar).

Suppose that we want to allow **`arg`** to hold any value <b>`x`</b> such that <b>`1 <= x <= 1e100`</b>.
We can follow the spectrum from left to right for a relatively natural progression of assumption strength.

| Type | Assumption |
| ---- | ---------- |
| **int/i32** | -2^31 <= x <= 2^31 - 1 |
| **size_t/usize** | 0 <= x <= 2^32 - 1 |
| **Integer** | -∞ <= x <= ∞ |
| **Natural** | 0 <= x <= ∞ |
| **NonZeroNatural** | 1 <= x <= ∞ |
| **(x:Natural \| 1 <= x <= 1e100)** |  1 <= x <= 1e100 |

In many ways, having stronger assumptions is a good thing: your code crashes less and is better documented (these are the pros of static types in general).
But as you move further along the spectrum, the more burdensome it becomes to maintain these assumptions.

While it might occasionally be practical to reach for a really strong assumption (e.g. "I know this number must be between 1 and 10")[^assertions], there comes a frequency at which doing so is undesirable because tracking every invariant in your code is incredibly computationally expensive[^rust].
This is why we don't code in proof languages all day.

[^assertions]: Assertions are a great hack to get these stronger assumptions without the rigidity of the type system (at the sacrifice of the compile-time check).

[^rust]: You know this feeling if you've ever battled with rust's borrow-checker.
You know that something should work, but the mechanisms of the code aren't quite lining up.

Further challenges with working with ideal types:

- We (as programmers) often *don't even know* what assumptions we should make (because the code might change over time, or we're focused on some other issue, or simply don't care).
- Physical hardware constraints (you can't index with big-int types, so **`size_t`** *has to be* a hard-coded hack - in the similar way, assembly language (and everything low-level) is a hack by necessity)
- Many languages don't even allow these asumptions
    - JavaScript doesn't (in the default case) even differentiate between int types and floats
    - Python (effectively[^py]) uses big-ints for everying
    - Rust/C only natively support the first two (int/size_t)
    - Haskell the first 3 (and pretty trivially the first 5)
    - Only a handful of proof-solving-type languages support the last one (e.g. Dafny or Liquid Haskell)
- Proving the last two can be a nightmare: *even Haskell* provides partial functions[^pf] for this reason

[^py]: Technically there's a promotion system under the hood, but we can effectively imagine all ints as big-ints.

[^pf]: E.g. **`head`** and **`tail`** are in the prelude, because it can be inconvenient to determine whether a list is empty at compile-time (this is *way* easier to do than reason about int values).

Sometimes **`int`** (the hack) is simply *good enough*.

## Structural Refactoring

The general structure of code (how data is stored, how structure interact, how algorithms are expressed, etc.) can be even more complex than types.
Refactoring structure is incredibly powerful (as it completely changes the invariants and interfaces you work with), but it's also expensive (in time, effort, and risks).
Structure demands uniformity (both by convention and by necessity[^necessity]), and if that structure is overly refined, it can be just as hard to work with as if it were under-refined.

[^necessity]: The structure of code defines how you interact with it, and code is highly interconnected, so restructuring one thing implies restructuring others.
Thus making one thing more extensible (often) implies making other things extensible.
This can easily be taken too far to get a plethora of Haskell-ish function signatures that mean everything and nothing at the same time.
This deserves a post of its own, but in short, there is a place for such generality (namely in atomic idioms (e.g. [Monads](/blog/monads), Functors, etc.)), but it's completely impractical to maintain on a large scale (and when you try, you get Haskell-ish libraries that are incredibly difficult to understand.
If you want to know how this feels,
try this challenge: using [documentation alone](https://hackage.haskell.org/package/random-1.2.1.2/docs/System-Random.html), write code to generate a random number between 1 and 10 in Haskell.
Try to do it in under an hour.
Good luck!

This [quote from The Primagen](https://youtu.be/5i_O6NLXYsM?si=71misb9VV3xqp4QI&t=725) describes this in a very relatable way:
> Perfectionism [is] such an enemy of good software.
> You want to create the greatest thing that will make all things work, and therefore you have to make these huge sweeping changes to your system to make a small change right here, because [just making local changes will not] be clean and [will] be hard to extend.
> But sometimes you need to say to yourself 'how far do I need to extend this?' maybe I don't need to extend this; maybe instead, this is just simply good enough, and when we need to extend it, maybe we can extend a bunch of things at once.

## Wicked Problems

The problems we set out to solve are often poorly defined, and even if we're given a complete definition, the larger the program, the harder it is to reason about all of the possible shapes it can take up.
"[Wicked Problems](https://en.wikipedia.org/wiki/Wicked_problem)"[^rudy] are problems whose solutions are so self-referential that they can't be solved without concretely attempting to solve them.

[^rudy]: Shout-out to [Rudy](https://github.com/rudyb2001) (endearingly nicknamed "the human RSS feed") who first introduced me to the idea of "Wicked Problems" (along with several other articles I've referenced in my other posts).
I eagerly await the release of his blog, which I hope to eventually link here.

Planning a solution to a wicked problem is like attempting to plan a path to climb up a mountain, while standing at the base.
Some mountains are so wickedly curved that details are obscured at the base, and you need to scaffold your way up (and often back-track) before you can find the right path.

Situations like these naturally produce code that is hack-y by nature.
This is good, because scaffolding is intended to be temporary (but it can often be modified and lightly refactored to work for longer).
And even if the scaffolding gets used as the final result, it's better to have a working hack than a non-existant ideal-program.

## Conclusion

Hack deliberately.
Accept that you must hack (to some extent), but step cautiously: make conscious choices of where you invest in pursuing ideals, and where you forego them.

Whichever choice you make, embrace it and enjoy it: both hacking and obsessing over ideals can be a lot of fun.
Things only become painful when the wrong choice is made.
When that happens, laugh and move on (but never forget).
