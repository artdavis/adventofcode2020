```python
import numpy as np
import itertools
from IPython.display import Markdown
from IPython.display import Image # Syntax: Image(filename='file', width=wd, height=ht)
```

# Day 1: Report Repair

Reference: https://adventofcode.com/2020/day/1

## Part 1

Find the two entries that sum to 2020; **what do you get if you multiply them together?**


```python
dat = np.loadtxt('day1_input.txt')
```


```python
# Let the two entries we seek equal a & b
# a + b = 2020
a = np.array(dat, dtype=np.uint)
# Let x equal the product we seek
# a * b = x ... rewrite as b = x/a and substitute for b in our first eq:
x = a * (2020 - a)
b = x // a

# The intersection of a & b is our solution
a0, b0 = set(a) & set(b)
```


```python
# display(Markdown("{} + {} = {}".format(a0, b0, a0 + b0)))
# display(Markdown("{} * {} = **{}**".format(a0, b0, a0 * b0)))
```

## Part 2

In your expense report, **what is the product of the three entries that sum to 2020?**


```python
# 2 equations and 3 unknowns now...
# Guess we try the hard way (all permutations)
for a1, b1, c1 in itertools.permutations(a, 3):
    if 2020 == a1 + b1 + c1:
        # Got it. Have a break:
        break
```


```python
# display(Markdown("{} + {} + {} = {}".format(a1, b1, c1, a1 + b1 + c1)))
# display(Markdown("{} * {} * {} = **{}**".format(a1, b1, c1, a1 * b1 * c1)))
```


```python

```
