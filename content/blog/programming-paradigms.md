---
title: "Programming Paradigms"
subtitle: "Why it's practical to learn a variety of languages"
notability: 7
tags: ["PL", "FP"]
date: 2024-06-28T10:03:14-06:00
draft: true
---

## Ideas and Code: Why Knowing Paradigms Matters

Writing code is a never-ending battle to translate abstract ideas into formal instructions.
This translation is lossy by nature: by making our ideas concrete, we are forced to face their inherent complexity, and the bewilderingly large set of ways to arrange their inner-workings[^hacking].

[^hacking]: For more thoughts on why this is and how we can best deal with it, see my post on [the necessity of hacking](/blog/hacking-is-necessary).

Programming is ultimately about coping with this complexity, often by imposing extra constraints, for example:

- Programming Language Syntax
- Style/Structural Conventions
- Idioms
- Design Patterns

The goal is for these rules to narrow the scope of possible translations, reducing the likelihood of out-of-control complexity.
However, the rules aren't perfect, and they almost always leave something to be desired.
This framework for expression of ideas is all too often at odds with the ideas themselves.
Furthermore, the framework itself can have unintentional influence on the ideas: as a programmer becomes more comfortable with a translation method, it becomes harder to draw the line between the ideas and their translations.
This might be harder to see in more moderate cases, but it's clearly true at the extremes:

- The 30-year COBOL developer who doesn't know what JavaScript is[^real]
- The FP enthusiast who swears by Haskell and scorns all industrial programming and SoyDevery
- The astronomer/scientist who solely uses Python for their studies
- The SoyDev who refuses to use a language that doesn't end in the word "script"

[^real]: I drafted this before I met people who actually fit this description.
Now I feel a little guilty, because it's so accurate that it might be seen as an insult.
Let it be known that I mean no offense to anybody who falls into these categories.
Having single-mindedness (when it comes to development) doesn't necessarily make a person bad at serving their respective role.
But it can definitely be detrimental in an environment that requires flexibility and nuanced perspective (most non-legacy development).

The common strand here is a *rigidity of mind*.
If you've worked with people who fit these descriptions, you'll know that, when posed with a problem, they have difficulty seeing it through any lens that isn't the one they're accustomed to.
This might be okay if that problem is effectively solved by their respective paradigm, but when it's poorly fitted[^fit], things don't turn out well.
This goes beyond coding too: communicating with single-paradigm devs is harder: they don't have intuition for ideas you explain to them, and they can't use your diverse intuition to more effectively explain ideas to you.

[^fit]: Or even slightly ill-fitted, where there's clearly a better alternative, but the sub-ideal paradigm can still be used: e.g. writing a NES emulator in JavaScript ([Sorry, Rudy](https://github.com/rudyb2001/js-nes-emulator)).

It's easy to see the problem here because these examples are so extreme.
But what about devs who are only familiar with two or three paradigms[^paradigm-count]?
The logical conclusion is that these flaws still hold, just to a lesser extent.

[^paradigm-count]: It's obviously way more nuanced then simply "count of known paradigms" (there aren't even that many widely adopted "paradigms" (that have well-known names), and the bounds and definitions of a paradigm are blurred).
You can instead imagine it as a spectrum of diversity of thought.

Therefore,

```
more-paradigms = more-diversity-of-thought = better-code = deeper-understanding = better-communication
```

These are all things I care deeply about, and things I strive to maximize.

## Case-Analysis: Beauty in Extremes

### Assembly

### Python

### Haskell

## Blending Paradigms

---

It's why I (like so many others) am so interested in trying to deeply understand less-popular programming paradigms[^fun-to-learn].
My goal is to train myself to deeply understand the formulation of ideas by analyzing their translations through a variety of constraint-sets.

[^fun-to-learn]: They're also a lot of fun to learn.
