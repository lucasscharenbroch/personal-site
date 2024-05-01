---
title: "Effective Anki Cards"
subtitle: "Against long flashcards"
notability: 3
tags: ["Workflow"]
date: 2023-07-08T19:35:55-05:00
---

TL;DR: long cards are bad; short cards are good.

---

The TL;DR says it all, but I want to elaborate on how I fell into the trap of making bad cards.

I have no excuse for not knowing what good cards look like: I first discovered [Anki](https://en.wikipedia.org/wiki/Anki_(software)) through [Derek Banas' tutorial](https://www.youtube.com/watch?v=5urUZUWoTLo), in which Derek makes suggestions to keep cards simple, avoid lists, and avoid grouping similar data.

While this was very natural in some contexts (i.e. vocabulary), I fell far from the mark when making cards for subjects that required more sequential knowledge. Consider this example from my [Linear Algebra class](/blog/uw-madison-computer-science/#340) (the {{c?::hidden-text}} strings are cloze-deletions: for each cloze-deletion in the note, a new card is created with that text hidden).

{{< highlight tex >}}

let L: V -> W be a linear tranformation, and S = {v[1] ... v[n]}.
L is completely determined by {{c1::{L(v[1]) ... L(v[n])}}}.

proof:
take any v in V - there exists linear coefficient a[1] ... a[n] such that
v = a[1]v[1] + ... + a[n]v[n]

it follows that

L(v) = L(a[1]v[1] + ... + a[n]v[n])
= a[1]L(v[1]) + ... + a[n]L(v[n])

so let A be the matrix with L(v[j]) as its j'th column. L(v) = A*[v][S]

two linear transformations with the same A are {{c2::identical}}

when S is the standard basis, A is called the {{c3::standard matrix representing L}}

{{< /highlight >}}

There are a number of "bad" aspects of this note: (1) there is a lot of [boilerplate](https://en.wikipedia.org/wiki/Boilerplate_text): much of the text on the note is not *immediately relevant* to the idea that the note is trying to show; (2) the note literally includes a proof of its claim: though it may be a good idea to remember the proof in general, the inclusion of the proof doesn't make the concept easier to remember (which is the point of the note); (3) the note includes related facts that (though relevant) could easily be made into other notes (i.e. the last two lines). All of this boils down to one central problem: the note is *way too long*.

This is an extreme example, but I wrote a significant amount of notes that look similar to this, and a lot more that were shorter but still longer than necessary. Even though I understood the facts and proof when I wrote it, trying to reiterate this giant block of logic was too much work for daily reviews, so I usually ended up spamming through the card or making a short and painful attempt at skimming the information. Ironically, my attempt make a detailed note (and to remember *more information*) resulted in a tougher and less-effective review session, through which the majority of information was lost.

In my defense, this information is hard to convey as a flashcard, and I don't think there's a lossless way to transfer it to flashcard-form. However, for the aforementioned reasons, I think that lossy conversion to easy-to-study is better than lossless conversion to cards that take high mental effort to decode, as the latter are effectively useless (as flashcards, at least).

Here's how I would refactor the bad note:

{{< highlight tex >}}
For any basis S and linear transformation L, L is completely determined by {{c1::L(v) for v in S}}
{{< /highlight >}}

{{< highlight tex >}}
Two linear transformations with the same representing matrix A are {{c1::identical}}
{{< /highlight >}}

{{< highlight tex >}}
The matrix A representing L with respect to the **standard basis S** is called {{c1::the standard matrix representing L}}
{{< /highlight >}}

Notes:
- 1 big note => 3 small notes
- The proof is completely discarded: the clutter from its presence outweighs the leap of understanding of its absence.
- The short notes are easy to read (=> take little energy to study)
- No redundant information is included
- The short notes follow a straightforward chain-of-thought (the question *leads to the answer*)

I've found that even big notes that are inherently simple and easy to answer (like lists) tend to be significantly less effective than their shorter variants. For example, consider this note from my vim deck.

{{< highlight tex >}}
Standard Digraphs

NO {{c1::¬}}
DG {{c2::°}}
+- {{c3::±}}
+Z {{c4::Σ}}
a', e', i', o', u' {{c5:: á, é, ...}}
a^, e^, i^,... {{c5::â ...}}
a:, e:, i:,... {{c5::ä}}
(-, -) {{c6::∈, the other way around}}
In {{c7::∫}}
Om {{c8::Ω}}
p* {{c9::π}}
.: {{c10::∴ (therefore)}}
e*, E* {{c11::ε, E}}
d*, D* {{c12::δ, Δ}}
h*, H* {{c13::θ, Θ}}
!<, !>, != {{c14:: as you might expect}}
(C, )C {{c15:: proper subset and proper superset}}
(_, )_ {{c16:: subset and superset}}
?2 {{c17::≈}}
AN {{c18::^}}
OR {{c19::V}}
(U, )U {{c20::union and intersection}}
<7, >7 {{c21:: left and right floor}}
7<, 7> {{c21:: left and right ceil}}
FA (for all), TE (there exists) {{c22::Ɐ ∃}}
/0 {{c23::empty set (∅)}}
00 {{c24:: ∞}}
{{< /highlight >}}

Each element in the list is concise, but I still found the note hard to work with, because the large body of text is distracting. By creating an individual note for each element in this list, though the information didn't change, I found the cards to be faster and easier to answer.

This same idea applies to cards with examples: even though an example (in theory) can be ignored, I found that the presence of the extra text convoluted the goal of memorization, and cued reflex for slower thinking, which slowed down review significantly. Note that this doesn't apply to examples that *are* the prompt: there are occasions when examples alone are easier to use than textual explanations, but if the example can be removed without changing the meaning of the note, it probably should be.

## Conclusion
The goal of Anki (and flashcards in general) is to continually review a maximal quantity of material in a minimal amount of time. Large cards are inherently difficult to process, which slows down review time and reduces review quality. Short cards are easy, which is ideal: they results in less time memorizing, leaving more time to work on the hard stuff.
