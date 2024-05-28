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

## Creational Patterns

<ul>
<li>Abstract Factory
<ul>
<li>An interface of constructor methods, which can be implemented by multiple "factories" who fulfill those constructors with their respective families of objects</li>
<li>This allows dynamic swapping of factories (and therefore dynamic selection of constructed families)</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

The book makes a compelling case for this one: abstract factories can be used as a mechanism for generalizing gui-widget-constructors: for example, say you have multiple families of gui widgets (button, scrollbar, window, etc), that all implement the same superclasses (or can be adjusted to do so).
Instead of mentioning any specific family in the application code, we can strictly use the superclasses, and we can use an abstract factory for constructing instances of them (each family gets a "factory", which extends the abstract factory by implementing the constructors; one of these factories is used at runtime).

I haven't written much code like this, so my POV is pretty narrow here, but it seems like a viable technique with few alternatives.
</details></li>
</ul></li>

<li>Builder
<ul>
<li>A class with methods that accumulates properties, which can then be "built" into a final result</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

I see this pattern used a lot in rust libraries as an easy means of constructing objects with mostly default fields[^builder].
The book displays much more algorithmic use-case: using a builder as a superclass, where multiple subclasses have different implementations for the accumulation of the supplied properties.

[^builder]: The methods effectively provide documentation, because they label each of the fields (as opposed to having a lengthy constructor call with unlabeled arguments).
</details></li>
</ul></li>

<li>Factory Method
<ul>
<li>A virtual/abstract method that constructs an object</li>
<li>The implementation can dynamically swap what object is constructed</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This is a narrower case of the Abstract-Factory pattern (with only a single constructor, in the form of a method rather than a member object (whose dynamic swap requires a different subclass rather than a different instance variable)).
</details></li>
</ul></li>

<li>Prototype
<ul>
<li>Using an instance of an object as a template to copy</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This one was a lot simpler than I expected: it's object cloning.
That's it.

This is a native feature of JS (replacing inheritance), which is a swimming example of its safety and intuition when used ubiquitously.
It's still safe and practical in smaller and more-contained cases, though.
</details></li>
</ul></li>

<li>Singleton
<ul>
<li>A class with exactly one instance</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This seems to be a common and well-known pattern.
It's basically global variables + OOP syntax.

The liberal use of global state is usually a bad thing, but in moderation, this can be a useful (enforcing that it remains a singleton adds some safety, at least along one dimension).
</details></li>
</ul></li>
</ul>

## Structural Patterns

<ul>
<li>Adapter
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>
</details></li>
</ul></li>


<li>Bridge
<ul>
<li>A superclass contains a member that holds dynamic implementation details</li>
<li>The subclass structure need not account for those details</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This pattern is basically just a factoring-out of functionality from one class structure to another, then using an instance from the latter class structure as a member of the former.
</details></li>
</ul></li>

<li>Composite
<ul>
<li>A subclass contains a member that is a list of the superclass</li>
<li>Any instance of the superclass can be viewed as a tree, whose nodes all have the same interface</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

ADTs are usually a better alternative to this.
The book does well explaining why: The composite pattern...

> "can make your design overly general. The disadvantage of making it easy to add new components is that it makes it harder to restrict the components of a composite. Sometimes you want a composite to have only certain components. With Composite, you can't rely on the type system to enforce those constraints for you. You'll have to use run-time checks instead."

ADTs also allow dispatch and unwrapping of children via case-statements (instead of unsafe casting), which is almost always necessary.
ASTs in compilers are a quintessential use-case: see [this blog post](/blog/rewriting-a-toy-compiler) for a comparison of the two tactics.

This pattern does offer more flexibility than the functional equivalent, though, or at least it feels more dynamic: since every node fulfills the superclass interface, it can be put anywhere a node is accepted (the same is typically not true for ADT trees); it's also more natural (though less safe) to mutate and pass around pointers in this way (during assembly: the opposite is true in disassembly).

The book is quick to admit these faults.
</details></li>
</ul></li>

<li>Decorator
<ul>
<li>A wrapper type that subclasses the type it wraps (the wrapped type can also be a wrapper)</li>
<li>This allows for composition of wrapper-method augmentations</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This is a less-generic case of the composite pattern (a subset of it, where the superclass list has exactly one element).
Because of this, the book highlights the decorator as a composition tool rather than a structuring mechanism.
Nevertheless, most of the pros and cons remain the same.
</details></li>
</ul></li>

<li>Facade
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>


<li>Flyweight
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>


<li>Proxy
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>
</ul>

## Behavioral Patterns

<ul>
<li>Chain of Responsibility
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>


<li>Command
<ul>
<li>Encapsulate an "action" (a (potentially) stateful function or set of functions) into a class</li>
<li>This "command" can then be used by a caller that has no knowledge of its state or purpose</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This is a use-case for the Strategy pattern.
It can be viewed as a more-extensible alternative to higher order functions.

> "It's hard to associate state with a function. For example, a function that changes
> the font needs to know which font."

This is only true when the language doesn't support closures.

There are some advantages of using an object over just a function, though, like the ability to have mutable state and multiple methods.
Objects also work nicely in the global-undo use-case.
</details></li>
</ul></li>

<li>Interpreter
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>Iterator
<ul>
<li>An interface for iterating through a collection</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

There isn't too much to say here: this pattern is ubiquitous.
The precise mechanics, as laid out by the book (as a class with four methods: `First`, `Next`, `IsDone`, and `CurrentItem`), are a bit sloppy, at least as compared to modern languages.
But you must work with what you're given.

It's useful to view iterators as streams (lazily computed, potentially infinite[^infinite] lists), which can then be transformed (e.g. filtering and mapping): this can be done quite naturally in most modern languages.

[^infinite]: Iterators need not be lazy or infinite to apply these operations, but it's often a useful option.
</details></li>
</ul></li>

<li>Mediator
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>Momento
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>Observer
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>State
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>Strategy
<ul>
<li>Use an instance of a class for its v-table (simulating higher-order functions)</li>
<li>Instances of different subclases can provide varying (interchangable) behavior</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

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
</details></li>
</ul></li>

<li>Template Method
<ul>
<li>TODO</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

</details></li>
</ul></li>

<li>Visitor
<ul>
<li>A strategy instance ("visitor") is passed into a "accept" method in the superclass, whose implementation calls the strategy method corresponding to the respective subclass (with the `this` pointer)</li>
<li>This refactors type-dispatch into the "accept" method (via polymorphism), where the case-handling code goes into the strategy ("visitor")</li>
<li>Recursion can optionally be refactored into the "visitor" (strategy) superclass or "accept" method</li>
<li class="no-bullet"><details>
<summary>(commentary)</summary>

This pattern is the solution to avoiding type-based dispatching in composite (polymorphic tree) traversals.
I've also seen this a lot in rust libraries and the rustc source code.
I really should have used this in some of my [earlier](/projects/computer-algebra-system/) ["compiler"](/projects/graphing-calculator) [projects](/blog/rewriting-a-toy-compiler).

ADTs are still arguably safer for this sort of operation (as they require a lot less boilerplate and are generally more compact and have more compiler guarantees[^adt-guarantees])

[^adt-guarantees]: The methods of the "visitor" class rely on manual naming and remembering to call the right method, and not forgetting to recurse. ADTs force destructuring, and directly use the type name, so it's harder to mess up with them on that front.

 One potential pro of this approach over ADTs is code repetition (once the boilerplate is set up): if the visitor inherits recursive methods from the superclass, writing a visitor that only handles one subclass can be very succinct (a single method).
The same might not be true with the ADT approach, assuming helpers aren't set up.
</details></li>
</ul></li>
</ul>

## General Thoughts

- useful, in moderation
    - use when necessary, but not otherwise
    - e.g. industrial fizz buzz
