---
title: "On Design Patterns"
subtitle: "Initial Thoughts on the Classic OOP Book"
date: 2024-05-26T11:33:05-05:00
notability: 0
tags: ["Pl", "Fp"]
draft: true
---

- I think OOP is still a central way of programming, an effective one, and one that's important to know
- hell, look at rust, even - it still exhibits this (the decomposition into structs still holds)

(11)
> The hard part about object-oriented design is decomposing a system into objects.The
> task is difficult because manyfactors comeinto play: encapsulation, granularity,dependency, flexibility, performance, evolution, reusability, and on and on. They allinfluence
> the decomposition, often in conflicting ways.


(20)
> That leads us to our second principle of object-oriented design:
> Favor object composition over class inheritance.

(31)
> Design patterns should not be applied indiscriminately
> Often they achieve flexibility and variability by introducing additional levels of indirection,
> and that can complicatea design and/or cost you some performance. A design pattern
> should only be applied when the flexibility it affords is actually needed.
