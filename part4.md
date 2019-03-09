For Part 4, your task is to render images with full global illumination, including a probabilistic estimate of infinite bounces of light. 
 
Your job will be to implement the functions called `est_radiance_global_illumination`, `zero_bounce_radiance`, `one_bounce_radiance`, and `at_least_one_bounce_radiance` (which is recursive) in *pathtracer.cpp*. 

* The function `est_radiance_global_illumination` is called to get an estimate of the total radiance with global illumination arriving at a point from a particular direction (e.g. on the image plane and going towards the image's center of projection). 
* The function `zero_bounce_radiance` should return light that results from no bounces of light, which is simply the light emitted by the given point and outgoing direction. This will be zero (black) unless the current point lies on a light.
* The function `one_bounce_radiance` is just the direct illumination that you implemented in Part 3.  You can just call your direct lighting function that uses importance sampling of the lights.
* The function `at_least_one_bounce_radiance` is the main implementation work for this Part 4.  At a high level, it should call the `one_bounce_radiance` function, and then recursively call itself to estimate the higher bounces. This recursive call should take one random sample from the BSDF at the hit point, trace a ray in that sample direction, and recursively call itself on the new hit point. 


Make sure you take a look at the pseudocode on slides 61-63 from lecture 17. 


We provide with the same setup code as in the direct lighting function:

    Matrix3x3 o2w;
    make_coord_space(o2w, isect.n);
    Matrix3x3 w2o = o2w.T();
    
    Vector3D hit_p = r.o + r.d * isect.t;
    Vector3D w_out = w2o * (-r.d);
    
As in part 3, `w2o` transforms world space vectors into a local coordinate frame where the normal is $(0,0,1)$, the unit vector in the $z$ direction. Thus `w_out` is the outgoing radiance direction in this local frame.

You need to implement the following steps:

1. For `zero_bounce_radiance`, use the BSDF's `get_emission()` function to determine if light is emitted from the intersection point.

2. For `one_bounce_radiance`, call a direct illumination function depending on the `direct_hemisphere_sample` program paramater.

3. For `at_least_one_bounce_radiance`: Take a sample from the surface BSDF at the intersection point using  `isect.bsdf->sample_f()`. This function requests the outgoing radiance direction `w_out` and returns the BSDF value as a `Spectrum` as well as 2 values by pointer. The values returned by pointer are

    * the probabilistically sampled `w_in` unit vector giving the incoming radiance direction (note that unlike the direction returned by `sample_L`, this `w_in` vector is in the *object* coordinate frame!) and
    * a `pdf` float giving the probability density function evaluated at the return `w_in` direction.

4. Use Russian roulette to decide whether to randomly terminate the ray. In theory, the probability of *not* terminating can be arbitrary. But in order to see the Russian roulette's effect more clearly, we recommend to use something like 0.6 or 0.7. You can use the `coin_flip` method from *random_util.h* to generate randomness.

5. Check other conditions of terminating a ray. Specifically, if indirect illumination is turned on (`max_ray_depth>1`), we always trace at least one indirect bounce regardless of the Russian roulette. Also, for the ray depth cutoff to work, you should initialize your camera rays' depths as `max_ray_depth` in `raytrace_pixel`, then decrease their depths by 1 at every bounce, until the depths drop to 1. 

6. If not terminating, create a ray that starts from `EPS_D` away from the hit point and travels in the direction of `o2w * w_in` (the incoming radiance direction converted to world coordinates). And remember to decrease its depth by 1.

7. Use `at_least_one_bounce_radiance` to recursively trace this ray, getting an approximation for its incoming radiance.

8. Convert this incoming radiance into an outgoing radiance estimator by scaling by the BSDF and a cosine factor and dividing by the BSDF `pdf` and one minus the Russian roulette termination probability.

You still can only render diffuse BSDFs until completing Part 5, however, you should now see some nice color bleeding in Lambertian scenes. You should be able to correctly render images such as 

    ./pathtracer -t 8 -s 64 -l 16 -m 5 -r 480 360 -f spheres.png ../dae/sky/CBspheres_lambertian.dae
    
<center>
<img src="https://i.imgur.com/rz40F6d.png" width="480px" />
</center>

Remember you will be running experiments for the [write-up](/article/21) -- since the renders will be taking some time, it's a good idea to start rendering now!