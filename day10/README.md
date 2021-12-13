```python
import numpy as np
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 10: Adapter Array

Reference: https://adventofcode.com/2020/day/10

## Part 1

**What is the number of 1-jolt differences multiplied by the number of 3-jolt differences?**


```python
dat = np.loadtxt('adapter_input.txt', dtype=np.int)
print("Device rating = {}".format(3 + max(dat)))
```

    Device rating = 172
    


```python
datsort = np.sort(dat)
datdiff = np.diff(datsort)
datdiff
```




    array([1, 1, 3, 1, 1, 1, 3, 1, 1, 1, 1, 3, 1, 1, 1, 1, 3, 1, 1, 1, 1, 3,
           1, 1, 3, 1, 1, 1, 1, 3, 1, 1, 1, 1, 3, 1, 1, 3, 3, 1, 1, 1, 1, 3,
           3, 1, 3, 3, 1, 1, 1, 3, 1, 1, 1, 1, 3, 1, 1, 1, 1, 3, 1, 3, 1, 3,
           1, 1, 1, 3, 1, 1, 1, 1, 3, 3, 3, 1, 1, 1, 3, 1, 1, 1, 1, 3, 3, 1,
           1, 3, 1, 1, 1, 3, 3, 1, 3, 1, 1, 1, 1, 3, 1, 1, 1, 1])




```python
onesum = sum(datdiff == 1) + 1
threesum = sum(datdiff == 3) + 1
```


```python
#Markdown("The number of 1-jolt differences times the number "
#         "of 3-jolt differnces is: **{}**".format(onesum * threesum))
```

## Part 2

**What is the total number of distinct ways you can arrange the adapters to connect the charging outlet to your device?**


```python
#dat = np.loadtxt('test1_adapter_input.txt', dtype=np.int)
#dat = np.loadtxt('test2_adapter_input.txt', dtype=np.int)
#dat = np.loadtxt('test3_adapter_input.txt', dtype=np.int)
dat = np.loadtxt('adapter_input.txt', dtype=np.int64)
dat.sort()
device_rating = dat[-1] + 3
device_rating

dat = np.append(dat, device_rating)
dat = np.insert(dat, 0, 0)
dat
```




    array([  0,   1,   2,   3,   6,   7,   8,   9,  12,  13,  14,  15,  16,
            19,  20,  21,  22,  23,  26,  27,  28,  29,  30,  33,  34,  35,
            38,  39,  40,  41,  42,  45,  46,  47,  48,  49,  52,  53,  54,
            57,  60,  61,  62,  63,  64,  67,  70,  71,  74,  77,  78,  79,
            80,  83,  84,  85,  86,  87,  90,  91,  92,  93,  94,  97,  98,
           101, 102, 105, 106, 107, 108, 111, 112, 113, 114, 115, 118, 121,
           124, 125, 126, 127, 130, 131, 132, 133, 134, 137, 140, 141, 142,
           145, 146, 147, 148, 151, 154, 155, 158, 159, 160, 161, 162, 165,
           166, 167, 168, 169, 172], dtype=int64)



### Strategy

Start by calculating the "skip difference" defined as the difference between every element
and the the element TWO ahead of this element.
Where the results are MORE THAN TWO, this element definitely
CANNOT be dropped (since then there would be a joltage difference of greater than
3 between adjacent elements).


```python
dat2 = np.roll(dat, -2)
diff2 = (dat2 - dat)[:-2]
diff2
```




    array([2, 2, 4, 4, 2, 2, 4, 4, 2, 2, 2, 4, 4, 2, 2, 2, 4, 4, 2, 2, 2, 4,
           4, 2, 4, 4, 2, 2, 2, 4, 4, 2, 2, 2, 4, 4, 2, 4, 6, 4, 2, 2, 2, 4,
           6, 4, 4, 6, 4, 2, 2, 4, 4, 2, 2, 2, 4, 4, 2, 2, 2, 4, 4, 4, 4, 4,
           4, 2, 2, 4, 4, 2, 2, 2, 4, 6, 6, 4, 2, 2, 4, 4, 2, 2, 2, 4, 6, 4,
           2, 4, 4, 2, 2, 4, 6, 4, 4, 4, 2, 2, 2, 4, 4, 2, 2, 2, 4],
          dtype=int64)



#### Standalone 2

Wherever there is a "standalone 2" in this array, we multiply the number
of possible arrays by 2. For example consider the sequence and its skip
difference:
```
... 20 23 24 25 28 ...
...    4  2  4 ...
```
In this case we can drop the `24` or not to maintain the required
minimum 3-jolt difference between elements doubling the possible
sequence combinations (multiplier = 2). Handily enough, we can just
leaves these 2's in the skip difference list and use them for multipliers
when we're done.

#### Pair of 2's

Wherever there is a "pair of 2's", we multiply the number of possible
arrays by 4. For example consider the following sequence and its skip difference:
```
... 14 17 18 19 20 23 ...
...    4  2  2  4 ...
```
The 4 possible valid sequences then are:
```
... 14 17 18 19 20 23 ...
... 14 17    19 20 23 ...
... 14 17 18    20 23 ...
... 14 17       20 23 ...
```
Since a pair of 2's conveniently multiply out to 4 we can also
just use them directly as multipliers.

#### Triplet of 2's

Wherever there is a "triplet of 2's", we multiply the number of possible
arrays by 7. For example consider the following sequence and its skip difference:
```
... 4 7 8 9 10 11 14 ...
...   4 2 2 2  4 ...
```
The 7 possible valid sequences would be:
```
... 4 7 8 9 10 11 14 ...
... 4 7   9 10 11 14 ...
... 4 7 8   10 11 14 ...
... 4 7 8 9    11 14 ...
... 4 7     10 11 14 ...
... 4 7   9    11 14 ...
... 4 7 8      11 14 ...
```
We will first run through the skip difference and tally up
the number of times there is a triplet of 2's (N). The
net multiplier for the number of possible sequences this corresponds
to will be $7^N$.

If we then strip all the triple 2's out of the skip difference
and remove any remaining "non 2's" our answer will be
the product of all of those 2's and $7^N$.


```python
i = 0
three_twos = 0
diff3 = list()
while i < len(diff2):
    if np.all(diff2[i:i+3] == 2):
        # There was a run of three 2's
        three_twos += 1
        i += 3
    else:
        diff3.append(diff2[i])
        i += 1
```


```python
diff3 = np.array(diff3)
twos = diff3[diff3 == 2]
soln = np.prod(twos) * 7 ** three_twos
```


```python
#Markdown("The total number of distinct adapter arrangements is **{}**".format(soln))
```


```python

```
