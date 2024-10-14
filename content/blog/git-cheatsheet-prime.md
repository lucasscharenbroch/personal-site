---
title: "Git Cheatsheet Prime"
subtitle: "An opinionated git CLI reference"
date: 2024-10-13T12:32:05-05:00
notability: 6
tags: ["Workflow"]
---

I'm an enthusiast and daily user of git's command line interface [^git-cli].
While GUIs offer advantages in visualization and intuition[^gui-better], they greatly fall short in power[^range], concision, and extensibility.

[^git-cli]: This is far from abnormal: I don't know the numbers, but I'd venture to say that git is the most popular version control system, and that a large percentage of developers use it through the command line (as opposed to some GUI).

[^gui-better]: (I'd even go as far as saying they're better for certain operations, like staging files and resolving merge conflicts).

[^range]: Namely range (sheer number) of possible operations. This is an inherent flaw of GUIs (and, conversely, an inherent power of CLIs): the screen space used for visualization (and the necessity of using visuals) directly impedes the speed and efficiency at which the user can communicate intent.
For example, most GUIs use tabs and dropdowns to allow a wide range of possible user inputs.
This is usually clumsy, and will never beat the speed of using a keyboard.
One possible solution to this is using keybindings with modifier keys (e.g. ctrl, alt, meta, ...).
Vim takes this a step farther by avoiding the modifiers; it's an extreme measure to allow the simultaneous use of visuals and rich  user input.

Like vim, learning to use git (or any other CLI tool with lots of input options) has a steep learning curve[^learning-curve] and many little shortcuts and conveniences that left out of tutorials and references[^references] (because they're not necessary for baseline usage), and thus never learned[^never-learned].

[^learning-curve]: As mentioned in the above footnote, the power of CLIs comes from the fact that they don't use visuals (the input is shaped in the mind of the user), so this learning curve is (in some ways) a necessity for power and speed of input.

[^references]: For example, the popular [github git cheat-sheet](https://education.github.com/git-cheat-sheet-education.pdf) (this post is meant to serve as an alternative to it, hence the title).

[^never-learned]: Or, more accurately, learned very slowly over time, with many remarks like "I can't believe I've been doing it this way all this time".

This post (much like [my vim one](/blog/vim-bindings)) includes the git subcommands (and options) I use on a regular basis, and their abbreviations (bash aliases) (which is how I use them).

I don't claim to be a master of git, and this list is far from complete, but it offers a wider range than the github one, and can be used as a richer starting point (for beginners) or as a sparse expansion pack (for everybody else).

I intend for it to evolve over time, as I uncover better ways of doing these things.

---

## Commands

### Commands With Abbreviations (The Most Frequent Ones)

(Key:

- `bash_alias` &mdash; `full_command` &mdash; explanation

)

- `gs` &mdash; `git status`  &mdash; ubiquitous
- `gd` &mdash; `git diff`  &mdash; diff
- `gds` &mdash; `git diff --staged`  &mdash; sanity-check after staging
- `g` &mdash; `git`  &mdash; saves two keystrokes for non-abbreviated commands
- `gb` &mdash; `git branch`  &mdash; ubiquitous
- `gbd` &mdash; `git branch -d`  &mdash; delete a branch
- `gbdf` &mdash; `git branch -D`  &mdash; force-delete a branch
- `gsw` &mdash; `git switch`  &mdash; switch to given branch
- `gswc` &mdash; `git switch -c`  &mdash; create a new branch and switch to it
- `gsh` &mdash; `git show`  &mdash; shows a commit's diff (...sometimes)
- `gc` &mdash; `git commit`  &mdash; list commit history
- `gca` &mdash; `git commit --amend`  &mdash; create a new commit off of `head^`, including the changes in `head`
- `gl` &mdash; `git log`  &mdash; ubiquitous
- `glg` &mdash; `git log --graph`  &mdash; ascii commit graph
- `glgo` &mdash; `git log --graph --oneline`  &mdash; ascii commit graph, minimal boilerplate
- `glmd` &mdash; `git log --remerge-diff`  &mdash; ascii commit graph, minimal boilerplate
- `ga` &mdash; `git add`  &mdash; ubiquitous
- `gai` &mdash; `git add -i`  &mdash; interactive add (add partial files)
- `gp` &mdash; `git push`  &mdash; ubiquitous
- `gpf` &mdash; `git push --force`  &mdash; set remote branch to match local
- `gf` &mdash; `git fetch`  &mdash; download (but don't merge) from remote
- `gpl` &mdash; `git pull`  &mdash; download and merge remote branch
- `gm` &mdash; `git merge`  &mdash; ubiquitous
- `grb` &mdash; `git rebase`  &mdash; ubiquitous
- `grbi` &mdash; `git rebase -i`  &mdash; interactive rebase
- `gcp` &mdash; `git cherry-pick`  &mdash; ubiquitous
- `gcpc` &mdash; `git cherry-pick --continue`  &mdash; continue after conflict/fixup/etc
- `grl` &mdash; `git reflog`  &mdash; list previous values of `HEAD`
- `gbs` &mdash; `git bisect`  &mdash; binary search to find when a bug was introduced
- `gr` &mdash; `git reset`  &mdash; ubiquitous
- `grh` &mdash; `git reset --hard`  &mdash; ubiquitous
- `grs` &mdash; `git restore`  &mdash; scrap unstaged changes
- `grss` &mdash; `git restore --staged`  &mdash; unstage changes
- `gmb` &mdash; `git merge-base`  &mdash; find the least-common-ancestor of two commits

### Other Commands

- `git init` &mdash; create new blank repository
- `git clone` &mdash; download repository from given url
- `git revert` &mdash; create a new commit that undoes the changes of the given commit
- `git blame` &mdash; list a file and the users/commits who last modified its lines
- `git stash [push|pop|list|drop|apply|...]` &mdash; temporary-commits
- `git rm` &mdash; remove a file, locally and in the git repository
- `git mv` &mdash; move a file, locally and in the git repository
- `git difftool` &mdash; use [vimdiff](https://vimdoc.sourceforge.net/htmldoc/diff.html) to visualize diffs
- `git mergetool` &mdash; use [vimdiff](https://vimdoc.sourceforge.net/htmldoc/diff.html) to resolve merge conflicts

## Idioms

- `git commit` &mdash; use the default behavior (enter message in text editor) instead of `-m`[^gcm]
- `git switch -` &mdash; switch to the last-switched-from branch
- `git switch -c <name> [<parent>]` &mdash; switch to a new branch off of `parent` (or `HEAD`, by default)
- `git reset --hard @~` &mdash; hard-reset to `HEAD`'s parent
- `git log -S <string>` &mdash; "Pickaxe": list commits whose diffs include the given `string`

[^gcm]: I feel pretty strongly about this, because it
(1) shows the files changed, branch, conventional commit message max line width,
(2) allows an abort,
(3) escapes special charcters, which would cause problems on the cli (even between double quotes), e.g. backtick,
(4) makes multi-line messages easier to write,
and (5) perhaps most importantly, is vim.

## Tricks

Locally ignore a file that is tracked by git:
- `git update-index --assume-unchanged <file>` &mdash; ignore changes to `file`
- `git update-index --no-assume-unchanged <file>` &mdash; undo the above

Setting up worktrees for a repository:
- (TODO)
- I haven't gotten into worktrees yet, or used them enough to flesh this out
- But it would be a crime not to mention them.
- See: [blog](https://matklad.github.io/2024/07/25/git-worktrees.html) [video](https://www.youtube.com/watch?v=2uEqYw-N8uE) [docs](https://git-scm.com/docs/git-worktree)

## Revision Specifiers

([the docs for this](https://git-scm.com/docs/gitrevisions) have a pretty low opportunity cost)

- `HEAD`[^head], `@` &mdash; `HEAD` (current "active" commit)
- `<rev>^<n>` &mdash; `n`th merge parent of `rev`
- `<rev>^` &mdash; parent of `rev`
- `<rev>~<n>` &mdash; `n`th generational parent of `rev`
- `<rev>~` &mdash; parent of `rev`

[^head]: (`HEAD` is case-insensitive; all-caps is used here for readability)

An elucidating example of `^` and `~` from the docs:
```
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A
A =      = A^0
B = A^   = A^1     = A~1
C =      = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```

### Ranges

- `<rev>^..<rev>` &mdash; inclusive range (kind of)[^cherry-pick-range]

[^cherry-pick-range]: Technically the `..` involves a set difference between reachable commits from the lhs and rhs, but I primarily use this for cherry-picking linear ranges of commits, so I think of it as a range (also note that the `^` is independent from the `..`).

---

The following **`git_alias`** function lives in my bashrc.
It sets up a bash abbreviation with git tab-completion.
There's probably a cleaner way to do it, but this works.

```bash
#!/bin/bash
git_alias() {
    alias "$1"="git $2"
    if (($# >= 3)) then
        __git_complete "$1" "$3"
    fi
}

git_alias g "" __git_main
git_alias gs status _git_status
git_alias gd diff _git_diff
git_alias gds "diff --staged" _git_diff
# ...
```
