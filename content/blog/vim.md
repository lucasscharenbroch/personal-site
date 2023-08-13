---
title: "Why I Use Vim"
subtitle: "How to edit text efficiently and why it matters"
date: 2023-08-13
---

## The Vim Lecture

I was initially hesitant to write a post about Vim, because most of what I have to say about it has already been said somewhere on the internet[^vim-videos].
My mind was recently changed when, in my [Programming III class](/blog/uw-madison-computer-science/#400), approximately 10 minutes of lecture were spent on "teaching Vim", which consisted entirely of using the *arrow keys* to move the cursor around, "i" for insert, escape to leave insert-mode, saving and quitting.
If I were one of the 299 other students in that lecture hall[^lec] who probably had no prior knowledge of Vim, I would have passed it off as a Nano[^nano] 2.0 that requires an extra keypress before you can start typing.
This is a shame, because there must have been at least one student (probably many) who would have had a much different attitude if they had seen what Vim could actually do.

[^vim-videos]: [These](https://www.youtube.com/watch?v=NzD2UdQl5Gc) [videos](https://www.youtube.com/watch?v=3yN6Q8I5KJA), [for](https://www.youtube.com/watch?v=X6AR2RMB5tE) [example](https://www.youtube.com/watch?v=IiwGbcd8S7I).

[^lec]: Okay, the number was probably more like 99, since the majority of students in that class (as with a lot of other CS classes) didn't attend lecture. Some of them may have seen the recording, though.

[^nano]: [GNU's Nano Text Editor](https://en.wikipedia.org/wiki/GNU_nano) is a dumbed-down (or perhaps "minimalist" if you prefer) terminal text editor, which had been our only method for editing text on the terminal prior to the Vim lecture.

In the defense of my professor, CS 400 covers a wide range of topics, and 10 minutes was probably close to the upper limit of the time that could have been allocated to Vim[^asm].
I don't think that 10 minutes is too little for a basic explanation *and* an honest demonstration, however.
Below is a transcription of an imaginary lecture that does just that.
For simplicity, I'll present it as if it were given in the same context as the CS 400 lecture that was actually given.

[^asm]: Assuming that the time (and subject) allocations for other lectures remain unchanged

## 10-Minute Demo

So far, we've used Nano to edit text from the terminal.
Now we will explore another program that does the same thing in a different way: Vim.
At its core, Vim is a text editor with bunch of key-bindings (kind of like keyboard shortcuts).
These shortcuts are designed to maximize editing capability while minimizing keystrokes.

{{< highlight bash >}}
lucas:~$ vim example.java
{{< /highlight >}}
*(Just like for Nano, passing a filename as an argument to Vim causes Vim to open that file)*

Vim is a "modal editor": the meaning of a pressed key depends on the "mode" that Vim is in.
"Normal-mode" is the default mode: pressing a key in normal-mode tells Vim to execute the command corresponding to that key (NOT to insert that character into the file).
For example, 'h', 'j', 'k', and 'l' are like the arrow keys for normal-mode (they move the cursor left, down, up, and right, respectively).
In "insert-mode", Vim works just like Nano does: it enters characters as you type.
Pressing 'i' in normal-mode enters insert-mode. Pressing the escape key in insert-mode returns Vim to normal-mode.

Normal-mode is the default mode for a reason: Vim is oriented towards efficient *editing*, and that implies the ability to move around the file and change text.
The typical way to do this in other text editors is to navigate with the mouse (or arrow keys) and use the mouse to highlight text and copy and paste it around the file.
Normal-mode allows you to do this entirely from the keyboard.
Once you're used to using normal-mode, the bindings become second nature, and using the mouse is comparatively annoying.

As I mentioned earlier, 'h', 'j', 'k', and 'l' are like the arrow keys for normal-mode.
There are a lot more ways to move the cursor, though.
'w' moves the cursor to the beginning of the next word, 'e' moves it to the end of the next word; 'b' moves it to the beginning of the last word.
"fx" ("find x") moves the cursor to the next instance of the character 'x' on the current line.
"fy" does the same thing, except for the character 'y', etc.
'0' moves to the beginning of the current line, '$' moves to the end of the line.

Other commands in normal-mode are associated with some action.
'd' means "delete", 'y' means "yank" (as in copy), and 'c' means "change".
Pressing these keys alone doesn't initially do anything, because Vim expects a motion key (one of the above cursor-movements) to be pressed immediately afterwards to specify *what* should be deleted/yanked/changed.
For example, "dw" deletes from the cursor up to the beginning of the next word. "c$" deletes the text from the cursor to the end of the line, and switches to insert-mode.
"yfx" yanks (copies) the text from the cursor's position to the next instance of 'x'.
'p' (paste) can then be used to paste that yanked text.

There are also special "motion" keys that have a special meaning when pressed after one of the above action commands.
For example, in most contexts pressing 'i' in normal-mode enters insert-mode, but when 'i' is pressed immediately after 'd', 'y', or 'c' (when Vim is expecting a motion), 'i' refers to the motion "in", meaning "apply this action 'in' some construct".
"di(" deletes the text in parenthesis (the cursor must be within the parenthesis).
"ci'" deletes the text in a single-quotes, and enters insert-mode.

You might be thinking "sure, that's pretty neat that you can do these things with your keyboard, but I can do them with my mouse almost as quickly."
This is true: *most of the time*, Vim will not save you more than a few seconds.
Vim's goal of "efficiency" doesn't just refer to *time saved*, though: you might instead think of it as *energy saved*.
Reaching for the mouse and making a precise highlight can get pretty tedious when you do it hundreds of times a day.
If you've ever used keyboard shortcuts (e.g. Ctrl-C, instead of right-click, copy), you know the joy of having an operation at your fingertips: it's not so much about the time saved, but rather that the operation can be done *immediately*, without thought, and devoted entirely to muscle-memory.
Vim is kind of like doing this in a bunch of little ways that combine to form an incredibly efficient method to edit text.

You may have noticed that many of the commands above (like '0', '$', and 'f') are generalized, and can do the same action on any line, regardless of its contents.
Vim's normal-commands turn out to naturally be a great programatic description of textual operations.
This opens the door for very easy use of macros.
The programmer can "record" commands on one location in a file, then apply that recording an arbitrary number of times in different locations around the file.
This is a case where Vim can actually save a significant amount of time[^sed].

[^sed]: It's worth noting that most things that can be done with Vim macros can be done just as well (if not better) with command-line text-manipulation utilities like Sed and AWK, but Vim allows these operations to be done directly in a file, with the familiar commands. It's also unlikely that an individual would know Sed and AWK but not know how to use Vim.

[ insert macro demonstration here if time > 10 mins ]

The functionality I've demonstrated here are a small subset of Vim's capabilities.
An easy way to learn more is to run the "vimtutor" command, which will open an interactive tutorial.

## *The Vim Editor* vs *The Vim Bindings*

I missed a key point in the above demonstration: how to exit Vim.
I did this to point out that my main goal isn't so much to teach *the Vim editor*: the point is to teach *the Vim bindings*.
The Vim editor (i.e. /usr/bin/vim) has its place as a convenient way to edit text from the terminal (this was the reason that Vim was a topic in CS 400 in the first place), but I think that the bindings themselves are the most important reason to teach Vim.
Personally, I love the /usr/bin/vim; I use it every day, and I'm familiar with a lot of functionality that is specific to the program (searching, command-mode, configuration, file i/o, registers, windows, tabs, etc.).
That being said, /usr/bin/vim is admittedly limiting when it comes to certain functionality (code completion (though I stand by CTRL-N and CTRL-P), debugging, filesystem navigation), which comes effortlessly with modern IDEs.

Luckily, there are plugins and settings that allow Vim bindings in almost every major IDE (even Emacs!); once you learn the bindings, there's little reason to ever stop using them.
