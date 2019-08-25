---
layout: post
title:  "Numba Guide for ArviZ"
date:   2019-08-23 
categories: jekyll update
---


## Developer Guide

As mentioned in my previous blog posts, the primary aim of my gsoc project was to apply numba across ArviZ. In this blog post, I'll be laying out a work plan so as to help the future developers on Numba related issues in the project. 

# The main components

The following methods in `utils.py` are at the core of Numba development for this project. Given that Numba, along with LLVM, is a reasonably large python package, it is not a necessary dependency, and this is where these methods become useful. 

* `conditional_jit`: A substitute for the numba jit decorator; accepts all the parameters as the regular numba jit method. Identical in function as the regular jit with numba installed, idempotent in the absence of it.

* `conditional_vect`: A substitute for the numba vectorize decorator; accepts all the parameters as the regular numba vectorize method. Identical in function as the ordinary vectorize when numba is installed, idempotent in the absence of it.

* `Numba class`: A class to toggle the numba status. It has the following attributes and methods:
    
    * `numba_flag`: True if numba is installed, else false.
    
    * `disable_numba`: A method to disable the `conditional_jit` and `conditional_vect` methods.
    
    * `enable_numba`: A method to enable `conditional_jit` and `conditional_vect` methods.
    
* `_numba_var`: A method to calculate variance. If `numba_flag` is true then it uses the custom methods(right now it is just the variance methods included in `stats_utils`) else it uses the standard numpy or scipy methods. Similar functions can be added by new developers in the future as well.

# Benchmarks
As mentioned in my previous blog post, airspeed velocity has ben used for benchmarking the numbified methods. Currently, importing arviz into `benchmarks.py` causes the build of the benchmarking suite to break abruptly. The workaround this is the following: The speedups gained by the entire method are roughly proportional to the speed up gained by the numbified part of the method. Hence, these benchmarks give us a good estimate of how much speedup the main method has gained overall. To get an idea of how much speedup your method has gained, do the following:

```python
def time_numba_func():
    data = {data to be used to benchmark your method}
    try:
        import numba
        @numba.jit(**kwargs)
        def your_custom_or_jitted_method(*args,**kwargs):
        # Your custom method
    except ImportError:
        return numpy or scipy equivalent of the numbified method
``` 

`CONTRIBUTING.md` contains the steps to run the asv benchmarks.

# Ahead of time compilation

As mentioned previously, numba is a reasonably heavy dependency. A simple alternative to this is to use the numba ahead of time compilation. More information about ahead of time compilation and its benefits can be found [here](https://numba.pydata.org/numba-doc/dev/user/pycc.html). The methods meant to be precompiled can be added in `aot.py`. The steps to add the generated binaries into the release are as follows (**These steps are only for Linux platforms**):

* Pull one the following docker image: `docker pull quay.io/pypa/manylinux1_x86_64` or `quay.io/pypa/manylinux1_i686`

* `docker run -it -v $(pwd):/io quay.io/pypa/manylinux1_x86_64`

* `PYBIN=/opt/python/cp36-cp36m/bin`

* ```for PYBIN in /opt/python/*/bin; do"${PYBIN}/pip" wheel /io/ -w wheelhouse/done```

* The manylinux wheel will be available in the wheelhouse directory.

* Upload it on PyPI via twine.

* The latest aot module can be installed via `pip install -i https://test.pypi.org/simple/ aot-arviz`. In the future, this will be available with ArviZ.

**Using the precompiled methods**

The current aot module has precompiled methods only for the variance methods included in the `stats_utils`. The package can then be imported under as `numba_compilations`. Here is an example:

```python
import numpy as np
from numba_compilations import stats_variance_2d as svar

data = np.random.randn(100,100)
print(svar(data,ddof=0,axis=0))
```

The variance methods in the `stats_utils` will be replaced by these precompiled methods shortly.

# Acknowledgement

Before this project, my programming experience was limited to writing small python scripts from time to time. This was a first attempt on working collectively on a project as a result of which I'm more familiar with programming collaboratively with others. From this project, I also learned a lot beyond programming skills, such as how to write a proposal, how is a Python package structured and how to build documentation. Working on a Python package gives me a deeper understand and encourages me to take one step further. Instead of installing a package and simply using it as instructed, now I would also like to look at the source code and maybe even customize it for my own requirements.

I would like to thank my mentors [**Ravin Kumar**](https://github.com/canyon289), [**Colin Carrol**](https://colindcarroll.com/)
for helping me throughout this project. Special thanks to [**Osvaldo Martin**](https://github.com/aloctavodia) and [**Ari Hartikainen**](https://github.com/ahartikainen) as well. It was a pleasure working with you all.
