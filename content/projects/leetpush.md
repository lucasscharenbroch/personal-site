---
title: "Leetpush"
subtitle: "A Chrome Extension for committing to Github from Leetcode (JS)"
notability: 2
date: 2024-02-11T11:46:13-06:00
---

{{% center-text %}}
<img src="/images/leetpush.jpg" alt="Leetpush extension popup" width="850px"/>
{{% /center-text %}}

### ([Github Link](https://github.com/lucasscharenbroch/leetpush))

## Background and Motivation

If you're unfamiliar with Leetcode, here's an elucidating quote from my [7/23 post on leetcode-problem-categorization](/blog/leetcode-4-types-of-problems):
> [Leetcode](https://leetcode.com/) is a popular platform for studying DSA (Data Structures and Algorithms), especially for [coding/technical interviews](https://en.wikipedia.org/wiki/Coding_interview), [competitive programming](https://en.wikipedia.org/wiki/Competitive_programming), and education in general. It offers a growing list of problems (~2,750 as of July 2023), which can be solved directly in the browser, typically in 10-50 lines of code.

I've solved quite a few Leetcode problems over the past few years, and Leetcode has (at times) become a part of my daily routine.
Immediately after solving problems, I always push my solutions to my [leetcode-solutions repo](https://github.com/lucasscharenbroch/leetcode-solutions) on github. I like to say that I do this because it makes them easily accessible for future reference, but I really do it because it makes [my contribution graph](https://github.com/lucasscharenbroch) look like a Christmas tree[^contribution-graph].

[^contribution-graph]: As superficial as it is, I've found that keeping a green-streak on my contribution graph has been very effective in motivating daily coding, even beyond Leetcode (and other toy problems/exercises). "The hardest part is getting started" holds for many aspects of programming, and this routine fosters the habit of making frequent (and atomic) progress, shrinking the difficulty of starting large tasks.

There are two ways[^2ways] to copy code from the leetcode website to github, and both involve way more clicks and keystrokes than I'd prefer.

Enter Leetpush.

[^2ways]: (1) copy it into a file, commit that file, push; (2) copy it into the github website, commit from there.

## Tools

This project is not technically challenging.
Its primary difficulty lies in the logistics of tying together APIs, namely:

- The Github API
- The Chrome API (extension permissions/ functionality)
- Scraping the Leetcode page

I used [Svelte](https://en.wikipedia.org/wiki/Svelte)[^svelte] (a web framework) for the main popup, as its intended functionality is pretty dynamic, and would be exceedingly annoying to try to handle that in vanilla JS.

[^svelte]: Svelte was chosen over the alternatives because I've been contributing (more accurately: fixing bugs) to [anki](https://github.com/ankitects/anki) recently, and Svelte is the framework used for some of anki's front-end.
I'd like to try to better understand it so I can contribute in less-trivial ways.

## The Chrome Extension

This is the first time I've written a Chrome Extension; it was simpler[^chrome-simple] than I thought.

[^chrome-simple]: The API is simple to use, not necessarily simple under the hood.

A hello-world is just a matter of writing a manifest.json file with a hand-full of lines, e.g.
(from [google's tutorial](https://developer.chrome.com/docs/extensions/get-started/tutorial/hello-world))

{{< highlight json >}}
{
  "manifest_version": 3,
  "name": "Hello Extensions",
  "description": "Base Level Extension",
  "version": "1.0",
  "action": {
    "default_popup": "hello.html",
    "default_icon": "hello_extensions.png"
  }
}
{{< /highlight >}}

From there, it's just a matter of adding "permissions", which give your script access to the [Chrome Extension API](https://developer.chrome.com/docs/extensions/reference/api/), which is pretty straight-forward given rudimentary knowledge of JavaScript and asynchronous execution.

{{< highlight diff >}}
9c9,14
<   }
---
>   },
>   "permissions": [
>     "activeTab",
>     "storage",
>     "scripting"
>   ]
{{< /highlight >}}

## The ~~Soy-Dev~~ Web-Dev Philosophy

Web APIs, Libraries, Frameworks and IDEs fit together quite nicely, and they all are heavily tied around one main idea: **all problems are solved with more JavaScript**[^more-js].

[^more-js]: "More JavaScript", as in "more boilerplate", "more dependencies", etc.

This is a double-edged sword.

Pros:
- *Easy*[^easy]: don't require a lot of effort to use (as I have seen in this project)
- *Powerful*: a lot can be done with extra code

[^easy]: My use of "easy" and "simple" follows Rich Hickey's definitions from ["Simple Made Easy"](https://www.youtube.com/watch?v=SxdOUGdseq4), namely: Simple: "One fold/braid" (opposite of complex); Easy: "Near, at hand".

Cons:
- *Complex* (not simple) - lots of layers of abstraction
- *Bloated* - lots of unnecessary baggage

It's hard to say to what degree the above cons are avoidable, especially with the inherent complexity of the web, and the speed of innovation and change in technology, but it's interesting to consider these flaws, their relationship to those in non-web tools, and their impact on software development and performance.
