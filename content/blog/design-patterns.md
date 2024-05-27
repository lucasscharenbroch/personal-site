---
title: "Design Patterns"
subtitle: "Notes and Commentary on the Classic OOP Book"
date: 2024-05-26T11:33:05-05:00
notability: 0
tags: ["Pl", "Fp"]
draft: true
---

- I think OOP is still a central way of programming, an effective one, and one that's important to know
- hell, look at rust, even - it still exhibits this (the decomposition into structs still holds)

(11)
> The hard part about object-oriented design is decomposing a system into objects.The
> task is difficult because manyfactors comeinto play: encapsulation, granularity,dependency, flexibility, performance, evolution, reusability, and on and on. They allinfluence
> the decomposition, often in conflicting ways.


(20)
> That leads us to our second principle of object-oriented design:
> Favor object composition over class inheritance.

(31)
> Design patterns should not be applied indiscriminately
> Often they achieve flexibility and variability by introducing additional levels of indirection,
> and that can complicatea design and/or cost you some performance. A design pattern
> should only be applied when the flexibility it affords is actually needed.

---

## In A Nutshell

- Creational Patterns
    - _ Abstract Factory
    - _ Builder
    - _ Factory Method
    - _ Prototype
    - _ Singleton
- Structural Patterns
    - _ Adapter
    - _ Bridge
    - x Composite
        - a subclass contains a member that is a list of the superclass
        - any instance of the superclass can be viewed as a tree, whose nodes all have the same interface
    - x Decorator
        - a wrapper type that subclasses the type it wraps (the wrapped type can also be a wrapper)
        - this allows for composition of wrapper-method augmentations
    - _ Facade
    - _ Flyweight
    - _ Proxy
- Behavioral Patterns
    - _ Chain of Responsibility
    - _ Command
    - _ Interpreter
    - _ Iterator
    - _ Mediator
    - _ Momento
    - _ Observer
    - _ State
    - x Strategy
        - use an instance of a class for its v-table (simulating higher-order functions)
        - instances of different subclases can provide varying (interchangable) behaivor
    - _ Template Method
    - _ Visitor

## Commentary

### Composite

- this was used in the 536 compiler, haha (ExprNode)
    - the below quote is accurate to how that went: hard to dispatch, and hard to do most tree operations because of that, and more information must be embedded into the superclass to accomplish it
    - "can make your design overly general. The disadvantage of making it easy to add new components isthat it makes it harder to restrict the components of a composite. Sometimes you want a composite to have only certain components. WithComposite, you can't rely on the type system to enforce those constraints for you. You'll have to use run-time checks instead."
    - the book recommends "getCommposite" as a solution to this, which isn't bad if there's only one type of composite and the type of leaves don't matter (their common interface is sufficient)
- this pattern does offer more flexibility than the functional equivalent, or at least it feels moer dynamic: since every node fulfills the superclass interface, it can be put anywhere a node is accepted (the same is typically not true for ADT trees); it's also more natural (though less safe) to mutate and pass around pointers
- but that generality must be aborted as soon as we want to dispatch (and the work-around is that getCommposite method)
- the book admits all of these faults (!)

### Strategy

This is basically just a hack to get higher-order functions.
It makes a little more sense when multiple functions are passed (in which case they would have to be wrapped up in some way anyway (perhaps via an product type or typeclass); in this case, the OOP and FP code looks pretty similar.

This can be viewed as an alternative to inheritance/subclassing, as it can eliminate a lot of boilerplate.

The book mentions using the strategy class as a template parameter, which is analagous to using a typeclass-instance-type as a parameter to the underlying type (i.e. the class that contains the strategy), e.g.

```cpp
template <class AStrategy>
class Context {
    void Operation() { theStrategy.doAlgorithm(); }
    // . . .
private:
    AStrategy theStrategy;
};

class MyStrategy {
public:
    void doAlgorithm();
}

Context<MyStrategy> aContext;
```

In C++ is equivalent to the following in Haskell:

```hs
{-# LANGUAGE ScopedTypeVariables #-}

import Data.Proxy (Proxy(Proxy))

data Context s = Context

class Strategy a where
    doAlgorithm :: Proxy a -> ()

operation :: forall s. Strategy s => Context s -> ()
operation _ = doAlgorithm (Proxy :: Proxy s)

data MyStrategy -- dummy type
instance Strategy MyStrategy where
    doAlgorithm _ = ()

aContext :: Context MyStrategy
aContext = Context
```

The utility of doing exactly this in Haskell is questionable, becaues the typeclass can be just as easliy constrained elsewhere (i.e. use the proxy (or ideally some other indicative type) as a direct argument to doAlgorithm or operation). This would eliminat the need for `ScopedTypeVariables`.

## Decorator

This is a less-generic case of the composite pattern (a subset of it, where the superclass list has exactly one element).
Because of this, the book highlights the decorator as a composition tool rather than a structuring mechanism.
Nevertheless, most of the pros and cons remain the same.
