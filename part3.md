## Task 1: Diffuse BSDF

[Relevant lecture slides](https://cs184.eecs.berkeley.edu/sp18/lecture/materials/slide_007)

In order to get render images with realisitic shading, we need a BSDF! First, implement the functions `DiffuseBSDF::f` and `DiffuseBSDF::sample_f` in *bsdf.cpp*. The function `DiffuseBSDF::f` takes as input the incoming solid angle `wi` and the outgoing solid angle `wo` and returns `f(wi -> wo)`. The function `DiffuseBSDF::sample_f` is slightly different; it must sample `wi` using `sample_f`  and return both it and the `pdf` of the sample that was chosen. (Use the BSDF's `sampler` object to get this sample.)

## Task 2: Direct lighting

Now that you have one material at your disposal, we can implement direct lighting estimations! You will implement two methods for estimation in *pathtracer.cpp*. The first is `estimate_direct_lighting_hemisphere`, in which we estimate the direct lighting on a point by sampling uniformly in a hemisphere, and the second is `estimate_direct_lighting_importance`, in which we instead use importance sampling by sampling all the lights directly. The functions are called by `one_bounce_radiance` in order to get an estimate of the direct lighting at a point that a ray hits - don't worry about this function yet.

### Uniform hemisphere sampling

Fill in `estimate_direct_lighting_hemisphere`. At a high level, it should take samples in a uniform hemisphere around the point of interest (`hit_p`), and for each ray that intersects a light source, compute the incoming radiance from that light source, then convert it to the outgoing radiance using the BSDF at the surface, and finally average across all samples.

The starter code at the top of the function uses the input intersection normal to calculate a local coordinate space for the object hit point. In this frame of reference, the normal to the surface is $(0,0,1)$ (so $z$ is "up" and $x,y$ are two other arbitrary perpendicular directions). We compute this transform because the BSDF functions you will define later expect the normal to be $(0,0,1)$. 

Additionally, in this coordinate system the cosine of the angle between a direction vector `w_in` and the normal vector `Vector3D(0,0,1)` is simply `w_in.z`. (Recall that the cosine of the angle between two unit vectors is equal to their dot product.) 

We store the transformation from local object space to world space in the matrix `o2w`. This matrix is orthonormal, thus `o2w.T()` is both its transpose and its inverse -- we store this world to object transform as `w2o`.

    Matrix3x3 o2w;
    make_coord_space(o2w, isect.n);
    Matrix3x3 w2o = o2w.T();

For your later convenience, we store a copy of the hit point in its own variable:

    const Vector3D& hit_p = r.o + r.d * isect.t;
    
We also calculate the outgoing direction `w_out` in the local object frame. Remember that this should be opposite to the direction that the ray was traveling.

    const Vector3D& w_out = w2o * (-r.d);

Lastly, we've included the number of samples (`num_samples`) you should take at this point. It is equal to the number of samples that you will take in the next part in importance sampling, so that we can clearly see the difference in de-noising power from the two sampling methods.

You need to implement the following steps:

1. To get each sample, use the member variable `hemisphereSampler`, and calculate the pdf.
2. Cast a ray from our hit point in the sample direction, testing against the `bvh` to see if it intersects the scene anywhere. Some things to note:
 
    * You should pass `bvh->intersect` a `Ray`, and a pointer to an `Intersection`, and it will not only return a bool representing if anything was intersecting, but it will also save information on the intersection in the `Intersection` you passed over.
    * The direction of the ray should be what you sampled from `hemisphereSampler`, transformed to world space: `Vector3D wi_world = o2w * wi`, where `wi` is the direction of the ray.
    * You should offset the origin of the ray from the hit point by the tiny global constant `EPS_D` (i.e., add `EPS_D * wi_world` to `hit_p` to get the shadow ray's origin). If you don't do this, the ray will frequently intersect the ray's origin triangle at the same spot again because of floating point imprecision.

3. If it hits something, get the intersected material's emitted light by calling `intersect.bsdf->get_emission()` on your `Intersection`.
4. Accumulate this result (with the correct multiplicative factors) inside `L_out`.

Factors to remember when summing up the radiance values in `L_out`:

* `get_emission()` returns an incoming radiance. To properly transform it to irradiance, you should multiply it by the BSDF and a cosine term.
* Make sure to average what you've accumulated correctly.
* Remember to weight by your pdf as described in the lecture on Monte Carlo Integration.

To test your results, update`est_radiance_global_illumination()` to return `estimate_direct_lighting_hemisphere()`. Now, you should be able to render any CB scenes with lambertian diffuse BSDFs, like CBspheres and CBbunny. 

At this point, here are the results of the commands

    ./pathtracer -t 8 -s 16 -l 8 -m 6 -H -f CBbunny_16_8.png -r 480 480 ../dae/sky/CBbunny.dae
    
<center>
<img src="https://i.imgur.com/XJuFCwn.png" width="480px" /> 
</center>

And
    
    ./pathtracer -t 8 -s 64 -l 32 -m 6 -H -f CBbunny_64_32.png -r 480 360 ../dae/sky/CBbunny.dae
    
<center>
<img src="https://i.imgur.com/oLYzGLl.png" width="480px" />
</center>


Now you should be able to render nice direct lighting effects such as area light shadows and ambient occlusion, albeit without full global illumination. However, you will only be able to render files with Lambertian diffuse BSDFs, as we have not yet implemented any of the other BSDFs. Also note that with this method, you can't render scenes with point light sources (like bunny.dae and dragon.dae), since our outgoing rays will never intersect with them! 

### Importance sampling by sampling over lights

Our results from uniform hemisphere sampling are quite noisy! While they will converge to the correct result, we can do better.

Fill in `estimate_direct_lighting_importance`. At a high level, it should sum over every light source in the scene, taking samples from the surface of each light, computing the incoming radiance from those sampled directions, then converting those to outgoing radiance using the BSDF at the surface. Namely, instead of uniformly sampling in a hemisphere, we sample each light directly.

You need to implement the following steps:

1. Loop over every `SceneLight` in the `PathTracer`'s vector of light pointers `scene->lights`.
2. For each light, check if it is a delta light. If so, you only need to ask the light for 1 sample (since all samples would be the same). Else you should ask for `ns_area_light` samples in a loop. 
3. To get each sample, call the `SceneLight::sample_L()` function. This function requests the ray hit point `hit_p` and returns both the incoming radiance as a `Spectrum` as well as 3 values by pointer. The values returned by pointer are

    * a probabilistically sampled `wi` unit vector giving the direction from `hit_p` to the light source,
    * a `distToLight` float giving the distance from `hit_p` to the light source, and 
    * a `pdf` float giving the probability density function evaluated at the returned `wi` direction.
    
4. The `wi` direction returned by `sample_L` is in world space. In order to pass it to the BSDF, you need to compute it in object space by calculating `Vector3D w_in = w2o * wi`. 
5. Check if the `z` coordinate of `w_in` is negative -- if so, you can continue the loop since you know the sampled light point lies behind the surface.
6. Cast a shadow ray from the intersection point towards the light, testing against the `bvh` to see if it intersects the scene anywhere. Two subtleties:

    * You should set the ray's `max_t` to be the distance to the light, since we don't care about intersections behidn the light source.
    * You should offset the origin of the ray from the hit point by the tiny global constant `EPS_D` (i.e., add `EPS_D * wi` to `hit_p` to get the shadow ray's origin). If you don't do this, the ray will frequently intersect the ray's origin triangle at the same spot again because of floating point imprecision.
    
7. If the ray does not intersect the scene, you can calculate the BSDF value at `w_out` and `w_in`. Accumulate the result (with the correct multiplicative factors) inside `L_out`.

Similar to hemisphere sampling, here are some factors to remember when summing up the radiance values in `L_out`:

* `sample_L` returns an incoming radiance. To properly transform it to irradiance, you should multiply it by the BSDF and a cosine term.
* You are summing up some number of samples per light. Make sure you average them correctly.
* The `sample_L` routine samples from a light. It does this using some probability distribution which can bias the sample towards certain parts of the light -- you need to account for this fact by using the returned `pdf` value when summing radiance values.

To test your results, update`est_radiance_global_illumination()` to return `estimate_direct_lighting_importance()`. 

At this point, here are the results of the commands

    ./pathtracer -t 8 -s 64 -l 32 -m 6  -f dragon_64_32.png -r 480 480 ../dae/sky/dragon.dae
    
<center>
<img src="https://i.imgur.com/FCcfZqX.png" width="480px" /> 
</center>

And
    
    ./pathtracer -t 8 -s 64 -l 32 -m 6  -f bunny_64_32.png -r 480 360 ../dae/sky/CBbunny.dae
    
<center>
<img src="https://i.imgur.com/5dcuGED.png" width="480px" />
</center>


Much better! We've drastically reduced noise, and solved our issue with intersecting with point light sources. Remember that you will be running experiments to compare these two methods for the [write-up](writeup). 
