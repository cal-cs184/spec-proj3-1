Since there are so many possible extensions for this assignment and since some of the extensions are not specific to one particular part, we've decided to accumulate all of our suggestions in one spot. We've grouped them in this rough ordering:

* **Challenge Level 1**: Similar to a medium-difficulty part of one of the previous assignments. Involves some digging around in the code and some googling, but changes should be reasonably localized.
* **Challenge Level 2**: Similar to a very hard part of one of the previous assignments (especially given the lack of detail provided here). Involves more significant research and structural changes to the code, perhaps also digging around in the parser or *.dae* files.
* **Challenge Level 3**: Advanced rendering techniques. Really only here to provide inspiration and potential final project ideas for the brave. Contact your GSI if you really would like to take them here.


### Challenge Level 1

* Create a more sophisticated pixel sampler that generates jittered or pseudorandom low-discrepancy samples. You'll probably want to make an extension to the `Sampler2D` class to implement this. Some potentially helpful slides to get you started are located [here](http://web.cs.wpi.edu/~emmanuel/courses/cs563/S10/talks/wk3_p1_wadii_sampling_techniques.pdf). Compare performance (in terms of noise and aliasing) to the random sampler.
* Implement the surface area heuristic for splitting in your BVH. Take some performance measurements on various scenes to determine when it performs better than more naive heuristics.
* Implement more efficient construction and intersection routines for the BVH that replace the recursive calls with a stack and a `while` loop (you should use the C++ standard library's `std::stack` class). Compare performance in terms of construction time and scene rendering time.
* Implement a more memory efficient BVH by storing all the `Primitive` pointers in one large vector and only keeping index references into small contiguous chunks of this vector inside the leaves. Alternatively, think of and research other ways to compress the tree. Compare memory usage to the original data structure.
* Add some GUI features to make our live-update interface better. You could use mouse clicks to focus the rendering around a particular area or to draw a rectangle around a region of interest that should get more samples, you could save incremental results to disk when rendering slow scenes, you could improve our sample buffer data structure to allow for multiple passes of sampling (so save every sample given instead of just the average), etc. Discuss what you added and show screenshots if applicable.
* Soup up our naive parallelization or use some clever arithmetic tricks in your inner loops (probably ray intersection) to speed things up. Profile your code and find the bottlenecks. Tell us about anything that gives a significant increase in rays/sec from your initial implementation.
* Explore fancier adaptive sampling heuristics and compare with our current method.

### Challenge Level 2

* Render motion blur effects. For this, you will have to adapt our Collada files or make some of your own. You will need to store start and end positions for every vertex in the scene. Once you make the required architecture changes, motion blur is fairly simple to implement, since you just need to deterministically or randomly sample a time for each ray and change the scene intersection code appropriately.
* Research alternative acceleration structures (k-d tree, octree, etc.) and implement one you think might be faster than the BVH. Take some measurements to compare performance of the two structures. Note that many of these structures partition space into disjoint subsets (rather than potentially overlapping boxes as in the BVH) and thus have trickier traversal code than the BVH.
* Upgrade your path tracer to perform photon mapping.

### Challenge Level 3

* Implement non-axis-aligned bounding boxes, and show it actually benefits ray traversal.
* Upgrade your path tracer to perform bidirectional path tracing.
