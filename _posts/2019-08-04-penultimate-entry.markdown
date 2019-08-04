---
layout: post
title:  "Numba and Data"
categories: jekyll update
---


## A Brief Recap

So, here we are, at the penultimate blog post of my journey.  I am fortunate enough to pass my second Google Summer of Code evaluation, and the experience has been nothing but educational.I am done with arviz `plots` and `stats` and I have been able to achieve substantial speedup on both of them.

I have learnt a plethora of new things during the past two months, but there is always something new to learn. Without any further delay, let's jump into the changes made since the last blog post :)

## Data


As mentioned in my previous blog post, after having completed plots, my next aim was to work on `arviz.data`. Luckily, like plots, data too had a base on which most of the methods were built. Unfortunately, most of the data is mainly I/O, and therefore, the majority of the methods are not compatible with numba and hence cannot be sped-up.
Working on Data
While working on `base.py`, the primary bottleneck which could be somewhat improved were `np.atleast_1d` and `np.atleast_2d`. Given that these methods were not compatible with numba nopython mode, I had to look into their source code to remove the possible bottlenecks. The major bottleneck in the methods as mentioned above was `np.asanyarray`. Given that in the majority of the cases in ArviZ, we deal with numpy arrays, this type of conversion was mainly redundant. However, to be safe I, I did add an additional provision to handle the rare instance of encountering something other than a numpy array. These changes worked well, and I was able to gain a few units of time in the data functions.

As always, the link to my pull request can be found [here](https://github.com/arviz-devs/arviz/pull/774). The pull request is complete and up for review.

## Next and Final Step


My next and final objective is to work on Numba ahead-of-time compilation about which you can read more [here](https://numba.pydata.org/numba-doc/dev/user/pycc.html).
