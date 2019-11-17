# Python

- [The Elements of Python Style](https://github.com/amontalenti/elements-of-python-style)

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
Last Modified: Sun 17 Nov 2019 07:41:24 AM CST
```
