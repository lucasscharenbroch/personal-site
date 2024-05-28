---
title: "Design Patterns"
subtitle: "Notes and Commentary on the Classic OOP Book"
date: 2024-05-26T11:33:05-05:00
notability: 7
tags: ["Pl", "Fp"]
draft: true
---

- I think OOP is still a central way of programming, an effective one, and one that's important to know
- look at rust, even - it still exhibits this (the decomposition into structs still holds)

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

### Creational Patterns

- [Abstract Factory](#abstract-factory-commentary)<a name="abstract-factory"></a>
    - an interface of constructor methods, which can be implemented by multiple "factories" who fulfill those constructors with their respective families of objects
    - this allows dynamic swapping of factories (and therefore dynamic selection of constructed families)
- [Builder](#builder-commentary)<a name="builder"></a>
    - a class with methods that accumulates properties, which can then be "built" into a final result
- [Factory Method](#factory-method-commentary)<a name="factory-method"></a>
    - a virtual/abstract method that constructs an object
    - the implementation can dynamically swap what object is constructed
- [Prototype](#prototype-commentary)<a name="prototype"></a>
    - using an instance of an object as a template to copy
- [Singleton](#singleton-commentary)<a name="singleton"></a>
    - a class with exactly one instance

### Structural Patterns

- _ [Adapter](#adapter-commentary)<a name="adapter"></a>
- x [Bridge](#bridge-commentary)<a name="bridge"></a>
    - a superclass contains a member that holds dynamic implementation details
    - the subclass structure need not account for those details
- x [Composite](#composite-commentary)<a name="composite"></a>
    - a subclass contains a member that is a list of the superclass
    - any instance of the superclass can be viewed as a tree, whose nodes all have the same interface
- x [Decorator](#decorator-commentary)<a name="decorator"></a>
    - a wrapper type that subclasses the type it wraps (the wrapped type can also be a wrapper)
    - this allows for composition of wrapper-method augmentations
- _ [Facade](#facade-commentary)<a name="facade"></a>
- _ [Flyweight](#flyweight-commentary)<a name="flyweight"></a>
- _ [Proxy](#proxy-commentary)<a name="proxy"></a>

### Behavioral Patterns
- _ [Chain of Responsibility](#chain-of-responsibility-commentary)<a name="chain-of-responsibility"></a>
- x [Command](#command-commentary)<a name="command"></a>
    - encapsulate an "action" (a (potentially) stateful function or set of functions) into a class
    - this "command" can then be used by a caller that has no knowledge of its state or purpose
- _ [Interpreter](#interpreter-commentary)<a name="interpreter"></a>
- x [Iterator](#iterator-commentary)<a name="iterator"></a>
    - an interface for iterating through a collection
- _ [Mediator](#mediator-commentary)<a name="mediator"></a>
- _ [Momento](#momento-commentary)<a name="momento"></a>
- _ [Observer](#observer-commentary)<a name="observer"></a>
- _ [State](#state-commentary)<a name="state"></a>
- x [Strategy](#strategy-commentary)<a name="strategy"></a>
    - use an instance of a class for its v-table (simulating higher-order functions)
    - instances of different subclases can provide varying (interchangable) behavior
- _ [Template Method](#template-commentary)<a name="template"></a>
- x [Visitor](#visitor-commentary)<a name="visitor"></a>
    - a strategy instance ("visitor") is passed into a "accept" method in the superclass, whose implementation calls the strategy method corresponding to the respective subclass (with the `this` pointer)
    - this refactors type-dispatch into the "accept" method (via polymorphism), where the case-handling code goes into the strategy ("visitor")
    - recursion can optionally be refactored into the "visitor" (strategy) superclass or "accept" method


## Commentary

### Composite<a name="composite-commentary"></a>

ADTs are usually a better alternative to this.
The book does well explaining why: The composite pattern...

> "can make your design overly general. The disadvantage of making it easy to add new components is that it makes it harder to restrict the components of a composite. Sometimes you want a composite to have only certain components. With Composite, you can't rely on the type system to enforce those constraints for you. You'll have to use run-time checks instead."

ADTs also allow dispatch and unwrapping of children via case-statements (instead of unsafe casting), which is almost always necessary.
ASTs in compilers are a quintessential use-case: see [this blog post](/blog/rewriting-a-toy-compiler) for a comparison of the two tactics.

This pattern does offer more flexibility than the functional equivalent, though, or at least it feels more dynamic: since every node fulfills the superclass interface, it can be put anywhere a node is accepted (the same is typically not true for ADT trees); it's also more natural (though less safe) to mutate and pass around pointers in this way (during assembly: the opposite is true in disassembly).

The book is quick to admit these faults.
[&#8617;](#composite)

### Strategy<a name="strategy-commentary"></a>

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

The utility of doing exactly this in Haskell is questionable, because the typeclass can be just as easily constrained elsewhere (i.e. use the proxy (or ideally some other indicative type) as a direct argument to `doAlgorithm` or operation). This would eliminate the need for `ScopedTypeVariables`.
[&#8617;](#strategy)

### Decorator<a name="decorator-commentary"></a>

This is a less-generic case of the composite pattern (a subset of it, where the superclass list has exactly one element).
Because of this, the book highlights the decorator as a composition tool rather than a structuring mechanism.
Nevertheless, most of the pros and cons remain the same.
[&#8617;](#decorator)

### Abstract Factory<a name="abstract-factory-commentary"></a>

The book makes a compelling case for this one: abstract factories can be used as a mechanism for generalizing gui-widget-constructors: for example, say you have multiple families of gui widgets (button, scrollbar, window, etc), that all implement the same superclasses (or can be adjusted to do so).
Instead of mentioning any specific family in the application code, we can strictly use the superclasses, and we can use an abstract factory for constructing instances of them (each family gets a "factory", which extends the abstract factory by implementing the constructors; one of these factories is used at runtime).

I haven't written much code like this, so my POV is pretty narrow here, but it seems like a viable technique with few alternatives.
[&#8617;](#abstract-factory)

### Bridge<a name="bridge-commentary"></a>

This pattern is basically just a factoring-out of functionality from one class structure to another, then using an instance from the latter class structure as a member of the former.
[&#8617;](#bridge)

### Command<a name="command-commentary"></a>

This is a use-case for the Strategy pattern.
It can be viewed as a more-extensible alternative to higher order functions.

> "It's hard to associate state with a function. For example, a function that changes
> the font needs to know which font."

This is only true when the language doesn't support closures.

There are some advantages of using an object over just a function, though, like the ability to have mutable state and multiple methods.
Objects also work nicely in the global-undo use-case.
[&#8617;](#command)

### Iterator<a name="iterator-commentary"></a>

There isn't too much to say here: this pattern is ubiquitous.
The precise mechanics, as laid out by the book (as a class with four methods: `First`, `Next`, `IsDone`, and `CurrentItem`), are a bit sloppy, at least as compared to modern languages.
But you must work with what you're given.

It's useful to view iterators as streams (lazily computed, potentially infinite[^infinite] lists), which can then be transformed (e.g. filtering and mapping): this can be done quite naturally in most modern languages.
[&#8617;](#iterator)

[^infinite]: Iterators need not be lazy or infinite to apply these operations, but it's often a useful option.

### Visitor<a name="visitor-commentary"></a>

This pattern is the solution to avoiding type-based dispatching in composite (polymorphic tree) traversals.
I've also seen this a lot in rust libraries and the rustc source code.
I really should have used this in some of my [earlier](/projects/computer-algebra-system/) ["compiler"](/projects/graphing-calculator) [projects](/blog/rewriting-a-toy-compiler).

ADTs are still arguably safer for this sort of operation (as they require a lot less boilerplate and are generally more compact and have more compiler guarantees[^adt-guarantees])

[^adt-guarantees]: The methods of the "visitor" class rely on manual naming and remembering to call the right method, and not forgetting to recurse. ADTs force destructuring, and directly use the type name, so it's harder to mess up with them on that front.

 One potential pro of this approach over ADTs is code repetition (once the boilerplate is set up): if the visitor inherits recursive methods from the superclass, writing a visitor that only handles one subclass can be very succinct (a single method).
The same might not be true with the ADT approach, assuming helpers aren't set up.
[&#8617;](#visitor)

### Builder<a name="builder-commentary"></a>

I see this pattern used a lot in rust libraries as an easy means of constructing objects with mostly default fields[^builder].
The book displays much more algorithmic use-case: using a builder as a superclass, where multiple subclasses have different implementations for the accumulation of the supplied properties.
[&#8617;](#builder)

[^builder]: The methods effectively provide documentation, because they label each of the fields (as opposed to having a lengthy constructor call with unlabeled arguments).

### Factory Method<a name="factory-method-commentary"></a>

This is a narrower case of the Abstract-Factory pattern (with only a single constructor, in the form of a method rather than a member object (whose dynamic swap requires a different subclass rather than a different instance variable)).
[&#8617;](#factory-method)

### Prototype<a name="prototype-commentary"></a>

This one was a lot simpler than I expected: it's object cloning.
That's it.

This is a native feature of JS (replacing inheritance), which is a swimming example of its safety and intuition when used ubiquitously.
It's still safe and practical in smaller and more-contained cases, though.
[&#8617;](#prototype)

### Singleton<a name="singleton-commentary"></a>

This seems to be a common and well-known pattern.
It's basically global variables + OOP syntax.

The liberal use of global state is usually a bad thing, but in moderation, this can be a useful (enforcing that it remains a singleton adds some safety, at least along one dimension).
[&#8617;](#singleton)

## General Thoughts

- useful, in moderation
    - use when necessary, but not otherwise
    - e.g. industrial fizz buzz
