```python
import numpy as np
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 10: Adapter Array

Reference: https://adventofcode.com/2020/day/10

## Part 1

Patched into the aircraft's data port, you discover weather forecasts of a massive tropical storm. Before you can figure out whether it will impact your vacation plans, however, your device suddenly turns off!

Its battery is dead.

You'll need to plug it in. There's only one problem: the charging outlet near your seat produces the wrong number of jolts. Always prepared, you make a list of all of the joltage adapters in your bag.

Each of your joltage adapters is rated for a specific output joltage (your puzzle input). Any given adapter can take an input 1, 2, or 3 jolts lower than its rating and still produce its rated output joltage.

In addition, your device has a built-in joltage adapter rated for 3 jolts higher than the highest-rated adapter in your bag. (If your adapter list were 3, 9, and 6, your device's built-in adapter would be rated for 12 jolts.)

Treat the charging outlet near your seat as having an effective joltage rating of 0.

Since you have some time to kill, you might as well test all of your adapters. Wouldn't want to get to your resort and realize you can't even charge your device!

If you use every adapter in your bag at once, what is the distribution of joltage differences between the charging outlet, the adapters, and your device?

For example, suppose that in your bag, you have adapters with the following joltage ratings:
```
16
10
15
5
1
11
7
19
6
12
4
```
With these adapters, your device's built-in joltage adapter would be rated for 19 + 3 = 22 jolts, 3 higher than the highest-rated adapter.

Because adapters can only connect to a source 1-3 jolts lower than its rating, in order to use every adapter, you'd need to choose them like this:

    The charging outlet has an effective rating of 0 jolts, so the only adapters that could connect to it directly would need to have a joltage rating of 1, 2, or 3 jolts. Of these, only one you have is an adapter rated 1 jolt (difference of 1).
    From your 1-jolt rated adapter, the only choice is your 4-jolt rated adapter (difference of 3).
    From the 4-jolt rated adapter, the adapters rated 5, 6, or 7 are valid choices. However, in order to not skip any adapters, you have to pick the adapter rated 5 jolts (difference of 1).
    Similarly, the next choices would need to be the adapter rated 6 and then the adapter rated 7 (with difference of 1 and 1).
    The only adapter that works with the 7-jolt rated adapter is the one rated 10 jolts (difference of 3).
    From 10, the choices are 11 or 12; choose 11 (difference of 1) and then 12 (difference of 1).
    After 12, only valid adapter has a rating of 15 (difference of 3), then 16 (difference of 1), then 19 (difference of 3).
    Finally, your device's built-in adapter is always 3 higher than the highest adapter, so its rating is 22 jolts (always a difference of 3).

In this example, when using every adapter, there are 7 differences of 1 jolt and 5 differences of 3 jolts.

Here is a larger example:
```
28
33
18
42
31
14
46
20
48
47
24
23
49
45
19
38
39
11
1
32
25
35
8
17
7
9
4
2
34
10
3
```
In this larger example, in a chain that uses all of the adapters, there are 22 differences of 1 jolt and 10 differences of 3 jolts.

Find a chain that uses all of your adapters to connect the charging outlet to your device's built-in adapter and count the joltage differences between the charging outlet, the adapters, and your device. What is the number of 1-jolt differences multiplied by the number of 3-jolt differences?


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
Markdown("The number of 1-jolt differences times the number "
         "of 3-jolt differnces is: **{}**".format(onesum * threesum))
```




The number of 1-jolt differences times the number of 3-jolt differnces is: **2432**



## Part 2

To completely determine whether you have enough adapters, you'll need to figure out how many different ways they can be arranged. Every arrangement needs to connect the charging outlet to your device. The previous rules about when adapters can successfully connect still apply.

The first example above (the one that starts with 16, 10, 15) supports the following arrangements:
```
(0), 1, 4, 5, 6, 7, 10, 11, 12, 15, 16, 19, (22)
(0), 1, 4, 5, 6, 7, 10, 12, 15, 16, 19, (22)
(0), 1, 4, 5, 7, 10, 11, 12, 15, 16, 19, (22)
(0), 1, 4, 5, 7, 10, 12, 15, 16, 19, (22)
(0), 1, 4, 6, 7, 10, 11, 12, 15, 16, 19, (22)
(0), 1, 4, 6, 7, 10, 12, 15, 16, 19, (22)
(0), 1, 4, 7, 10, 11, 12, 15, 16, 19, (22)
(0), 1, 4, 7, 10, 12, 15, 16, 19, (22)
```
(The charging outlet and your device's built-in adapter are shown in parentheses.) Given the adapters from the first example, the total number of arrangements that connect the charging outlet to your device is 8.

The second example above (the one that starts with 28, 33, 18) has many arrangements. Here are a few:
```
(0), 1, 2, 3, 4, 7, 8, 9, 10, 11, 14, 17, 18, 19, 20, 23, 24, 25, 28, 31,
32, 33, 34, 35, 38, 39, 42, 45, 46, 47, 48, 49, (52)

(0), 1, 2, 3, 4, 7, 8, 9, 10, 11, 14, 17, 18, 19, 20, 23, 24, 25, 28, 31,
32, 33, 34, 35, 38, 39, 42, 45, 46, 47, 49, (52)

(0), 1, 2, 3, 4, 7, 8, 9, 10, 11, 14, 17, 18, 19, 20, 23, 24, 25, 28, 31,
32, 33, 34, 35, 38, 39, 42, 45, 46, 48, 49, (52)

(0), 1, 2, 3, 4, 7, 8, 9, 10, 11, 14, 17, 18, 19, 20, 23, 24, 25, 28, 31,
32, 33, 34, 35, 38, 39, 42, 45, 46, 49, (52)

(0), 1, 2, 3, 4, 7, 8, 9, 10, 11, 14, 17, 18, 19, 20, 23, 24, 25, 28, 31,
32, 33, 34, 35, 38, 39, 42, 45, 47, 48, 49, (52)

(0), 3, 4, 7, 10, 11, 14, 17, 20, 23, 25, 28, 31, 34, 35, 38, 39, 42, 45,
46, 48, 49, (52)

(0), 3, 4, 7, 10, 11, 14, 17, 20, 23, 25, 28, 31, 34, 35, 38, 39, 42, 45,
46, 49, (52)

(0), 3, 4, 7, 10, 11, 14, 17, 20, 23, 25, 28, 31, 34, 35, 38, 39, 42, 45,
47, 48, 49, (52)

(0), 3, 4, 7, 10, 11, 14, 17, 20, 23, 25, 28, 31, 34, 35, 38, 39, 42, 45,
47, 49, (52)

(0), 3, 4, 7, 10, 11, 14, 17, 20, 23, 25, 28, 31, 34, 35, 38, 39, 42, 45,
48, 49, (52)
```
In total, this set of adapters can connect the charging outlet to your device in 19208 distinct arrangements.

You glance back down at your bag and try to remember why you brought so many adapters; there must be more than a trillion valid ways to arrange them! Surely, there must be an efficient way to count the arrangements.

What is the total number of distinct ways you can arrange the adapters to connect the charging outlet to your device?


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
Markdown("The total number of distinct adapter arrangements is **{}**".format(soln))
```




The total number of distinct adapter arrangements is **453551299002368**




```python

```
