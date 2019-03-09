## Task 1: Filling in the sample loop

Fill in `PathTracer::raytrace_pixel()` in *pathtracer.cpp*. This function returns a `Spectrum` corresponding to the integral of the irradiance over this pixel, which you will estimate by averaging over `ns_aa` samples. Since we are taking `ns_aa` samples per pixel, think about its relationship to antialiasing.

The inputs to this function are integer coordinates $(x,y)$ in pixel space. When `ns_aa > 1`, you should generate `ns_aa` random rays through this pixel by randomly sampling points $(s,t) \in [0,1]^2$ and casting a ray through $(x + s, y + t)$. When `ns_aa == 1`, you should instead generate your ray through the center of the pixel, i.e., $(x+.5,y+.5)$. You can evaluate the radiance of a ray `r` with `est_radiance_global_illumination(r)`.




Notes:
* PathTracer owns a `gridSampler`, which has a method  you can use to get random samples in $[0,1]^2$ (see *sampler.h/cpp* for details).
* You will most likely want to pass a location to the camera that has been scaled down to $[0,1]^2$ coordinates. To do this, you might want to use the width and height of the pixel buffer, stored in `sampleBuffer.w` and `sampleBuffer.h`.
* You can generate a ray by calling  `camera->generate_ray()` (which you will implement in Task 2).
* Remember to be careful about mixing *int* and *double* types here, since the input variables have integer types.

## Task 2: Generating camera rays

Fill in `Camera::generate_ray()` in *camera.cpp*. The input is a 2D point you calculated in Task 1. Generate the corresponding world space ray as depicted in [this slide](https://cs184.eecs.berkeley.edu/sp19/lecture/9-5/raytracing). 

The camera has its own coordinate system. In camera space, the camera is positioned at the origin, looks along the $-z$ axis, has the $+y$ axis as image space "up". Given the two field of view angles `hFov` and `vFov`, we can define a sensor plane one unit along the view direction with its bottom left and top right corners at 
    
    Vector3D(-tan(radians(hFov)*.5), -tan(radians(vFov)*.5),-1)
    Vector3D( tan(radians(hFov)*.5),  tan(radians(vFov)*.5),-1)

respectively. Convert the input point to a point on this sensor so that $(0,0)$ maps to the bottom left and $(1,1)$ maps to the top right. This is your ray's direction in camera space. We can convert this vector to world space by applying the transform `c2w`. This vector becomes the direction of our ray, `r.d` (don't forget to normalize it!!). The origin of the ray, `r.o`, is simply the camera's position `pos`. The ray's minimum and maximum $t$ values should be the `nClip` and `fClip` camera parameters. 

Tasks 1 and 2 will be tricky to debug before implementing part 3, since nothing will show up on your screen!

## Task 3: Intersecting Triangles

Fill in both `Triangle::intersect()` methods in *static_scene/triangle.cpp*. You are free to use any method you choose, but we recommend using the Moller Trumbore algorithm from [this slide](https://cs184.eecs.berkeley.edu/sp19/lecture/9-22/raytracing). Make sure you understand the derivation of the algorithm: [here](http://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-rendering-a-triangle/moller-trumbore-ray-triangle-intersection) is one reference.

Remember that not every intersection is valid -- the ray has `min_t` and `max_t` fields defining the valid range of `t` values. If `t` lies outside this range, you should return `false`. Else, update `max_t` to be equal to `t` so that future intersections with farther away primitives will be discarded.

Once you get the ray's $t$-value at the intersection point, you should populate the `Intersection *isect` structure in the second version of the function as follows:

* `t` is the ray's $t$-value at the hit point.
* `n` is the surface normal at the hit point. Use barycentric coordinates to interpolate between `n1, n2, n3`, the per-vertex mesh normals.
* `primitive` points to the primitive that was hit (use the `this` pointer).
* `bsdf` points to the surface bsdf at the hit point (use `get_bsdf()`).

After implementing this, you should be able to run your pathtracer on *dae/sky/CBspheres_lambertian.dae* to test your first three tasks! Don't forget to press *R* to start rendering. The spheres won't show up (we'll implement this in the next part!), so you should see an empty box:

<center>
<img src="https://i.imgur.com/IU96zao.png" width="480px" />
</center>

## Task 4: Intersecting Spheres

Fill in both `Sphere::intersect()` methods in *static_scene/sphere.cpp*. Use the quadratic formula and [this slide](https://cs184.eecs.berkeley.edu/sp19/lecture/9-23/raytracing). You may wish to implement the optional helper function `Sphere::test()`. As with `Triangle::intersect()`, set `r.max_t` in both routines and fill in the `isect` parameters for the second version of the function. For a sphere, the surface normal is a scaled version of the vector pointing from the sphere's center to the hit point.

Now the spheres should appear in *dae/sky/CBspheres_lambertian.dae*:

<center>
<img src="https://i.imgur.com/U0tj5MV.png" width="480px" />
</center>
