---
title: "RS-Paint"
subtitle: "A light-weight image editor (Rust)"
date: 2024-07-27T10:19:56-05:00
notability: 8
loc: 12000
draft: true
---

{{% center-text %}}
<img src="/images/rs-paint/rs-paint.png" alt="RS-Paint program screenshot" width="850px"/>
{{% /center-text %}}

### ([Github Link](https://github.com/lucasscharenbroch/rs-paint))

## Motivation

I wrote this program to be "<u>*my* ideal image editor</u>".
I don't edit images very often, and when I do, I often only reach for a handful of simple operations.
I want those operations to be **intuitive and highly ergonomic[^ergonomic]** (1).
That being said, I still need an editor that's **relatively powerful** (2) so I won't be held back by a lack of features.
Finally, I expect my editor to be **free and natively compatible with Linux** (3).

[^ergonomic]: I won't get into the details of what really qualifies as "ergonomic", as it's highly subjective and not a very interesting subject to reason about logically.
It'll suffice to say that I've shaped the program to my preferences for UI.

I've a tried few mainstream editors, and each of them fails on at least one of these criteria.

- **Gimp** is quite powerful, but it's a far cry from ergonomic[^gimp-not-ergo] (1).
- **MS-Paint** is relatively ergonomic for simple drawing, but it has a clumsy interface in other ways (1),
lacks several fundamental features (namely transparency and magic wand) (2), and only runs on Windows (3).
- **Paint.NET** was my editor of choice on Windows, but doesn't run natively on Linux (3).
- **Photoshop** is not free, and I don't think it runs on Linux (3).

[^gimp-not-ergo]: Gimp's UI is notoriously unintuitive, and often involves way more clicks than necessary.
[Here's an Eric Murphy video that demonstrates exactly what I mean](https://youtu.be/nHQv4blla7g?si=Z0866gsd4OgKag4K&t=352)

Thus I set out to write my own editor, RS-Paint[^name].

[^name]: The name "RS-Paint" is a play on the names of the subpar editors that inspired it.
It's only one letter away from "MS-Paint", and, like "Paint.NET", it contains a reference to the tooling that was used to create it ("RS" for "Rust").
Perhaps this isn't very original, but I figured it more clever than yet another random fake-word or word-play like "imigia" or "photon", or whatever.

## Project Nature

This is officially the biggest project I've written.
It's 12k LoC.

Much of what I wrote was relatively straight-forward, allowing bursts of very rapid development.
This was really satisfying, and It's helped me better understand why people like web development and ui-coding (and other high-volume, low-effort tasks).

Another curious aspect of this project is that there was virtually no completion criteria, and the minimal functionality was laid out very early on.
This made the development process revolve around adding and improving rather than solely implementing a giant skeleton[^skeleton].
This has a vastly different feel, and it was refreshing.

One thing that arose from this (combined with the other logistics of this project (namely rust and GTK)) was a large volume of sub-ideal ("hacked") solutions.
This project opened my eyes to how [there is little use in avoiding these](/blog/hacking-is-necessary).

[^skeleton]: "Implementing a giant skeleton" is an accurate description of all of my compiler-related projects (which follow the same skeleton: lex, parse, evaluate).

## Problems : Solutions : Commentary

Here I'll discuss some of the more interesting problems/challenges I faced throughout development.
I'll order them in nonincreasing order with respect to the key:

```
actual_complexity * (1.0 / expected_complexity)
```

That is, the most complex (especially the unexpectedly complex) will come first[^ordering].

[^ordering]: I choose this ordering because I suspect the few people who open this post will likely not get too far (I don't blame them; I wouldn't either), so I might as well peak curiosity early on.

### Image Storage, Layering, Undo

It turns out that all of the following are inherently interweaved.

1. Storing and updating the image (as the user modifies it)
2. Maintaining multiple layers
3. Maintaining the image's history
4. Drawing the image efficiently

(4) is probably the least obvious.
It turns out that cairo (the drawing library I used) must be given its images-to-draw in a particular format.
For efficiency, we'd like that format to line up exactly with how we natively store the image so a raw pointer can be given to cairo to avoid conversion on every draw[^inefficient].
But the format it demands ([pre-multiplied alpha](https://en.wikipedia.org/wiki/Alpha_compositing#Straight_versus_premultiplied)) is seriously undesirable[^lossy] as a native image format.

[^inefficient]: Which would obviously be incredibly inefficient.

[^lossy]: This is because color values are stored as integers (one byte per color channel; I chose this arbitrarily, but decided to stick with it for efficiency), so using storing colors with pre-multiplied alpha is pretty lossy.

The solution is to maintain two image structs: one regular **`Image`** (to be read and written as normal), and one **`DrawableImage`** to be directly given to cairo.
The two can then be wrapped in a **`FusedImage`**, which forwards writes to both the **`Image`** and **`DrawableImage`**.

Adding layers is a matter of having an **`Image`** and **`DrawableImage`** per ever layer, and adding machinery into **`TrackedLayeredImage`** that handles the lazy blending of pixels into the global **`DrawableImage`**.

This machinery is intertwined with change-tracking code that allows for undo-saves (more on undo later).

The resulting code:

```rs
pub struct Pixel {
    // the order of the fields is in the unsafe cast in Image::to_file
    r: u8, g: u8, b: u8, a: u8,
}

pub struct DrawablePixel {
    // order of the fields corresponds to cairo::Format::ARgb32
    // (this struct is used for directly rendering the cairo pattern)
    b: u8, g: u8, r: u8, a: u8,
}

pub struct Image {
    pixels: Vec<Pixel>,
    width: usize,
    height: usize,
}

pub struct DrawableImage {
    pixels: Vec<DrawablePixel>,
    width: usize,
    height: usize,
}

pub struct LayerProps {
    layer_name: String,
    locked: bool,
    visible: bool,
}

struct Layer {
    image: Image,
    props: LayerProps,
}

pub struct FusedLayer {
    image: Image,
    drawable: DrawableImage,
    props: LayerProps,
}

pub enum LayerIndex {
    BaseLayer, // The bottom layer
    Nth(usize), // The (n + 1)'th from bottom layer (0 = first from bottom)
}

pub struct FusedLayeredImage {
    drawable: DrawableImage,
    base_layer: FusedLayer,
    /// Non-base layers, increasing in height
    other_layers: Vec<FusedLayer>,

    active_layer_index: LayerIndex,

    // Only one layer is active at a time:
    // the below keep track of changes made to
    // the currently-active layer
    pix_modified_since_draw: HashMap<usize, Pixel>,
    pix_modified_since_save: HashMap<usize, (Pixel, Pixel)>,
}

/// An interface of `FusedLayeredImage` that only exposes
/// undoable operations (used by `DoableAction`)
pub trait TrackedLayeredImage {
    fn pix_at(&self, r: i32, c: i32) -> &Pixel;
    fn pix_at_mut(&mut self, r: i32, c: i32) -> &mut Pixel;
    fn try_pix_at(&self, r: i32, c: i32) -> Option<&Pixel>;
    fn try_pix_at_mut(&mut self, r: i32, c: i32) -> Option<&mut Pixel>;
    fn width(&self) -> i32;
    fn height(&self) -> i32;
}
```

The **`pix_modified_since_draw`** and **`pix_modified_since_save`** hash-maps are used for lazily updating the **`drawable`** and lazily accumulating changes, respectively.
**`pix_modified_since_draw`** is populated on calls to **`[try_]pix_at_mut`**, and **`pix_modified_since_save`** is populated when **`pix_modified_since_draw`** is flushed (in the accessor to **`drawable`**).
This is one many possible schemes for accomplishing the same result.
This particular arrangement is good at avoiding unnecessary computations (because of its laziness).

### Drawing

Given a set of points that the cursor followed, create a smooth path across those points, and rasterize that path.

The two core difficulties here are:

1. Path creation
2. Rasterization

(1) is hard because the input path is, in pathological cases, jagged (the sampling rate is never fast enough to track the sufficiently fast user's cursor, so the resulting point-trace ends up being un-rounded).

One possible solution is to ignore the problem and live with the jagged edges (MS-Paint does this).

{{% center-text %}}
<figure>
<img src="/images/rs-paint/ms-paint-jagged-edges.jpg" alt="MS-Paint pencil pencil stroke with jagged edges" width="850px"/>
<figcaption style="text-align:center">Bad (straight segments)</figcaption>
</figure>
{{% /center-text %}}

{{% center-text %}}
<figure>
<img src="/images/rs-paint/rs-paint-smooth-edge.jpg" alt="MS-Paint pencil pencil stroke with jagged edges" width="850px"/>
<figcaption style="text-align:center">Good (spline)</figcaption>
</figure>
{{% /center-text %}}

The other approach is to assume that quickly-drawn strokes are probably meant to be round, and to take the given points and compute a smooth curve that connects them.
[Splines](https://en.wikipedia.org/wiki/Spline_(mathematics)) are the mathematical formula to approximate a polynomial connecting several points, so they're a natural solution to this problem.
The drawback is that more than two points are needed to draw a non-linear spline, and waiting for a third point adds a bit of a lag to the display of the line.

{{% center-text %}}
<figure>
<img src="/images/rs-paint/pencil-lag.gif" alt="Pencil stroke lags slightly behind cursor" width="850px"/>
<figcaption style="text-align:center">(1/3 speed)</figcaption>
</figure>
{{% /center-text %}}

Some programs address this by first rendering the jagged line, then, upon release (when the stroke is "committed"), the jagged line is replaced by the smoothed one.
I prefer the (very slight) lag over the re-render, so I chose to keep it.

(2) is a little more obvious: given the formula for the curve, how do we draw it onto the image?
This naturally leads to the question: "what is a brush?"

I chose the maximally trivial definition: "a brush is an image".
And therefore strokes are rasterized via periodic sampling (that is, the brush-image is drawn at evenly-spaced points along the curve).
The spacing of the samples is slightly scaled up as the size of the brush increases (to avoid especially slow performance on big brushes).

This sampling strategy leads to brush-samples being drawn on top of each other (or at least partially overlapping).
When the color we're drawing is transparent, this causes the overlapping parts of the samples to be darker.
To fix this, we can keep a mask of drawn-on-pixels for each stroke, and prevent re-drawing pixels we've already touched.

The possibility for (semi-)transparent strokes brings up another issue with rasterization: how should transparent colors be blended?
For example, consider a fully transparent pencil stroke.
There are at least two potential interpretations:

1. The stroke should clear out all opaque pixels in its path
2. The stroke is transparent, and should have no effect

Neither interpretation is wholly correct, so to support both, we'll add the notion of "blending modes".

1. Overwrite: the color of drawn pixels replaces the old color
2. Paint: the color of the drawn pixel is "blended onto" the old color, with respect to transparency; this is the same as as `Overwrite` when the new pixel is opaque
3. Average: all color channels of the drawn color and old color are averaged

The allowance of transparency as "a valid color" (in the `Overwrite` blending-mode) causes a problem with the definition of a brush as "just an image": we need a new notion of "absence of color" other than transparency (else transparent brushes cannot have non-rectangle shape).

The following definition results.

```rs
pub struct BrushImage {
    pixel_options: Vec<Option<Pixel>>,
    width: usize,
    height: usize,
}
```

### Multi-Level Undo

Ever since watching [this legendary talk by Sean Parent](https://www.youtube.com/watch?v=bIhUE5uUFOA) I've been itching to implement multi-undo.
This project poses the perfect opportunity to do so in a non-trivial context.

My implementation is about as vanilla as it gets.
I toyed with using some sort of immutable trie-like data structure[^closure], but it turned out to be a lot tricker than I expected, and I gave up quickly.
I ended up using a tree of diffs.
Getting everything to line up is a little tricky, because nodes of the tree represent diffs, not states, but it's otherwise straight-forward.

[^closure]: [Like the persistent data structures from Clojure](https://www.youtube.com/watch?v=wASCH_gPnDw&t=1818s).

```rs
struct UndoNode {
    parent: Option<Weak<UndoNode>>,
    children: RefCell<Vec<Rc<UndoNode>>>,
    recent_child_idx: RefCell<usize>, // index of the most recently visited child
    value: Rc<RefCell<ImageStateDiff>>,

    // below is gui-related
    widget: gtk::Box,
    label: Label,
    button: Button,
    container: Rc<gtk::Box>, // possibly inherited from parent
}
```

The implementation is a little verbose because pointer-based trees are annoying in Rust.
If you don't know Rust, ignore every instance of **`Weak`**, **`Rc`**, and **`RefCell`**, and pretend it's Java.

Undo involves moving upward into the parent; redo downward (into **`recent_child_idx`**).

Traversing to a specific commit involves finding a least-common-ancestor (I naively used bfs to do this), then applying (or unapplying) the diffs, in order, along the path.

Layers (namely the currently active layer) also have to be tracked by undo.
All operations that impact layers (reorders, merges, clones, layer creation) must be logged as undo actions.
Single-layer modifications (which act on the currently-active-layer) must be constrained to not resize the image, and all resizing operations must operate on all layers.

Many diffs are manually implemented (for time and space efficiency), but automatic change tracking is possible too (see the section on image storage for more on this).
There are a variety of "**`...Action`**" structures/traits that allow wrapping code with a variety of different change-allowing APIs.

```rs
/// An action that uses the automatic undo/redo in `FusedLayeredImage`
/// (exposed through the `TrackedLayeredImage` interface)
pub trait AutoDiffAction {
    fn name(&self) -> ActionName;
    fn exec(self, image: &mut impl TrackedLayeredImage);
    // undo is imlpicit: it will be done by diffing the image
}

/// An action with a manual undo that modifies the image
/// through the `ImageLikeUncheckedMut` interface. The action
/// will only be used on a single layer (and is free to store
/// mutable data tied to that layer).
pub trait SingleLayerAction<I>
    where I: ImageLikeUncheckedMut
{
    fn name(&self) -> ActionName;
    fn exec(&mut self, image: &mut I);
    fn undo(&mut self, image: &mut I); // explicit undo provided
}

/// An action with a manual undo that is given full access
/// to the `Image` (including resizing). The action is
/// executed/undone to each layer individually.
/// It's assumed that both `undo` and `exec` are effectively
/// pure in terms of the size of the input and output image (so
/// layers sizes will not become mismatched).
pub trait MultiLayerAction {
    /// Layer-Specific undo data provided mutably to both
    /// `exec` and `undo`
    type LayerData;
    fn new_layer_data(&self, image: &mut Image) -> Self::LayerData;

    fn name(&self) -> ActionName;
    fn exec(&mut self, layer_data: &mut Self::LayerData, image: &mut Image);
    fn undo(&mut self, layer_data: &mut Self::LayerData, image: &mut Image);
}
```

Every type of history-changing (undoing/redoing/traversing) is an application of diffs which change the entire **`FusedLayeredImage`**.
This means that the <b>`DrawableImage`</b>s must also be updated.
The two naive ways to do this are inefficient in their respective pathological cases:

1. Update all changed pixels/layers, as applied. (inefficient when a large chain of diffs is applied and the intermediate drawable-changes are overwritten before the drawable is rendered)
2. Update all layers, once at the end. (inefficient when a single-layer change is applied to a many-layer image)

My solution is to accumulate (but not apply) all changes (to the drawables) when the diffs are applied, then to apply the accumulated changes once at the end.
Layering makes this a little hairy, because rearranging of layers, mid-accumulation, must be accounted for.

```rs
/// Specifies which drawables/pixels of
/// a `FusedLayeredImage` should be re-computed. This
/// can be used to lazily accumulate changes before updating.
/// The inclusion of any pixel implicitly includes that
/// pixel on the "main-drawable" (`FusedLayerImage::drawable`).
struct DrawablesToUpdate {
    full_layers: HashSet<LayerIndex>,
    /// pixels_in_layers[i] <=> set of flat indices into LayerIndex::from_usize(i)
    pixels_in_layers: Vec<HashSet<usize>>,
    /// If this is `true`, the main drawable will always be updated.
    /// If not, it will be updated if any full layer is included;
    /// if none are, it'll be updated at any pixels in `pixels_in_layers`.
    definitely_main: bool,
}
```

### Bitmask Outlining

{{% center-text %}}
<img src="/images/rs-paint/magic-wand-selection.jpg" alt="A magic wand selection: outlined pixels with the selection shaded" width="850px"/>
{{% /center-text %}}

The drawing of a bitmask's outline (and filling the selected area) is yet another non-trivial problem.

Luckily, I was able to ride off cairo's back[^cairo] for most of the geometric calculations (in this case, the fill[^cairo-fill]).
But cairo never does all of the work.

[^cairo]: As you'll soon see, I rely on cairo quite a bit.

[^cairo-fill]: The fill algorithm used for bitmask is the [even-odd rule](https://en.wikipedia.org/wiki/Even%E2%80%93odd_rule), which (as I understand) draws a line from any given point outward, then counts the intersections of the shape's edges to determine if it's "in the shape" (even), or "out of the shape" (odd).

The outline alone isn't too bad: just look at every pixel and check its borders.
Unfortunately, cairo isn't able to fill a path (shape) that's assembled with the border segments in arbitrary order.

So I used a relatively straight-forward algorithm to, given a bitmask, find the outline (list of points, connected by grid-aligned segments) each distinct "shape".

- Find each segment by scanning each row and column and checking for boundaries
- Place the segments into hash maps so they can be looked up by their endpoints
- Start from an arbitrary endpoint of an arbitrary segment, and repeatedly explore unvisited segments that end at the same point
- Repeat this for each "strongly connected component" (shape) of segments in the bitmask (until there are no unvisited segments)
- The order in which the segments were visited is the order they should be traced in cairo to allow for filling

(see **`image::bitmask::ImageBitmask::gen_edge_path`** for the full code)

This algorithm might fall apart when given pathological bitmasks (e.g. ones with dithered selection), but since all brushes and magic-wand-selections form "nice" shapes, I haven't ran into any problems with it.

If it were a little less sloppy, it would probably make a good leetcode problem.

### Free Transform

I dreaded this problem for a long time, and put it off so it was one of the last tools I implemented.

It turned out to be quite fun[^math] to solve, because my insistance on generality exposed the core of the problem, leading to a relatively simple solution.

[^math]: The non-arithmetic parts were fun, at least.

The idea was that free-transform should work on all of the following:

1. Rectanglular Images
2. Image Bitmasks
3. Shapes
4. Text

(1) and (2) were later combined, but that fact is irrelevant.

So what's the least common denominator of all of these things (that is necessary for free transform)?

1. Sampling: look up a pixel value at a given coordinate[^unit] (used for rasterization)
2. Drawing (display the thing with cairo; kept sepearte for efficiency)

[^unit]: Coordinates are relative to the unit square (between the points (0.0, 0.0) and (1.0, 1.0)) for complete generality: shapes and text are effectively vector until they're rasterized, so it doesn't make sense to assign them any specific height/width, so we use 1.0.

```rs
pub trait Samplable {
    /// Get a pixel value at given (x, y)
    /// (coords should be in the unit square)
    fn sample(&self, x: f64, y: f64) -> Pixel;
}

pub trait Transformable {
    /// Draw the untransformed thing within the unit square: (0.0, 0.0) (1.0, 1.0)
    fn draw(&mut self, cr: &cairo::Context, pixel_width: f64, pixel_height: f64);
    fn gen_sampleable(&mut self, pixel_width: f64, pixel_height: f64) -> Box<dyn Samplable>;

    // (this was later added to allow for different scaling algorithms of selections)
    fn try_image_ref(&self) -> Option<&Image>;
}
```

**`gen_sampleable`**[^spelling] is used instead of an accessor because the underlying data might not be inherently sampleable (and might need extra computation).

[^spelling]: Please forgive the inconsistent spelling of "sampl[e]able": I still haven't made up my mind.
This is one of the cons of really good autocomplete: it's easy to ignore a typo.

From here, it's a matter of simultaneously maintaining translation, scale, and rotation of the transformation, displaying the transformed result, and modifying it with respect to its current transformation.

It's amazing how easy this sounds if you know the trick and how difficult it sounds if you don't[^graphics].

[^graphics]: This sentence is the probably one of the strongest arguments for getting a CS degree.
Sometimes, it's useful to have a certain baseline of knowledge, which would otherwise be hard to obtain through [lazy learning](/blog/lazy-vs-eager-learning).
I remember thinking that it would be useful to wait to start this project until I took a graphics class (UW's CS 559).
That was generally false, with two major exceptions: (1) this, and (2) splines (see above).

This trick is the [transformation matrix](https://en.wikipedia.org/wiki/Transformation_matrix), which (in a nutshell) maintains those three things (translation, scale, and rotation), and makes them easy to apply, invert, and combine (with other matrices).

The implementation also required a few hundred lines of geometry logistics for...

- Determining "what the cursor should do" (translate? scale? (which direction?) rotate?) based on its position relative to the transformation
- Calculating the correct amounts to translate/scale/rotate to (independently)...
    1. Clamp the transformation to the grid
    2. Maintain the aspect ratio of the transformation
- Draw the free-transform UI thing (the box around the transformation, the points on the corners, etc.)

### Shapes and Text

I expected the implementation of **`gen_sampleable`** for shapes and text to be tricky.

For shapes it would be a matter of maintaining a vector-image (which I have no clue how to do), or doing the math to determine if a given point's distance from a line segment[^icpc] (perhaps they're isomorphic).

[^icpc]: Which I learned (from ICPC (training)) is way harder than it sounds.

For text, I would have to find some tool to render a given font as a vector image, then do a point lookup (sample) on that image.

Luckily, I had a tool at my fingertips that could roughly do both: cairo.
I was a little hesitant to do this at first (because it feels a little too much like a hack), but I eventually decided that there's no[^no-shame] shame in the hack, so long as it works well[^hacked].

[^no-shame]: Perhaps "no shame" is a little too strong. There is always some shame, but if the hack is deliberate and calculated, it should be embraced. See my [blog on hacking](/blog/hacking-is-necessary) for more on this.

[^hacked]: So many other parts of this project were already "tainted" (in my POV) by hacking anyway, and I was losing patience, so the hacked option started seeming more and more attractive.

So with a handful of lines, I added a default implementation of **`gen_sampleable`** that rode off the back of cairo to make a **`Samplable`** solely by using the **`draw`** method.
This made implementing both shapes and text as easy as[^shapes-and-text-easy] implenting **`draw`** (to draw the thing to a cairo context).

[^shapes-and-text-easy]: There was more code involved in both (neither was "trivial" after this, per se), but no extra work for drawing or sampling.

```rs
pub trait Transformable {
    fn draw(&mut self, cr: &cairo::Context, pixel_width: f64, pixel_height: f64);
    fn gen_sampleable(&mut self, pixel_width: f64, pixel_height: f64) -> Box<dyn crate::transformable::Samplable> {
        // default implementation: ride off the back of cairo,
        // use the `draw` method, then sample off of the resulting context

        let width = pixel_width.ceil() as usize;
        let height = pixel_height.ceil() as usize;

        let surface = cairo::ImageSurface::create(
            cairo::Format::ARgb32,
            width as i32,
            height as i32,
        ).unwrap();

        let cr = cairo::Context::new(&surface).unwrap();
        cr.scale(pixel_width, pixel_height);

        self.draw(&cr, pixel_width, pixel_height);

        std::mem::drop(cr);
        let raw_data = surface.take_data().unwrap();
        let drawable_image = DrawableImage::from_raw_data(width, height, raw_data);

        Box::new(drawable_image)
    }
    fn try_image_ref(&self) -> Option<&Image> { None }
}
```

## Rust

This is my first major[^major] project using Rust[^new-and-major].
I learned a lot and developed a lot of opinions throughout the process.
The following are some of them.

[^major]: Major = non-trivial ~= >1000 LoC.
I would describe my work with Rust before this as "dabbling".
I would now describe Rust as one of my "primary languages".

[^new-and-major]: Ironically, it's also the biggest project (by LoC) I've done, by far (>12K LoC).
The second biggest project ([APL Interpreter](/projects/apl-interpreter), which was >3500 LoC) was also my first stab at a language (Haskell at that case).
I never intended to make this a habit, but it's proven quite effective.
If my next big project isn't in Rust, it'll likely be in a language I've never used before.

### Learnings

#### Borrow Checking, Basic Semantics (All Hail The LSP)

I think that some degree of eager learning is desirable for Rust[^primagen] (I read the [book](https://doc.rust-lang.org/book/), and I'm glad I did), but, as with most languages, true understanding only comes from using the thing.

[^primagen]: The Primagen says something of the same sentiment [in this video](https://youtu.be/JeoYA7wABAA?si=X-okVi1Y5ZoxF9CJ&t=463).

I owe the speed of my understanding of the Rust's ubiquitous mechanics of Rust to the tight feedback loop of my LSP and IDE.

And the faster you receive feedback from your IDE, the faster you can learn how to use the language.[^preach-to-the-choir]
In almost no time at all, you'll be anticipating what the red underline means before the error text even appears.
Soon after that, the you'll be anticipating a red underline before it even shows up.
This same process happens when you wait for compilation for feedback, but it takes much longer.

[^preach-to-the-choir]: I'm probably preaching to the choir to some extent here (basically everybody uses VSCode nowadays), but I figure that if I convince one stubborn vim user to switch to neovim, I've done a great service.

You can imagine the shrinking of the feedback loop as technology progresses.

- Punch Cards (wait for the operator to hand you the results)
- Teletype (wait for the terminal to type the result)
- Terminal Emulator (wait until you're ready to compile to see the results)
- Modern IDEs (receive the results as you type)

Each step is quite substantial.
Ignoring the last one is naive.

It's hard to imagine what might be next.

#### Interior Mutibility

I finally figured out what **`Rc`** and **`RefCell`** do (after being seriously perplexed when I first saw them in a leetcode problem, then the next hundred times I saw them).

**`Rc<T>`** is a immutable reference-counting pointer.
It's immutable in the sense that you're not allowed to mutably borrow the **`T`** (only immutable borrows are allowed).
Cloning the pointer is allowed, and doing so copies the address and increments the reference count.

**`RefCell<T>`** isn't a pointer at all; it's a "mutable memory location with dynamically checked borrow rules".
This means two things:

1. You can mutate the **`T`** (get a **`&mut T`**) when you only have immutable access to the **`RefCell`** (a **`&RefCell<T>`**)
2. Doing so will cause a run-time panic if **`T`** is (or becomes) borrowed anywhere else

Cloning a **`RefCell<T>`** is only allowed if the **`T`** can be cloned, and the clone will be a new memory location.
This is because (again) **`RefCell<T>`** is not a pointer; it's much like holding onto a **`T`**, just with these special borrowing rules.

**`Rc`** and **`RefCell`** are often used together (as **`Rc<RefCell<T>`**) because their combined behavior is a clonable pointer with interior mutability (which mirrors the behavior of pointers in virtually every other popular modern language).

Since **`Rc<T>`** implements **`Deref<Target = T>`**, methods of **`&T`** can be directly used on the **`Rc<T>`**, and therefore methods of **`RefCell<T>`** can be used on **`Rc<RefCell<T>>`**.

```rs
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let seven: Rc<i32> = Rc::new(7);

    {
        let same_seven = Rc::clone(&seven); // points to the same memory location;
                                            // the reference count is incremented to 2
    } // `same_seven` is dropped, the reference count is decremented to 1

    let num: RefCell<i32> = RefCell::new(1); // note that `num` is NOT declared as mutable...

    // ... despite this, we can still change its internal value
    *num.borrow_mut() = 2;
    println!("{}", num.borrow()); // => 2

    // but `num` is not a pointer, so we can't have shared ownership of it;
    // cloning works, but makes a "deep copy"
    let num_clone = num.clone(); // `num_clone` is an entirely different memory location

    *num.borrow_mut() = 3;
    println!("{}", num_clone.borrow()); // => 2

    // to make a "shallow copy", we need to wrap the RefCell in a Rc
    let num2: Rc<RefCell<i32>> = Rc::new(RefCell::new(4));
    let num2_clone = Rc::clone(&num2);

    // borrow_mut() is a method on RefCell; this call works, because
    // Rc<RefCell<T> can dereference into RefCell<T>
    *num2.borrow_mut() = 5;
    println!("{}", num2_clone.borrow()); // => 5
} // the reference counts of `seven` and (`num2`/`num2_clone`) are decremented to 0, the memory is freed
```

#### Lifetimes

When trying to eagerly learn Rust, I found lifetimes to be very elusive.
I think (much like monads and other similarly complex topics) that they're best understood when you're forced (or encouraged) to use them in context[^you-not-me].

[^you-not-me]: Emphasis on **you** (**you** have to use them, not watch somebody else use them).
This means that the following explanation might not be too helpful, but I'll include it anyway, because it demonstrates thoughts that I had that helped in my understanding.

The situation when I most often reached for lifetimes (besides `'static`) was when I wanted to use a reference in a struct.
Doing so is not allowed without explicit lifetimes.

```rs
pub struct SampleableCommit<'s, 'i> {
    sampleable: &'s dyn Samplable,
    matrix: cairo::Matrix,
    scale_method: ScaleMethod,
    image_option: Option<&'i Image>,
    culprit: ActionName,
}

impl<'s, 'i> AutoDiffAction for SampleableCommit<'s, 'i> {
    fn name(&self) -> ActionName { /* ... */ }
    fn exec(self, image: &mut impl crate::image::TrackedLayeredImage) { /* ...  */ }
}

// code that uses `SampleableCommmit` (simplified)
fn commit_transformable_no_update(&mut self) {
    let selection = /* ... */; // 'i begins (the `Image` is stored somewhere within `selection`)
    let sampleable = selection.transformable.gen_sampleable(width, height); // 's begins
    let commit_struct = SampleableCommit::new(
        &*sampleable, // pass in 's
        selection.matrix,
        self.ui_p.borrow().toolbar_p.borrow().get_free_transform_scale_method(),
        selection.transformable.try_image_ref(), // pass in 'i
        selection.culprit.clone(),
    );

    // consumes `commit_struct` ('s and 'i both alive, so this is fine)
    self.image_hist.exec_doable_action_taking_blame(commit_struct);

    // 's and 'i end
}
```

For example, the above struct (**`SampleableCommit`**) is used to draw a **`Samplable`** to the image.
We want to wrap up all of the information necessary for drawing into a single struct (to allow it to implement **`AutoDiffAction`**), yet we don't want to clone the **`Sampleable`** and the **`Image`** (in this case, we couldn't even if we wanted to[^why-not]).
Since we know that the **`SampleableCommit`** struct will be dropped before the **`Sampleable`** and **`Image`** (`'s` and `'i`), there's no reason we can't wrap the references into a struct.
The lifetimes are how we express this to the compiler.

[^why-not]: We can't because **`&dyn Sampleable`** exposes no method for cloning.

### Opinions

#### Type System

Clumsy at times, but in general, godsend

So many of the right defaults

## GTK

- rust has trade-offs
- gtk is a little odd to use with it, because it's a c library
- interior mutability
    - gtk uses it, which is confusing at first (you're passing around references as if they're pointers)
    - I also used it a lot to get static lifetimes - to pass around pointers to my data structures to gtk hooks
    - perhaps not rustic, but idk if there's a rustic way to make a ui that isn't elm-style (requiring some type of framework)
- also gotta use glib and gio and all that
- there's a push to use ".ui" files and css and all that - to work around it, it's a bit ugly
- sometimes figuring out how to do super low-level stuff in gtk is odd (requries those other libs, often doing runtime and pointer stuff)

## Retrospective

- a project I've wanted to do a long time
- took a long time, was fulfilling - the duality
- frontier of viability
