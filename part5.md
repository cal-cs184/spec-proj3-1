## Introduction

So far, we've seen that Monte Carlo path tracing is very powerful in generating realistic images! However, you may have noticed that it also always results in a large amount of noise.

Of course, noise can be eliminated by increasing the number of samples per pixel. However, it turns out that we usually don't have to do this uniformly for all pixels. Some pixels converge faster with low sampling rates, while other pixels require many more samples to get rid of noise.

Adaptive sampling tries to avoid the problem of using a fixed (high) number of samples per pixel, by concentrating the samples in the more difficult parts of the image. 

## Algorithm

We provide a very simple algorithm to enable adaptive sampling. It works for each pixel individually based on statistics, detecting whether the pixel has converged as we trace ray samples through it.

Suppose that you've already traced $n$ samples through a pixel. We can immediately get their mean $\mu$ and standard deviation $\sigma$. Let's define a variable 
$$I = 1.96\cdot\frac{\sigma}{\sqrt{n}}$$ 
to measure the pixel's convergence.

Intuitively, $I$ is small only when the samples' variance $\sigma^2$ is small, or the number of samples $n$ is large enough. So, the smaller $I$ is, the more confidently we can conclude that the pixel has converged. Specifically, we check if 
$$I \leq maxTolerance \cdot \mu,$$ 
where `maxTolerance=0.05` by default. If the condition above is satisfied, we assume that the pixel has converged and stop tracing more rays for this pixel. If not, we continue the tracing-and-detecting loop.

You may wonder why the measure $I$ is defined this way and where the magic number $1.96$ comes from. In fact, we're calculating a *confidence interval* in statistics. Specifically, we are trying to solve for $I$ so that, with 95% confidence, the average illuminance in this pixel is between $\mu-I$ and $\mu+I$, based on our $n$ samples so far. The number $1.96$ comes from the 95% confidence. To learn more about this, you can check out [this article](http://www.dummies.com/education/math/statistics/how-to-calculate-a-confidence-interval-for-a-population-mean-when-you-know-its-standard-deviation/) as a nice introduction, and [the wikipedia page of z-test](https://en.wikipedia.org/wiki/Z-test) for a deeper understanding.

## Implementation

For this part, you will work in `PathTracer::raytrace_pixel()` in `pathtracer.cpp`.

* Tip 1: The `Spectrum` class is essentially a three-channel vector storing R, G, B values. When calculating the statistics, you may use `Spectrum::illum()` to get its illuminance.
* Tip 2: You don't have to keep track of every sample's illuminance $x_k$ to compute $\mu$ and $\sigma$. You only have to keep these two variables:
$$s_1=\sum_{k=1}^n x_k,$$
$$s_2=\sum_{k=1}^n x_k^2,$$
by adding $x_k$ and $x_k^2$ for each new sample. Then the mean and variance of all $n$ samples so far can be expressed as 
$$\mu=\frac{s_1}{n}$$
$$\sigma^2=\frac{1}{n-1}\cdot\left(s_2-\frac{s_1^2}{n}\right)$$

* Tip 3: You don't have to check a pixel's convergence for each new sample. This can be costly and, as such, we want to avoid computing it any more frequently than we need to. Instead, we provide a variable `samplesPerBatch=32` by default, so that you can check whether a pixel has converged every `samplesPerBatch` pixels, until you reach `ns_aa` samples.

* Tip 4: Make sure to fill the `sampleCountBuffer` with your actual number of samples per pixel, so that you can see the output sampling rate image.

## Result

You can enable adaptive sampling from the command line with `-a <samplesPerBatch> <maxTolerance>`. Note that, once the `-a` argument is specified, the previous sampling rate assigned by argument `-s` now represents the *maximum* number of samples.

For example, when you run

    ./pathtracer -t 8 -s 2048 -a 64 0.05 -l 1 -m 5 -r 480 360 -f bunny.png ../dae/sky/CBbunny.dae

<center>
<img src="https://i.imgur.com/yrMCcJZ.png" width="480px"/><img src="https://i.imgur.com/7UmgYse.png" width="480px"/>
</center>

You should be able to see an image `bunny.png`, along with an image `bunny_rate.png` showing the sample rate of every pixel. We use red and blue colors to represent high and low sampling rates. Sampling rates are computed as the ratio between the actual number of samples and the maximum number of samples allowed.

Note that since the adaptive sampling aims at generating noise-free images, it can take very long to converge. For example, this bunny takes more than 700 seconds to finish on our powerful desktop with 12 threads. (However, judging from the sampling rate image, using a uniform 2048 samples per pixel should take even longer.) So, it is recommended that you test it with a low resolution image or on a small patch of an image to verify its correctness before you start generating final images for your write-up.
