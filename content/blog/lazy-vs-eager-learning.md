---
title: "Lazy vs Eager Learning"
subtitle: "What's the best way to learn programming skills?"
date: 2024-03-13T09:53:10-05:00
---

There are a lot of opinions about the "best ways" to learn programming on the internet, but they're usually some combination of the following:

1. "Just write code" ("learn by doing")
2. Learn "the basics" (e.g. DSA)
3. Learn certain technologies (e.g. React)
4. Use resource *x* (e.g. books, websites, courses, video tutorials)
5. Get a degree

By far, (1) seems to be the consensus.
It's also is also the most generic: (2)-(5) all refer to some specific resource or concept, while (1) is more of an abstract strategy.

When people recommend (1), usually usually mean the following:

- Gaining skills through writing code (in general)
- **Lazy Learning** - Waiting to acquire information until it is necessary to know

The two are very tightly intertwined.
Writing code almost always implies some degree of lazy learning;
Lazy learning is much more general: it encompasses a lot of things, because the "necessary to know" clause can be imposed by anything[^procrastination].
The most useful way to use lazy learning is to purposefully make the target information "necessary to know" by beginning to write a program that requires it.

[^procrastination]: By this definition, procrastinating studying for a test (or skipping lectures and letting lecture-videos-to-be-watched accumulate) is both lazy and eager learning.
It's lazy because learning is deferred until it's necessary for the test, but it's also eager because the act of learning it is proactive (since it's a class, and it's knowledge intended to be used in the future).

So, for the purposes of this post, we'll use these domain-specific definitions:
- **Lazy Learning** - Acquiring information on-the-fly by writing programs
- **Eager Learning** - Acquiring information proactively, before applying it

## Efficiency and Exclusivity

Lazy learning is incredibly efficient in most situations.
It sets up a dichotomy between learning and failing: either you learn what's necessary to write the program, or the program isn't written.
This is ideal, so long as you succeed.
By its nature, lazy learning makes it easy to...

- See the concrete practical utility in the concepts you are learning
    - Get excited about building/learning (tight feedback loop)
- Understand how to implement these concepts in practice
- Avoid spending energy learning unnecessary facts

But all of this only happens if the learning succeeds: it's possible (and sometimes likely) for this strategy to fall on its face.
It's only feasible to learn on-demand when that learning is *relatively easy*: if the knowledge-being-acquired is sufficiently novel or complex, the barrier to learning becomes so high that it can't effectively be intertwined with coding.
For instance, imagine a beginning programmer attempting to write an operating system from scratch: they wouldn't know where to begin.
The same idea holds for more experienced developers when attempting to learn concepts of similar novelty.
Lazy learning can handle a finite amount of complexity before it cracks and eager learning is required.

Eager learning (e.g. reading books, watching tutorials, going to lectures) has its own set of benefits.

- Practicality (implementation details) is not an impediment
    - This makes it possible to explore complex problems incrementally, which is infeasible in lazy learning
    - The deferral of implementation also makes it possible to reason about especially abstract ideas
- Long-form materials make it easier to gain a wider, deeper, more high-level understanding
- Materials often group information together (and point to other materials), making extended exploration easy and likely

And drawbacks.

- It's easy to fall into a cycle of discovery and end up spending little time actually applying the acquired knowledge (some people are more prone to this than others)
- Unnecessary and unhelpful information is more likely to (unknowingly) cause distractions
- The abstract natures makes it easy to get demotivated or detached from the goal (to write programs)

## Both Are Necessary

If you want to become a proficient developer, you must have a wide range of knowledge and practical experience.
To gain these, you must employ both eager and lazy learning.

If you avoid eager learning, you will[^well] be stuck working with the technologies and ideas you are familiar with, and you will be naive and bind to the world of possibilities within technologies unknown to you.
You will have no mechanism for discovering new ideas, and you will be stubborn to change.

[^well]: These statements are admittedly extreme, but I use them in attempt to make my point clear.
Reality is more complex than this, but these are relatively true in generality.

If you avoid lazy learning, you probably aren't writing much code, which means the code you do write probably isn't very refined.
What you do learn is completely hypothetical and will likely soon be forgotten.

## What's the Right Balance?

Do whatever feels best for you, so long as you are avoiding the above pitfalls.
Doing what you are excited about is the easiest way to make quick and effortless progress.

In general, lazy learning should occupy more time, because practical experience cannot be gained without writing code.
This is likely the reason lazy learning is so highly recommended by internet programmers.
Another reason might be the tendency for beginner programmers to find their way into into cycles of eager learning ("tutorial hell"), as lazy learning can be particularly difficult at first.
