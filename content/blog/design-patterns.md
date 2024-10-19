---
title: "Design Patterns"
subtitle: "Notes and commentary on the classic OOP book"
date: 2024-05-30T11:33:05-05:00
notability: 7
tags: ["Pl", "Fp", "Book Review"]
---

{{% center-text %}}
<img src="/images/design-patterns.jpg" alt="The cover of 'Design Patterns' by the 'gang of four'" height="500px"/>
{{% /center-text %}}

## All Hail The Object

_Design Patterns_ is a classic book from the 90s that centers around low-level OOP idioms.
While I'm sometimes quick to criticize OOP[^oop], I still think there's a lot of utility in understanding its principles and learning to work with it.

[^oop]: Usually in favor of FP...

This is partially because OOP is so widely used: even in languages that claim to be multi-paradigm (e.g. Python and Rust), objects play a huge role in code organization.
In order to work with these languages at max-capacity, it's critical to work with the standard idioms, not to fight against them.

But perhaps more importantly, I think there's something inherently effective about OOP: its widespread adoption cannot solely be a fluke.
While there are certainly flaws in thinking in terms of "everything as an object", and the mechanics around that, there are many advantages too, which allow for increased code efficiency and productivity.
My goal is to find those advantages across every paradigm, OOP included.

I've spent the past few days reading the book, and I've synthesized the key details (and some of my thoughts surrounding them) into this post.

## Quotes

Here are some compelling quotes from the introduction of the book: I think they do a nice job motivating the use of design patterns (and also reservations).

> "The hard part about object-oriented design is decomposing a system into objects.The
> task is difficult because many factors come into play: encapsulation, granularity,dependency, flexibility, performance, evolution, reusability, and on and on. They all influence
> the decomposition, often in conflicting ways" (11).

> "That leads us to our second principle of object-oriented design:
> Favor object composition over class inheritance" (20).

> "Design patterns should not be applied indiscriminately
> Often they achieve flexibility and variability by introducing additional levels of indirection,
> and that can complicated design and/or cost you some performance. A design pattern
> should only be applied when the flexibility it affords is actually needed" (31).

The last two quotes highlight the important idea that these patterns must be used in moderation.
The authors hold this sentiment throughout the book, and it voids the vast majority of the criticisms I expected to have.
I'm glad that this didn't turn out to be a manifesto for [Enterprise FizzBuzz](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition).
Anybody who thinks that probably hasn't actually read it.

## Creational Patterns

<ul>
<li>Abstract Factory
<ul>
<li>An interface of constructor methods, which can be implemented by multiple "factories" who fulfill those constructors with their respective families of objects</li>
<li>This allows dynamic swapping of factories (and therefore dynamic selection of constructed families)</li>
<li class="no-bullet"><details>
<summary>...</summary>

The book makes a compelling case for this one: abstract factories can be used as a mechanism for generalizing gui-widget-constructors: for example, say you have multiple families of gui widgets (button, scrollbar, window, etc), that all implement the same superclasses (or can be adjusted to do so).
Instead of mentioning any specific family in the application code, we can strictly use the superclasses, and we can use an abstract factory for constructing instances of them (each family gets a "factory", which extends the abstract factory by implementing the constructors; one of these factories is used at runtime).

I haven't written much code like this, so my POV is pretty narrow here, but it seems like a viable technique with few alternatives.
</details></li>
</ul></li>

<li>Builder
<ul>
<li>A class with methods that accumulates properties, which can then be "built" into a final result</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
<summary>...</summary>

This is a narrower case of the Abstract-Factory pattern (with only a single constructor, in the form of a method rather than a member object (whose dynamic swap requires a different subclass rather than a different instance variable)).
</details></li>
</ul></li>

<li>Prototype
<ul>
<li>Using an instance of an object as a template to copy</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
<summary>...</summary>

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
<li>A wrapper class that converts one interface to another</li>
<li class="no-bullet"><details>
<summary>...</summary>

It's a wrapper.

The OOP version uses inheritance (and a new subtype), but this can be avoided using typeclasses/ad-hoc polymorhpism.

There are advantages of using a wrapper object when the thing being wrapped becomes more complex, though (e.g. contains multiple objects as state).
In this case, typeclasses can still be used, but not without introducing a new type.

- "object adapter": adapter class *contains* the adaptee, inherits from the target, forwarding requests to the adaptee field
- "class adapter": adapter class inherits from both the adaptee and target, forwarding requests to the inherited adaptee methods
</details></li>
</ul></li>


<li>Bridge
<ul>
<li>A superclass contains a member that holds dynamic implementation details</li>
<li>The subclass structure need not account for those details</li>
<li class="no-bullet"><details>
<summary>...</summary>

This pattern is basically just a factoring-out of functionality from one class structure to another, then using an instance from the latter class structure as a member of the former.
</details></li>
</ul></li>

<li>Composite
<ul>
<li>A subclass contains a member that is a list of the superclass</li>
<li>Any instance of the superclass can be viewed as a tree, whose nodes all have the same interface</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
<summary>...</summary>

This is a less-generic case of the composite pattern (a subset of it, where the superclass list has exactly one element).
Because of this, the book highlights the decorator as a composition tool rather than a structuring mechanism.
Nevertheless, most of the pros and cons remain the same.
</details></li>
</ul></li>

<li>Facade
<ul>
<li>An interface that unifies and abstracts a complex subsystem</li>
<li class="no-bullet"><details>
<summary>...</summary>

It's a bit of a stretch to consider this a design pattern: it's basically the definition of "abstraction", plus an implied context of OOP.
</details></li>
</ul></li>


<li>Flyweight
<ul>
<li>Use a set of shared objects to avoid the costs of excess instantiation</li>
<li class="no-bullet"><details>
<summary>...</summary>

("Flyweight" is the lightest weight-class in fighting sports, opposite to "heavyweight".)

This is effectively just an optimization via caching.

While caching in general is certainly useful, this pattern encourages using caching at a higher level than the ideal.

The book uses the example of sharing `Character` objects, which each have a single method: `Draw()`.
We'll assume that these objects are very expensive to store (perhaps they contain lots of font information).
Since these `Character` objects are shared, they (by necessity) "cannot make assumptions about the context in which they operate" (they can only contain "intrinsic" (non-context-specific) state, and must be passed their extrinsic (context-specific) state).

I would argue that the ability to hold extrinsic information is one of the primary advantages of using objects in the first place, and that one of the following would be a cleaner and safer way to implement this:

1. Make a singleton class with a `DrawCharacter()` method, which takes (as parameters) the character to draw and the extrinsic information, and handles the caching internally.
    - This removes the potential for mistaking `Character`s as value-types, and factors out the entire caching process from the user's (the caller of `Draw`'s) code.
2. Turn the `Character` class into a proxy for the shared data and an adapter/facade for the global caching mechanism.
    - The wrapper can contain a pointer to the shared object or the minimal information necessary to retrieve it (or make calls that implicitly use it, e.g. `DrawCharacter()` from the singleton).
    - The wrapper can contain extrinsic information, and abstracts all caching: from the user's perspective, `Character` is a value type.
    - The wrapper can have multiple methods, and fulfills all of those OOP conventions you so strongly desire.

Again, all of this only applies if the shared class really is expensive, and can't be represented with a value type (or there's some other reason to link the instances (although that probably wouldn't be considered the same pattern anymore)).

Perhaps this is implied by the book (and left out because it makes the pattern more complicated), but I am compelled to point it out, as the idea of shared objects (except when absolutely necessary) is appalling to me.

"Sharing terminal symbols with the Flyweight pattern. ... In the Motivation example, a sentence can have the terminal symbol dog (modeled by the LiteralExpression class) appearing many times."
I believe this process can be referred to as "interning", and it's widely used in compilers.
</details></li>
</ul></li>


<li>Proxy
<ul>
<li>A wrapper class that (usually) defers or transforms its requests before forwarding them</li>
<li class="no-bullet"><details>
<summary>...</summary>

This is a special case of the Adapter pattern with an emphasis on transformation/augmentation of input (rather than strict translation/forwarding).
Proxies usually have the same interface as their target, which is not true for adapters.

Proxies could also be viewed as narrower cases of Facades, as they serve as a layer of abstraction.
</details></li>
</ul></li>
</ul>

## Behavioral Patterns

<ul>
<li>Chain of Responsibility
<ul>
<li>A linked list of event-handlers</li>
<li class="no-bullet"><details>
<summary>...</summary>

This one is pretty straight-forward.
A signal is propagated across objects.

More complexity (like conditional propagation/handling can easily be added by changing the propagation code (which can be inherited from a `Handler` superclass, or written manually)).
</details></li>
</ul></li>


<li>Command
<ul>
<li>Encapsulate an "action" (a (potentially) stateful function or set of functions) into a class</li>
<li>This "command" can then be used by a caller that has no knowledge of its state or purpose</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
<li>Code that interprets a language</li>
<li class="no-bullet"><details>
<summary>...</summary>

I'm not sure how this qualifies as a design pattern, any more than other types of programs (say, "arithmetic library", "database", or even "react app"[^react-app]).

[^react-app]: I'd be willing to bet that most of us have written more react apps than interpreters.

Perhaps DSLs/macros/structural-data is common enough within code to consider this different.

I suppose the structure of interpreters is usually relatively uniform, so it was easy to jump on.
</details></li>
</ul></li>

<li>Iterator
<ul>
<li>An interface for iterating through a collection</li>
<li class="no-bullet"><details>
<summary>...</summary>

There isn't too much to say here: this pattern is ubiquitous.
The precise mechanics, as laid out by the book (as a class with four methods: `First`, `Next`, `IsDone`, and `CurrentItem`), are a bit sloppy, at least as compared to modern languages.
But you must work with what you're given.

It's useful to view iterators as streams (lazily computed, potentially infinite[^infinite] lists), which can then be transformed (e.g. filtering and mapping): this can be done quite naturally in most modern languages.

[^infinite]: Iterators need not be lazy or infinite to apply these operations, but it's often a useful option.
</details></li>
</ul></li>

<li>Mediator
<ul>
<li>An object that directs interactions between a group of objects</li>
<li>The interacting objects only need to know about the mediator (and not the objects they interact with)</li>
<li class="no-bullet"><details>
<summary>...</summary>

This is a necessary pattern when dealing with highly-complected systems (which come up frequently in GUIs).

The DOM, for example, is a mediator.

Functional web frameworks (e.g. Elm) embrace this even more heavily, removing almost all direct connections between objects in favor of using global handlers.
</details></li>
</ul></li>

<li>Momento
<ul>
<li>An object used to store/back-up the encapsulated state of another</li>
<li>Ideally, the momento can only be read/written by the object whose state it represents (maintaining encapsulation)</li>
<li class="no-bullet"><details>
<summary>...</summary>

Object serialization with encapsulation.

As the book points out, this might be hard to enforce in some languages (C++ uses a friend class to do this: I'm surprised those were around back in the day).
</details></li>
</ul></li>

<li>Observer
<ul>
<li>Multiple objects can "subscribe to" (observe) another, and will be notified (a method will be called) when it's updated</li>
<li>To accomplish this, the object being observed maintains a list of its observers</li>
<li class="no-bullet"><details>
<summary>...</summary>

Yet another ubiquitous pattern in complected systems.

The [GTK libraries](https://www.gtk.org/) are littered with this terminology ("notify", "subscribe", etc.)
</details></li>
</ul></li>

<li>State
<ul>
<li>Using polymorphism to form <a href="https://en.wikipedia.org/wiki/Tagged_union">sum types</a></li>
<li>Each variant is a subclass, and can have its own method implementations and associated data</li>
<li class="no-bullet"><details>
<summary>...</summary>

This can be viewed as a refactoring of state data (and the code that dispatches on and handles each state accordingly) up into the class structure.
While it's better than the pre-refactored version, real sum types (e.g. enums in Rust) do this in a much more natural way.

This pattern is the base structure required for the visitor pattern, and it's frequently used in combination with the composite pattern.
</details></li>
</ul></li>

<li>Strategy
<ul>
<li>Use an instance of a class for its v-table (simulating higher-order functions)</li>
<li>Instances of different subclases can provide varying (interchangable) behavior</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
<li>A placeholder method whose implementation is deferred to subclasses</li>
<li class="no-bullet"><details>
<summary>...</summary>

> "Template methods lead to an inverted control structure that'ssometimes referred
> to as 'the Hollywood principle,' that is, 'Don't call us, we'll call you' [Swe85]."


"Hook" is a nice term to refer to this: the superclass suspects that a subclass might want to inject some code in a certain place within the superclass's (pre-defined) methods, so it exposes a virtual template method that gets called in that place and can be overridden to add perform that injection.

This pattern is used frequently in big systems for easy extension.
Two examples from things I've worked on are the TI-8X calculators (which expose hooks as a way of triggering code upon certain operating system events[^ti]) and [Anki](https://github.com/ankitects/anki) (which uses hooks as the primary mechanism for implementing add-ons).

[^ti]: This could also be viewed as an instance of the observer pattern, where there is only a single observer, and it is notified via calling a single function pointer.
The book/OOP-POV also implies that the template method is used in the context of inheritance/subtyping.

Curiously, Haskell and Rust come close to allowing "inheritance" through template methods in typeclasses/traits.
That is, typeclasses-functions can be defined in terms of other type-class functions, and instances can fulfill the typeclass by implementing a certain subset of those functions (those implemented functions can be viewed as the template methods).
</details></li>
</ul></li>

<li>Visitor
<ul>
<li>A strategy instance ("visitor") is passed into a "accept" method in the superclass, whose implementation calls the strategy method corresponding to the respective subclass (with the `this` pointer)</li>
<li>This refactors type-dispatch into the "accept" method (via polymorphism), where the case-handling code goes into the strategy ("visitor")</li>
<li>Recursion can optionally be refactored into the "visitor" (strategy) superclass or "accept" method</li>
<li class="no-bullet"><details>
<summary>...</summary>

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
