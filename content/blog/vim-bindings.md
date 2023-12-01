---
title: "Every Vim Binding I Know"
subtitle: "An extensive list"
date: 2023-10-01T08:58:26-05:00
---

I love [vim](/blog/vim). It makes editing text fast and fun.
The learning-curve is admittedly steep, however, and the return-on-investment for seeking new bindings looks too much like the reciprocal function.
I have nevertheless spent several hours reading the majority of the [vim user manual](https://vimdoc.sourceforge.net/htmldoc/usr_toc.html) and searching for new vim tips online.
That information was terribly uncompressed, though, so it took way longer than it could have.
The goal of this post is to compress all the information I've collected to make it readily available for fellow vim users.
Please note:

1. Explanation is omitted in favor of conciseness
2. Angle-brackets (\<\>) denote meta-meaning (except when they're used literally)
3. Some especially specialized bindings have been excluded (for example, the list of marks is purposely not exhaustive): consider this a list of "generally useful" bindings
4. This list is far from complete, especially the /usr/bin/vim section
5. If you notice a useful binding that isn't on the list, please [contact me](/contact).

# Generalized Vim Bindings[^generalized]

[^generalized]: This section should apply to ports of vim bindings that aren't necessarily /usr/bin/vim (i.e. other IDEs).

## Normal Mode

### Motions
- h, j, k, l - left, down, up, right
- \<arrow keys\> - a poor alternative to hjkl
- g\<j/k\> - move up or down one "screen line" (instead of an actual line)
- '*&lt;mark&gt;* - go to row of *mark*
- `*&lt;mark&gt;* - go to *row and column* of *mark*
- \<ctrl-o\>, \<ctrl-i\> - go to sequentially older/newer jumped-from positions
- gg - go to top of buffer
- G - go to bottom of buffer
- H, M, L - move cursor to a high/middle/low row on the screen
- \<number\>G - go to line \<number\> (analagous to ":\<number\>")
- \<number\>% - go to given percentage of file
- \<ctrl-f\>, \<ctrl-b\> - move one screen down, one screen up
- \<ctrl-u\>, \<ctrl-d\> - move one half-screen up, one half-screen down
- $ - cursor to last character in line
- g_ - cursor to last non-whitespace character in line
- 0 - cursor to first character in line (column 0)
- ^ - cursor to first non-whitespace character in line
- *, # - find next occurrence of word under cursor (forwards and backwards, respectively)
- w, b - move fowards/backwards to beginning of word
- W, B - move fowards/backwards to beginning of words (including punctuation)
- e, ge - move forwards/backwards to end of words
- E, gE - move forwards/backwards to end of words (including punctuation)
- {, } - move to previous/next paragraph
- (, ) - move to previous/next sentence
- /\<pattern\>, ?\<pattern\> - search forwards/backwards for pattern
- f\<char\>, F\<char\> - move cursor to next/previous occurrence of \<char\> in current line
- t\<char\>, T\<char\> - move cursor to the character "before" next/previous occurrence of \<char\> in current line
- n, N - continue search in same/opposite direction
- ;, \<literal comma\> - continue f/F/t/T character-search in same/opposite direction
- % - move to matching paren/quote (matched to character under cursor)
- [{, ]} - move cursor to opening/closing brace of current block
- [(, ]) - move cursor to opening/closing paren of current block
- [/, ]/ - move cursor to beginning/end of current c-style multi-line comment

### Special Motions (only valid when vim is expecting a motion)
- i\<quote or bracket\> - inner quote/bracket construct
- ip, is, iw - inner paragraph/sentence/word
- a\<quote or bracket\> - around quote/bracket construct
- ap, as, aw - around paragraph/sentence/word

### Actions
- x - delete character under cursor
- X - delete character to the left of cursor
- ~ - switch case of character under cursor, then move cursor to the right
- \<ctrl-a\> - increment integer under cursor
- \<ctrl-x\> - decrement integer under cursor
- s - delete character under cursor, enter insert mode
- D - delete from cursor to the end of the line
- C - delete from cursor to the end of the line, enter insert mode
- S - delete entire line, enter insert mode
- J - join current line with line below (space between)
- gJ - join current line with line below
- i, a - enter insert mode before/after cursor
- I, A - enter insert mode at beginning/end of current line
- o, O - create a new line below/above the current line, enter insert mode
- R - enter replace mode (type over text)
- P, p - put yanked text before/after cursor
- u - undo
- \<ctrl-r\> - redo
- cc - delete current line, then enter insert mode
- dd - delete current line
- yy - yank current line
- @@ - re-run last executed macro
- ZZ - save and quit
- ZQ - quit without saving
- : - enter command mode
- v - enter visual mode
- V - enter visual line mode
- \<ctrl-v\> - enter visual block mode
- . - repeat last command
- gv - re-select last visual selection
- gi - enter insert mode in position of last insert-mode-edit

### Parameterized Actions
- @\<register\> - execute macro in \<register\>
- m\<mark\> - set \<mark\> to cursor position
- q\<register\> - begin recording macro into \<register\>
- "\<register\>\<action\> - perform \<action\> on \<register\> (\<action\> can be 'x' for cut, 'y' for yank, 'p' for paste...)
- y\<motion\> - yank text
- d\<motion\> - delete text
- c\<motion\> - delete text, then enter insert mode
- r\<char\> - replace character under cursor with \<char\>
- \>\<motion\> - shift text right
- \<\<motion\> - shift text left
- =\<motion\> - smart-indent text
- gw\<motion\> - format text
- gq\<motion\> - format text, move cursor to the end of formatted text
- gU\<motion\>, gu\<motion\> - capitalize/un-capitalize text
- g?\<motion\> - filter given text through rot13
- /\<pattern\>\<enter\> - search forwards for \<pattern\>
- ?\<pattern\>\<enter\> - search backwards for \<pattern\>

## Insert Mode
- \<ctrl-v\>\<char\> - type \<char\> literally (e.g. don't expand tab)
- \<ctrl-t\>, \<ctrl-d\> - add/remove a tab at beginning of line
- \<ctrl-o\>\<normal mode command\> - execute given normal mode command (without leaving insert mode)
- \<ctrl-k\>\<two letters\> - insert digraph associated with given letters
- \<ctrl-a\> - insert same text as last insert
- \<ctrl-y\> - copy character from same column in line above
- \<ctrl-e\> - copy character from same column in line below
- \<ctrl-w\> - delete from cursor to start of last word
- \<ctrl-r\>\<char\> - insert contents of given register
- \<ctrl-n\>, \<ctrl-p\> - next/previous word in autocomplete

## Visual Mode (and Visual Line Mode)
- o, O - cursor to opposite side of block
- g \<ctrl-a\>, g \<ctrl-x\> - cascading increment/decrement of selected numbers
- \<parameterized action\> - execute action on selected text instead of supplying a motion
- \<other actions\> - roughly analagous to their corresponding action in normal mode

## Visual Block Mode
- o, O - cursor to opposite corner / side of selected block
- I\<text\>\<Esc\> - insert given text on left side of block on lines that are long enough
- A\<text\>\<Esc\> - insert given text on right side of the block, regaurdless of line-length
- C\<text\>\<Esc\> - change text from left of block to end of line on lines that are long enough
- $ - extend 'block' to the end of each line
- \<other actions\> - roughly analagous to their corresponding action in normal mode

## Key Combinations
- xp - transpose
- ea - append

## Marks
- ', ` - last jumped-from position
- . - last edited position
- " - position upon last file exit
- [ - start of last change
- ] - end of last change

# /usr/bin/vim

## Normal Mode
- zt, zz, zb - move screen position to align cursor near the top/center/bottom of screen
- \<ctrl-y\>, \<ctrl-e\> - move screen position up/down one line
- K - search manual for word under cursor
- gt - move one tab to the right
- gT - move one tab to the left
- !\<motion\>\<shell command\>\<enter\> - filter given text through \<shell command\>
- ga - display ascii value of character under cursor
- [I - list all lines in current file that contain word under cursor
- gD - goto definition (first occurrence in file) of word under cursor
- gf - goto file under cursor (close current file, open filename under cursor)
- g+, g- - go forward/backward one change in undo tree
- zm, zM, zr, zR - open/close some/all folds
- zi - open all folds / close all folds (toggle between)
- zd - delete fold under cursor
- zf\<motion\> - fold given text
- zo, zc - open/close fold under cursor
- dp, do - diff put / diff obtain - put/obtain text in diff mode (see vimdiff)

## Command Mode
- :q - quit (fail upon unsaved change)
- :w - write (save)
- :wq - write, then quit
- :(w)qa(ll) - (write and) quit all tabs/windows/buffers
- :q! - force quit without saving
- :w ! sudo tee % - force save
- :X - enable encryption
- :norm\<commands\> - execute given normal mode commands on selected (or current) line(s)
- :file \<filename\> - keep current buffer, but change save-file to \<filename\>
- :edit \<filename\> - change current buffer and save-file to \<filename\>
- :n, :p - next/previous buffer in queue
- set tabstop=\<n\> - set the tab-stop to \<n\>
- set shiftwidth=\<n\> - number of spaces shifted with '\>' and '\<'
- set expandtab - expand tabs to spaces (when typed)
- set textwidth=\<n\> (tw=\<n\>) - set the row-width (in columns) at which to wrap around
- :sp \<filename\> (split \<filename\>), new \<filename\> - open new horizontal window with \<filename\>
- :vsp \<filename\> (vslipt \<filename\>), vnew \<filename\> - open new vertical window with \<filename\>
- :sp, :vsp - open a duplicate buffer in a horizontal/vertical window
- \<ctrl-w\>+, \<ctrl-w\>- - increase/decrease height of current window by 1 row
- \<ctrl-w\>_ - maximize height of current window
- \<height>\<ctrl-w\>_ - set height of current window to \<height\> lines
- \<ctrl-w\>\<, \<ctrl-w\>\> - decrease/increase width of current window by 1 row
- \<ctrl-w\>| - maximize width of current window
- \<width\>\<ctrl-w\>| - set width of current window to \<width\> columns
- \<ctrl-w\>\<hjkl\> - move cursor between windows (directional)
- \<ctrl-w\>\<ctrl-w\> - move cursor between windows (cycle)
- \<ctrl-w\>\<HJKL\> - move current window
- \<ctrl-w\>q - close current window
- :close - like :q, but don't close the last tab/window
- :only - close all windows except the current one
- :tabnew \<filename\> - open a new tab with \<filename\>
- :tab \<cmd\> - execute \<cmd\> in a new tab
- :tabm \<n\> - move current tab \<n\> positions to the right
- :tabonly - close all tabs but the current one
- :set ignorecase - ignore case in search
- :set hlsearch - persistent highlight of search matches
- :noh - remove highlight from search matches
- :set incsearch - execute search as it is being typed
- :set list - highlight tabs/trailing spaces
- :terminal (:ter) - open virtual terminal in new window
- :set virtualedit=all - allow cursor to move over non-existent characters
- :help \<text\> - search manual for \<text\>, and open matched page in a window
- :jumps - list jumped-from/jumped-to locations
- :digraphs - list all digraphs
- :set foldmethod=indent - automatically folds the file based on indentation

## Searching
- //\<offset\> - repeat last search with given offset

### Relevant Regex
- \\\<, \\\> - match beginning/end of word
- \\d \\D - digit, non-digit
- \\x, \\X - hex, non-hex
- \\s, \\S - space, non-space
- \\l, \\L - lowercase letter / non-lowercase-letter
- \\u, \\U - uppercase letter / non-uppercase-letter
- \<most standard regular expressions\> - work as expected

### Offsets
- \<n\> - \<n\> lines below match
- b, b+\<n\>, b-\<n\> - beginning of match (+/- \<n\> characters)
- e, e+\<n\>, e-\<n\> - end of match (+/- \<n\> characters)

## Tools / Command Line Options
- vim -p \<files\> - open \<files\> in tabs
- vim -o \<tiles\> - open \<files\> in windows
- vimdiff \<a\> \<b\> - side-by-side comparison of differences in \<a\> and \<b\>
