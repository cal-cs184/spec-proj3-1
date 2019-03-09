<center>
<img src="/cs184_sp17_content/article_images/12_.jpg" width="800px" />
</center>

## Due Date
Tue March 11, 11:59pm

## Overview
You will implement the core routines of a physically-based renderer using a pathtracing algorithm. This assignment reinforces many of the hefty ideas covered in class recently, including ray-scene intersection, acceleration structures, and physically based lighting and materials. By the time you are done, you'll be able to generate some stunning pictures (given enough patience and CPU time). You will also have the chance to extend the assignment in a plethora of technically challenging and intellectually stimulating directions.

**This project is much longer than the other projects. Rendering images for your writeup will take several *hours* at the minimum, so start early! Make sure you read the "Experiments, Report, and Deliverables" article ahead of time.**

## Project Parts
This time, we've split off each part into its own article:

* Part 1: [Ray Generation and Scene Intersection](/article/15)
* Part 2: [Bounding Volume Hierarchy](/article/16)
* Part 3: [Direct Illumination](/article/17)
* Part 4: [Global Illumination](/article/18)
* Part 5: [Adaptive Sampling](/article/19)

All parts are equally weighted at 20 points each, for a total of 100 points.

You'll also want to read these articles:

* [Experiments, Report and Deliverables](/article/21)
* [Optional Extra Credit Opportunities](/article/20)

In particular, please read the website writeup and deliverables page before beginning the project. Several parts of this assignment ask you to compare various methods/implementations, and we don't want you to be caught off guard!




## Using the program

First, accept the assignment on Github Classroom. Then, clone *your* private repo. *DO NOT* clone the skeleton. 

    git clone <YOUR_PRIVATE_REPO>
    
As before, use *cmake* and *make* inside a *build/* directory to create the executable ([how to build and submit](/article/7)).

### Command line options

Use these flags between the executable name and the *dae* file when you invoke the program. For example, to simply run the regular GUI with the *CBspheres.dae* file and 8 threads, you could type (note these commands will not ):

    ./pathtracer -t 8 ../dae/sky/CBspheres_lambertian.dae
    
If you wanted to save to the *spheres_64_16_6.png* file on the instructional machines with 64 samples per pixel, 16 samples per light, 6 bounce ray depth, and 480x360 resolution, you might rather use something like this:

    ./pathtracer -t 8 -s 64 -l 16 -m 6 -r 480 360 -f spheres_16_4_6.png ../dae/sky/CBspheres_lambertian.dae
    
For this assignment, we've provided a windowless run mode, which is triggered by providing a filename with the `-f` flag. The program will run in this mode when you are *ssh*-ed into the instructional machines. 

This means that when trying to generate high quality results for your final writeup, you can use the windowless mode to farm out long render jobs to the s349 machines! You'll probably want to use [*screen*](https://www.howtoforge.com/linux_screen) to keep your jobs running after you logout of *ssh*. After the jobs complete, you can view them using the *display* command, assuming you've *ssh*-ed in with graphics forwarding enabled, or *scp* them back to your local machine.

Also, please take note of the `-t` flag! We recommend running with 4-8 threads almost always -- the exception is that you should use `-t 1` when debugging with print statements, since `printf` and `cout` are not guaranteed to be thread safe. Furthermore, if you are using a virtual machine, **make sure that you allocate multiple CPU cores to it**. This is important in order for your `-t` flag to work. In practice, it's good to assign $N-2$ cores to the virtual machine, where $N$ is the number of your physical machine's number of CPU cores or hyperthreaded cores.



| Flag and parameters                              | Description                     |
| ------------------------------------ |:--------------------------------------------------:|
|`-s  <INT>` |        Number of camera rays per pixel (default=1, should be a power of 2)  |
|`-l  <INT>` |       Number of samples per area light (default=1) |
|`-t  <INT>` |       Number of render threads (default=1) |
|`-m  <INT>` |        Maximum ray depth (default=1) |
|`-f  <FILENAME>`|   Image (.png) file to save output to in windowless mode |
|`-r  <INT> <INT>`|  Width and height in pixels of output image (if windowless) or of GUI window |
|`-p <x> <y> <dx> <dy>`| Used with the -f flag to render a cell |
|`-c  <FILENAME>` |               Load camera settings file &#40;mainly to set camera position when windowless&#41; |
|`-a  <INT> <FLOAT>`|   Samples per batch and tolerance for adaptive sampling |
|`-H` | Enable hemisphere sampling for direct lighting |
|`-h` |               Print command line help message |


### Moving the camera (in edit and BVH mode)

| Command                              | Action                                                |
| ------------------------------------ |:--------------------------------------------------:|
| Rotate                 | Left-click and drag                                       |
| Translate             | Right-click and drag                                   |
| Zoom in and out              | Scroll                                          |
| Reset view           | Spacebar                                       |


### Keyboard commands

| Command                              | Keys                                                |
| ------------------------------------ |:--------------------------------------------------:|
| Mesh-edit mode (default) | `E`|
| | | 
| BVH visualizer mode | `V`|
| Descend to left/right child (BVH viz) | `LEFT/RIGHT`|
| Move up to parent node (BVH viz) | `UP` |
| | |
| Start rendering | `R` |
| Save a screenshot | `S`|
| Decrease/increase area light samples | `- +`|
| Decrease/increase camera rays per pixel | `[ ]`|
| Decrease/increase maximum ray depth | `< >`|
| Toggle cell render mode | `C`|
| Toggle uniform hemisphere sampling | `H`|
| Dump camera settings to file |`D`|

**Cell render mode lets you use your mouse to highlight a region of interest so that you can see quick results in that area when fiddling with per pixel ray count, per light ray count, or ray depth.**


## Basic code pipeline

What happens when you invoke *pathtracer* in the starter code? Logistical details of setup and parallelization:

1. The `main()` function inside *main.cpp* parses the scene file using a `ColladaParser` from *collada/collada.h*.
2. A new `Viewer` and `Application` are created. `Viewer` manages the low-level OpenGL details of opening the window, and it passes most user input into `Application`. `Application` owns and sets up its own `pathtracer` with a camera and scene. 
3. An infinite loop is started with `viewer.start()`. The GUI waits for various inputs, the most important of which launch calls to `set_up_pathtracer()` and `PathTracer::start_raytracing()`. 
4. `set_up_pathtracer()` sets up the camera and the scene, notably resulting in a call to `PathTracer::build_accel()` to set up the BVH.
5. Inside `start_raytracing()` (implemented in *pathtracer.cpp*), some machinery runs to divide up the scene into "tiles," which are put into a work queue that is processed by `numWorkerThreads` threads.
6. Until the queue is empty, each thread pulls tiles off the queue and runs `raytrace_tile()` to render them. `raytrace_tile()` calls `raytrace_pixel()` for each pixel inside its extent. The results are dumped into the pathtracer's `sampleBuffer`, an instance of an `HDRImageBuffer` (defined in *image.h*).

Most of the core rendering loop is left for you to implement.

1. Inside `raytrace_pixel()`, you will write a loop that calls `camera->generate_ray(...)` to get camera rays and `est_radiance_global_illumination(...)` to get the radiance along those rays.
2. Inside `est_radiance_global_illumination`, you will check for a scene intersection using `bvh->intersect(...)`. If there is an intersection, you will accumulate the return value in `Spectrum L_out`, 
    * adding the BSDF's emission with `zero_bounce_radiance` which uses `bsdf->get_emission()`, 
    * adding global illumination with `at_least_one_bounce_radiance`, which calls `one_bounce_radiance` (which calls a direct illumination function), and recursively calls itself as necessary

You will also be implementing the functions to intersect with triangles, spheres, and bounding boxes, the functions to construct and traverse the BVH, and the functions to sample from various BSDFs. 

Approximately in order, you will edit (at least) the files

* *pathtracer.cpp* (part 1)
* *camera.cpp* (part 1)
* *static_scene/triangle.cpp* (part 1)
* *static_scene/sphere.cpp* (part 1)
* *bvh.cpp* (part 2)
* *bbox.cpp* (part 2)
* *bsdf.cpp* (part 3)
* *pathtracer.cpp* (parts 3-5)

You will want to skim over the files

* *ray.h*
* *intersection.h*
* *sampler.h/cpp*
* *random_util.h*
* *static_scene/light.h/cpp*

since you will be using the classes and functions defined therein.

In addition, *ray.h* contains a defined variable `PART`. This is currently set to 1. You may use this variable if you like to test separate parts independently. For example, you can write things like `if (PART != 4) { return false; }` to easily "revert" parts of your code. **In particular, you will need to set `PART` to 5 to get your rate sampling images in Part 5.**

## Speeding up rendering tests: setting field of view (cropping) and number of samples

High quality renders take a long time to complete! During development and testing, *don't* spend a long time waiting for full-sized, high quality images. **Make sure you use the parameters specified in the writeup for your website, though!**

Here's some ways to get quick test images:

* Render with fewer samples!
* Utilize the cell rendering feature - start a render with `R`, then hit `C` and click-drag on the viewer. The pathtracer will now only render pixels inside the rectangular region you selected.
* Set a smaller window size using the -r flag (example: `./pathtracer -t 8 -s 64 -r 120 90 ../dae/sky/CBspheres_lambertian.dae`). Zoom out until the entire scene is visible in the window, then start a render with `R`. 
