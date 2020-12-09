```python
import numpy as np
import itertools
from IPython.display import Markdown
from IPython.display import Image # Syntax: Image(filename='file', width=wd, height=ht)
```

# Day 1: Report Repair

Reference: https://adventofcode.com/2020/day/1

## Part 1

After saving Christmas five years in a row, you've decided to take a vacation at a nice resort on a tropical island. Surely, Christmas will go on without you.

The tropical island has its own currency and is entirely cash-only. The gold coins used there have a little picture of a starfish; the locals just call them stars. None of the currency exchanges seem to have heard of them, but somehow, you'll need to find fifty of these coins by the time you arrive so you can pay the deposit on your room.

To save your vacation, you need to get all fifty stars by December 25th.

Collect stars by solving puzzles. Two puzzles will be made available on each day in the Advent calendar; the second puzzle is unlocked when you complete the first. Each puzzle grants one star. Good luck!

Before you leave, the Elves in accounting just need you to fix your expense report (your puzzle input); apparently, something isn't quite adding up.

Specifically, they need you to find the two entries that sum to 2020 and then multiply those two numbers together.

For example, suppose your expense report contained the following:

```
1721
979
366
299
675
1456
```

In this list, the two entries that sum to 2020 are 1721 and 299. Multiplying them together produces 1721 * 299 = 514579, so the correct answer is 514579.

Of course, your expense report is much larger. Find the two entries that sum to 2020; what do you get if you multiply them together?


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

The Elves in accounting are thankful for your help; one of them even offers you a starfish coin they had left over from a past vacation. They offer you a second one if you can find three numbers in your expense report that meet the same criteria.

Using the above example again, the three entries that sum to 2020 are 979, 366, and 675. Multiplying them together produces the answer, 241861950.

In your expense report, what is the product of the three entries that sum to 2020?


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
