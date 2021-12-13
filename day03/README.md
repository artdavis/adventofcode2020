# Day 3: Toboggan Trajectory

Reference: https://adventofcode.com/2020/day/3

## Part 1

Starting at the top-left corner of your map and following a slope of right 3 and down 1, **how many trees would you encounter?**


```python
import itertools
import numpy as np
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt

```

### Approach 1: BRUTE FORCE!

March the toboggan down the hill and check for any crashes.
Uses itertools.cycle to repeat the map input pattern as
needed.


```python
def brute_crashes(dx, dy=1):
    """
    Parameters
    ----------
    dx: int
        Number of spaces right the tobaggon moves
    dy : int (default: 1)
        Number of spaces down the tobaggon moves
    """
    x = dx
    collisions = 0
    tr = {'.': False, '#': True}
    with open('map_input.txt', 'r') as fid:
        # Don't need the first line for testing tree collision
        fid.readline()
        for y, line in enumerate(fid):
            # Throw away any lines skipped because of dy
            if dy > 1 and not(y % dy):
                continue
            level = itertools.cycle([tr[_] for _ in line.strip()])
            for _ in range(x + 1):
                loc = next(level)
            if loc:
                collisions += 1
            x += dx
    return collisions
```

### Approach 2: BINARY SHENANIGANS!

Treat the terrain map as a list of binary numbers
and the tobaggon position as a
binary number shift. Whenever the `bitwise_and` of the
tobaggon with the terrain gives you back the value
of the tobaggon, a tree crash has ocurred.


```python
def getasnum(strdat, ntype='int', endian='little'):
    """
    Convert '.' '#' maps to an integer assuming '.' values are
    binary 0 and '#' are binary 1.
    
    Parameters
    ----------
    strdat: str
        String containing characters of '.' and '#'
    ntype: {'int', 'bin', 'str'} (default: 'int')
        Identifier for the type conversion to return
    endian: {'little', 'big'} (default: 'little')
        Endianess of strdat. Assuming first character is
        least significant that's 'little' endian. If first
        character is most significant that's 'big' endian
    """
    bmap = {'.': '0', '#': '1'}
    bstr = ''.join([bmap[x] for x in strdat])
    if 'little' == endian:
        bstr = bstr[::-1] # Reverse order in the string
    elif 'big' == endian:
        pass # Already big
    else:
        raise ValueError("Unhandled endian: {}".format(endian))
    if 'str' == ntype:
        return bstr
    bint = np.uint32(np.int(bstr, 2))
    if 'int' == ntype:
        return bint
    if 'bin' == ntype:
        return np.binary_repr(bint, width=32)
    raise ValueError("Unhandled ntype: {}".format(ntype))

def get_collisions(dx, dy=1):
    """
    Return the number of collisions for a tobbogan trajectory
    that moves at a slope of dx / dy
    
    Parameters
    ----------
    dx : int
        Number of spaces right the tobaggon moves
    dy : int (default: 1)
        Number of spaces down the tobaggon moves
    """
    x = 1
    collisions = 0
    with open('map_input.txt', 'r') as fid:
        # Don't need the first line for testing tree collision
        # Use it to get the repeating pattern length
        plen = len(fid.readline().strip())
        for y, line in enumerate(fid):
            # Throw away any lines skipped because of dy
            if dy > 1 and not(y % dy):
                continue
            x += dx
            lnum = getasnum(line.strip())
            xmod = 2**((x - 1) % plen)
            collision_test = np.bitwise_and(xmod, lnum)
            if xmod == collision_test:
                # If the bitwise_and yielded back the same value as the
                # toboggon location, then there had to be a tree there and
                # a collision has ocurred
                collisions += 1
    return collisions
```


```python
def compute_crashes(dxlist, dylist, fn=get_collisions):
    """
    Compute the number of crashes for the supplied
    lists of dx, dy and also the product of the
    number of crashes
    
    Parameters
    ----------
    dxlist: list of ints
    dylist: list of ints
    fn: {brute_crashes, get_collisions} (default: get_collisions)
    """
    report = dict()
    crashlist = list(map(fn, dxlist, dylist))
    for i, crashes in enumerate(crashlist):
        report[(dxlist[i], dylist[i])] = crashes
    return report

def report_crashes(crashdict):
    """
    Report the crashes
    
    Parameters
    ----------
    crashdict: dict
        Dictionary returned from compute_crashes
    """
    crashprod = 1
    for (dx, dy), crashes in crashdict.items():
        display(Markdown("({}, {}) crashed {} times".format(dx, dy, crashes)))
        crashprod *= crashes
    #crashprod = np.prod(list(crashdict.values()), dtype=np.uint64)
    display(Markdown("Product of tree crashes: **{}**".format(crashprod)))

```

### Results


```python
# Part 1 parameters
dxlist = [3]
dylist = [1]
display(Markdown("#### Brute Force"))
%time crashdict1 = compute_crashes(dxlist, dylist, fn=brute_crashes)
#report_crashes(crashdict1)
display(Markdown("#### Binary Shenanigans"))
%time crashdict2 = compute_crashes(dxlist, dylist, fn=get_collisions)
#report_crashes(crashdict2)
```


#### Brute Force


    Wall time: 18 ms
    


#### Binary Shenanigans


    Wall time: 3 ms
    

## Part 2

**What do you get if you multiply together the number of trees encountered on each of the listed slopes?**

### Results


```python
# Part 2 parameters
dxlist = [1, 3, 5, 7, 1]
dylist = [1, 1, 1, 1, 2]
display(Markdown("#### Brute Force"))
%time crashdict1 = compute_crashes(dxlist, dylist, fn=brute_crashes)
#report_crashes(crashdict1)
display(Markdown("#### Binary Shenanigans"))
%time crashdict2 = compute_crashes(dxlist, dylist, fn=get_collisions)
#report_crashes(crashdict2)
```


#### Brute Force


    Wall time: 118 ms
    


#### Binary Shenanigans


    Wall time: 16 ms
    


```python

```
