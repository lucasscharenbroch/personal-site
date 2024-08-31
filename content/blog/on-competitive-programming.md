---
title: "On Competitive Programming"
subtitle: "My experience and reflections"
date: 2024-08-31T15:05:45-05:00
notability: 6
tags: ["learning"]
---

{{% center-text %}}
<img src="/images/on-competitive-programming/icpc-23.jpg" alt="'Sphagetti Sort' [sic] at ICPC NCNA 23-24 (11/23)" height="500px"/>
{{% /center-text %}}

## What is Competitive Programming?

"Competitive Programming" (by its name) encompasses all programming done in competitive nature.
This post will specifically focus on (and use the term "competitive programming (CP)" to refer to) data-structures/algorithm (DSA) style contests.

These contests usually have a similar format: solve as many problems as possible, as fast as possible.
Where "problem"s are "solved" by providing a program that transforms some specified input to some specified output under some specified constraints (i.e. run time and memory).

## My Experience

I practiced for competitive programming CP pretty consistently for about a year.
While the training I did wasn't nearly enough to be "good" (to excel in competitions), it was enough to develop a solid idea of what skills CP trains, and thus to have a meaningful opinion on the usefulness of CP in general.
I'll outline my experience here to (1) flesh out the weight of my opinion[^reader], and (2) keep this post from being too short.

[^reader]: To allow the reader to judge my opinion based on my knowledge of CP.

I've solved...[^problem-count]

[^problem-count]: Problem count is just as flawed a metric as LoC, but it gives an idea.

- [450 Leetcode Problems](https://github.com/lucasscharenbroch/leetcode-solutions)
- [150 Kattis Problems](https://github.com/lucasscharenbroch/kattis-solutions)

Hardcore CP enthusiasts will be quick to point out that Leetcode problems are easier than the average CP problem ([codeforces](https://codeforces.com/)), and some might not even consider Leetcode CP, because it's more heavily oriented towards interview prep.

This is true, but I'll include them anyway, because their format is very similar, and they train many of the same muscles (which makes them significant in my experience).

Several of those Kattis problems were done for the [UW ICPC Training](https://pages.cs.wisc.edu/~dieter/ICPC/training.html), which introduced a wider (than Leetcode) range of algorithmic topics.

I competed in the ICPC NCNA in all 2[^all-2] years of college.

[^all-2]: [I graduated in 2 years](/blog/speedrunning-college/).

- [NCNA 2022-2023: 14th/116](https://ncna22.kattis.com/contests/ncna22/standings)
- [NCNA 2023-2024: 9th/129](https://ncna23.kattis.com/contests/ncna23/standings)

The placing might seem more impressive than it is.
Compared to the average team (at the regional), we probably seemed pretty good, but compared to most "serious competitive programmers", we were average, at best.[^no-offense]
We would have gotten creamed at nationals, but in both years, at least 3 teams from our school placed higher than us.

[^no-offense]: This isn't to be negative or to insult my teammates (or anybody who placed under us), but as an effort to paint an accurate picture to unfamiliar readers.

In mid-2023, I slowed (then eventually stopped) my training in favor of studying other topics (that become more interesting to me at that time).

## Reflections

Now that my perspective is clear, I'll discuss the advantages/disadvantages of CP as a means of improving as a developer[^goal].

[^goal]: I'm not suggesting that everybody who does CP does it solely to become a better developer.
But I think that there is a lot of overlap, and that these ideas are useful to consider before/after diving in.

### Advantages

The greatest benefit of CP is the sheer amount of non-trivial code it makes you write.
By doing CP, you develop a wicked intuition for the various shapes[^within-the-paradigm] code (at the smallest scale) can take[^may-seem-trivial].
Furthermore, it gives you a deep understanding of DSA (way past what is practical in normal development), so there's virtually no common coding problem you can't effortlessly solve.

[^within-the-paradigm]: More specifically, you gain the ability to quickly find the local minima (probably within the respective paradigm/language): not necessarily the "best" way to code the solution [(which might be found in another paradigm)](/blog/programming-paradigms).

[^may-seem-trivial]: This may seem trivial, but it's remarkably easy not to develop this skill by (1) not coding very much, and (2) coding with very little variety.
Marginal differences in local-mastery of code can also magnify in larger programs.

A (perhaps more superficial) skill CP develops is a strong intuition for program efficiency and run speed.
Since submissions fail when they take too long to run, competitive programmers must learn how to accurately compute the run-time of a program based on its asymptotic efficiency[^how].

[^how]: This is usually done by substituting the input `N` into the asymptotic notation (e.g. `N lg(N)`, or `N^2`), then assuming computers do about `10^8` (pessimistic) "atomic instructions" (iterations, usually) per second, mostly ignoring constant factors.

### Disadvantages

- It can take a lot of time
- The diversity of learning steeply drops off (as you get better and focus more on tuning your ability to recognize patterns and solve problems quickly)
- It can create the illusion of progress (when you're beating other people) when you aren't (necessarily) learning many new things

### Summary

There's a lot to be gained from CP (even outside esoteric algorithmic knowledge).
If you're excited about CP, and you don't have any other pressing priorities, then it's a great way to spend your energy.

{{% center-text %}}
<figure>
<img src="/images/on-competitive-programming/solution-accepted.gif" alt="Submission judge being ran, checkboxes slowly filling up" height="500px"/>
<figcaption>There is little more thrilling then waiting (in anticipation) for the result of the judge.</figcaption>
</figure>
{{% /center-text %}}

That being said, don't feel obligated to keep grinding if you have other interests or lose excitement, as the general knowledge and coding ability benefit drop off steeply.
