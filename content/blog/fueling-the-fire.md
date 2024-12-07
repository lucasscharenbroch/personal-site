---
title: "Fueling the Fire"
subtitle: "The role of motivation in development"
date: 2024-08-30T11:30:04-05:00
notability: 6
tags: ["workflow", "analogy", "non-technical"]
---

{{% center-text %}}
<img src="/images/feeding-the-fire/feed-the-fire.jpg" alt="Man chucks wood into a forest fire" height="500px"/>
{{% /center-text %}}

## Intro

Motivation, though often overlooked, is arguably the strongest influence on a developer's[^dev] productivity.
While skills, obstacles, and environments define the challenges we face, motivation defines the speed and fervor with which we approach them.

[^dev]: This argument generalizes far beyond programming, but it's easier to speak in concrete terms.

Motivation is dually the great enabler and the great disabler.
With sufficient motivation, no challenge is too difficult to overcome; without motivation, no challenge is easy enough to begin.

## Decomposition

I conceptualize motivation as a combination of three forces.
All three are required to become motivated.

1. **Inherent Fire** - a deep, primitive desire to obtain something (related to the task at hand)
2. **Reachability** - an intuitive understanding of how to make immediate progress
3. **Whim** - the uncountable implicit factors that control a human's desires

(1) is pretty straight-forward.
You aren't motivated to do things you don't inherently want.
I don't know where this fire comes from[^fire-comes-from] (it's probably influenced by personality), or how mutable it is, but it's hard to deny its existence when you feel it at full-strength.
Though the origin of the feeling is usually out of control, it can be directed[^direct].

[^fire-comes-from]: It's quite possible that it's also the derivative or repurposing (or even interpreting) of more primitive desires.

[^direct]: For example, if you're inherently driven to play competitive video games, you might instead decide to direct that fire towards competitive programming.

(2) is mostly a matter of practicality.
If you don't know what the next step is (or if it's too big), it's hard to get revved up to do it (because you don't know how to direct your energy), even if you're inherently driven.
Furthermore, when the next step is obvious and easy, it's natural for us to want to do it.
This usually leads to satisfaction, and thus more motivation, causing a snowballing (or, in this case, forest-fire) [positive feedback loop](https://en.wikipedia.org/wiki/Positive_feedback)[^acorns].

[^acorns]: This ties in with the idea of little acorns and [the power of commits](/blog/the-power-of-the-commit).

(3) is admittedly imprecise, but I think that imprecision necessary because it's a little different for everybody.
In a nutshell, there are a lot of tiny (even subconscious) influences on the brain and body that, when added up, dictate what you "feel like" doing.
These influences might include:

- novelty of the task
- mood / mental state
- environment
- bodily state (sickness/pains)

When these factors go wrong, then often result in a mildly ill feeling ("[malaise](https://en.wikipedia.org/wiki/Malaise)").
Sometimes, you sit down at your computer, and even if (1) and (2) align, it's hard to bring yourself to the task (and hard to stay focused on it).

## Thinking and Doing

When motivation is lacking, there are two primary ways of addressing it[^asm].

[^asm]: Here we'll assume that inherent fire isn't lacking; if that is the case, the solution isn't as interesting (either find a way to direct some existing fire towards the challenge at hand, or find some other challenge to face).

1. Changing your mindset and "tricking your brain" into becoming motivated
2. Molding the challenge (and the surrounding factors) into a form that is more motivating

The effectiveness of (1) lies in the ability to "correct" the inefficiency of the mind.
In theory, reachability and whim are entirely internal to the mind[^straw], and there's no reason why we can't use mental gymnastics (logic, meditation, etc.) to undo (at least some of) their negative effects.

[^straw]: Of course this is necessarily true, because, by definition, all affairs of humans are governed by internal workings of the mind.
But this is interesting because we *do* have some control over what happens in our heads.

(2) is a matter of changing your workflows (how you approach the challenge), or even choosing different (or rephrased) challenges in the first place.

Both of these tools are useful, but can be counterproductive when used in the wrong place.
(1) is more fit for fighting short-term inconveniences[^jb].
It's the "white-knuckle" approach - necessary to resolve immediate, non-recurring difficulties, but otherwise unsustainable.
You can't trick your brain forever.
Conversely, (2) should only be used for relatively long-term fixes.
Changing procedures for a single occasion will cause distraction.

[^jb]: As Jonathan Blow describes in [this video](https://www.youtube.com/watch?v=i7kh8pNRWOo).

## Conquering Whim

The following are strategies I've found helpful for battling misaligned whim (the most evasive aspect of motivation).

- Fight to stay excited about your current pursuit
    - Finish projects/studies quickly, before they go sour
        - Don't spend 80% more time for 20% extra gains
            - ["Have a goal; accomplish it, then leave"](https://youtu.be/NtCmZ8uA5DY?si=w665ZPNbQN8U3m9Y&t=1134)[^nuance]
    - Try to do what you feel like doing, not what you feel obligated to do[^personal]
        - Or otherwise try to find a balance[^vg-eg]
    - Accept Imperfection ([the cult of done](https://www.youtube.com/watch?v=bJQj1uKtnus))
    - Spend your [kreplits](https://www.urbandictionary.com/define.php?term=Kreplit) wisely
        - Don't spend a kreplit when you'll get cut off or won't use it (e.g. in an environment you can't focus)
        - Don't let novelty rot (if you're excited about something, make it a priority to address before it goes stale)
- Encourage novelty (both in development and life in general)
    - Expose yourself to new technical topics
    - Spend time doing things that require different mental frames than development

[^nuance]: This "know what you want to get done" mentality works better in the long-term (e.g. for deciding when to finish a project, not when to end the day).
In programming, when dealing with novel challenges, it's often hard to predict how far you might get in a day, and it's confining to cut yourself off early (failing to [punch through the board](/blog/punching-through-the-board)) and frustrating if you set a goal that's too ambitious.

[^personal]: This obviously applies to personal pursuits, not work.

[^vg-eg]: For example, if you're into game development, you might like the process of designing games more than writing code.
You feel obligated to (apart from making games) practice you coding skills.
I argue that it's a bad idea to put aside your passion for obligation, but it might be helpful to spend *some* time on that obligation (which, in this case, would eventually strengthen the passion).

## Relative Motivation

At any given moment, you can score all possible activities by your motivation to do them.

Generally, the activity you choose to do (when given the choice, e.g. in your free time) will rank relatively highly.
And when you force yourself to do something that doesn't rank very high, it will be difficult to sustain (because more-highly-ranked activities will draw you in).

For an action to be considered (or high enough) in the ranking, it must have a sufficient level of reachability (e.g. you aren't motivated[^def] to play xbox when you're driving to work).
You can imagine that a few "baseline" actions are always reachable, e.g.

[^def]: By the definition of "motivation" above (you might feel like (by whim) playing xbox when driving to work, but you won't actually do it, because it's not reachable).

- doing nothing at all[^depression]
- thinking (about whatever comes to mind)
- checking your phone

[^depression]: Depression is characterized by all (or most) motivations being below this baseline.

This can be a useful framework for understanding why you are drawn to certain actions.

If your highest-motivated actions are undesirable, you might want to consider decreasing their reachability to bring them below the motivation-level of the actions you'd rather do.
