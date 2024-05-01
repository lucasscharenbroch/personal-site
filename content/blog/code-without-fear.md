---
title: "Code Without Fear"
subtitle: "Minimizing the development-time cognitive load"
notability: 6
tags: ["Workflow", "FP", "PL"]
date: 2024-04-21T10:28:06-05:00
---

Programming is hard.
It's hard to work with tools, to formulate ideas into code, and to make that code work.
As developers, we are inevitably forced to manage many responsibilities at once, more than can be reasonably managed by a single human.
This can have a huge psychological strain: when one responsibility deeply focused on, the others go into subconscious[^subconscious], yet the programmer still feels obligated to handle them - this results in a perpetual state of fear that feels a lot like wading through a [Norwegian peat bog](https://youtu.be/kwzIwFPEGbA?si=BXPUzWCBni53ZRdR&t=735)[^peat-bog].
It results in fragile programs and a stressful programming experience.

[^subconscious]: I mean "subconscious" in the least technical way possible: more accurately, the programmer functions a lot like a single-core CPU running multiple threads: they must juggle many tasks, only working on one at a time.

[^peat-bog]: I think the peat bog is actually a really accurate extended metaphor.
The bog (some development task) doesn't seem too menacing at first, but as you wade into it, you begin to realize that it's thicker and deeper than it originally seemed (because of all the hidden development-time cognitive loads)
You're no longer in such an adventurous mood, and you begin to worry that you might not make it through to the other side, or perhaps you still think you'll make it, but you know it'll be more difficult than it was planned to be
The bog keeps getting deeper (you fight more and more unseen tasks, which arise out of nowhere and compound on each other).
The depths of the bog press on you, and you realize that your body is being held down under pure mud.
Each step is incredibly slow and difficult.
Eventually, you're so close to getting stuck (your progress slows), that you panic, and frantically make a last-ditch effort to grab onto the side of the bog, pulling the grass one handful at a time (unraveling the difficulties, one at a time), until you miraculously heave yourself out of the bog, covered in sludge (the baggage/technical-debt picked up from going so deeply into the bog).

In my few years of programming experience, I've been thrilled to find a few practices[^some] which greatly reduce these fears.
This has led me into an obsession of finding new tools to further reduce these development strains in hopes of attaining a more directed workflow.

[^some]: Some of these are widely accepted, others are still viewed as fringe.

## Source Control

Let's begin with something that is minimally controversial.
Rapid modification and refactoring of code leads to a fear of <span style="color:red">losing progress</span>.
This adds an extra cost for experimentation and rolling back: every change to the code must be done very carefully to avoid messing things up and losing the ability to revert, or the information of what was changed.

<span style="color:green">Version control systems</span> (especially <span style="color:green">Git</span>), though imperfect (it's still technically possible to lose progress), work wonders on this front.

## Memory Safety

### Memory Management

It's also generally agreed upon that <span style="color:red">manual memory management</span> (i.e. C's malloc/free, C++'s new/delete) is dangerous and difficult to do correctly.

<span style="color:green">Garbage Collection</span> is used by most modern languages, and completely removes the need to worry about this; <span style="color:green">RAII (Resource Acquisition Is Initialization) and destruction guarantees/borrow checkers<span> are a more busy (but more efficient) method.

### Buffer Overflows

Languages that don't enforce array boundaries and allow/encourage pointer arithmetic make avoiding <span style="color:red">buffer overflows</span> the responsibility of the programmer.

<span style="color:green">Language-level bounds checks (done internally and automatically)</span> are the obvious solution to this, and they're done by most languages.

## Efficiency

### Asymptotic Efficiency

It's pretty hard to avoid worrying about <span style="color:red">asymptotic efficiency</span> in general, but it's ideal to <span style="color:green">factor out</span> the algorithm (or data structure) into a minimal size so its complexity can be best analyzed and performance best tuned.
This also separates the focus of efficiency from the remainder of the code.

### Constant-Factors

Worrying about <span style="color:red">constant-factor efficiency</span> when writing code is a major distraction.

Ideally, non-library code shouldn't have to change at all to gain this efficiency: using <span style="color:green">optimized compilers and libraries</span> is usually much more effective doing fancy (and usually bug-prone) tricks with code.

## Null

In languages with <span style="color:red">ubiquitous nullable types</span> (which have ignorable null-variants), programmers must decide between (1) checking if every variable is null before using it, or (2) attempting to mentally track invariants of which variables are null, and which functions can return null, almost certainly making mistakes, which lead to runtime errors.

Luckily, the solution is incredibly easy and natural in languages with ADTs: make a data-type that is used for nullable values (i.e. <span style="color:green">Option</span>/<span style="color:green">Maybe</span>), and make the associated value only accessible after a null-check.


## Exception Propagation

<span style="color:red">Traditional try/catch/throw-style exceptions</span> are flawed in the same way as null: the programmer must live in fear of functions unexpectedly throwing exceptions.

The solution follows the same idea: embed the variable behavior within the types rather than adding it as a language feature.
Namely, create a data-type for values that might cause errors (i.e. <span style="color:green">Result</span>/<span style="color:green">Either</span>).

## Syntactic Correctness

<span style="color:red">Syntax errors</span>, though less and less common with greater language fluency, are a delayed-feedback burden (a lot like type errors), because, in editors that don't check for them, they are only caught at compile/run time.
It can also be more difficult to understand the error by the time compilation happens, since large swaths of (previously) untested code are sent to the compiler all at once.

<span style="color:green">In-IDE Linting</span> is also the solution to this fear: syntax errors can be exposed as they are written, giving the programmer verification that their code is correct when they write it.

## Type Alignment

<span style="color:red">Type errors</span> are omnipresent in dynamically-typed languages.
Though these languages may offer greater flexibility and a lower barrier to compilation/running, they also completely remove many type-alignment guarantees, and therefore force their burden on the programmer, which causes extreme mental strain when writing non-trivial programs.

<span style="color:green">Compilation</span> (with type checking) and, <span style="color:green">linting</span> (with type checking) are both solutions to this: a downside of compilation is that it is frequently delayed (must be manually triggered), and is often presented in a sub-ideal format[^cli-compiler-err].
Linting is also imperfect for many languages, so the ideal antidote to this issue is using both linters and compilers.

[^cli-compiler-err]: Namely a vomiting of compiler errors to stdout. This is frequently harder to read - line numbers must be followed, and error messages are frequently cryptic (an infamous example of this is GCC C++, which, in will frequently spit out hundreds (or even thousands) of lines of gibberish for a single error). Inline display of errors in the IDE, immediately after they are typed, greatly tightens the feedback loop and increases confidence before manual compilation.

## API Usage
<span style="color:red">Type-alignment and name matching in terms of API usage</span> is yet another task <span style="color:green">IDE-integration</span> and <span style="color:green">linters</span> can facilitate.
Primitive auto-completion[^vim] can't come up with function names you haven't yet used, so the name must be manually typed, and feedback is delayed until compilation-time/run-time.

[^vim]: i.e. the completion built into vim, notepad++, or any other text-editor that uses words in the buffer for completion.

## Off-By-One Errors

<span style="color:red">Off-By-One Errors</span> are infamous annoyances: they always seem to happen around loops (especially while loops): this is because, by their nature, they are a mismatch of the data that is being transformed/read/written/created and some assumption about that data (usually some local variable).

There are two primary ways to avoid such mismatch, both involve eliminating loops (imperative style): (1) make no assumptions about (and hold no state related to) the data: instead do a <span style="color:green">dispatch on the data itself</span> to determine which path to execute (usually this involves recursion), (2) use <span style="color:green">high-level sweeping operations</span> (i.e. map, filter, fold, zip, etc.) which are pre-coded to avoid such errors (and often employ (1) under the hood).

## Managing State

### Global/Shared Variables

<span style="color:red">Global and shared[^shared] variables</span> are often frowned upon, because their state is hard to track over time.
Globals are rarely necessary, and there's typically an isomorphic way to <span style="color:green">write the programs without global variables</span> that is no less natural, and no more complex.

[^shared]: Shared across functions and blocks

### Local Variables, over time

While the above idea is very common, most developers don't seem to mind <span style="color:red">mutable local variables</span> at all, which surprises me.
Mutable local variables exhibit many of the same flaws that global variables do (especially as the function grows in complexity): their state can unexpectedly change during refactors and reordering of statements, and their state is often difficult to trace over time.
I think most developers (including myself) have grown to accept these complexities simply because they're idiomatic to imperative style.

It's possible to avoid this complexity, though, namely by using <span style="color:green">immutability</span>.
Minimizing state changes of local variables reduces the complexity of function bodies, and makes them inherently easier to understand, taking mental load off of the programmer.

### Complex Data Structures

In many languages, <span style="color:red">it's very unnatural to represent [sum types](https://en.wikipedia.org/wiki/Tagged_union)</span> (union-types with associated data) (it's impossible to restrict access to the data in a struct given the state of an associated enum).
As such structures grow in complexity, it becomes very difficult to work with it in a clean and bug-free way (this is basically the combined fears of mutable state and detachment of state and data).

<span style="color:green">ADTs ([Algebraic Data Types](https://en.wikipedia.org/wiki/Algebraic_data_type))<span>, by definition, allow enumerations with associated data, and come with natural syntax to do a dispatch (and unwrapping of data) according to that enum.
They eliminate the mental burden of trying to bodge together a type that simulates this, and deal with whatever shoddily-glued invariants were used to implement the type every time it is used.

## Functional Manifesto in Disguise

If you haven't already noticed, the above points almost entirely argue moving away from imperative and towards functional style.
I don't intend for this post to be solely an argument for functional, though, but rather a call to be aware of these fears that we frequently ignore, and to actively search for ways to avoid them.
I think functional is a big step towards doing this, but it is by no means the essence of it: I suspect that, even if functional is more widely adopted, we would find more fine-grained fears that also arise in functional programming, and any style we could adopt for that matter.

My hope is that, over time, as we eliminate as many development-time burdens as possible, we can focus less and less on trivial mechanical struggles and more on the problems at hand, on higher and higher levels of abstraction.

## TL;DR

As developers, we (often unknowingly) tolerate unnecessary mental burdens while coding.
Use the following tools to avoid them:

- Source Control
- Linters + IDE integration
- Functional Style
    - Aim for high-level transformation chains rather than iteration and state
    - Avoid mutability
- ADTs (no null or exceptions, no bodged sum types)
- Languages that guarantee...
    - Memory Safety
    - Type Safety (static typing)
- Optimized compilers and libraries rather than optimized (overall) code

The language and the tools you use should take a maximal load off of your shoulders.
