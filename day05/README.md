# Day 5: Binary Boarding

Reference: https://adventofcode.com/2020/day/5

## Part 1

You board your plane only to discover a new problem: you dropped your boarding pass! You aren't sure which seat is yours, and all of the flight attendants are busy with the flood of people that suddenly made it through passport control.

You write a quick program to use your phone's camera to scan all of the nearby boarding passes (your puzzle input); perhaps you can find your seat through process of elimination.

Instead of zones or groups, this airline uses binary space partitioning to seat people. A seat might be specified like FBFBBFFRLR, where F means "front", B means "back", L means "left", and R means "right".

The first 7 characters will either be F or B; these specify exactly one of the 128 rows on the plane (numbered 0 through 127). Each letter tells you which half of a region the given seat is in. Start with the whole list of rows; the first letter indicates whether the seat is in the front (0 through 63) or the back (64 through 127). The next letter indicates which half of that region the seat is in, and so on until you're left with exactly one row.

For example, consider just the first seven characters of FBFBBFFRLR:

    Start by considering the whole range, rows 0 through 127.
    F means to take the lower half, keeping rows 0 through 63.
    B means to take the upper half, keeping rows 32 through 63.
    F means to take the lower half, keeping rows 32 through 47.
    B means to take the upper half, keeping rows 40 through 47.
    B keeps rows 44 through 47.
    F keeps rows 44 through 45.
    The final F keeps the lower of the two, row 44.

The last three characters will be either L or R; these specify exactly one of the 8 columns of seats on the plane (numbered 0 through 7). The same process as above proceeds again, this time with only three steps. L means to keep the lower half, while R means to keep the upper half.

For example, consider just the last 3 characters of FBFBBFFRLR:

    Start by considering the whole range, columns 0 through 7.
    R means to take the upper half, keeping columns 4 through 7.
    L means to take the lower half, keeping columns 4 through 5.
    The final R keeps the upper of the two, column 5.

So, decoding FBFBBFFRLR reveals that it is the seat at row 44, column 5.

Every seat also has a unique seat ID: multiply the row by 8, then add the column. In this example, the seat has ID 44 * 8 + 5 = 357.

Here are some other boarding passes:

    BFFFBBFRRR: row 70, column 7, seat ID 567.
    FFFBBBFRRR: row 14, column 7, seat ID 119.
    BBFFBBFRLL: row 102, column 4, seat ID 820.

As a sanity check, look through your list of boarding passes. What is the highest seat ID on a boarding pass?


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

Ding! The "fasten seat belt" signs have turned on. Time to find your seat.

It's a completely full flight, so your seat should be the only missing boarding pass in your list. However, there's a catch: some of the seats at the very front and back of the plane don't exist on this aircraft, so they'll be missing from your list as well.

Your seat wasn't at the very front or back, though; the seats with IDs +1 and -1 from yours will be in your list.

What is the ID of your seat?


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
