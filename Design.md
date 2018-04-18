# Rectindex - a C++11 approach to finding intersecting rectangles

----

## Purpose

- Rectindex stores rectangles.
- It facilitates just one kind of query:
  - Given a query rectangle,
  - Find all stored rectangles which intersect the query.

----

## Opinionated design

- It requires C++11, possibly higher.
- It is low-level, and useless on its own.
  - It is used to build other higher-level data structures.
- It *does not* store any other information, not even a key.
  - ***The rectangle itself is the key***. *Read the next point*.
- Embrace ***hash tables***.
  - With C++11 hash collections (```unordered_map```), looking up other columns is very easy, once the ***exact value*** in any indexed column is given.
  - This utility seizes upon this opportunity, by converting 2D range query into a lookup that returns ***exact rectangles***.
- Embrace ***spuriousness***.
  - Algorithms can be designed so that spurious intermediate data do not affect correctness of outcome.
  - This is typically done by having additional checks (typically equality checks) at later stages of the algorithm.
  - Case-in-point: Hash table.
    - Hash table is great because it embraces spuriousness.
    - Hash functions are purposefully non-injective.
    - Different records can hash into the same value.
    - The hash value is further compressed (non-injectively) to fit a table size.
- Coordinates are ***integer-valued***.
  - Adapting this utility to floating-point is trivial. Just use the integer-valued version by
    dropping the fractional part via ```floor``` or ```ceil``` as needed.
  - The coordinate modifications introduce spurious results; but *re-read the point above on embracing spuriousness*.

----

## Algorithm design

- This utility is based on quad-tree. *See side-note 1.*
- At each node, a soon-to-be-added rectangle is classified as either...
  - Straddle - the rectangle cannot be classified into any child node, therefore it must be stored at the current node;
  - Splittable - the rectangle is contained wholly within one of the child notes.
- Straddle is stored in the current node.
- Straddle does not influence the node-splitting decision, because such items can't be stored at child nodes.
- Splittable may be stored either way, depending on the status of node-splitting.
- Rectangles are stored in ```std::vector<Rect>```.
- Straddle and splittable items are stored separately, because it is fairly likely the splittable collection
  will need to be garbage-collected in node-splitting; whereas the straddle collection will remain.
- Node-splitting consists of:
  - Creating one or more child nodes; *See side-note 2*
  - Distributing all splittable items into child nodes;
  - Garbage-collecting the splittable collection at the current node.
- Removal is handled by compacting a ```std::vector<Rect>```. However, it is not bad because:
  - The number of rectangles stored at a node is capped; otherwise node-splitting is triggered.
  - By the design purpose of this utility, it is permitted to store spurious rectangles, because the
    higher-level user of this library can cross-reference with another lookup table
    (typically ```unordered_map```, as discussed in the Purpose section) to remove spurious results.

### Side-note 1 - Different flavors of quadtrees

When comparing quad-tree implementations, be mindful there are two flavors: those that store points;
those that store rectangles. An implementation that store rectangles can be trivially adapted to
store points by converting each input point into a minimally-sized rectangle.

### Side-note 2 - Creating all child nodes or only as needed

This involves a lengthy discussion on memory management. **TODO** *add details later*.

----

## Advanced applications notes

These advanced applications notes are *not* implemented within this utility. These are supposed to
be implemented in a higher-level utility built on top of this utility.

### Storing rectangles with high aspect ratio

#### Background and motivation

In typical implementations of quad-tree, rectangles with high aspect ratio are almost always counted as straddles,
therefore they will pile up at some nodes closer to the root. As a result, performance degrades for every query
because these nodes are visited by nearly every query, and intersection tests must be performed with these
rectangles (with high aspect ratio), regardless of their chance of intersection.

#### Solution

It is recommended that any higher-level utility which builds on top of this utility (rectindex) implement a
high-aspect-ratio rectangle splitting strategy.

For example, a rectangle with top-left coordinate (100, 100) and size (1000, 10) can be split into several
subdivided rectangles along its horizontal coordinate range. The smaller rectangles can now be pushed deeper
into the tree, which reduces performance degradation for the majority of queries.

The mapping between the original (high-aspect-ratio) rectangle and the subdivided rectangles can follow the
same opinionated design principle as discussed earlier.

----

## Enhancements reserved for future implementation

### Thread safety

Being a low-level utility (*see the opinionated design point above*), thread-safety belongs to elsewhere,
namely the higher-level utility. It may potentially use reader-writer-lock, or ```std::shared_mutex```
***available in C++17***.

### SIMD speed-up for flat list rectangle intersection tests

To achieve speed-up with SIMD for this task, the rectangles need to be stored differently. In particular,
instead of storing as (x, y, width, height) *a la OpenCV-style*, the flat list need to be stored as
(left, top, negated(right), negated(bottom)) instead. This will cause some difficulty with code readability
and live-debugging.

### Multi-threaded query

Queries involving fewer than millions of results do not benefit from multi-threading, because the latency
from the time of request to the activation of a worker thread (whether created anew or reused from a worker
thread pool) is likely measured on the scale of milliseconds.

To enable multi-threaded query in a beneficial way, the quadtree nodes need accounting to estimate the
number of items stored in the subtree. In addition, the higher-level utility must manage its own spatial
density estimate, not as a property of subtree nodes, because for each node that is partially
inside the query rectangle, the subtrees that are completely outside are simply not visited.

### Spatial hotspots

Because of the opinionated design principles (*see section in the beginning*), spatial hotspots can be
handled either by a higher-level utility, or as a special kind of rectangle list *in addition to*
splittables and straddles.

----
