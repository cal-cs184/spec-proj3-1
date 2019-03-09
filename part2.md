## Task 1: Constructing the BVH
You may notice that any *.dae* files with even mildly complicated geometry take a very long time to "render," even with only the simple normal shading. In this part we'll implement a Bounding Volume Hierarchy (BVH) to speed up our pathtracter so that it can render these *.dae* files efficiently.

For example, try running your pathtracer on a mesh from the previous project:

    ./pathtracer -t 8 ../dae/meshedit/cow.dae
    
And hit `R` to render.
 
**This will take quite some time to complete!** The reason this takes so long is because the starter code creates a one-node BVH by storing all nodes directly into a leaf node. As we will see, a BVH constructed with simple heuristics will perform much better, taking ray intersection complexity from linear to log, for those who care.

For your write-up (see the write-up instructions page for details), note how long it takes for the mesh to render (you can do this as informally as just keeping an eye on a clock, or more precisely if you wish) and the statistics printed out at completion (rays traced, intersection tests performed, etc.). We'll compare this to how long it takes to render once we implement our BVH. Let's get to it!

Implement the function `BVHAccel:construct_bvh()` inside *bvh.cpp*. A BVH is a binary tree: the `BVHAccel` class itself only contains the root node `BVHNode *root`. Each node in the tree contains a bounding box `bb`, left and right children `l` and `r`, and a pointer `vector<Primitive *> *prims` to a list of actual scene primitives. For interior nodes, `l` and `r` are non-`NULL`, and for leaf nodes, `prims` is non-`NULL`.

The following utility functions will be useful:

1. `Primitive::get_bbox()` returns the bounding box of a primitive.
2. `BBox::expand()` expands a bounding box to include the function argument, which can either be a `Vector3D` or another `BBox`.
3. Check out the `Vector3D` members variables inside a `BBox`: `min, max, extent`, where `extent = max-min`.

We recommend that you first attempt to construct a BVH with the following simple (but slightly inefficient) recursive function:

1. Compute the bounding box of the primitives in `prims` in a loop. 
2. Initialize a new `BVHNode` with that bounding box.
3. If there are at most `max_leaf_size` primitives in the list, this is a leaf node. Allocate a new `Vector<Primitive *>` for node's primitive list (initialize this with `prims`) and return the node.
4. If not, we need to divide the primitives into two sets: left and right, and then recurse. First, pick the axis to split on by choosing the largest dimension of the bounding box's extent.
5. Calculate the split point you are using on this axis (try using the midpoint of the bounding box, or the average of the centroids of the primitives).
6. Split all primitives in `prims` into two new vectors based on whether their bounding box's centroid's coordinate in the chosen axis is less than or greater than the split point. (`p->get_bbox().centroid()` is a quick way to get a bounding box centroid for `Primitive *p`.)
7. Recurse, assigning the left and right children of this node to be two new calls to `construct_bvh()` with the two primitive lists you just generated.
8. Return the node.

**One potential problem to consider**: if the split point is not chosen well (depending on your implementation) all the primitives may lie on one side of the split point. In this scenario, we will get a segfault because of infinite attempted recursive calls. With that in mind, you may need some logic to handle the case where either the left or right primitive vector is empty. One possible solution is to recursively move the split point to the midpoint of the half that contains all the primitives. Do this until both the left and right vectors are not empty.  
 
Feel free to experiment with other BVH construction methods. Some ideas include changing the splitting heuristic (using the median object rather than the bounding box's midpoint, using a surface area heuristic, etc.), or improving the algorithmic or memory efficiency of your BVH. See the extra credit section for more. 


## Task 2: Intersecting `BBox`

Implement the function `BBox::intersect()` inside *bbox.cpp*, using the simple ray and axis-aligned-plane intersection equation [here](https://cs184.eecs.berkeley.edu/sp19/lecture/9-35/raytracing) and the ray and axis-aligned-box intersection method [here](https://cs184.eecs.berkeley.edu/sp19/lecture/9-34/raytracing). Note that this function returns an *interval* of `t` values for which the ray lies inside the box.



## Task 3: Intersecting `BVHAccel`

Using the previous two parts, implement the two `BVHAccel::intersect()` routines inside *bvh.cpp*. 


`BVHAccel::intersect(const Ray& ray, BVHNode *node)` returns true if this ray intersects with any primitive inside the given BVH tree given by `node`.

`BVHAccel::intersect(const Ray& ray, Intersection* i, BVHNode *node)` returns true if this ray intersects with any primitive inside the given BVH tree given by `node`, and returns the closest hit in `i`.

The starter code assumes that *root* is a leaf node and tests the ray against every single primitive in the tree. Your improved method should implement [this](https://cs184.eecs.berkeley.edu/sp19/lecture/9-74/raytracing) recursive traversal algorithm. 

***Keep the following points in mind***:

* You can safely return `false` if a ray intersects with a `BBox` but its `t` interval has an empty intersection with the ray's interval from `min_t` to `max_t`. 
* In the version with no `isect` parameter, you can safely return `true` after a single hit. However, the other version must return the *closest* hit along the ray, so you need to check every `BBox` touched by the ray. (Why is this true?)
* If all primitives update `i` correctly in their own intersection routines, you don't need to worry about filling in `i` in `BVHAccel::intersect()`.
* If all primitives update `r.max_t` correctly in their own intersection routines, you don't need to worry about making sure the ray stores the closer hit in `BVHAccel::intersect()` since it will be taken care of automatically. (Why does this work?)

We've provided a helpful BVH visualization mode which you can use to debug your implementation. To enter this mode, press `V`. You can then navigate around the BVH levels using the left, right, and up keys. The BVH viz view looks like this:

<center>
<img src="https://i.imgur.com/umnQABt.jpg" width="600px" />
</center>

Let's go back and see how long it takes that cow mesh to render now! 

    ./pathtracer -t 8 ../dae/meshedit/cow.dae
    
And hit `R` to render.
 
Compare how long it takes to render the mesh now, versus how long it took originally. 

Once this part is complete, your intersection routines should be fast enough to render any of our scene files in a matter of seconds (with normal shading only), even ones like *dae/meshedit/maxplanck.dae* with tens of thousands of triangles:

<center>
<img src="https://i.imgur.com/akYhmoR.png" width="480px" />
</center>

Or *dae/sky/CBlucy.dae*, with hundreds of thousands of triangles:

<center>
<img src="https://i.imgur.com/AkpIWnC.png" width="480px" />
</center>

Take a moment to refresh yourself on the [write-up and deliverables](writeup) requirements and conduct some performance comparison experiments of your newly optimized intersection algorithm!
