## Website report guidelines

The goals of your write-up are for you to (a) think about and articulate what you've built and learned in your own words, (b) have a write-up of the project to take away from the class.  Your write-up should include:

* An overview of the project, your approach to the various parts, what problems you encountered and how you solved them. Strive for clarity and succinctness. 
* On each part, make sure to include the results described in the Deliverables section for each Part.  If you failed to generate any results correctly, provide a brief explanation of why.
* Clearly indicate any extra credit items you completed, and provide a good write-up and illustration. 

The writeup is our main method of evaluating your work, so it is important to spend the time to do it correctly.  Plan ahead to allocate time for the writeup well before the deadline.

Another link to [how to build and submit](/article/7).

### Technical details

Do not convert or resize your *.png*. files. Use either the command line rendering mode or the `S` key to save screenshots, not your own OS's utility (like Project 1 but not like Project 2). Keep your images in a subdirectory called `images/` in the `website/` directory. We recommend using the `-r 480 360` command line flag to set resolution at 480 by 360 for all your screenshots.

## Deliverables

### Part 1

* Walk through the ray generation and primitive intersection parts of the rendering pipeline.
* Explain the triangle intersection algorithm you implemented in your own words.
* Show the normal shading for a few small *dae* files.

### Part 2

* Walk through your BVH construction algorithm. Explain the heuristic you chose for picking the splitting point.
* Walk through your BVH intersection algorithm.
* Show the normal shading for a few large *dae* files that you couldn't render without the acceleration structure.
* Perform some rendering speed experiments on the scenes, and present your results and a 1-paragraph analysis.

### Part 3

* Walk through both implementations of the direct lighting function.
* Show some images rendered with both implementations of the direct lighting function.
* Focus on one particular scene (with at least one area light) and compare the noise levels in soft shadows when rendering with 1, 4, 16, and 64 light rays (the `-l` flag) and 1 sample per pixel (the `-s` flag) using light sampling, not uniform hemisphere sampling.
* Compare the results between using uniform hemisphere sampling and lighting sampling, including a 1-paragraph analysis.

### Part 4

* Walk through your implementation of the indirect lighting function.
* Show some images rendered with global (direct and indirect) illumination. Use 1024 samples per pixel.
* Pick one scene and compare rendered views first with *only* direct illumination, then *only* indirect illumination. (You'll have to edit `at_least_one_bounce_radiance` in your code to generate these.) Use 1024 samples per pixel.
* For `CBbunny.dae`, compare rendered views with `max_ray_depth` equal to 0, 1, 2, 3, and 100 (the `-m` flag). Use 1024 samples per pixel.
* Pick one scene and compare rendered views with various sample-per-pixel rates, including at least 1, 2, 4, 8, 16, 64, and 1024. Use 4 light rays.
* You will probably want to use the instructional machines for the above renders in order to not burn up your own computer for hours.

### Part 5

* Walk through your implementation of the adaptive sampling.
* Pick one scene and render it with the maximum number of samples per pixel at least 2048. Show a good sampling rate image with clearly visible differences in sampling rate over various regions and pixels. Include both your sample rate image (which shows your how your adaptive sampling changes depending on which part of the image we are rendering) and your noise-free rendered result. Use 1 sample per light and at least 5 for max ray depth.
