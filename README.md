# Rectindex - a C++11 approach to finding intersecting rectangles

----

## Purpose

- Rectindex stores rectangles.
- It facilitates just one kind of query:
  - Given a query rectangle,
  - Find all stored rectangles which intersect the query.

*Read more at [Design.md](./Design.md)*

----

## A sneak peek of its opinionated design

- It requires C++11, possibly higher.
- It is low-level, and useless on its own.
- It *does not* store any other information, not even a key.
- Embrace ***hash tables***.
- Embrace ***spuriousness***.
- Coordinates are ***integer-valued***.

*Read more at [Design.md](./Design.md)*

----

## License

This header file contains code that is work-in-progress.
This file is currently not part of OpenCV project, but its license terms shall be chosen
to be compatible with the policy of OpenCV code contribution guideline.

The license for the OpenCV project can be found at http://opencv.org/license.html

----

## Why

*See [WHY.md](./WHY.md)*

*My resume at [https://kinchungwong.github.io](https://kinchungwong.github.io)*

----
