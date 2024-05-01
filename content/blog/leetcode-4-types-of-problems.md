---
title: "Leetcode: 4 Types of Problems"
subtitle: "Practical categorization"
notability: 6
tags: ["Workflow", "Learning"]
date: 2023-07-04T15:03:41-05:00
---

## What and Why

[Leetcode](https://leetcode.com/) is a popular platform for studying DSA (Data Structures and Algorithms), especially for [coding/technical interviews](https://en.wikipedia.org/wiki/Coding_interview), [competitive programming](https://en.wikipedia.org/wiki/Competitive_programming), and education in general. It offers a growing list of problems (~2,750 as of July 2023), which can be solved directly in the browser, typically in 10-50 lines of code.

I've solved over 400 problems on Leetcode (roughly 15% of the problems available), and I've noticed that problems tend to draw their difficulty from a few set sources, and can be classified accordingly. Some of these categories are more desirable to learn than others, and practicing random problems without regard to category can cause unnecessary difficulties and inefficiency.

## Problem Types

### 1. Basic DSA
The problem's difficulty lies in a reduction to "basic" DSA (the definition of "basic" is intentionally loose here). The majority of Leetcode problems (especially the popular ones) fall into this category, and Leetcode's "topics" (the categorization for its problems) consists mainly of subcategories of basic DSA, i.e. dynamic programming, greedy, search, graph, binary search, tree, list, etc.

Examples:
- [15. 3Sum](https://leetcode.com/problems/3sum/)
- [72. Edit Distance](https://leetcode.com/problems/edit-distance/)
- [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)


### 2. Implementation
The problem requires little knowledge of DSA and derives difficulty from the inherent difficulty of programming. It's often obvious what needs to be done and the basic approach of how to do it, but doing so usually involves a lot of lines, and is sometimes nontrivial.

Examples:
- [12. Integer to Roman](https://leetcode.com/problems/integer-to-roman/)
- [54. Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)
- [2512. Reward Top K Students](https://leetcode.com/problems/reward-top-k-students/)

### 3. Advanced DSA
The problem's difficulty lies in a *relatively complex* reduction to basic DSA or a reduction to advanced DSA. Once again, this definition very subjective, but the main idea is "basic DSA, but harder". See the examples for explanation.

Examples:
- [212. Word Search II](https://leetcode.com/problems/word-search-ii/) &mdash; requires use of a trie and graph search in a nuanced way
- [312. Burst Balloons](https://leetcode.com/problems/burst-balloons/) &mdash; non-intuitive substring-dp
- [1349. Maximum Students Taking Exam](https://leetcode.com/problems/maximum-students-taking-exam/) &mdash; requires network-flow/matching or nontrivial dp

### 4. Math
The problem's difficulty lies in nontrivial mathematics (once again, this definition is subjective).

Examples:
- [50. Pow(x, n)](https://leetcode.com/problems/powx-n/) &mdash; exponentiation algorithm
- [116. Fraction to Recurring Decimal](https://leetcode.com/problems/fraction-to-recurring-decimal/) &mdash; long division
- [319. Bulb Switcher](https://leetcode.com/problems/bulb-switcher/) &mdash; reduction to number of factors

## Distribution

Even if the definitions were more clearly defined, it would still be difficult to get an accurate measurement of what percentage of Leetcode's problems fall into which categories (especially with the unavoidable overlaps), so the following estimate is based completely on my experience: 60% Basic DSA, 30% Implementation, 7.5% Advanced DSA, 2.5% Math.

It's also worth noting that the style of problems changes over time (with newer problems tending to be more convoluted, and more often falling into implementation and advanced DSA). This is apparent by looking at the length of problem names over time.

{{% center-text %}}
<img src="/images/lc-graph.jpg" alt="Leetcode problem name lengths graphed over time"/>
{{% /center-text %}}

## How to Study

My method for studying was to pick problems at random, and only looking at the solution as a last-resort. I tried doing this before I had a solid understanding of basic DSA, and as a result, I spent many hours trying to "debug" bogus solutions to advanced DSA problems. Several months later, when I had a better understanding of DSA, I spent a lot of time solving implementation problems, some of which were too easy to be worth bothering with. While I think that practicing implementation was good for my general programming skills, it didn't help me much with learning DSA, which was my main goal in using Leetcode. The bottom line is that studying problems at random is inefficient unless your areas-of-improvement exactly match Leetcode's category distribution. Lists like the [Blind 75](https://leetcode.com/list/oizxjoit/) are made for this purpose, and I've found them to be convenient, though they might not be ideal for everybody.
