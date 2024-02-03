---
title: "Scheme to JS"
subtitle: "A Scheme-to-JavaScript Transpiler (Haskell)"
date: 2024-02-03T09:27:32-06:00
---

### ([Github Link](https://github.com/lucasscharenbroch/scheme-to-js))

## Motivation

As a part of my dive into the functional rabbit-hole, I've read a lot of [SICP](https://en.wikipedia.org/wiki/Structure_and_Interpretation_of_Computer_Programs), which has a lot of big ideas[^big-ideas] about how programs execute and interract.
I decided to do this project in hopes of fleshing out what these ideas might look like in a non-trivial program, and seeing to what extent I could get such a program to eat itself.

[^big-ideas]: Big ideas include: procedures vs processes, immutability vs mutability, scoping/closures, and code as data.

JavaScript is the target language for two main reasons:
1. It's flexible => easy to get something working
2. If it's practical to transpile Scheme to anything, it's probably JavaScript

Haskell is the language of choice for transpilation becuase it's (comparatively) safe and fun to write parsers in, and I wanted to learn [Parsec](https://hackage.haskell.org/package/parsec).

## The Central Pipeline

The transpiler itself is a CLI program called **s2j**, which works a lot like other cli compilers (e.g. **gcc**).
Given source files as command-line-arguments, the below steps are followed.

- Read and lex each source file ("lexing" produces a list of tokens and a list of "preprocessor directives"[^preprocess])
- Recursively gather included source files, then sort them into a topological order
- For each tokenized file,
    - Parse ([Token] -> AST)
    - Optimize Tail-Calls (AST -> AST)
    - Generate (AST -> String)
- Concatenate the resulting code, along with the JS library/boilerplate (js-lib/core.js)
- Write the result to the output file

[^preprocess]: I'll use the term "preprocessing" for handling include-directives (e.g. **;#include std.scm**), but it's worth noting that this notion of "preprocessing" doesn't exactly match C's notion of preprocessing (C allows for much more advanced logic and textual substitution, while this program solely uses it for determing which files to include, and the order to include them).

## Generation

### Mangling

Scheme is pretty liberal when it comes to characters allowed in identifiers: it allows many characters that JS doesn't[^id-chars].
Furthermore, globally defined names could potentially share identifiers with names in compiled code[^global-scope].

Prefixing each name with "s2j_" fixes the latter problem, but the former problem is a little trickier, because all valid JS identifier characters are valid Scheme identifier characters, so there is no obvious escape character[^escape-underscore].

[^escape-underscore]: One way to solve this is to choose a valid identifier character (e.g. '_') as the designated escape character, then to use a sort of huffman-code to determine what the characters following _ mean (much like the use of '\' as an escape character).

Luckily, it turns out that Scheme literals are supposed to be case-insensitive, so Scheme names of varying case can (and should) be mapped deterministically to a single JS identifier.
The natural way to do this is to map all scheme identifiers to lowercase.
Now upper-case letters are free, and they can be used to represent special characters.

{{< highlight haskell >}}
mangleChar :: Char -> String
mangleChar '!' = "Bang"
mangleChar '$' = "Dollar"
mangleChar '%' = "Perc"
mangleChar '&' = "Amp"
mangleChar '*' = "Ast"
mangleChar '/' = "Slash"
mangleChar ':' = "Colon"
mangleChar '<' = "Lt"
mangleChar '=' = "Eq"
mangleChar '>' = "Gt"
mangleChar '?' = "Question"
mangleChar '~' = "Tilde"
mangleChar '^' = "Hat"
mangleChar '+' = "Add"
mangleChar '-' = "Sub"
mangleChar '.' = "Dot"
mangleChar c = [toLower c]

mangle :: String -> String
mangle id = "s2j_" ++ concatMap mangleChar id
{{< /highlight >}}


[^id-chars]: In addition to letters, (non-leading) digits, and underscores, "!$%&*/:<=>?~^+-." are allowed.

[^global-scope]: This could be fixed by adding a new scope around all transpiled code, but this adds unnecessary complexity.



### Wrapper Types

I opted to use wrapper-types instead of directly using native JS types types.
These (or some equivalent) are necessary for:

- Run-time arity checking
- Truthy Values (in Scheme, #f and '() are falsy, and everything else is truthy)
- Representing data types specific to scheme (e.g. symbols)

They also make printing, debugging, and ambiguity a lot easier to deal with.

{{< highlight javascript >}}
// js-lib/core.js

class SchemeObject {
    type = "object";

    constructor(val) { this.val = val; }
    truthy() { return true; }
    call() { throw new Error("application: not a procedure: " + this.print()); }

    and(other) {
        if(!this.truthy()) return this;
        else return other();
    }

    or(other) {
        if(this.truthy()) return this;
        else return other();
    }

    print() { return "" + this.val; }
}

class SchemeBool extends SchemeObject {
    type = "bool";

    truthy() { return this.val; }
}

class SchemeNil extends SchemeObject {
    type = "nil";

    truthy() { return false; }
    constructor() { super(null); }
    print() { return "nil"; }
}

class SchemeProcedure extends SchemeObject { type = "procedure"; /* ... */ }
class SchemePair extends SchemeObject { type = "pair"; /* ... */ }
class SchemeString extends SchemeObject { type = "string"; /* ... */ }
class SchemeVector extends SchemeObject { type = "vector"; /* ... */ }
class SchemeNum extends SchemeObject { type = "number"; }
class SchemeChar extends SchemeObject { type = "char"; }
class SchemeSymbol extends SchemeObject { type = "symbol"; }


{{< /highlight >}}

I couldn't find a way to get JS to use obj.truthy() to determine the truthiness in a short-circuited (&&)/(||) expression, but functions can be used as a work-around to this:

```
; source
(and 1 (print "test") 3)
```

```
// output
(new SchemeNum(1.0)).and(() => ((s2j_print).call(new SchemeString("test"))).and(() => new SchemeNum(3.0)));
```

### Everything is a Value

In Scheme, each expression is either a primitive, special form (special function call), or function call.
Each of these yields a value, and the latter two are constructed by operations on the values of their constituent expressions.
This consistency makes expression generation very simple: the output of the **genExpr** function is JS source whose value is the value resulting from that expression.


{{< highlight haskell >}}
genExpr :: Expression -> String
genExpr (ExprVar id) = mangle id
genExpr (ExprNumber num) = scmNum num
genExpr (ExprChar ch) = scmChar ch
genExpr (ExprString str) = scmString str
genExpr (ExprBool b) = scmBool b
genExpr (ExprQuotation datum) = genQuote datum
genExpr (ExprProcedureCall p args) = callMember (genExpr p) "call" (map genExpr args)
genExpr (ExprTailCall paramNames args) = call ("() => { " ++ intercalate "\n" [overwriteArgs, setRec, "}"]) []
    where overwriteArgs = "[" ++ intercalate ", " (map mangle paramNames) ++ "] = [" ++ intercalate ", " (map genExpr args) ++ "];"
          setRec = "rec = true;"
genExpr (ExprList exprs) = call "arr_to_list" ["[" ++ intercalate ", " (map genExpr exprs) ++ "]"]
genExpr (ExprLambda args body) = genLambda args body
-- (special forms)
genExpr (ExprIf cond conseq alt) = parenthesize (callMember cond' "truthy" []) ++ " ? " ++ conseq' ++ " : " ++ alt'
    where cond' = genExpr cond
          conseq' = genExpr conseq
          alt' = genExpr alt
genExpr (ExprAssignment id rhs) = mangle id ++ " = " ++ genExpr rhs ++ ";"
genExpr (ExprCond clauses) = intercalate "\n" (map genClause clauses ++ [scmNil])
    where genClause (CondIf cond conseq) = callMember (genExpr cond) "truthy" [] ++ " ? " ++ genSeq conseq ++ " : "
          genClause (CondElse conseq)  = "true ? " ++ genSeq conseq ++ " : "
          genSeq [] = scmNil
          genSeq seq = genBody $ Body [] seq
genExpr (ExprAnd args) = case args of
    [] -> scmBool True
    as -> foldr1 (\a b -> callMember a "and" ["() => " ++ b]) $ map genExpr as
genExpr (ExprOr args) = case args of
    [] -> scmBool True
    as -> foldr1 (\a b -> callMember a "or" ["() => " ++ b]) $ map genExpr as
genExpr (ExprLet bindings body) = call (mkLambda (map definitionName bindings) body) $ map (genExpr . definitionRhs) bindings
    where mkLambda args body = parenthesize (intercalate "," $ map mangle args) ++ " => " ++ genBody body
genExpr (ExprLetStar bindings body) = genExpr $ recurse bindings body
    where recurse [] body = ExprProcedureCall (ExprLambda (FormalArgList []) body) []
          recurse (d:ds) body = ExprProcedureCall (ExprLambda (FormalArgList [definitionName d]) (Body [] [recurse ds body])) [definitionRhs d]
genExpr (ExprLetRec outerBindings (Body innerBindings exprs)) = genBody $ Body (outerBindings ++ innerBindings) exprs
genExpr (ExprBegin exprs) = genBody $ Body [] exprs
{{< /highlight >}}

## Libraries

A set of standard functions[^std] is necessary to perform non-trivial operations on the wrapper types.
Implementing these functions in Scheme is much nicer than in JS, because the latter requires a lot of extra boilerplate (mangling, calling with ".call()", etc.), however some JS is necessary to provide the baseline functionality (Scheme on its own knows nothing about the wrapper types).
For this reason, I separated the "standard" functions into two sets: *js-lib/core.js*, which contains the wrappers themselves and a minimal set of standard functions, and *scm-lib/std.scm*, which defines the remainder of the "standard" functions.

[^std]: The "standard functions" I refer to are a subset of those from  the [Third Revised Report on Scheme](https://standards.scheme.org/official/r3rs.pdf) (those which I found the most useful). In general, this entire project is heavily guided by that report.

*core.js* is automatically included in all transpilations, while *std.scm* must be explicitly included.

{{< highlight javascript >}}
// js-lib/core.js

/* ... wrapper definitions ... */

/* ... */

// types

let s2j_booleanQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "bool"));
let s2j_pairQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "pair"));
let s2j_nullQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "nil"));
let s2j_symbolQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "symbol"));
let s2j_numberQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "number"));
let s2j_stringQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "string"));
let s2j_charQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "char"));
let s2j_vectorQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "vector"));
let s2j_procedureQuestion = new SchemeProcedure(1, false, x => new SchemeBool(x.type == "procedure"));

// conversion

let s2j_symbolSubGtstring = new SchemeProcedure(1, false, s => s.type != "symbol" ? err("symbol->string: expected symbol")
                                                                                  : new SchemeString(s.val));
let s2j_stringSubGtsymbol = new SchemeProcedure(1, false, s => s.type != "string" ? err("string->symbol: expected string")
                                                                                  : new SchemeSymbol(s.val));

let s2j_charSubGtinteger = new SchemeProcedure(1, false, c => c.type != "char" ? err("char->integer: expected char")
                                                                               : new SchemeNum(c.val.charCodeAt(0)));
let s2j_integerSubGtchar = new SchemeProcedure(1, false, i => i.type != "number" ? err("integer->char: expected integer")
                                                                                 : new SchemeChar(String.fromCharCode(i.val)));

/* ... */
{{< /highlight >}}

{{< highlight scheme >}}
; scm-lib/std.scm

;;;;; misc

(define (not b)
  (if b #f #t))

(define eqv? eq?) ; alias for eq?

(define (equal? x y) ; recursive eq? (pairs and vectors)
  (cond ((or (pair? x) (pair? y))
         (and (pair? y)
              (pair? x)
              (equal? (car x) (car y))
              (equal? (cdr x) (cdr y))))
        ((or (vector? x) (vector? y))
         (and (vector? x)
              (vector? y)
              (equal? (vector->list x) (vector->list y))))
        (else (eq? x y))))

; ...

;;;;; functional

(define (compose f g)
  (lambda (x) (f (g x))))

(define (foldl f init l)
  (cond ((null? l) init)
        ((not (pair? l)) (error "foldl: expected pair"))
        (else (foldl f (f init (car l)) (cdr l)))))

; ...
{{< /highlight >}}

## Evaluation

A downside to compilation is that the code-to-execution translation is removed from runtime.
It's not possible to have an *eval* function that re-uses the transpilation code (the Haskell program).

The beauty of Scheme (or Lisp in general) is that its syntax is so simple that it is trivially represented by primitive types.
An implementation for *eval* (which, with its helpers, is only abut 150 lines) is provided in *std.scm*.

{{< highlight scheme >}}

(define (eval expr env)
  (cond ((or (null? expr)
             (number? expr)
             (boolean? expr)
             (string? expr)
             (vector? expr)
             (char? expr)
             (procedure? expr))
         expr)
        ((symbol? expr) (lookup expr env))
        ((list? expr) (let ((sf (assq (car expr) special-forms)))
                        (if sf
                            ((cdr sf) (cdr expr) env)
                            (apply (eval (car expr) env)
                                   (eval-list (cdr expr) env)))))
        (else (error "eval: can't evaluate" expr)
              (error expr))))

;;; apply

(define (apply f args)
  (define (closure? x) (and (pair? x) (eq? (car x) 'closure)))
  (cond ((not (list? args)) (error "apply: expected list as second argument"))
        ((procedure? f) (apply-procedure f args))
        ((closure? f) (eval (cadadr f) (bind (caadr f) args (cddr f)))) ; eval function body in new env
        (else (error "apply: expected procedure or closure as first argument"))))

{{< /highlight >}}

This implementation is almost identical to the one from SICP, except it supports more syntax.

It's worth noting that, in the definition of *apply*, both *closure*s and regular *procedure*s are accepted as valid functions.
Closures are dynamically constructed (during evaluation), while procedures are assumed to be higher-level (in the scope from which the outermost *eval* is being called).
Closures themselves are simply a list of *'closure* (the symbol closure), a procedure, and an environment (*env*) (a list of association-lists of variable bindings in progressively higher scopes).

Transpiled code rides off of JavaScript's scoping, but there's no clear way to do this dynamically with the eval-apply model.
Thus we manually keep track of scope with *env*, but still allow for lookups/modifications in the JS scope when the *env* runs out.
This is admittedly an ugly kludge.

{{< highlight scheme >}}
(define (lookup sym env)
  (if (null? env)
      (js-eval-symbol sym) ; call `eval(sym.val)` in javascript (look up in the current JS scope)
      (let ((pair (assq sym (car env))))
           (if pair (cdr pair)
                    (lookup sym (cdr env))))))
{{< /highlight >}}

Other cons of using this model for evaluation:
- Difficult to debug[^debug]
- No tail-call optimization (because of the mutual recursion)
    - Only ~ 250 recursions before the call stack runs out (using node.js)

[^debug]: I spent a couple of hours dealing with problems that cascaded from my mistaken use of "car" instead of "cadr". This sort of thing doesn't happen in Haskell.

## Tail-Call Optimization

As with other functional languages, Scheme requires [tail-call optimization](https://en.wikipedia.org/wiki/Tail_call) to execute recursive functions of sizable depth without overflowing the call-stack.

This means changing something of the form (iterative process, recursive procedure):

{{< highlight javascript >}}
function add_nat(x, y) {
    if(x == 0) return y;
    else return add_nat(x - 1, y + 1);
}
{{< /highlight >}}

Into something like (iterative process, iterative procedure):

{{< highlight javascript >}}
function add_nat(x, y) {
    while(true) {
        if(x == 0) return y;
        x = x - 1;
        y = y + 1;
    }
}
{{< /highlight >}}

The key thing to note here is there must be some path of execution where the function's return value is a call to itself.
If that's the case, then the entire function-body can be surrounded in an infinite-loop, and the recursive call can be replaced by an overwriting-assignment to the arguments (re-using the stack-space for those arguments, and avoiding allocating another stack frame).

This rule doesn't quite work for the everything-is-a-value framework for generation, because tail-calls/return-values might be arbitrarily nested in other expressions.
For example, consider the transpilation of the Scheme equivalent to the above example.

{{< highlight scheme >}}
(define (add_nat x y)
  (if (= x 0)
      y
      (add_nat (- x 1) (+ y 1))))
{{< /highlight >}}

The (non-optimized) result:

{{< highlight javascript >}}

let s2j_add_nat = new SchemeProcedure(2, false, (s2j_x, s2j_y) => (() => {
    return (((s2j_Eq).call(s2j_x, new SchemeNum(0.0))).truthy()) ? s2j_y :
           (s2j_add_nat).call((s2j_Sub).call(s2j_x, new SchemeNum(1.0)), (s2j_Add).call(s2j_y, new SchemeNum(1.0)));
})());

{{< /highlight >}}

The outer "() => { return ... })());" is the result of generating the *body* of *add_nat*.
The return and curly brackets are unnecessary[^redundant] in this case (because there is only one expression in the body of *add_nat*), but they remain for generality.

[^redundant]: Redundancy and generality were favored over clarity of output.

It's not clear how the "s2j_y" and "(s2j_add_nat).call(...)" expressions can locally decide whether to return or not, because they are values, not statements.

An easy way around this is to avoid trying to use return-statements in the first place, and just set a local variable when tail-recursion should happen.
If the variable isn't set, then the value of the expression is returned.
It's not beautiful, but it works.

{{< highlight javascript >}}

let s2j_add_nat = new SchemeProcedure(2, false, (s2j_x, s2j_y) => (() => {
    while(true) {
        let rec = false;
        let res = (((s2j_Eq).call(s2j_x, new SchemeNum(0.0))).truthy()) ? s2j_y : 
                  (() => { [s2j_x, s2j_y] = [(s2j_Sub).call(s2j_x, new SchemeNum(1.0)), (s2j_Add).call(s2j_y, new SchemeNum(1.0))];
                           rec = true;
                         }
                  )();
    if(!rec) return res;
    }
})());

{{< /highlight >}}

## Testing

This projects' tests are entirely written in Scheme.
They mostly focus on testing the core/standard library: the fact that they run and return the expected result seems to be enough to show that the transpiler works.

Assertions make use of quoted expressions and *eval* to enable printing of failed assertions.
In this way, quotation is a lot like macros.

{{< highlight scheme >}}
; test/lib-test.scm

;#include std.scm

(define (assert pred)
  (if (eval pred '()) ; evaluate pred (a quoted expression) in the empty environment
    '()
     (error "assertion failed" pred))) ; prints the failed pred

(define (assert= x y)
  (assert (list 'equal? x y))) ; constructs a new expression: (equal? {x} {y}), and asserts it
{{< /highlight >}}

## IDEs, Tooling

I primarily used VSCode for this project (as opposed to /usr/bin/vim, which I've used on all of my previous work with Haskell), and it worked swimmingly.
The real-time type-checking by HLS (Haskell Language Server) practially eliminated compiler errors (which are almost always cryptic).
It probably cut my development time in half.

Hlint also fixed a lot of syntactic inconveinences in my code caused by my lack of knowledge of library functions.
