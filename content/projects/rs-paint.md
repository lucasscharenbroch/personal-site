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
<figcaption style="text-align:center">(~1/3 speed)</figcaption>
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

### Free Transform

### Shapes

### Text

### Copy/Paste

---

## Rust, GTK

- rust has trade-offs
- gtk is a little odd to use with it, because it's a c library
- interior mutability
    - gtk uses it, which is confusing at first (you're passing around references as if they're pointers)
    - I also used it a lot to get static lifetimes - to pass around pointers to my data structures to gtk hooks
    - perhaps not rustic, but idk if there's a rustic way to make a ui that isn't elm-style (requiring some type of framework)
- also gotta use glib and gio and all that
- there's a push to use ".ui" files and css and all that - to work around it, it's a bit ugly
- sometimes figuring out how to do super low-level stuff in gtk is odd (requries those other libs, often doing runtime and pointer stuff)
