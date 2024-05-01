---
title: "Graphing Calculator"
subtitle: "Recursive descent parser and graphing tool (C++, WASM)"
notability: 3
date: 2023-03-25T18:14:48-05:00
---

## Note: this post was updated after a major refactor in 8/2023; [click here](/misc/graphing-calculator-original) for the original post.

<iframe src="https://lucasscharenbroch.github.io/graphing-calc-cas/"></iframe>

### ([Github Link](https://github.com/lucasscharenbroch/graphing-calc-cas))

I've used a variety of calculators over the years (namely for math homework), but none of them have everything I want. The TI-83/84 provides most of the desired functionality, but since it is a physical calculator, it comes with all sorts baggage: weight in backpack, battery (=> need to charge or buy batteries), expensive replacement cost, non-ergonomic button layout (any keyboard that isn't a keyboard is not ergonomic in my opinion), non-ergonomic user interface (manually setting the window, navigating menus), etc. As a result, even though I own a TI-84, I didn't think it was worth it to carry it around campus. Instead, I would use [Desmos](https://www.desmos.com/calculator) for graphing when I needed that, and I would type equations into the Google search bar for basic calculations. Desmos was convenient and good for graphing, but it didn't have integration with the numerical calculations, and though Google's calculator was surprisingly decent, it was not ideal for obvious reasons.

Even though I was dissatisfied with the options, I wanted surprisingly little:

- Command-line-like interface for numerical calculation (easy copy/paste/scroll from history)
- Convenient operators for programming (lg for log2, % for "modulus", scientific notation)
- Support for binary and hex literals
- Basic graphing and tracing
- Seamless integration between command line and graphing (i.e. "What does this function look like? I'll just graph it")

I realized that there is no reason I couldn't have it all in the browser. I also wanted it to be fast (and I prefer to avoid dealing with Javascript directly), so C++ compiled to WASM was the chosen toolkit.

## Recursive Descent Parsing
The goal of a calculator is to correctly evaluate mathematical expressions.
We'd like to take input in the form of a character-string (e.g. "2 + 3 * 4"), and output a real number.
However, it's not really obvious how the computer should start evaluating, and it's especially unclear how to evaluate in a way that obeys the order of operations.

Ideally, we'd like input in a form that has no ambiguity in order of evaluation.
That is, we want the corresponding equation tree to be trivially extractable from the input.

It turns out to be a little tricky to do this, and as a result, historic programs and calculators used [Reverse Polish Notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation), which uses a recursive "LEFT RIGHT OP" structure.

```
  Infix Notation                         Reverse-Polish Notation                         Equation Tree

    1 + 2 + 3                                   1 2 + 3 +                                      +
                                                                                              / \
                                                                                             +   3
                                                                                            / \
                                                                                            1 2


  (1 + 2) * (3 + 4)                           1 2 + 3 4 + *                                    *
                                                                                              / \
                                                                                             +   +
                                                                                            / \ / \
                                                                                           1  2 3  4

   1 / 2 ^ 3 ^ 4                               1 2 3 4 ^ ^ /                                  /
                                                                                             / \
                                                                                            1   ^
                                                                                               / \
                                                                                              2   ^
                                                                                                 / \
                                                                                                 3  4

```

Ultimately, the problem is converting Infix Notation into an equation tree.
Though I'm aware of a few different ways to approach this problem, such as using the [Shunting yard Algorithm](https://en.wikipedia.org/wiki/Shunting_yard_algorithm) or other parsing techniques/ using [yacc](https://en.wikipedia.org/wiki/Yacc),
[Recursive Descent Parsing](https://en.wikipedia.org/wiki/Recursive_descent_parser)
is my favorite by far because of its elegance and versitility.
The idea is simple: define the structure of an equation *entirely recursively*.
This can be done by defining a heierarchical structure of *nonterminal symbols* which are reduced to each other depending on their context in the equation; this structure is called the *grammar*.
If we set up this grammar in a very particular way, it can correctly reduce the input into equation trees that obey the order of operations.
It's kind of hard to explain without an example, so here's the grammar for the calculator.

```
S -> E$

E -> T = E                                  (T must be a variable or function)
  -> T {(+|-) T} [(==|!=|<|<=|>|>=) E]

T -> F {(*|/|//|%|<nothing>) F}

F -> [-]X [{(^|**) F]                       (don't parse negation when parsing implicit multiplication)

X -> (E)
  -> NUM
  -> VAR[{'}(ARGS)]

ARGS -> {E {, E {, ...}}}
```

*S,* *E*, *T*, *F*, *X*, and *ARGS* are all called *nonterminal symbols*: they're non-terminal because they correspond to groups of tokens that are already reduced.
In this case, the names of the nonterminals are abbreviations for Statement, Expression, Term, Factor, eXponential, and ARGument liSt, respectively.
The "->" represents a reduction, e.g. "S -> E$" means that "E$" reduces to "S".
Another way to think about it: when parsing an "S", since "S" only has one reduction, we must first parse an "E", then a $ (the end-of-string must be mached).
If "S" had multiple reductions, we would attempt to match each, in the order given.

*VAR* and *NUM* are *terminal symbols*: they represent variable and numeric constants respectively; note that input is "lexed" (the string is turned into a series of "tokens") before given to the parser, so the parser need not know exactly what makes up a variable or number: that can be defined elsewhere. They're called *terminals*, because they are parsed directly, not recursively (unlike non-terminals); more on this later.

The parenthesis/braces/brackets have special meaning here (with the exception of "X -> (E)", where the parenthesis are intended literally) in a similar way to regular expressions;
parenthesis are used for grouping, usually with the "|" operator, which implies that multiple symbols may be matched (e.g. "(+|-)" matches either "+" or "-").
"{...}" means "0 or more of ...", and "[...]" means "0 or 1 of ...".
This use of symbols is completely arbitrary: this grammar definition isn't a program, nor is it ever interpreted by a program (though there are some programs that can interpret grammars with a certain syntax), but rather a descripton of how to write the parser.

The remaining symbols ("+", "-", "'", "%", etc.) represent their literal values; these are also *terminal symbols* (like *VAR* and *NUM*).

From here, the algorithm is pretty straight-forward: it defines a recursive routine for each non-terminal (e.g. parseS(), parseE(), parseT(), etc...), which take (implicitly) the current index in the token-list, attempt to match each reduction, in order, by recursively calling other non-terminal routines or matching literal tokens with the input stream; the index in the toke-list is advanced accordingly, and the resuntant tree (or null, if no match is unsuccessful) is returned. Here's an example of such function from src/calc/parser.cpp with added comments.


{{< highlight cpp >}}
unique_ptr<TreeNode> parseT() {
    unique_ptr<TreeNode> lhs = parseF();
    if(lhs == nullptr) return nullptr;      // all valid terms must begin with an F (see the grammar),
                                            // so if there is no F, then there is no T.

    while(true) {                           // {(*|/|//|%|<nothing>) F}
        unique_ptr<Token>& op = next_tok(); // next_tok() is a helper defined exactly as follows:
                                            // unique_ptr<Token>& next_tok() {
                                            //     return i == tokens.size() ? eos : tokens[i++];
                                            // }
                                            // where i is the current position in the token stream

        if(&op == &eos) return lhs;         // reached end of input, but we have a valid match, so return it
        bool is_implicit = false;

        if(!op->is_op() ||                  // current token isn't an operand: perhaps a variable or number
           op->str_val() != "*"  &&
           op->str_val() != "/"  &&
           op->str_val() != "//" &&
           op->str_val() != "%") {
            unget_tok();                    // void unget_tok() {
                                            //     i--;
                                            // }
            is_implicit = true;
        }

        if(is_implicit) parsing_impl_mult = true;  // parsing_impl_mult is a global variable: it needs to be set
                                                   // to ensure that the binary '-' isn't interpreted as unary
                                                   // (2-3 != 2*-3); it needs to be saved and restored upon
                                                   // recursive calls to parseE, however, since 2(-3) = 2*-3.
                                                   // see the full source for more details
        unique_ptr<TreeNode> rhs = parseF();
        parsing_impl_mult = false;

        if(rhs == nullptr) {
            if(is_implicit) return lhs;
            else throw invalid_expression_error("expected operand after `" +  op->str_val() + "`");
        }
        lhs = unique_ptr<TreeNode> {new BinaryOpNode(std::move(lhs), std::move(rhs),       // combine lhs and rhs into
                                                     is_implicit ? "*" : op->str_val())};  // lhs; note that this
                                                                                           // preserves left-
                                                                                           // associativity.
    }
}
{{< /highlight >}}

The other parsing functions are similar in nature: the main idea is that they use recursive calls and keep close track of their state to ensure that the described structure is exactly preserved.

To parse the entire input string into an equation tree, parseS is called. The following are a few traces of the whole routine; hopefully they make clear why it's called "recursive descent".
Indentation signifies level of recursion.

```
input: "1 + 2 + 3"

parseS:
    parseE:
        parseT:
        |   parseF:
        |       first token is '-'? no: put it back
        |       parseX:
        |           return 1
        |       check next token: it is '+'; that's not '^' or '**', so return 1
        |   check next token: it is '+'; that's not '*' or '/' or '//' or '%', so assume implicit multiplication
        |   parseF:
        |       first token is '-'? no: put it back
        |       parseX:
        |           nothing matches '+', so return null
        |       got null from parseX, so return null
        |   got null from parseF, so there's no implicit multiplication after all; return 1
        match '+': expect another term
        parseT:
        |   // same as above, except "2" instead of "1"
        match "+": expect another term
        parseT:
        |   // same as abmove, "3" instead of "2"
        nothing left to match, so return the current tree (((1 + 2) + 3))
    match '$' is successful, so return the tree

```
```

input: "(1 + 2) * (3 + 4)"

parseS:
    parseE:
        parseT:
            parseF:
            |   first token is '-'? no: put it back
            |   parseX:
            |       matched '('
            |       parseE:
            |           parseT:
            |               parseF:
            |                   first token is '-'? no: put it back
            |                   parseX:
            |                       return 1
            |                   check next token: it is '+'; that's not '^' or '**', so return 1
            |               check next token: it is '+'; not '*' or '/' or '//' or '%'; assume implicit multiplication
            |               parseF:
            |                   first token is '-'? no: put it back
            |                   parseX:
            |                       nothing matches '+', so return null
            |                   got null from parseX, so return null
            |               got null from parseF, so there's no implicit multiplication after all; return 1
            |           match "+": expect another term
            |           parseT:
            |               // same as above, except "2" instead of "1"
            |           nothing left to match, so return the current tree ((1 + 2))
            |       match ')' is successful, so return the tree
            |   check next token: it is '*'; that's not '^' or '**', so return the current tree ((1 + 2))
            match '*': expect F
            parseF:
            |   // same as above, except (3 + 4) is returned instead of (1 + 2).
            nothing left to match, to return current treee ((1 + 2) * (3 + 4))
        nothing left to match, to return current treee ((1 + 2) * (3 + 4))
    match '$' is successful, so return the tree
```

Despite the simplicity of recursive-descent, it can be tough to implement: grammars are not very intuitive to write, and subtle differences can lead to unexpected behavior (see [this video by hhp3](https://www.youtube.com/watch?v=SToUyjAsaFk&t=1297s) for a good example of this).
One such subtlty is the difference between left-associative and right-associative operators, e.g. '+' and '^' respectively.
Left-associative operators have to be handled iteratively (keep checking if there is one more, e.g. "T -> {(+|-) T} ..."), while right-associative operators can be handled recursively, e.g. "X (^|\*\*) F"

Additionally, it's pretty much impossible to debug without writing the whole thing, since, as the name suggests, each parsing routine calls the one below it.
Any failure to put back a token when it was unsuccessfully matched can have unintended consequences and unexpected results because of the recursion involved (something might get messed up in a call, but it won't cause a problem until the stack unrolls a call or two).
I used *cout* more than I care to admit while debugging, and I spent way too long tracking down one or two forgotten calls to *unget_tok()*;
This is one case where I can really see the merit of a debugger.

## Object Oriented Programming


For simplicity, the parsing-functions for a numerical calculator could just return the real-number that results from the equation instead of the tree itself. This doesn't work very well in generality, though, and in order to have graphing and other desired functionality, the full tree must be returned.
I used this as an opportunity to apply some of those OOP skills (unique_ptr&lt;TreeNode&gt; => polymorphism!).
I also was able to use similar functionality for the Token (terminal symbol) classes.
This provides convenient ways to get strings and double values for any expression: the parent node (in general) doesn't need to know the type of its children; it can simply call eval() or to_string() to get the corresponding value from the child subtree.

{{< highlight cpp >}}
struct TreeNode { // Abstract superclass for all other node types
    virtual string to_string() = 0;
    virtual double eval() = 0;
    virtual bool is_var() { return false; }
    virtual ~TreeNode() { }
};

struct Token {
    virtual string to_string() = 0;
    virtual string str_val() { return ""; }
    virtual double dbl_val() { return NAN; }
    virtual bool is_var() { return false; }
    virtual bool is_num() { return false; }
    virtual bool is_op() { return false; }
    virtual ~Token() { }
};
{{< /highlight >}}

## Functions and Variables

All functions and variables are kept in a hashtable with their names (strings) as keys. These tables are separate, so a variable and function can share an identifier. Functions can be defined from the command-line with the straightforward notation: "f(x, y) = x + y + z". The parsed tree of the right-hand-side of the equal-sign is kept internally in the function table. When the function is called, the tree is evaluated as normal, except identifier lookups to the parameter names are caught and redirected to return the argument values. Any non-argument parameters are treated as usual variables, so in the case of f, z's value would be evaluate as if it were typed literally by the user.

Built-in functions are more straightforward, as their argument values can be passed directly into a backend function. I was able to make use of some [template metaprogramming](https://en.wikipedia.org/wiki/Template_metaprogramming) with the class "NDoubleFunction" (that, as you might expect, takes n doubles and returns 1 double).

## Macros

Some non-calculation functions are also defined for user-interaction with the backend (e.g. "graph", "ans", and "clear").
These functions have some funny implications when used in the same context as standard math functions (e.g. trying to graph the graph function; trying to take the derivative of the clear function, etc.).
I initially let these remain, since they don't usually come up by accident, but when revisiting this project to add algebraic manipulation, I needed a means of manipulating the tree before evaluating it. That is, given an input like "graph(deriv(x^2))", "deriv(x^2)" should be replaced by the symbolic derivative of "x^2" ("2x") before it is graphed (the symbolic derivative shouldn't be computed each time a point is drawn).

I thus fabricated the concept of a "macro": a function that changes its own tree-structure.
There's probably a better name for this than "macro", but the changing of syntax before execution kind of mimics the functionality of C's preprocessor-macros, and "macro" is a short term.
Macros are kind of hard to implement cleanly, since TreeNodes don't have pointers to their parents, and calling a macro almost certainly results in mutilating the tree-structure and removing the function call.
I eventually oped for adding a new function to TreeNode called *exe_macros()*; in which every non-function TreeNode simply replaces its children with *exe_macros()* called on the children (i.e. *left = left->exe_macros()*).
This conveniently fixes the problem of graphing the graph function and other silly tricks like that, because by making all special-functions macros that evaluate to NAN or something, we can ensure that we never graph a function without executing its macros and thus ensure that we never graph a special function.



## Math

Even though my degree's calculus requirements are behind me, I still wanted to program numerical derivative with [Lagrange's Notation](https://en.wikipedia.org/wiki/Notation_for_differentiation#Lagrange's_notation) instead of the ugly nderiv notation used by the TI-84. It was a simple addition to the parser, and it works on any built-in function (even the non-differentiable ones!).

Numerical differentiation is surprisingly trivial: the formula is: (f(x + step) - f(x - step)) / (2 * step), where step is some small constant, which is pretty much the definition of the derivative. Floating point rounding errors are an issue, of course, and in this case, I tried a few values of step to find one that worked sufficiently (1e-6). The step size can be changed from the command-line by changing the "DERIV_STEP" variable.

Higher-order derivatives don't seem to work well with this formula, however (floating point rounding errors magnify), so anything higher than a third-derivative is pretty much guaranteed to be garbage.

I also implemented numeric integrals, which are calculated as Midpoint Riemann Sums.

## Graphing

Graphing uses a similar parameter replacement procedure as user-defined functions, except that x is the only parameter, and it is always replaced.

The graphing algorithm is intuitive: imagine a vertical line being scanned along the graph window from left to right. The intersection of this vertical line with the graphed function at each horizontal pixel is plotted. The only issue with this is that steep curves become dotted lines if only one pixel is drawn. To prevent this, a vertical segment of pixels, with height equal to the change in y value (from the previous intersection), is drawn along the respective part of the vertical line. The other computations in the draw function are to determine the real x and y values (on the plane) correspond to the pixels.

{{< highlight cpp >}}
// draws graphed_functions[index] to graph_buffer
void draw(int index) {
    double old_x_value = get_id_value("x"); // x is used as the drawing variable - save its old val

    // x_c means "x on canvas" (uses int units), x_p means "x on plane" (uses float units)

    double x_ratio = (x_max - x_min) / (graph_width);
    double y_ratio = (graph_height) / (y_max - y_min);
    vector<int> y_c_vec; // holds the value of y_c at each x_c.

    for(int x_c = 0; x_c < graph_width; x_c++) {
        double x_p = x_min + (x_c * x_ratio);
        set_id_value("x", x_p);
        double y_p = graphed_functions[index]->eval();
        int y_c;
        if(isinf(y_p) || isnan(y_p)) y_c = INT_MAX;
        else {
            y_c = (y_p - y_min) * y_ratio;
            y_c = graph_height - y_c; // 0 = bottom => 0 = top
        }

        y_c_vec.push_back(y_c);
        draw_point_vector(y_c_vec, index); // draws the points in y_c_vec, connecting them (with vertical segments)
    }

    set_id_value("x", old_x_value);
}
{{< /highlight >}}
