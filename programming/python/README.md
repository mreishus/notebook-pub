# Python

- [The Elements of Python Style](https://github.com/amontalenti/elements-of-python-style)

## Books

Best non-beginner books for python:

- [Fluent
  Python](https://www.amazon.com/Fluent-Python-Concise-Effective-Programming/dp/1491946008)
- [Effective Python, 2nd
  Ed](https://www.amazon.com/Effective-Python-Specific-Software-Development/dp/0134853989)
- [Python Cookbook, 3rd
  Ed](https://www.amazon.com/Python-Cookbook-Third-David-Beazley/dp/1449340377)

## Conda

from internet comment:

I have been using a consistent setup that hasn't yet failed me for the past 2 years.

1. Install Anaconda to your home user directory.
2. create environment using (`conda create --name myenv python=3.6`).
3. Switch to the environment using (`conda activate myenv`).
4. Use (`conda install mypackage`), (`pip install mypackage`) in that priority
   order.
5. Export environment using (`conda env export > conda_env.yaml`).
6. Environment can be created on an other system using (`conda env create -f conda_env.yaml`).

Links to [Anaconda](https://www.anaconda.com/distribution/#download-section)
and [Dockerized
Anaconda](https://docs.anaconda.com/anaconda/user-guide/tasks/docker/)

## Numpy

- [Python Data Science Handbook,
  Amazon](https://www.amazon.com/Python-Data-Science-Handbook-Essential/dp/1491912057)
  - Price is currently higher than usual
- [Python Data Science Handbook,
  Online](https://jakevdp.github.io/PythonDataScienceHandbook/)
- [Scipy Lecture notes](https://scipy-lectures.org/intro/)
- [From python to
  numpy](https://www.labri.fr/perso/nrougier/from-python-to-numpy/) (More
  advanced)

### Blockshaped

Helper for taking square subgrids of a 2d grid. Let's say you have
a `m = np.arange(36).reshape(6, 6)` and you want to split it up into 3x3 subgrids,
the first one being `m[:3,:3]`, the second being `m[:3,3:6]`, etc. You can use
`blockshaped(m,3,3)` to turn it into a _stack_ of 3x3s, then `unblockshaped`
to _merge_ them back.

```python
import numpy as np
def blockshaped(arr, nrows, ncols):
    """
    Return an array of shape (n, nrows, ncols) where
    n * nrows * ncols = arr.size

    If arr is a 2D array, the returned array looks like n subblocks with
    each subblock preserving the "physical" layout of arr.
    """
    h, w = arr.shape
    return (arr.reshape(h//nrows, nrows, -1, ncols)
               .swapaxes(1,2)
               .reshape(-1, nrows, ncols))


def unblockshaped(arr, h, w):
    """
    Return an array of shape (h, w) where
    h * w = arr.size

    If arr is of shape (n, nrows, ncols), n sublocks of shape (nrows, ncols),
    then the returned array preserves the "physical" layout of the sublocks.
    """
    n, nrows, ncols = arr.shape
    return (arr.reshape(h//nrows, -1, nrows, ncols)
               .swapaxes(1,2)
               .reshape(h, w))
```

## Advent of Code Tips

Be familiar with:

### stdlib

- binascii
- collections (especially defaultdict)
- fractions (gcd)
- itertools
  - combinations
  - combinations_with_replacement
  - permutations
  - product
  - izip
  - chain
- md5
- re
- os/sys

### outside stdlib

- networkx (graphs and shortest path)
- sympy (isprime())
- numpy (efficient representation of large arrays)

### Tricks

It's useful to have a canned exhaustive tree search prepared
(breadth/depth/"best"-first search).

You can use complex numbers to represent 2-dimensional coordinates and
direction; turning the unit vector left and right involves multiplying by 1j
and -1j, respectively, and movement is addition.

Oh, and sometimes pypy is faster enough that it can make a Python solution
viable where otherwise it would have been too slow.

### Examples

- [Norvig's AOC](https://github.com/norvig/pytudes/blob/master/ipynb/Advent%202017.ipynb)

```
Created:       Sun 17 Nov 2019 07:35:56 AM CST
Last Modified: Sun 05 Jan 2020 10:34:56 AM CST
```
