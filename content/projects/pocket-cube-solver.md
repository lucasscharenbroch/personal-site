---
title: "Pocket Cube Solver"
subtitle: "2x2x2 Rubik's Cube solver and visualizer (C++, WASM)"
date: 2023-02-05T14:37:08-05:00
---

# Pocket Cube Solver

<iframe src="https://lucasscharenbroch.github.io/PocketCubeSolver/"></iframe>

### ([Github Link](https://github.com/lucasscharenbroch/PocketCubeSolver))

## The Solving Algorithm

I've always been intrigued by the difficulty of solving a [Rubik's Cube](https://en.wikipedia.org/wiki/Rubik's_Cube). As a young child, I desperately failed at solving even the first layer which, though I didn't bother to realize it, would have been the easiest part. Several years later, with the help of the internet, I was able to successfully solve a 3x3, and with casual practice over a few weeks, I was able to bring my time down to a little under 2 minutes.

I had little consistency, though, and since I was still using the beginner method, the clear path to cutting my time was memorizing and drilling tens of "algorithms" (move combinations) for the second and third layers. I, of course, gave up at this point, because playing video games was much more fun than memorization.

Thus the problem of quickly solving a cube was on my mind when I first started programming. Since my favorite part of programming is trivializing obscenely difficult problems, I naturally thought of the Rubik's Cube when brainstorming project ideas. I had a few early (but hopeless) attempts at writing a solver; I managed to come up with some ideas of how to *represent* the cube in a programatic structure, but I didn't yet understand a way to approach core problem of solving, and I could tell that the (brute-force) ideas I had would doubtfully work, even if I could manage to implement them successfully.

When I was in high school, I first discovered Data Structures and Algorithms through [leetcode](https://github.com/lucasscharenbroch/LeetcodeSolutions), and I later came across [MIT's 6006: Introduction to Algorithms](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/). I ended up watching the course lectures around 3 times over the next few years, with increasing understanding. The course logo contains a pocket cube (2x2 Rubik's Cube), which immediately reminded me of my quest to programatically solve a Rubik's Cube, but I didn't make an attempt until I had cemented my knowledge of basic Graph Theory, in late 2022.

The problem is actually pretty trivial when thought of as a graph. Imagine each state of the cube as a node. We can add an edge between each pair of states that are different by one turn. The solve with the fewest moves is the shortest path from the current state to any solved state (there are 6: one for each orientation of the cube). The shortest path can be found with Breadth-First-Search from the current state, as each edge is assigned equal (e.g. 1) weight. BFS alone is a bad idea, however, as the search-space is exponential (there are 42e18 = 42 quintillion permutations of a 3x3 rubik's cube, and each new breadth multiplies the fronteir by the number of possible turns). We can use Meet-in-the-Middle (BFS from the initial state and from the destination) to cut the exponent in half. [It's been proven](https://www.cube20.org/) that any permutation can be solved in 20 moves or less, so with 12 possible moves (6 faces, turned in both directions), the total number of explored states with a vanilla BFS is order of 12^20 which is about 4e21, which, assuming a modest 1e8 operations per second, is 4e13 seconds, more than a million years. With Meet-in-the-Middle, the total number of states explored is order of 12^10 (10 breadths from each state) which is around 6e10, around 600 (pessemistic) seconds = 10 minutes. This is still somewhat steep for a casual program (which is why I (and MIT) opted for the 2x2 version (which takes no more than 11 turns to solve any permutaion)), but at least it terminates within this millennium.

Though the algorithm is relatively straightforward, it was a little tricky to implement in a concise way, as I couldn't find a convenient representation of states of states of a cube. In any 2d matrix arrangement (mesh) of the cells,
some of the 12 possible moves cause non-intuitive color shifts (look at the 2d mesh and press the turn buttons: no matter the orientation, some moves cause color swaps across non-contiguous sections of the mesh) on any 2d array representation, and graph/tree/list representations cause inefficiency with the overhead of extra pointers and memory allocation.

In the end, I opted for a messy-looking set of recursively-called helper functions that perform bitwise operations on the cube's faces. This proved to be a nightmare to debug, as minor errors were tricky to spot during testing, because the 2x2 is so small.

{{< highlight cpp >}}
/* ~ ~ ~ ~ Turning Methods ~ ~ ~ ~ */
// rotates U face clockwise (and bordering cells on L, F, R, and B)
void PocketCube::turnU() {
    rotateC(U);

    shiftUpperCells(L,
    shiftUpperCells(F,
    shiftUpperCells(R,
    shiftUpperCells(B, (byte) (state[L] >> 8)))));
}

/* ... */

void PocketCube::turnLP() {
    rotateCC(L);

    shiftLeftCells(U,
    shiftLeftCells(F,
    shiftLeftCells(D,
    invert(shiftRightCells(B, invert((byte) (state[U] >> 4)))))));
}

/* ... */
{{< /highlight >}}

The high-level search algorithm itself is also convoluted due to similar implementation difficulties; perhaps that is why I opted to search from only one (of the six) solved states (I'm not sure if I realized at the time that the constant factor of 6 would be well worth it, as in nearly all cases, a few moves can be saved by going to a closer solved state, which results in a lowering of the breadths searched, which is in the exponent).

## Graphics

When writing the graphics for this program, I had just finished my Calc II course (whose full name is "Calculus and **Analytic Geometry** II"), in which I learned the (very) basics of Computational Geometry. I had previously heard of Raycasting in terms of old video games, so I was able to put this together without having to read any mathematical formulas online (phew!). I probably should have, though, as the projection is far from pretty. The resolution is pretty low, because anything much higher would be terribly slow (see below), and the borders on the cube are inconsistent because they are drawn using a geometric hack (face pixels are drawn as black if the ray's intersection point with the cube is almost equidistant the two cube corners closest to that point), which is why the centers of the faces have a bold cross, while the middles of the lines fade out when turning the cube (the distance difference shrinks moving towards the inside of the cube). In retrospect, I realize that this could be fixed by calculating distance from the horizontal/vertical edges instead of the corners (this wasn't my first thought, because the distance from the corners was already being considered in determining the pixel's color).

## Web Assembly and Optimization

This project was my first time using web-assembly and [emscripten](https://emscripten.org/) (a c++ to WASM compiler), and it turned out to be pretty easy after the initial debugging stage. The source compiles into a .WASM and .js file pair (instead of the usual .out file). There a few catches, namely that all C++ functions called from Javascript must be specified in the build options. I also ran into an issue with the heap overflowing because the search algorithm makes so many allocations, but the heap size can also be changed in the build options. I'm not sure how much faster WASM actually is (as compared to JS), and I'm not eager to try to write anything significant in Javascript to find out, but the convenience of C++ alone sold me on using WASM.

Even after full compiler optimization, the graphics still felt really sluggish (my shoddy raycasting implementation is probably to blame here). After several test runs, I found that the bottleneck was the lineIntersection subroutine, which is called by the raycasting algorithm for each pixel.

{{< highlight cpp >}}
// lineIntersection: computes the point of intersection between *this and l; if they do not
// intersect, Point::notAPoint is returned.
Point Rect::lineIntersection(const Line& l) const {
    Point planeIntersection = plane.lineIntersection(l);
    if(planeIntersection == Point::notAPoint) return Point::notAPoint;

    // dot product of point and all sides
    const double dot1 = Vector{corners[0], corners[1]}.dot(Vector{corners[0], planeIntersection});
    const double dot2 = Vector{corners[1], corners[2]}.dot(Vector{corners[1], planeIntersection});
    const double dot3 = Vector{corners[2], corners[3]}.dot(Vector{corners[2], planeIntersection});
    const double dot4 = Vector{corners[3], corners[0]}.dot(Vector{corners[3], planeIntersection});

    // ensure that the point lies within the rect
    if(dot1 < 0 || dot2 < 0 || dot3 < 0 || dot4 < 0) return Point::notAPoint;

    /* ... Set color ... */
}
{{< /highlight >}}

Removing the dot-product lines caused the program's lag to disappear, so the extra work of allocating 12 Vector objects (though they only consisted of 3 doubles each) was dragging the program down significantly. Adding the "inline" modifier to Vector's constructor and the dot function fixed the problem. Moral of the story: heap allocation vs stack allocation makes a big difference, especially in nested loops.
