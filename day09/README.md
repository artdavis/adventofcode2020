```python
import itertools
import numpy as np
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 9: Encoding Error

Reference: https://adventofcode.com/2020/day/9

## Part 1

The first step of attacking the weakness in the XMAS data is to find the first number in the list (after the preamble) which is not the sum of two of the 25 numbers before it. **What is the first number that does not have this property?**


```python
with open('encoding_input.txt', 'r') as fid:
    dat = [int(line) for line in fid]
```


```python
box = 25
for i in range(box, len(dat)):
    for x in itertools.combinations(dat[i - box:i], 2):
        if sum(x) == dat[i]:
            # Verified
            #print("{} + {} = {}".format(x[0], x[1], sum(x)))
            verified = True
            break
        else:
            verified = False
    if not verified:
        # Found a number which does not verify
        #print("{}: {}".format(i, dat[i]))
        invalid = dat[i]
        break
```


```python
#Markdown("The first invalid number is **{}**".format(invalid))
```

## Part 2

**What is the encryption weakness in your XMAS-encrypted list of numbers?**


```python
# Iteratively shift the numbers by 1 and add them to
# the running sum array. As soon as the target invalid number
# appear in the running sum array, we've found the solution
d0 = np.array(dat)
arrs = [d0.copy()]
dsum = d0.copy()
cond = False
while(not cond):
    d0 = np.roll(d0, -1)
    d0[-1] = 0
    arrs.append(d0.copy())
    dsum += d0
    cond = invalid in dsum
```


```python
# The index of the invalid number in the running
# sum array is the index in all of the composition
# arrays for the numbers that summed to the invalid number
i = int(np.where(dsum == invalid)[0])
#print("Solution starts at index: {}".format(i))
soln = [a[i] for a in arrs]
```


```python
#Markdown("The sum of the min and max values in the solution is: **{}**"
#        .format(min(soln) + max(soln)))
```
