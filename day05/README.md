# Day 5: Binary Boarding

Reference: https://adventofcode.com/2020/day/5

## Part 1

**What is the highest seat ID on a boarding pass?**


```python
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

### Brute force the paritioning


```python
def binpart(code, segment):
    """
    Binary partion the supplied segment according to code character

    Parameters
    ----------
    code: str
        Single code charachter of:
            'F' or 'L' for lower half or
            'B' or 'R' for upper half
    segment: list
        List for binary segmentation
    """
    mid = len(segment) // 2
    if code in 'FL':
        return segment[:mid]
    elif code in 'BR':
        return segment[mid:]
    else:
        raise ValueError("Unhandled code: {}".format(code))

def consume(codestr, segment, result=None):
    """
    Consume a codestr and partition segment recursively
    to yield the identifier
    
    Parameters
    ----------
    codestr: str
        Valid sequential characters in the codestr are:
            'F' or 'L' for lower half or
            'B' or 'R' for upper half
    segment: list
        List for binary segmentation
    """
    if 1 == len(segment):
        return segment[0]
    else:
        segment = binpart(codestr[0], segment)
        return consume(codestr[1:], segment)
    
def code2seat(codestr):
    """
    Convert codestr to seatid and return
    
    Parameters
    ----------
    codestr: str
        First 7 characters of 'F' or 'B' for lower or upper half
        respectively of 0 through 127.
        Last three characters of 'L' or 'R' for lower or upper half
        respectively of 0 through 7
    """
    row = consume(codestr[:7], list(range(128)))
    col = consume(codestr[7:], list(range(8)))
    return row * 8 + col # seatid
```


```python
#%%timeit
# Read in the seat input converting to seat IDs
seatids = list()
with open('seats_input.txt', 'r') as fid:
    for line in fid:
        seatids.append(code2seat(line.strip()))
# timeit results:
print('8.51 ms ± 15.5 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)')
```

    8.51 ms ± 15.5 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
    


```python
#Markdown("The highest seat on a boarding pass is **{}**".format(max(seatids)))
```

### Binary representation of the partitioning

Turns out binary partitioning is just "counting by binary".
Convert FRs to 0s and BLs to 1s and that binary number is the answer!
This is way simpler.


```python
ckeys = {'F': '0', 'L': '0', 'B': '1', 'R': '1'}
def bincode2seat(codestr):
    return int(''.join([ckeys[x] for x in codestr]), 2)
```


```python
#%%timeit
# Read in the seat input converting to seat IDs
seatids = list()
with open('seats_input.txt', 'r') as fid:
    for line in fid:
        seatids.append(bincode2seat(line.strip()))
# timeit results:
print('1.6 ms ± 12.3 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)')
```

    1.6 ms ± 12.3 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
    


```python
#Markdown("The highest seat on a boarding pass is **{}**".format(max(seatids)))
```

## Part 2

**What is the ID of your seat?**


```python
# Range of seats
s0, s1 = min(seatids), max(seatids)
allseats = set(range(s0, s1+1))
# Our seat is the one missing from seatids
ourseat = allseats - set(seatids)
```


```python
#Markdown("Our seat ID is: **{}**".format(ourseat.pop()))
```
