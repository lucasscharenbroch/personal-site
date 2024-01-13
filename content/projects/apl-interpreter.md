---
title: "APL Interpreter"
subtitle: "A REPL for APL on the command-line (Haskell)"
date: 2024-01-11T12:44:11-06:00
---

### ([Github Link](https://github.com/lucasscharenbroch/apl-interpreter))

## Why APL?

[APL](https://en.wikipedia.org/wiki/APL_(programming_language)) is an *array* programming language[^name].
Its **only data type** is the (multidimensional) *array*.
While this might seem like a huge limitation, the generality it provides leads to a syntax that is incredibly compact and expressive, which forces the programmer to approach problems at a higher level.

[^name]: APL actually stands for "A Programming Language" (not "array programming language"), which was the name of a book by Ken Iverson in the 60's, back when the term "programming language" had a very different connotation.

I was first drawn to APL by the nature of its syntax: with the exception of user-defined variables, all built-in functions and operators are single[^single] unicode symbols.
As a result, code looks like this.

[^single]: Outer-product (∘.) (two symbols) is an exception

{{< highlight apl >}}
⍝ My solution to day 7 of Advent of Code '23

c2n ← '23456789TJQKA'∘⍳
classify ← (5-≢⍤∪),(⌈/(+/∘.=⍨)),((5≡∪)×(⌈/c2n))
h2n ← 13⊥classify,(¯1∘+c2n)

hands bids ← ↓⍉↑((' '∘≠)⊆⊢)¨ input
bids ← ⍎¨ bids

⎕ ← +/ (⍳≢bids) × bids[⍋ h2n¨ hands]

modes ← { ⍵≡⍬ : ⍬ ⋄ ∪⍵⌷⍨⊂(⍸⌈/=⊢)+/∘.=⍨⍵ }
change_joker ← { (⊃ 'A' ,⍨ modes (⍵~'J'))@(⍸⍵='J') ⊢ ⍵ }

c2n ← 'J23456789TQKA'∘⍳
h2n ← 13⊥(classify change_joker),(¯1∘+c2n)

⎕ ← +/ (⍳≢bids) × bids[⍋ h2n¨ hands]
{{< /highlight >}}

As you might expect, learning to write programs like this[^trad] requires a totally different mind-set.
Array programming is similar to functional programming -- the primary way to control execution involves composition of functions -- but APL tends to encourage the reliance on global properties and sweeping operations rather than low-level recursion[^fp-vs-ap].

[^fp-vs-ap]: Functional-style also uses global operations, but it encourages pattern-matching and recursive solutions; implementing such an approach in APL is like pulling teeth: it's possible, but annoying.

[^trad]: APL has a "traditional function" syntax that is more C-like, but this is not implemented in my interpreter.

I'd like to think that learning to approach problems this way provides greater insights for programming in general.

## Why Haskell?

I originally planned this project to be a deep-dive into APL, and learning Haskell was more of a side-quest.
It ended up the other way around: the hardest aspect of this project, by far, was learning work with Haskell[^hard].

[^hard]: More specifically, the core difficulty was learning to use a new paradigm (pure functional) to approach problems that would have been nontrivial to solve using the familiar toolkit (imperative). 

In all honesty, Haskell probably isn't an ideal tool for an array-language interpreter:
it makes parsing and combination of functions much more elegant, but at the expense of
ease of working with state, data structures, and performance.

As a result, this project isn't intended to be especially practically useful, nor to be a replacement for existing interpreters.

## The Big Picture

The program at large works exactly how you would expect any interpreter to work.

- Read Text As Input
- Convert Raw Text to Tokens (Lexing/Scanning)
- Convert Token Stream Into Syntax Tree (Parsing)
- Evaluate the parsed tree
- Print the result
- Repeat

The interpreter state (a mapping from variable names to values) is read/updated throughout.

## Parsing

Haskell has some powerful parsing libraries, but I decided to write my parser from scratch, partly to make sure I really understood what the code was doing, and partly because I was scared off by the type signatures[^type-signatures] of said libraries.
I initially attempted to write the parser in a similar style to those in my [other](/projects/bash-with-floats) [projects](/projects/graphing-calculator), but was forced into using a series of helper "match-functions" to translate the imperative code into a functional style.
Then, through a series of refactors, the majority of those helpers dissolved into calls to standard Haskell functions, resulting in syntax very similar to the aforementioned parsing libraries.

[^type-signatures]: Haskell documentation has a very steep learning curve.

### Parser, Version 1 (Context-Free)

The core unit of logic in the parser if the **`MatchFn`**. Its definition changed across the versions of the parser, but its main purpose did not: it takes a list of tokens as input, and possibly returns the thing that was matched, along with the new list of tokens.

{{< highlight haskell >}}
type MatchFn a = [Token] -> Maybe (a, [Token])
{{< /highlight >}}

**`MatchFn`**'s for [nonterminals](https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols) can now be trivially described.

{{< highlight haskell >}}
data Token = NumTok Double
           | StrTok String
           | IdTok String
           | ChTok Char

matchCh :: Char -> MatchFn Char
matchCh c (ChTok c':ts)
    | c == c' = Just (c, ts)
    | otherwise = Nothing
matchCh _ _ = Nothing

matchId :: MatchFn String
matchId (IdTok s:ts) = Just (s, ts)
matchId _ = Nothing

matchStrLiteral :: MatchFn String
matchStrLiteral (StrTok s:ts) = Just (s, ts)
matchStrLiteral _ = Nothing

matchNumLiteral :: MatchFn Double
matchNumLiteral (NumTok n:ts) = Just (n, ts)
matchNumLiteral _ = Nothing
{{< /highlight >}}
[^conv]

[^conv]: By convention, I used the prefix "match" for both terminal-matching functions and matching helper-functions, and the prefix "parse" for nonterminal-matching functions.

From here, it isn't really obvious how to combine the above functions to parse arbitrary nonterminals.
This is addressed by adding a few more helpers that combine (chain) **`MatchFn`**'s.


{{< highlight haskell >}}
matchOne :: [MatchFn a] -> MatchFn a
-- return first successful match, else Nothing
matchOne fns toks = foldl try Nothing fns
    where try (Just x) _ = Just x -- already found match
          try Nothing f = f toks

matchAll :: [MatchFn a] -> MatchFn [a]
-- match every function in list (sequentially), returning their results, else Nothing
matchAll fns toks = chFst (reverse) . foldl try (Just ([], toks)) $ fns
    where try Nothing _ = Nothing
          try (Just (rs, ts)) f = case f ts of
              Just (r, ts') -> Just(r:rs, ts')
              _ -> Nothing

matchMax :: [MatchFn a] -> MatchFn [[a]]
-- match 0 or more repetitions of the entire function list
matchMax fns toks = case matchAll fns toks of
    Nothing -> Nothing
    Just (r, ts) -> case matchMax fns ts of
        Nothing -> Just ([r], ts)
        Just (rs, ts') -> Just (r:rs, ts')

matchAllThenMax :: [MatchFn a] -> MatchFn [[a]]
-- ... similar definition
{{< /highlight >}}

The **`chFst`** function is a convenient way to conditionally apply an arbitrary function to the result of a MatchFn (**`Maybe (a, [Token])`**) without doing a case-analysis.


{{< highlight haskell >}}
chFst :: (a -> b) -> Maybe (a, c) -> Maybe (b, c)
-- chain first: apply function to first element of maybe-wrapped tuple
chFst f m = case m of
    Nothing -> Nothing
    Just (x, y) -> Just (f x, y)

{{< /highlight >}}

There's also another variant, **`mchFst`**, which allows the chaining-function to fail (return Nothing), in which case the entire construct also becomes nothing.

{{< highlight haskell >}}
mchFst :: (a -> Maybe b) -> Maybe (a, c) -> Maybe (b, c)
-- maybe chain first
mchFst f m = case m of
    Nothing -> Nothing
    Just (x, y) -> case (f x) of
        Nothing -> Nothing
        Just z -> Just(z, y)
{{< /highlight >}}

All of the above helper match-functions only deal with combining homogeneous **`MatchFn`**'s (i.e. **`MatchFn Int`** and **`MatchFn Int`**, not **`MatchFn String`** and **`MatchFn Int`**).
Tuples are one solution to this.

{{< highlight haskell >}}
matchT2 :: (MatchFn a, MatchFn b) -> MatchFn (a, b)
matchT2 (fa, fb) ts = case fa ts of
    Nothing -> Nothing
    Just (a, ts') -> case fb ts' of
        Nothing -> Nothing
        Just (b, ts'') -> Just ((a, b), ts'')

matchT3 :: (MatchFn a, MatchFn b, MatchFn c) -> MatchFn (a, b, c)
matchT3 (fa, fb, fc) ts = case fa ts of
    Nothing -> Nothing
    Just (a, ts') -> case fb ts' of
        Nothing -> Nothing
        Just (b, ts'') -> case fc ts'' of
            Nothing -> Nothing
            Just (c, ts''') -> Just ((a, b, c), ts''')

matchT4 :: (MatchFn a, MatchFn b, MatchFn c, MatchFn d) -> MatchFn (a, b, c, d)
-- ... etc.
{{< /highlight >}}

These helpers alone make for a relatively elegant[^efficient] parser, with syntax kind of similar to [BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) (or at least a lot closer than an average imperative implementation).

[^efficient]: Albeit not necessarily the most efficient.

{{< highlight haskell >}}
-- der_arr => train der_arr
--         => arr der_fn der_arr
--         => arr

parseDerArr :: MatchFn ArrTreeNode
parseDerArr = matchOne [
        chFst (\(t, da) -> ArrInternalMonFn t da) . matchT2 (parseTrain, parseDerArr),
        chFst (\(lhs, f, rhs) -> ArrInternalDyadFn f lhs rhs) . matchT3 (
            parseArr,
            parseDerFn,
            parseDerArr
        ),
        parseArr
    ]
{{< /highlight >}}
[^full-v1]

[^full-v1]: (Full code for Parser
[version 1](https://github.com/lucasscharenbroch/apl-interpreter/tree/e6a0bb3d3577b19321fe5ddd9fc56ab022b855e9/src/Parse.hs),
[version 2](https://github.com/lucasscharenbroch/apl-interpreter/blob/653cf82481be1ef23ec931745184fde11b1c8313/src/Parse.hs),
[version 3](https://github.com/lucasscharenbroch/apl-interpreter/blob/89d5d67867b16819aedb7935bbaed141a7d76b96/src/Parse.hs)
[version 4](https://github.com/lucasscharenbroch/apl-interpreter/blob/e2ed6f655f88ec9c94bb1f0bbe0c6455dff3cb75/src/Parse.hs)
)

### Parser, Version 2 (+Context)
It turns out that APL doesn't have a [context-free grammar](https://en.wikipedia.org/wiki/Context-free_grammar).[^cfg]
This means that it requires more information than the tokens alone to form the syntax tree, namely the values of variables.

[^cfg]: This is because any variable name could be one of the following: {array, function, monadic operator, dyadic operator}, each of which have different implications during parsing.

This foils the current definition of **`MatchFn`**: because Haskell is purely functional, **`MatchFn`**'s must solely deal in their arguments[^comb] (otherwise they have side-effects), so the global state (**`IdMap`**) must be added as an argument to **`MatchFn`**. Parsing shouldn't modify the **`IdMap`**, so the return value can remain the same.

[^comb]: And in pure functions applied to their arguments.

{{< highlight haskell >}}
type MatchFn a = (IdMap, [Token]) -> Maybe (a, [Token])
{{< /highlight >}}
[^bundle]

[^bundle]: The choice to bundle **`IdMap`** and **`[Token]`** into a tuple is totally arbitrary: the main motivation to do this is ease of composition (one-argument functions are more natural to compose). **`MatchFn a`** could jut as well be **`IdMap -> [Token] -> Maybe (a, [Token])`**.

The beauty of this change is that only the terminal matching functions and the matching helpers (only the functions with "match" as a prefix) need to be changed -- this is because the nonterminal parsing ("parse-") functions never actually directly touch to the arguments to MatchFn: they solely serve as *definitions of combinations* of helper functions[^data-abstraction].

[^data-abstraction]: This can be though of as a *[data abstraction](https://en.wikipedia.org//wiki/Abstraction_(computer_science)#Data_abstraction)*, with functions (**`MatchFn`**'s) as the data being abstracted. Pretty neat.

The "match-" functions can be easily updated, with some minor syntactic inconvenience.

{{< highlight diff >}}

 matchCh :: Char -> MatchFn Char
-matchCh c (ChTok c':ts)
+matchCh c (_, (ChTok c':ts))
     | c == c' = Just (c, ts)
     | otherwise = Nothing
 matchCh _ _ = Nothing


 matchStrLiteral :: MatchFn String
-matchStrLiteral (StrTok s:ts) = Just (s, ts)
+matchStrLiteral (_, (StrTok s:ts)) = Just (s, ts)
 matchStrLiteral _ = Nothing

 -- ... etc.

 matchOne :: [MatchFn a] -> MatchFn a
 -- return first successful match, else Nothing
-matchOne fns toks = foldl try Nothing fns
+matchOne fns args = foldl try Nothing fns
     where try (Just x) _ = Just x -- already found match
-          try Nothing f = f toks
+          try Nothing f = f args

 matchAll :: [MatchFn a] -> MatchFn [a]
 -- match every function in list (sequentially), returning their results, else Nothing
-matchAll fns toks = chFst (reverse) . foldl try (Just ([], toks)) $ fns
+matchAll fns (idm, toks) = chFst (reverse) . foldl try (Just ([], toks)) $ fns
     where try Nothing _ = Nothing
-          try (Just (rs, ts)) f = case f ts of
+          try (Just (rs, ts)) f = case f (idm, ts) of
               Just (r, ts') -> Just(r:rs, ts')
               _ -> Nothing

 -- ... etc.

 matchT2 :: (MatchFn a, MatchFn b) -> MatchFn (a, b)
-matchT2 (fa, fb) ts = case fa ts of
+matchT2 (fa, fb) (idm, ts) = case fa (idm, ts) of
     Nothing -> Nothing
-    Just (a, ts') -> case fb ts' of
+    Just (a, ts') -> case fb (idm, ts') of
         Nothing -> Nothing
         Just (b, ts'') -> Just ((a, b), ts'')
 -- ... etc.
{{< /highlight >}}

### Parser, Version 3 (+Monads)

Throughout writing and refactoring the parser, while I was generally happy with its functionality, there were three main inconveniences that bothered me.

1. The existence of the **`parseT`** functions: they are too hard-coded. There must be a more general way to fix this.
2. The existence and syntactic clumsiness of **`chFst`** and **`mchFst`**.
3. **`parseDerFn`**

The majority of the "parse-" functions were relatively concise and readable, but there was one particular function that had a heavy reliance on the global state (IdMap) that made its implementation monstrous.

{{< highlight haskell >}}
-- der_fn => (f|a) op [f|a] {op [f|a]}       (where (f|a) is fn or arr; match the
--                                            optional iff op is dyadic)

parseDerFn :: MatchFn FnTreeNode
parseDerFn (idm, ts) = matchOne [
        (=<<) (parseDerFnRec) . (=<<) (finishOpMatch) . matchT2 (_parseArg, parseOp),
        parseFn
    ] (idm, ts)
    where _parseArg = matchOne [parseFn, chFst (FnLeafArr) . parseArr]
          finishOpMatch :: ((FnTreeNode, Operator), [Token]) -> Maybe (FnTreeNode, [Token])
          finishOpMatch ((lhs, op@(DyadOp _ _)), toks) = chFst (FnInternalDyadOp op lhs) $ _parseArg (idm, toks)
          finishOpMatch ((lhs, op), toks) = Just (FnInternalMonOp op lhs, toks)
          parseDerFnRec :: (FnTreeNode, [Token]) -> Maybe (FnTreeNode, [Token])
          parseDerFnRec (lhs, toks) = case (=<<) (finishOpMatch) . chFst (\op -> (lhs, op)) $ parseOp (idm, toks) of
              Nothing -> Just (lhs, toks)
              Just res -> parseDerFnRec res
{{< /highlight >}}

**`parseDerFn`** is the exception to the above statement that "parse-" functions never directly manipulate the argument to **`MatchFn`** (**`(IdMap, [Token])`**).
**`parseDerFn`** breaks this rule because it needs to have the **`IdMap`** to check whether the parsed operator is monadic or dyadic.
This can't be done with **`chFst`** or **`mchFst`**, because neither takes the **IdMap** as an argument, which **`finishOpMatch`** requires.

At this point, I had already realized the similarity between **`mchFst`** and **`(=<<)`**.

{{< highlight haskell >}}
(=<<) :: (a -> m b) -> m a -> m b
-- (let a = (a, c); b = (b, c); m = Maybe in the definition of (=<<))
-- (=<<) :: ((a, c) -> Maybe (b, c)) -> Maybe (a, c) -> Maybe (b, c)
-- let c = [Token]
-- (=<<) :: ((a, [Token]) -> Maybe (b, [Token])) -> Maybe (a, [Token]) -> Maybe (b, [Token])

mchFst :: (a -> Maybe b) -> Maybe (a, c) -> Maybe (b, c)
-- let c = [Token]
-- mchFst :: (a -> Maybe b) -> Maybe (a, [Token]) -> Maybe (b, [Token])

-- (=<<) ::  ((a, [Token]) -> Maybe (b, [Token])) -> Maybe (a, [Token]) -> Maybe (b, [Token])
-- mchFst :: (a            -> Maybe b           ) -> Maybe (a, [Token]) -> Maybe (b, [Token])
{{< /highlight >}}

So **`mchFst`** is just a specialized version of **`(=<<)`** that assumes that the token-list remains unchanged.

I decided I ought to try to figure out what other patterns in my parser were mimicking those of built-in monadic functions[^why], and attempt to refactor accordingly.

[^why]: Using built-in functions is not only more concise, but it also forces increased flexibility and generality, along with a combinatorial explosion of library functions that need not be-written.

**`chFst`** and **`mchFst`** could not directly be substituted for monadic functions, because they heavily rely on the knowledge that the value they receive is a tuple (wrapped in a Maybe), and that the first element of that tuple is the one that gets modified, while the second element (the **`[Token]`**) stays the same[^the-same].

[^the-same]: The **`[Token]`** stays the same through calls of chFst and mchFst, but it is modified by other functions.

The key point here is that **`chFst`** and **`mchFst`** are *not general enough*. Their main use is to (in conjunction with composition) convert a **`MatchFn`** parameterized by one type to a **`MatchFn`** parameterized by another (see below).
But they assume that that function (**`f`**) only deals with the type parameter of **`MatchFn`** (**`a`**/**`b`**), and has no effect on the **`[Token]`**, and cannot read the **`IdMap`**.

{{< highlight haskell >}}
-- (MatchFn from version 2)
type MatchFn a = (IdMap, [Token]) -> Maybe (a, [Token])

mfFmap  f = (.) (chFst  f) :: (a ->       b) -> MatchFn a -> MatchFn b
mfFBind f = (.) (mchFst f) :: (a -> Maybe b) -> MatchFn a -> MatchFn b
{{< /highlight >}}
[^fmap-fliped-bind]

[^fmap-fliped-bind]: These functions (mfFmap, mfFBind) weren't used in the parser, because their definitions concise enough to use inline. They're laid out here to display their types.

This lack of generality is clearly the cause of inconvenience #3.

One solution to this is to expand the definition of **`MatchFn`** to also return an IdMap (so chained functions can access it), and to write a few new chaining helper-functions that allow access to different parts of the returned tuple.

{{< highlight haskell >}}
type MatchFn a = (IdMap, [Token]) -> Maybe (a, [Token], IdMap)

mfFmap            :: (         a ->         b) -> MatchFn a -> MatchFn b
mfFBindMaybe      :: (         a -> Maybe   b) -> MatchFn a -> MatchFn b
mfFBindIdm        :: (IdMap -> a ->         b) -> MatchFn a -> MatchFn b
mfFBindIdm'       :: (IdMap -> a -> MatchFn b) -> MatchFn a -> MatchFn b
mfFBindIdmMaybe   :: (IdMap -> a -> Maybe   b) -> MatchFn a -> MatchFn b
mfFBind           :: (         a -> MatchFn b) -> MatchFn a -> MatchFn b
{{< /highlight >}}

This isn't a very good solution, though, as it adds a lot of boilerplate (much of which is rarely used), and it doesn't fix the other inconveniences.
Additionally, by adding **`IdMap`** to the return type, it introduces the (albeit unlikely) possibility that a parsing function may modify the **`IdMap`**.

A better solution is to change **`MatchFn`** into a monad whose functions exactly match the desired behavior, then to use generic monadic functions instead of chFst and mchFst.


{{< highlight haskell >}}
type MatchFn a = StateT [Token] (MaybeT (Reader IdMap)) a
{{< /highlight >}}

It's probably a little steep to try to completely explain this type, so I'll keep it high-level.
The names that end with 'T' are [Monad Transformers](https://en.wikipedia.org/wiki/Monad_transformer), which take another monad as one of their type parameters, to form a single resulting monad. Each of the following is a monad.

- **`Reader IdMap`**
- **`MaybeT (Reader IdMap)`** 
- **`StateT [Token] (MaybeT (Reader IdMap))`**

Monad transformers preserve the behavior of the monad they receive, and add their own behavior on top[^lift].

[^lift]: Functions can be applied on successively deeper monads by applying [lift](https://hackage.haskell.org/package/transformers-0.5.6.2/docs/Control-Monad-Trans-Class.html#v:lift).

It's interesting to note that **`MatchFn`** itself is no longer a function: it is a monad whose state internal state is a function[^internal-state-fn].
[^internal-state-fn]: More specifically, its internal state is the function **`[Token] -> MaybeT (Reader IdMap) (a, [Token])`**, where the return type encapsulates a value: **`Reader IdMap (Maybe (a, [Token]))`**, which encapsulates the function: **`IdMap -> Maybe (a, [Token])`**. So, in effect, **`MatchFn`** stores a function of type **`[Token] -> IdMap -> Maybe (a, [Token])`**, which its interface provides fine-grained control over. It's worth noting that this effective state is isomorphic to **`MatchFn`** from version 2.


This requires another reworking of the "match-" functions, but the "parse-" functions can (again) stay mostly the same, except **`(<$>)`** (fmap) replaces **`chFst`**, and **`(=<<)`** (or **`(>>=)`** or do-notation) replaces **`mchFst`**.

{{< highlight diff >}}
+evalMatchFn :: IdMap -> [Token] -> MatchFn a -> Maybe (a, [Token])
+evalMatchFn idm toks = (flip runReader) idm . runMaybeT . (flip runStateT) toks

+maybeMatch :: MatchFn a -> MatchFn (Maybe a)
+maybeMatch f = do
+    idm <- getIdm
+    toks <- get
+    case evalMatchFn idm toks f of
+        Nothing -> return Nothing
+        Just (x, toks') -> do
+            put toks'
+            return $ Just x

 matchOne :: [MatchFn a] -> MatchFn a
 -- return first successful match, else Nothing
-matchOne fns args = foldl try Nothing fns
-    where try (Just x) _ = Just x -- already found match
-          try Nothing f = f args
+matchOne [] = mzero
+matchOne (f:fs) = do
+    mb <- maybeMatch f
+    case mb of
+        Nothing -> matchOne fs
+        Just x -> return x

 matchAll :: [MatchFn a] -> MatchFn [a]
--- match every function in list (sequentially), returning their results, else Nothing
-matchAll fns (idm, toks) = chFst (reverse) . foldl try (Just ([], toks)) $ fns
-    where try Nothing _ = Nothing
-          try (Just (rs, ts)) f = case f (idm, ts) of
-              Just (r, ts') -> Just(r:rs, ts')
-              _ -> Nothing
+matchAll [] = return []
+matchAll (f:fs) = do
+    a <- f
+    as <- matchAll fs
+    return $ a:as

 -- ... etc.

 matchT2 :: (MatchFn a, MatchFn b) -> MatchFn (a, b)
-matchT2 (fa, fb) (idm, ts) = case fa (idm, ts) of
-    Nothing -> Nothing
-    Just (a, ts') -> case fb (idm, ts') of
-        Nothing -> Nothing
-        Just (b, ts'') -> Just ((a, b), ts'')
+matchT2 (fa, fb) = do
+    a <- fa
+    b <- fb
+    return (a, b)

 -- .. etc.

 parseOpOrFn :: MatchFn (Operator, FnTreeNode)
 parseOpOrFn = matchOne [
-        chFst (\_ -> (oReduce, FnLeafFn fReplicate)) . matchCh '/',
-        chFst (\_ -> (oScan, FnLeafFn fExpand)) . matchCh '\\',
-        chFst (\_ -> (oReduceFirst, FnLeafFn fReplicateFirst)) . matchCh '⌿',
-        chFst (\_ -> (oScanFirst, FnLeafFn fExpandFirst)) . matchCh '⍀'
+        (\_ -> (oReduce, FnLeafFn fReplicate)) <$> matchCh '/',
+        (\_ -> (oScan, FnLeafFn fExpand)) <$> matchCh '\\',
+        (\_ -> (oReduceFirst, FnLeafFn fReplicateFirst)) <$> matchCh '⌿',
+        (\_ -> (oScanFirst, FnLeafFn fExpandFirst)) <$> matchCh '⍀'
     ]

 parseArr :: MatchFn ArrTreeNode -- parse an entire literal array
-parseArr = chFst (_roll) . matchT2 (
+parseArr = (_roll) <$> matchT2 (
         parseArrComp,
-        chFst (concat) . matchMax [ matchOne [
-            chFst (Left) . parseArrComp,
-            chFst (\(_, il, _) -> Right il) . matchT3 (
+        (concat) <$> matchMax [ matchOne [
+            (Left) <$> parseArrComp,
+            (\(_, il, _) -> Right il) <$> matchT3 (
                 matchCh '[',
                 parseIdxList,
                 matchCh ']'
             )
         ]]
     )

 -- .. etc.
{{< /highlight >}}

Evaluating **`mzero`** in the monad at any time causes the internal function to unconditionally return Nothing (effectively short-circuit).
This behavior comes from **`MaybeT`**.

The new version of parseDerFn:

{{< highlight haskell >}}
parseDerFn = matchOne [_parseOpExpr, parseFn]
    where _parseOpExpr = do
              lhs <- _parseArg
              otn <- parseOp
              _parseOpExprRec lhs otn
          _parseArg = matchOne [parseFn, FnLeafArr <$> parseArr]
          _parseOpExprRec :: FnTreeNode -> OpTreeNode -> MatchFn FnTreeNode
          _parseOpExprRec lhs otn = do
              df <- case unwrapOpTree otn of
                        (MonOp _ _) -> return $ FnInternalMonOp otn lhs
                        (DyadOp _ _) -> do rhs <- _parseArg
                                           return $ FnInternalDyadOp otn lhs rhs
              mb <- maybeMatch parseOp
              case mb of
                   Nothing -> return $ df
                   (Just otn2) -> _parseOpExprRec df otn2
{{< /highlight >}}

### Parser, Version 4 (+Applicative)

My favorite part about Haskell: realizing a built-in function does precisely what you want.

It turns out that, [circa 2014](https://wiki.haskell.org/Functor-Applicative-Monad_Proposal), all Monads in Haskell are also [Applicative Functors](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Applicative.html).
**`(<*>)`**, **`(<*)`**, and **`(*>)`** remove the need for the **`MatchT`** functions, and the vast majority of lambdas (typically as the LHS's of **`<$>`**).

{{< highlight diff >}}
 parseDerArr :: MatchFn ArrTreeNode
 parseDerArr = matchOne [
         parseArrAss,
-        (\(f, da) -> ArrInternalMonFn f da) <$> matchT2 (parseDerFn, parseDerArr),
-        (\(lhs, f, rhs) -> ArrInternalDyadFn f lhs rhs) <$> matchT3 (
-            parseArr,
-            parseDerFn,
-            parseDerArr
-        ),
+        ArrInternalMonFn <$> parseDerFn <*> parseDerArr,
+        (flip ArrInternalDyadFn) <$> parseArr <*> parseDerFn <*> parseDerArr,
         parseArr
     ]

 parseArrAss :: MatchFn ArrTreeNode
 parseArrAss = matchOne [
-        (\(id, _, da) -> ArrInternalAssignment id da) <$> matchT3 (
-            matchId,
-            matchCh '←',
-            parseDerArr
-        ),
-        (\(id, df, _, da) -> ArrInternalModAssignment id df da) <$> matchT4 (
-            matchId,
-            parseDerFn,
-            matchCh '←',
-            parseDerArr
-        )
+        ArrInternalAssignment <$> matchId <*> (matchCh '←' *> parseDerArr),
+        ArrInternalModAssignment <$> matchId <*> parseDerFn <*> (matchCh '←' *> parseDerArr)
     ]

 -- ... etc.
{{< /highlight >}}

Version 4 is over 100 lines shorter than version 3.

## Evaluation

This project was relatively large in scope (at least in comparison to what I'm used to), so there are a lot of nuances in the evaluation of the syntax trees.
I've picked a few of the more-interesting ones to highlight.

### Functions as Data
APL makes combination of functions very natural; adjacent functions and operators alone form trees (even before application to arrays)[^tacit].

[^tacit]: Operators are higher-order functions (which naturally form trees when they are applied); functions placed next to each other are combined into "trains" (this is called [tacit programming](https://aplwiki.com/wiki/Tacit_programming)).

For example, here's the tree for the function **`h2n`** (from an earlier example):

```
    c2n ← '23456789TJQKA'∘⍳
    classify ← (5-≢⍤∪),(⌈/(+/∘.=⍨)),((5≡∪)×(⌈/c2n))
    h2n ← 13⊥classify,(¯1∘+c2n)
    h2n
 ┌──┼────────────┐
 13 ⊥ ┌──────────┼────────────────────────┐
  ┌───┼────────┐ ,                   ┌────┴─────┐
┌─┼─┐ ,    ┌───┼───────┐             ∘          ∘
5 - ⍤    ┌─┴─┐ ,   ┌───┼─────┐      ┌┴─┐ ┌──────┴──────┐
   ┌┴┐   / ┌─┴─┐ ┌─┼─┐ × ┌───┴────┐ ¯1 + 23456789TJQKA ⍳
   ≢ ∪ ┌─┘ /   ⍨ 5 ≡ ∪   /        ∘
       ⌈ ┌─┘ ┌─┘       ┌─┘ ┌──────┴──────┐
         +   ∘.        ⌈   23456789TJQKA ⍳
           ┌─┘
           =
```
[^tut]

[^tut]: For an explaination of how to interpret this tree, see my [APL Tutorial](/blog/apl-tutorial).

Since functions are higher-order in Haskell, it's natural to store them as normal data.
Thus, upon evaluation (in this case, evaluation happens when `h2n` is assigned to[^lazy]), the variable **`h2n`** no longer has access to the syntax tree, it only holds a primitive haskell function, a string which represents the scrapped tree, and a few other values describing the behavior of the function.

[^lazy]: Technically, evaluation of the tree might be delayed until Haskell needs the value of the [`Function`](https://github.com/lucasscharenbroch/apl-interpreter/blob/master/src/GrammarTree.hs#L146) (my name for the container type of the primtive Haskell function, the text representation of the tree, and some other function-related data) is actually needed (to call the primitive function, or to display the string). This has to do with Haskell's laziness, which will be discussed later.

{{< highlight haskell >}}
type FuncM = Array -> StateT IdMap IO Array
type FuncD = Array -> FuncM

data Function = MonFn FnInfoM FuncM
              | DyadFn FnInfoD FuncD
              | AmbivFn FnInfoA FuncM FuncD

-- "function tree": a tree that makes up a derived function:
-- the internal nodes are operators, and the leaves are functions or (derived) arrays
data FnTreeNode = FnLeafFn Function
                | FnLeafVar String
                | FnLeafArr ArrTreeNode
                | FnInternalMonOp OpTreeNode FnTreeNode
                | FnInternalDyadOp OpTreeNode FnTreeNode FnTreeNode
                | FnInternalAtop FnTreeNode FnTreeNode
                | FnInternalFork FnTreeNode FnTreeNode FnTreeNode
                | FnInternalAssignment String FnTreeNode
                | FnInternalQuadAssignment FnTreeNode
                | FnInternalAxisSpec FnTreeNode ArrTreeNode
                | FnInternalDummyNode FnTreeNode

evalFnTree :: FnTreeNode -> StateT IdMap IO (Either Array Function)
{{< /highlight >}}

### Practical Typeclasses

Like the parser, evaluation uses monads to handle state.
Since any arbitrary function might have full control over the program state, in order to have a universal function type, that type (**`Function`** (see above)) must be impure[^impure].

[^impure]: Impure in the sense that it returns a monad which has access to the entire global state.

The majority of built-in **`Functions`** don't touch the global state, and the ones that do usually only use it for a very narrow purpose.
I wanted a mechanism that allowed me to define these **`Functions`** with maximally constraining types, yet to easily convert them into the full (non-constrained) monad (**`EvalM`**) without causing a combinatorial explosion of helper functions.
I used the typeclass **`SubEvalM`** to do this.

{{< highlight haskell >}}
{- SubEvalM (subset of EvalM): typeclass for wrapper monads -}

type EvalM = StateT IdMap IO

class (Monad m) => SubEvalM m where
    toEvalM :: m a -> EvalM a

instance SubEvalM Identity where
    toEvalM = return . runIdentity

newtype IdxOriginM a = IdxOriginM { unIdxOriginM :: Reader Int a }
    deriving (Functor, Applicative, Monad, MonadReader Int) via (Reader Int)

instance SubEvalM IdxOriginM where
    toEvalM iom = do
        idm <- get
        let iO = case mapLookup "⎕IO" idm of
                  Just (IdArr a)
                      | ScalarNum n <- a `at` 0 -> Prelude.floor $ n
                  Just _ -> undefined -- unexpected val for ⎕IO
                  _ -> undefined -- no val for ⎕IO
        return . (flip runReader) iO . unIdxOriginM $ iom

newtype RandAndIoM a = RandAndIoM { unRandomAndIoM :: StateTStrict.StateT StdGen (Reader Int) a }
    deriving (Functor, Applicative, Monad, MonadReader Int, MonadState StdGen) via StateTStrict.StateT StdGen (Reader Int)

instance SubEvalM RandAndIoM where
    toEvalM rm = do
        gen <- lift $ newStdGen
        toEvalM . IdxOriginM . runStateGenT_ gen $ \_ -> unRandomAndIoM rm


{- Monad Wrappers -}

mkMonFn :: SubEvalM m => FnInfoM -> (Array -> m Array) -> Function
mkMonFn i f = MonFn i (\a -> toEvalM $ f a)

mkDyadFn :: SubEvalM m => FnInfoD -> (Array -> Array -> m Array) -> Function
mkDyadFn i f = DyadFn i (\a b -> toEvalM $ f a b)

mkAmbivFn :: (SubEvalM m0, SubEvalM m1) => FnInfoA -> (Array -> m0 Array) -> (Array -> Array -> m1 Array) -> Function
mkAmbivFn ia fm fd = AmbivFn ia (\a -> toEvalM $ fm a) (\a b -> toEvalM $ fd a b)

-- (mkMonOp, ... etc.)

{- Pure Wrappers -}

pureMonFn :: FnInfoM -> (Array -> Array) -> Function
pureMonFn i f = mkMonFn i (Identity . f)

pureDyadFn :: FnInfoD -> (Array -> Array -> Array) -> Function
pureDyadFn i f = mkDyadFn i (Identity .: f)

pureAmbivFn :: FnInfoA -> (Array -> Array) -> (Array -> Array -> Array) -> Function
pureAmbivFn ia fm fd = mkAmbivFn ia (Identity . fm) (Identity .: fd)

-- (pureMonOp, ... etc.)
{{< /highlight >}}

### Selective Assignment

APL has a feature called "selective assignment" where the left-hand-side of an assignment can be an expression, so long as it only uses (pre-verified) functions that solely permute/select items of the argument array.
For example:

```
    x ← 2 2⍴⍳4
    x
1 2
3 4
    (,⌽x) ← ⍳4 ⍝ assign ⍳4 to the ravel (,) of
    x          ⍝ the reversed (along the last axis) (⌽) x
2 1
4 3
```

At first glance, this seems like implementing this will require a re-implementation of each selectable function, which can be called to determine which cells of the variable are selected, and their shape.
However, after making the above assumption (all selectable functions do not mutate the cells of their argument), the selected array (x, in the above example) can simply be replaced by an array of the same shape whose cells are the indices of x (⍳⍴x), then, after applying the selecting functions, the result will be some permutation/selection of the indices of (⍳⍴x), which can then be assigned to.
So, for the above example, we have

```
    x ← 2 2⍴⍳4
    x
1 2
3 4
    ⍴x
2 2
    ⍳⍴x
┌───┬───┐
│1 1│1 2│
├───┼───┤
│2 1│2 2│
└───┴───┘
    ⌽⍳⍴x
┌───┬───┐
│1 2│1 1│
├───┼───┤
│2 2│2 1│
└───┴───┘
    ,⌽⍳⍴x
┌───┬───┬───┬───┐
│1 2│1 1│2 2│2 1│
└───┴───┴───┴───┘
    x[,⌽⍳⍴x]
2 1 4 3
    x[,⌽⍳⍴x] ← ⍳4
    x
2 1
4 3
```

### Higher-Dimensional Arrays: It's Just Indexing

One of the hardest aspects of implementing the built-in functions/operators was making them work on higher-dimensional arrays.
The majority APL primitives are trivially implemented on vectors and scalars, but their behavior becomes much harder to understand when they operate on n-dimensional arrays.

For example, dyadic (**`,`**) ([con]catenate).
Just put the two arrays together, no problem, right?

{{< highlight haskell >}}

catenate :: Double -> Array -> Array -> IdxOriginM Array
catenate ax x y
    | isIntegral ax = ask >>= \iO -> let ax' = (Prelude.floor ax) - iO + 1
                                     in return $ _catenate ax'
    | otherwise = ask >>= \iO -> let ax' = (Prelude.ceiling ax) - iO
                                 in return $ _laminate ax'
    where _catenate ax'
              | ax' <= 0 || ax' > rank = throw . RankError $ "(,): invalid axis"
              | x == zilde = y
              | y == zilde = x
              | (shape'' x') /= (shape'' y') = throw . LengthError $ "(,): mismatched argument shapes"
              | otherwise = zipVecsAlongAxis ax'' ax'' ax'' (++) x' y'
              where (x', y') = _rankMorph (x, y)
                    rank = arrRank x'
                    _rankMorph (a, b)
                        | shape a == [1] && arrNetSize b > 0 = (shapedArrFromList (shape'1 b) $ Prelude.replicate (foldr (*) 1 $ shape'1 b) (a `at` 0), b)
                        | shape b == [1] && arrNetSize a > 0 = (a, shapedArrFromList (shape'1 a) $ Prelude.replicate (foldr (*) 1 $ shape'1 a) (b `at` 0))
                        | arrRank a == arrRank b = (a, b)
                        | arrRank a == arrRank b + 1 && ax' <= arrRank a = (a, b {shape = take _ax' (shape b) ++ [1] ++ drop _ax' (shape b)})
                        | arrRank a + 1 == arrRank b && ax' <= arrRank b = (a {shape = take (_ax' + 1) (shape a) ++ [1] ++ drop (_ax' + 1) (shape a)}, b)
                        | otherwise = throw . RankError $ "(,): mismatched argument ranks"
                        where _ax' = ax' - 1
                    shape'1 a = take (ax' - 1) (shape a) ++ [1] ++ drop ax' (shape a)
                    shape'' a = take (ax'' - 1) (shape a) ++ drop ax'' (shape a)
                    ax'' = if arrRank x' == arrRank x + 1 then ax' + 1 else ax'
          _laminate ax'
              | ax' < 0 || ax' > rank = throw . RankError $ "(,): invalid axis"
              | (shape x') /= (shape y') = throw . LengthError $ "(,): mismatched argument shapes"
              | otherwise = unAlongAxis (ax' + 1) [x', y']
              where (x', y') = _rankMorph (x, y)
                    rank = arrRank x'
                    _rankMorph (a, b)
                        | shape a == [1] && arrNetSize b > 0 = (shapedArrFromList (shape b) $ Prelude.replicate (arrNetSize b) (a `at` 0), b)
                        | shape b == [1] && arrNetSize a > 0 = (a, shapedArrFromList (shape a) $ Prelude.replicate (arrNetSize a) (b `at` 0))
                        | shape a == shape b = (a, b)
                        | otherwise = throw . RankError $ "(,): mismatched argument ranks"

{{< /highlight >}}

The majority of this code is actually just edge-cases (which are difficult in their own right), but the key functions to note are **`zipVecsAlongAxis`** and **`unAlongAxis`** (which are applied after the edge-cases are resolved).
All primitives functions that operate on higher-dimensional arrays can be expressed using a handful of helper functions.
All of those helpers are built on top of three core functions.

{{< highlight haskell >}}
alongAxis :: Int -> Array -> [Array]
alongAxis ax a
    | ax - _iO >= (length . shape $ a) = throw $ RankError "invalid axis"
    | (length . shape $ a) == 0 = []
    | arrNetSize a == 0 = []
    | otherwise = map (subarrayAt) [0..(n - 1)]
        where n = (shape a) !! (ax - _iO)
              shape' = if (length . shape $ a) == 1
                       then [1]
                       else take (ax - _iO) (shape a) ++ drop (ax - _iO + 1) (shape a)
              sz = arrNetSize a
              subarrayAt i = shapedArrFromList shape' . map (arrIndex a) $ indicesAt i
              indicesAt i =  map (\is -> take (ax - _iO) is ++ [i] ++ drop (ax - _iO) is) $ map (calcIndex) [0..(sz `div` n - 1)]
              indexMod = tail . scanr (*) 1 $ shape'
              calcIndex i = map (\(e, m) -> i `div` m `mod` e) $ zip shape' indexMod
              _iO = 1 -- axis supplied is with respect to _iO = 1, not actual ⎕IO

alongRank :: Int -> Array -> Array
alongRank r a
    | foldr (*) 1 (shape a) == 0 = zilde
    | n <= 0 = listToArr [ScalarArr a]
    | n >= (length $ shape a) = a
    | otherwise = shapedArrFromList outerShape . map (ScalarArr . shapedArrFromList innerShape) . groupsOf groupSz . arrToList $ a
        where outerShape = take n $ shape a
              innerShape = drop n $ shape a
              n = if r >= 0 then (length $ shape a) - r else -1 * r
              groupSz = foldr (*) 1 innerShape

arrReorderAxes :: [Int] -> Array -> Array
arrReorderAxes targetIdxs a
    | length targetIdxs /= (length $ shape a) = undefined
    | otherwise = shapedArrFromList shape' vals'
        where shape' = map (foldr min intMax . map snd) . groupBy (on (==) fst) . sortBy (on compare fst) $ zip targetIdxs (shape a)
              n = foldr (*) 1 shape'
              vals' = map ((a`arrIndex`) . calcIndex) [0..(n - 1)]
              calcIndex i = map ((idxList!!) . (+(-1))) targetIdxs
                  where idxList = calcIndex' i
              calcIndex' i = map (\(e, m) -> i `div` m `mod` e) $ zip shape' indexMod
              indexMod = tail . scanr (*) 1 $ shape'
{{< /highlight >}}

The details of the implementation doesn't matter so much as the fact that these functions only do two main things:
1. Index arithmetic
2. Manipulation of shapes (array dimensions)

That's all it takes to work with multi-dimensional arrays.

## Mimicking Dyalog

[Dyalog](https://www.dyalog.com/) APL is the de-facto modern implementation of APL.
This project is heavily based off of Dyalog APL.
All of the syntax and glyphs come directly from Dyalog, and
the behavior of the functions and operators was almost entirely taken/tweaked from the [Dyalog Reference Guide](https://docs.dyalog.com/latest/Dyalog%20APL%20Language%20Reference%20Guide.pdf).

This was convenient in many ways, because it provided an oracle to test against[^testing], and at any point when I was uncertain of the behavior of something, it gave me a definitive answer, or at least a model which I could analyze to come to my own answer.

[^testing]: I wrote a shell-script to compare the output of my interpreter with Dyalog's output.

However, attempting to clone Dyalog made the project harder in a lot of ways, namely
- Getting printing of arrays, function trees, and (high-precision, scientific-notation, floating-point) numbers to exactly match Dyalog
- Trying to match Dyalog's behavior in weird edge-cases
- The sheer generality of many of the glyphs (catenate (above) is a good example of this)

I ended up making a lot of compromises between what I thought was doable (or even possible: some behavior of Dyalog was seemingly incomprehensible), what seemed the most simple[^rich], and what Dyalog did.

[^rich]: The most *simple*, not the most *easy*.

The following is a (far from complete) laundry-list of discrepancies between my interpreter and Dyalog's.

- Differing properties:
    - Amount of whitespace when printing arrays with non-zero rank, but with 0 ∊ shape
    - Whitespace and some number formatting when printing matrices of real-numbers
    - Printing format of arrays as leaves of function trees
    - Printing format of arrays as axis specifications
    - Printing of dfns/dops (∇)
    - Exact rules for sorting order
    - Propigation rules for function properties (i.e. identity, ability to select, axis specification) on derived functions
    - Fine-grained behavior of execute (⍎) and format (⍕)
- In Dyalog, ...
    - Variables have nameclasses and weird behavior after reassignment[^nameclass]
    - Some dops can't be used inline
    - Dyadic operators can't be in the right-hand-side of assignments
    - The default ⍺ can be a function in dops
- In my interpreter, ...
    - Dops/dfns are not strongly short-circuited: if the condition in a guard is false, the rhs isn't evaluated, but it is still parsed (in Dyalog, it isn't parsed)
    - Scalars are not 0-rank: they are vectors with a single element (this approach seems more simple[^simple], but it causes some incompatibilities in the behavior of certain functions)
    - Arrays with non-1 rank cannot have zero as an element in their shape (Dyalog allows this)
    - LCM, GCD, factorial, and binomial only work on integers (Dyalog allows reals and complex numbers)
    - The circle function (dyadic ○) only supports 1 2 3, ¯1, ¯2, ¯3
    - Dfns cannot *modify* global variables
    - ⎕IO can be *any integer* (not just 0 or 1)
    - Axis spec operator is more limited (several functions don't support it (e.g. ⊂)), it also only takes singleton numbers, never vectors
    - Dyadic ⍒/⍋ is limited to vector (not higher-dimensional array) collation sequences
    - Reduce can't take a negative or zero argument on left (window)
    - Encode only works on integers
- My interpreter does not support ...
    - Complex numbers
    - Dyadic thorn (⍎) and dyadic hydrant (⍕)
    - &, ⌸, ⌺, and ⌹
    - Monadic squad (⌷)
    - I-Beam (⌶)
    - Traditional Functions
    - Array prototypes, fill elements other than 0
    - Many System Names
    - Many other Dyalog features

[^nameclass]: As a result, my interpreter has a higher likelihood of a runtime error when the value of a name changes class (e.g. from an array to a function).

[^simple]: Making scalars 0-rank is sometimes flat-out confusing. In Dyalog, the result of "(≡¨,≢¨) 1 ⍬" is "0 1 1 0", where my interpreter yields "1 1 1 0". How can ⍬ have depth 1, but tally 0?

## Haskell: the Good and the Ugly

### The Good

#### The Compiler

I am generally a fan of the guarantees the compiler gives, and the Haskell complier has given me more guarantees[^comp] than any other I have used.
The time I spent debugging runtime errors in Haskell was significantly smaller in proportion to imperative languages, and there were many times where I was astonished[^work] that my code worked after I fixed the compilation errors.

That being said, runtime errors still happened, and many of them had similar nature to those in other languages (typically logic errors, or panics). Such errors grew more common and more difficult to debug as the project grew in scale[^err].

It was also relatively difficult to get code to compile (and writing code in general) (in comparison to in other languages).
It's hard to tell how much of this has to do with the fact that I'm new to Haskell, and how much easier it will become over time.
It's hard to gage my progress over the course of project, because the nature of the problems I'm solving (and their complexity) has fluctuated a lot.
I've certainly become more fluent in Haskell, but I've continued to battle compilation errors.

[^comp]: Namely: it guarantees that there are no side-effects that that all types line up (these rules have a lot of implications, because almost everything is a function, and everything has a type).

[^work]: This has been a very rare occurrence, and I've grown to expect disappointment when running code for the first time.

[^err]: This could partially be a coincidence, as I spent more time writing heavily error-prone code (functions and operators) as the project grew.

#### The Libraries

The standard libraries have a lot of useful functions.
I made frequent use of Data.List, Control.Monad, and Data.Function, among others.

Most simple operations you might want to do are often well within reach, and it's a lot of fun using standard functions to replace boilerplate.

#### Currying, Combinators, Tacit

I probably used these way too much, but they're a lot of fun.

### The Ugly

#### The Elephant In The Room

Haskell is incomprehensible if you haven't spent time with it (or languages like it).
Functional style alone is probably a big enough gap from imperative to make it difficult[^read-write], but combine that with all the category theory and crazy type stuff, and the learning curve becomes a cliff.

[^read-write]: It certainly was enough to make writing code difficult for me -- the same might not be true for reading.

I'm not suggesting that Haskell should be watered down -- I see the utility in its more esoteric style -- but this doesn't change the fact that it makes communication (one of the most important aspects of programming) difficult in many situations.

#### Over-Generalization

Perhaps this is a contradiction (you can't have your cake and eat it), since I just raved about the benefits of generality, but I found that it goes both ways: generality can be bad in excess.

My best example of this is the System.Random (and System.Random.Stateful) library: even after I felt like I had a solid grasp on monads and monad transformers, it took me several hours (and even reading some of the source) to comprehend the library[^subset] enough to feel comfortable using it[^comfort].

[^comfort]: I admittedly have a tendency to insist on complete (to a degree) understanding, so I could probably have gotten something up and working more quickly, but the point stands.

[^subset]: Not the whole library: just the subset of types I thought I might need (after narrowing down which type I needed, which probably took an hour or two in itself).

#### Efficiency

This goes without saying.
Apparently Haskell's performance isn't *that bad*, but I don't plan on using Haskell for anything that is remotely performance-sensitive.

Trying to optimize Haskell code does sound like an interesting problem, but it might be a lost cause.

#### Laziness

Generally it doesn't matter *when values are evaluated*, so long as you have them when you need them, but since I made excess use of *exceptions* and *undefined*'s in this program, understanding laziness became important.

It's funny how, when using Haskell, laziness creeps into your program where you don't expect it.

After implementing assignment, it took me a while to realize that, if the RHS of an expression threw an error when evaluated, that error wouldn't show itself until I tried to *print* the variable to which the value was given. For example:

```
    x ← 1 + 'c' ⍝ this statement succeeds
    1 + 2       ⍝ execution continues as normal
3
    y ← x + 1   ⍝ this is fine as well
    y           ⍝ x is only evaluated when the concrete value of y is needed for printing
DOMAIN ERROR: expected number
```

Funnily enough, I thought I fixed this, but after running this example, the current version of the interpreter produces the same result.

Laziness can be convenient when you want to avoid throwing errors unless absolutely necessary (which is usually the case), but when you want to deliberately throw an error, you must force execution into a single path towards that error.

This uncertainty of time of evaluation also makes catching errors difficult, because calling **`catch`** on the function that throws the error *will not necessarily catch that error*.
Catch must be called on the function that *forces evaluation* on that error.
This is something that is hard to trace, and something that types don't help much with.

The solution to this is probably to use monads instead of exceptions.

#### Debugging

Haskell's lazy evaluation also makes debugging much harder.
Evaluation is not linear: stepping doesn't go line-by-line: it jumps around between functions as their values are needed.

GHCi does have a built-in debugger (which is pretty impressive), but for this reason, it's not really the most useful.

As I mentioned earlier, runtime errors are much less prevalent due to strict typing, but when they do arise, this could become an issue.
