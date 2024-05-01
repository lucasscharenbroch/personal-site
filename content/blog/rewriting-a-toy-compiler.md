---
title: "Rewriting a Toy Compiler"
subtitle: "Java to Haskell (-xx% LoC)"
date: 2024-04-25T07:59:41-05:00
draft: true
tags: ["Refactoring", "PL", "FP"]
---

- my 536 class is an intro to compilers
- the central project of the course is a compiler for a very simplified C-style language
- it uses lexer and parser generators (jlex and cup)
- the development of the compiler is dispensed in six parts, framed as projects
    - parts:
        - symbol table data structure
        - lexing
        - parsing
        - name analysis
        - type checking
        - code generation
    - each part is released, due, then graded, and the next part is released with the correct code for the last part
    - this basically means that, while the student technically has some control of the architecture on a single project, across all six, there is virtually no control over it in the long-run
    - so we have to use the java code provided, which is, in my opinion, complete and utter trash
    - the resulting code amounts to xxxx lines of java alone, and xxxx lines of lexer/parser generator code
    - there are null-checks, exceptions being thrown, and casting happening all over the place, and it's a complex mess
    - having written a few compilers in haskell recently, this made me curious just how much better it could be written in haskell, so I did just that

    - describe the haskell version (xxx lines of code, used parsec, since it's only fair, because the java code also used a library)

- walk through the differences
    - data structures!
    - lexing
    - parsing
    - name analysis
    - type checking
    - code generation
