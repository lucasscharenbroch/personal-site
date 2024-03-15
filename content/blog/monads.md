---
title: "Monads"
subtitle: "An attempt to precisely describe a notorious concept"
date: 2024-03-06T07:34:10-06:00
---

Monads are **data types**[^category-theory] that expose methods for *chaining*, typically for the sake of **abstracting state**.

[^category-theory]: The term "monad" comes from category theory, where it has a slightly different definition (it's defined in terms of categories, not data). All definitions of category-theory constructs in this post refer to their Haskell equivalents.

More concretely, a monad is a type m where there exist the two[^more-than-2] functions:

[^more-than-2]: In Haskell, **`Monads`** must also be instances of **`Applicative`**.

{{< highlight haskell >}}
class Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
    return :: a -> m b
{{< /highlight >}}

The above signatures might be confusing, because they make use of several of Haskell's language features.
- *Infix Operators*:**`(>>=)`** is a two-argument function that can be called with infix notation (i.e. **`x >>= y`**)
- *Parametric* Polymorphism: m is a generic type that takes single type parameter[^param]; in this case, the paramaters themselves (**`a`** and **`b`**) are also generic
- *Currying*: every function in Haskell takes a single argument; the illusion of multiple-argument functions is made when functions return other functions: **`(>>=)`** doesn't actually take two arguments

[^param]: The type itself (when used in a signature) takes the parameter: this is another way of saying "m has [kind](https://wiki.haskell.org/Kind) * -> *"

The bottom line is that **`m`** is a data type that is generic over exactly one other type (i.e. the "vector" in "vector\<T\>", the "Maybe" in "Maybe a", or the "[]" in "[a]"), and the class statement declares that any **`m`** is a monad so long as it declares itself an instance of **`Monad`** and provides the functions **`(>>=)`** and **`return`**.

This is all pretty abstract, and it's not clear how these type-signatures imply chaining or state-abstraction.

## Contextualization

Say we have a chain of functions that transform some data of type **`A`** to type **`D`** by sequentially applying the functions f, g, and h.

```
    f        g         h
A -----> B ------> C -----> D

f :: A -> B
g :: B -> C
h :: C -> D
(\x -> h (g (f x))) :: A -> D
```

Say we want to maintain some shared state of type **`E`** over these function calls (**`f`**, **`g`**, and **`h`**, can read or modify that data, as if it were a global variable).

```
    f        g         h
A -----> B ------> C -----> D
E _/ \______/ \_______/ \__ E

f :: A -> B
g :: B -> C
h :: C -> D
```

This drawing doesn't quite work out, though, since the external arrows hovering below the function calls (passing **`E`** between functions whose signatures don't involve **`E`**) are impure.
It isn't possible to have a pure function **`f :: A -> B`** that can read/write external data.
So in order to have this "global" state, the passing of **`E`** must be embedded within the type-signatures/function-calls.

In this case, this can be done by introducing a tuple.

```
         f             g             h
(A, E) -----> (B, E) -----> (C, E) -----> (D, E)

f :: (A, E) -> (B, E)
g :: (B, E) -> (C, E)
h :: (C, E) -> (D, E)
(\x -> h (g (f x))) :: (A, E) -> (D, E)
```

We can also define a monad called **`EState`** that can do a similar computation.

```
    f             >>= g           >>= h
A -----> EState B -----> EState C -----> EState D

f :: A -> EState B
g :: B -> EState C
h :: C -> EState D

(>>= g) :: EState B -> EState C
(>>= h) :: EState C -> EState D
(\x -> f x >>= g >>= h) :: A -> EState D
```

This almost exactly lines up with the tuple version, except the definitions of **`f`**, **`g`**, and **`h`**, namely that their parameters are not wrapped in **`EState`** (which creates an assymmetry in the composition).
This is explained by the definition of **`EState`**.


{{< highlight haskell >}}
newtype EState a = EState (E -> (a, E))

runEState :: EState a -> E -> (a, E)
runEState (EState run) = run

-- The above could be replaced by (using record notation):
-- newtype EState a = EState { runEState :: (E -> (a, E)) }

instance Monad EState where
    -- (>>=) :: EState a -> (a -> EState b) -> EState b
    (>>=) esa f = EState internal
        where -- internal :: E -> (b, E)
              internal state = let (a, state') = runEState esa state
                               in runEState (f a) state'
    -- return :: a -> EState a
    return x = EState (\state -> (x, state))
{{< /highlight >}}

The key realization here is that **`EState A`** does not store the same data as **`(A, E)`**. **`(A, E)`** stores a concrete value of **`A`** and **`E`**, while **`EState a`** is a stores a function of type **`(E -> (a, E))`**.
Bind (**`(>>=)`**) tacks on a function call (the RHS) onto the function encapsulated into the **`EState`**.

The following more clearly illustrates the underlying type of the monad.
The resulting **`EState D`** value encapsulates a function which is the (indirect) composition of **`f`**, **`g`**, and **`h`**.
Applying that underlying function (to the initial state) passes the state through the whole **`f`**-**`g`**-**`h`** pipeline, and yields the resulting value and state.

{{< highlight haskell >}}

f :: A -> EState B
x :: A

(f x) :: EState B -- (E -> (B, E))

(>>=) :: Monad m => m b -> (b -> m c) -> m c
g :: B -> EState C

(f x >>= g) :: EState C -- (E -> (C, E))
-- f x >>= g = EState internal
--     where internal state = let (a, state') = runEState (f x) state
--                            in runEState (g a) state' -- add g to the chain

h :: C -> EState D

(f x >>= g >>= h) :: EState D -- (E -> (D, E))
-- f x >>= g >>= h = EState internal
--     where internal state = let (b, state') = runEState (f x >>= g) state
--                            in runEState (h b) state' -- add h to the chain

-- which is equivalent to...

-- f x >>= g >>= h = EState internal
--     where internal state = let (a, state') = runEState (f x) state
--                                (b, state'') = runEState (g a) state'
--                            in runEState (h b) state'' -- add h to the chain
{{< /highlight >}}

The only piece that's left is how the state can be read/written.
This is also done by chaining <b>`EState`</b>s.

{{< highlight haskell >}}
get :: EState E
get = EState (\state -> (state, state))

put :: E -> EState ()
put newState = EState (\_ -> ((), newState))
{{< /highlight >}}

**`get`** and **`put`** use their knowledge of the underlying structure of **`EState`** to make an interface[^interface] for users to reach into the internal value (in this case, adding functions to the chain that read/write the state instead of just ignoring it).

Below is an example of how they can be used.

{{< highlight haskell >}}
type E = Int

-- square the state, then return it
squareTheState :: EState Int
squareTheState = get >>= \s -> (put (s * s) >>= \_ -> return (s * s))

-- get >>= \s -> (put (s * s) >>= \_ -> return (s * s) = EState internal
--     where internal state = let (a, state') = runEState get state
--                            in runEState (lambda1 a) state'
--           lambda1 s = EState internal1
--               where internal1 state1 = let (a1, state1') = runEState (put (s * s)) state1
--                                        in runEState (lambda2 a1) state1'
--           lambda2 _ = EState internal2
--               where internal2 state2 = runEState (return (s * s)) state2
{{< /highlight >}}

[^interface]: In general, due to the definition of Monad, only the monadic value (the **`a`** in **`m a`**) is accessible using the built-in Monadic functions (**`>>=`**, **`return`**, and their variants).

### Do-Notation
Bind (**`(>>=)`**) is so commonly used (and creates such a syntactic inconvenience) that there's a special syntax for it.

{{< highlight haskell >}}
do x <- y
   (...)

-- is the same as

y >>= \x -> (...)
{{< /highlight >}}

And

{{< highlight haskell >}}
do y
   (...)

-- is the same as

y >>= \_ -> (...)
{{< /highlight >}}

The following is an equivalent definition of **`squareTheState`**, written with do-notation instead of bind.

{{< highlight haskell >}}
squareTheState :: EState Int
squareTheState = do s <- get
                    put (s * s)
                    return (s * s)
{{< /highlight >}}

### Key Takeaway

Monads allow us to abstract away the management of "state" via chaining.
We can have the effect of the below ascii-image (where an external factor controls the execution of pure functions) by wrapping the return-values of functions in monads, and, as a result, not have to worry about the underlying chaining-details when actually writing the functions.

```
    f        g         h
A -----> B ------> C -----> D
  _/ \______/ \_______/ \__
```

## Monads in Action

### Non-Function Data

Like **`EState`**, many monads in standard libraries encapsulate functions:

- IO
- State
- Reader
- Writer

There's no constraint on what can be a monad, though, so long as it fulfills the minimal required functions.

**`Maybe`** and **`Either`** are both ambivalent: when chained together, they always propagate the first failure.

{{< highlight haskell >}}
instance  Monad Maybe  where
    (Just x) >>= k      = k x
    Nothing  >>= _      = Nothing

instance Monad (Either e) where
    Left  l >>= _ = Left l
    Right r >>= k = k r
{{< /highlight >}}

### Not Confusing At All

There are some particularly fringe monad[^applicative-too] instances in the standard libraries too: it seems that if there is any remotely[^trivial] logical implementation for a given data type, it becomes a monad.

[^trivial]: A long-time Haskeller might argue that most instances are trivially derivable from the type signatures.
This might be true, but that rarely implies that they're trivially understood in context of other complex code.
With great power comes great responsibility.

[^applicative-too]: The same is true for Applicative and Functor, and probably many more such typeclasses.
Some of these actually are pretty useful: the instance of Applicative for functions defines the **`<*>`** operator to be the S combinator, and **`liftA2`** to be the Φ combinator.
See [Conor Hoekstra's Combinator Table](https://combinatorylogic.com/table.html) for more.

This can occasionally be useful, but it seems dangerously close to obfuscation at times.

For instance, **`[]`** is a Monad.
Try to guess what the following lines do without looking at [the definition](https://hackage.haskell.org/package/base-4.19.1.0/docs/src/GHC.Base.html#line-1306).

{{< highlight haskell >}}
[[1.3, 5.6, 7.8], [1.0, 3.4, 1.4]] >>= reverse
[1, 2, 3] >>= \x -> replicate x x
["abc", "def", "ghi"] >>= id
{{< /highlight >}}
[^answer]

[^answer]: The type-signature makes it easier: **`[a] -> (a -> [b]) -> [b]`**. It's a flipped version of concatMap.

Functions (the type **`((->) r)`**) are also monads: **`(>>=)`** is the flipped Σ combinator (which is not all that useful).

{{< highlight haskell >}}
(>>=) ::  m    a  -> (a -> m    b) -> m    b
(>>=) :: (r -> a) -> (a -> r -> b) -> r -> b -- for ((->) r)

duplicateHead :: [a] -> [a]
duplicateHead = head >>= (:)

adjacentDifference :: Num a => [a] -> [a]
adjacentDifference = tail >>= zipWith (-)
{{< /highlight >}}
