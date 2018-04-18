# Why

## Why create rectindex

- It is a *distillation* from something else, a higher-level rectangle collection that is actually useful.
- From the earlier implementation, I realized that a lower-level *distillation* is also useful, not to
  end-users, but to implementers of high-level utility.

## What did I use the rectangle collection for

- It was part of my implementation of "remap engine" (*side note*), a higher-level wrapper around
  OpenCV's ```cv::remap()``` API.
- The "remap engine" handles tiling, working around limitations in existing implementations (*side note 2*),
  distributing tiled tasks to concurrency workers, generating remap matrices on-the-fly from formulae
  (if applicable) and within each coucnrrency worker (which reduces memory usage, enhances cache locality,
  makes butterflies more beautiful, among other things).
- These things are very important for the "remap engine".
- To do its work, the "remap engine" must know the coordinate transformation very well, mathematically.
  This is what is called diffeomorphism.
- The rectangle collection is a key part in algorithmically analyzing a coordinate transformation.
  There are many key parts actually; these will be contributed piece-by-piece into OpenCV.

*Side note* it has nothing to do with diesel engine ECUs.

*Side note 2* see http://answers.opencv.org/question/22079/volunteer-for-fixing-bug-1337-rotate-remap-crash-when-widthstep-exceeds-15-bits/
