---
title: "Anki Curses"
subtitle: "A front-end for the powerful flashcard tool, Anki (Python)"
notability: 4
loc: 900
date: 2023-07-16
---

{{% center-text %}}
<img src="/images/anki-curses.jpg" alt="Anki in Curses"/>
{{% /center-text %}}

### ([Github Link](https://github.com/lucasscharenbroch/anki-curses))

## The Problem

As a student and a programmer, I have a lot of stuff to memorize. [Anki](https://apps.ankiweb.net/) allows me to do this efficiently, but there's a problem: the only vanilla way to study is through AnkiWeb (a web app), or Anki's desktop program (a [qt](https://www.qt.io/) (GUI) app). Neither of these options allow me to do my daily reviews without leaving the terminal, and they both require excessive use of *the mouse*.

My first thought was to search for a [curses](https://en.wikipedia.org/wiki/Curses_(programming_library)) alternative, but apparently not many people have the same problem as me, because I couldn't find one.

## The Solution

Anki turns out to be open-source! So begins this project.

## Anki's Architecture

The codebase has three main components: Rust (back-end), Python (front-end), and TypeScript (working with cards). The TypeScript component mainly involves specialized card features (like image occlusion and LaTeX), which I didn't think were feasible for a minimalist curses program, and since the qt program is written in Python, there is a Python interface for the Rust back-end. Therefore the curses program can be written entirely in Python, and (in theory) has no less power than the qt GUI, since it has access to the same library.

## Implementation

The implementation was surprisingly painless. The main challenge was becoming familiar with the codebase (which was *lightly* documented). I was able to get a working version in around 15 hours; a lot of that time was spent learning curses (as I've never used it for a non-toy project).

## HTML

The biggest "technical" issue was parsing HTML: Anki stores cards and notes in subtly different HTML formats, and I also used HTML to style formatted text drawn on the screen[^attr-ex]. The use of multiple layers of HTML caused a few minor difficulties.

[^attr-ex]: For example, *print_styled_mu(window, 0, 0, "&lt;b&gt;bold&lt;/b&gt; &lt;r&gt;red&lt;/r&gt;")* prints "**bold** <span style="color:red">red</span>" at (0, 0) in the given window.

Consider a note with the following text:

**Front**:
><i>this</i> is the
>
><b>question</b>

**Back**:
>&amp; <i>this</i> is the
>
><b>answer</b>

This is stored in the database as:

**Front**:
{{< highlight tex >}}
<i>this</i> is the<br><b>question</b>
{{< /highlight >}}

**Back**:
{{< highlight tex >}}
&amp; <i>this</i> is the<br><b>answer</b>
{{< /highlight >}}

During review, to print the card, this text[^q-a] is checked for Cloze Deletions[^cloze], *br* and *div* tags are converted into newlines, and bold/italic/underline tags and HTML escape sequences (e.g. *&amp;amp;*) are left unchanged:

[^q-a]: The note-dictionary (from the database) is actually not parsed directly, because this throws away the generality of question-answer formatting for notes with more than two fields. Anki's backend note object provides *.question()* and *.answer()* methods, which provide HTML in a slightly different format, which is then weeded of redundant tags, and processed as described above.
[^cloze]: Cloze Deletions allow for removal of specified text on the card: see the image at the top of this post as an example.

**Front**:
{{< highlight tex >}}
<i>this</i> is the
<b>question</b>
{{< /highlight >}}

**Back**:
{{< highlight tex >}}
&amp; <i>this</i> is the
<b>answer</b>
{{< /highlight >}}

The text is then passed to the print function, which parses it as HTML again, converting it to a (plain-text, attribute-list) pair (for printing), which looks ike this.


**Front**:
{{< highlight python >}}
("this is the\nquestion", [italic] * 5 + [normal] * 8 + [bold] * 8)
{{< /highlight >}}

**Back**:
{{< highlight python >}}
("this is the\nanswer", [italic] * 5 + [normal] * 8 + [bold] * 6)
{{< /highlight >}}

This form can now be easily printed onto a terminal screen.

The procedure for producing note text to edit is slightly different, because the style tags (italics, bold, underline) should remain in the text. This can't be done directly, however, because this allows multiple db-texts to convert to the same plain-text, leading to an ambiguous reverse-conversion[^asm]. For example, consider a note whose text is "*text*" (italicized). It is stored in the database as "&lt;i&gt;text&lt;/i&gt;", and converted to editable text as the same string. Now consider a note whose text is literally "&lt;i&gt;text&lt;/i&gt;": it is stored in the database as "&amp;lt;i&amp;gt;text&amp;lt;/i&amp;gt;", and converted to editable text as "&lt;i&gt;text&lt;/i&gt;", the same string as above. Therefore, if both notes are edited and saved as-is, at least one of them will be changed.

[^asm]: Assuming angle brackets and ampersands are not escaped in the plain-text

To make a representation with no ambiguity, the db-text to editable-text conversion must be one-to-one. The obvious way to do this is to edit the db-text format directly, but this is really inconvenient if your cards contain a lot of angle brackets and ampersands (which is true in my case). I decided to instead escape the bold/italic/underline tags with backslashes (and escape the backslash with two backslashes). The above examples would convert to "\itext\I", and "&lt;i&gt;text&lt;/i&gt;", respectively. "\i" corresponds to "&lt;i&gt;", and "\I" corresponds to "&lt;/i&gt;". This conversion is a little ugly (the backslash-escape tags are harder to differentiate from the text than html tags), but it works.

## Practical OOP

The entire program is made up of only two (2) distinct "screen types"[^editor] (ui layouts): reviewer and select-from-list. The reviewer only has one use, but there are several uses of select-from-list (deck manager, note browser (:find, :findin), choose-a-note-type (:new)). The *SelectFromList* class[^lambda] is used for all of them.

[^editor]: With exception of the text-editor, but all text-editing is done by calling $EDITOR on a tempfile; this is easy to implement and ideal for workflow.

[^lambda]: I opted for lambda functions instead of overridden methods to allow SelectFromList to be used directly (without override).

{{< highlight python >}}
class SelectFromList(KeyHandler):
    choices: list[T]
    is_match: list[bool]

    def init_keybinds(self) -> None:
        self.keybind_map = \
        {
            'h': self.no_selection,
            'q': self.no_selection,
            'j': self.move_down,
            'k': self.move_up,
            '/': self.search,
            'n': self.next_match,
            'N': self.prev_match,
            'f': lambda: self.screen_down(curses.LINES - 4),
            'b': lambda: self.screen_up(curses.LINES - 4),
            'd': lambda: self.screen_down((curses.LINES - 4) // 2),
            'u': lambda: self.screen_up((curses.LINES - 4) // 2),
            'g': lambda: self.screen_up(1e18), # go to top
            'G': lambda: self.screen_down(1e18), # go to bottom
            'l': lambda: True,
        }

        self.keys_handled_by_parent = [':']

    def __init__(self,
        mm, # main menu
        parent: KeyHandler,
        prompt: str,
        choices: list[T],
        elem_to_strs: Callable[[T], tuple[str, str, str]], # (left, center, right)
        elem_is_match: Callable[[T, str], bool] = lambda e, s: False, # for searching
        keybind_help = "hq=back  jk=navigate  l=select"):

        self.init_keybinds()

        # ...

    # ...
{{< /highlight >}}

As a result, adding vim-like bindings (navigation keys, command mode, search) to this one class makes those bindings applicable almost everywhere in the program.
