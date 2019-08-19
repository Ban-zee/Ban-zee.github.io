---
layout: post
title:  "Final Submission"
date:   2019-08-19 
categories: jekyll update
---


## GSoC 2019 Submission Details

As required by Google Guidelines, the commit history and the pull request list have been provided in this blog post. To allow the reader to understand the changes made during the work period, I have provided a detailed account of the vital pull requests. For those who just want to see the code changes without going into details, here is a list of my pull requests:

* [Stats and Diagnostics](https://github.com/arviz-devs/arviz/pull/710)
* [Plots](https://github.com/arviz-devs/arviz/pull/742)
* [Data](https://github.com/arviz-devs/arviz/pull/774)
* [AOT](https://github.com/arviz-devs/arviz/pull/785)
* [Code profiling and bottleneck identification](https://github.com/arviz-devs/arviz/pull/687)

To read the necessary documentation, refer to the my final blog post(Link has been provided at the end of this post)
### Working on numba stats and diagnostics:

In line with the timeline agreed upon by my mentors, I started my work with Numba implementation on stats. After going through different resources(both video and official documentation), the following method of work was adopted by me:

* Profile the given function on centered_school and a custom random dataset in a jupyter notebook.
* Identify bottlenecks.
* Try to implement a given numpy or scipy function from scratch in python and jit it.
* Compare their respective times.
* Put the modified methods in the primary method and then compare it's speed to the one included in ArviZ. 

The task was completed successfully with an approx speed gain of almost 40% in most cases.

The merged pull request is available [here](https://github.com/arviz-devs/arviz/pull/710)


A feature to enable and disable numba-jit as and when required, was also added along with the benchmark tests.
### Working on numba plots: 

Just like with stats, the first thing to do was profiling every single plot via [line_profiler](https://github.com/rkern/line_profiler). The best thing about working on plots is that the majority of them are based on [`plot_kde`](https://arviz-devs.github.io/arviz/generated/arviz.plot_kde.html#arviz.plot_kde), which can be optimised substantially using numba. 


This task  was completed successfully, and the required merged pull request is available [here](https://github.com/arviz-devs/arviz/pull/742)


### Working on numba data:

Luckily, like plots, data too had a base on which most of the methods were built. Unfortunately, most of the data is mainly I/O, and therefore, the majority of the methods are not compatible with numba and hence cannot be sped-up.
Working on Data
While working on `base.py`, the primary bottleneck which could be somewhat improved were `np.atleast_1d` and `np.atleast_2d`. Given that these methods were not compatible with numba nopython mode, I had to look into their source code to remove the possible bottlenecks. The major bottleneck in the methods as mentioned above was `np.asanyarray`. Given that in the majority of the cases in ArviZ, we deal with numpy arrays, this type of conversion was mainly redundant. However, to be safe, I, I did add a provision to handle the rare instance of encountering something other than a numpy array. These changes worked well, and I was able to gain a few units of time in the data functions.

The pull request can be found [here](https://github.com/arviz-devs/arviz/pull/774)


## What's done and what's left?

As jitting tests has no meaning, all of my work, as mentioned in the project proposal, has been completed. The necessary benchmarks tests have been added as well. However, on the request of my mentors, I was asked to work on ahead of time numba compilation which too has been completed and the only work left is the integration of aot into the main code; which would be done at a later stage. The aot pr is available [here](https://github.com/arviz-devs/arviz/pull/785)


### Other essential links:

[Link to my blog for a more detailed analysis](https://ban-zee.github.io/)

[Commit History](https://github.com/arviz-devs/arviz/commits?author=Ban-zee)

[Code profiling and bottleneck identification](https://github.com/arviz-devs/arviz/pull/687)

