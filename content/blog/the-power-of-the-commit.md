---
title: "The Power of the Commit"
subtitle: "Atoms of progress"
date: 2024-05-17T07:47:11-05:00
notability: 7
tags: ["Workflow"]
---

{{% center-text %}}
<img src="/images/commits.jpg" alt="The output of git-log (a list of commits)" height="500px"/>
{{% /center-text %}}

I don't know of a developer who doesn't appreciate version control.
The undo-on-steroids it provides is convenient and invaluable.
But there's a much more subtle aspect of version control (perhaps even a by-product) that I think has equal (if not greater) effects on development: *commits*.

## Atoms (Peeling Apart)

Every git repository is made up solely of commits.
Assuming that each commit contains one atomic change[^ideal-commit]; this fact proves that all software can be taken apart and synthesized.
In a world of giant, unfathomable programs, this is a powerful idea we must hold onto: no piece of software is too complex to understand, it's solely a matter of unraveling.
Commits are one thread we can pull on to do this.

[^ideal-commit]: This ideal isn't always followed, of course, but I don't think there's any technical reason why it cannot be (so long as the programmer periodically commits as they work): some sections of programs can be more inherently intertwined than others, but they can always can be broken into related (though perhaps not independent) chunks.

## Rhythm (Building Up)

Commits provide structure: they're discrete steps towards solving problems: bricks in the building of a program.
It's easy to get lost in the complexity and sheer volume of code that must be written: commits are an antidote to this: they're a proof of progress[^little-acorns].
This makes committing inherently satisfying, even if it's trivial.
They give intermediate gratification[^leetcode], and they lead into each other: they help develop a rhythm of advancement.

[^little-acorns]: This observation can be applied to most other facets of life too; it's often helpful to break problems into little pieces that can be individually tackled.
The intro to [the White Stripes song "Little Acorns"](https://www.youtube.com/watch?v=KSvGOKH5zxk) makes a compelling case for this.

[^leetcode]: Not unlike seeing that sweet green text after solving a leetcode problem.
There's something about the color green...

## Streak (Progressing Persistently)

This rhythm works on a larger scale, too.
Sites like github can track your daily commits (and other contributions) across all repositories, which can lead to a streak of green squares.
Once you have a streak, it becomes your duty to maintain it, which creates a daily obligation towards committing.
This might seem a little superficial[^superficial], but (for me) it's effective nevertheless.
It's a wonderful way to encourage getting the ball rolling, which I've found is often the hardest part.

[^superficial]: You might expect that this abstract rule to encourage making dummy commits (like fixing typos, or other trivial changes) rather than doing work of consequence.
While this is sometimes true, I've found that this is not an issue for me: I only rarely have such days, and they only happen when I truly am short on time.
For me, maintaining the streak is a matter of pride, and making trivial commits feels like cheating.

{{% center-text %}}
<img src="/images/commit-graph.jpg" alt="A github commit graph that looks much like a christmas tree" width="750px"/>
{{% /center-text %}}
