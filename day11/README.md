```python
import numpy as np
import itertools
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 11: Seating System

Reference: https://adventofcode.com/2020/day/11

## Part 1

Your plane lands with plenty of time to spare. The final leg of your journey is a ferry that goes directly to the tropical island where you can finally start your vacation. As you reach the waiting area to board the ferry, you realize you're so early, nobody else has even arrived yet!

By modeling the process people use to choose (or abandon) their seat in the waiting area, you're pretty sure you can predict the best place to sit. You make a quick map of the seat layout (your puzzle input).

The seat layout fits neatly on a grid. Each position is either floor (.), an empty seat (L), or an occupied seat (#). For example, the initial seat layout might look like this:
```
L.LL.LL.LL
LLLLLLL.LL
L.L.L..L..
LLLL.LL.LL
L.LL.LL.LL
L.LLLLL.LL
..L.L.....
LLLLLLLLLL
L.LLLLLL.L
L.LLLLL.LL
```
Now, you just need to model the people who will be arriving shortly. Fortunately, people are entirely predictable and always follow a simple set of rules. All decisions are based on the number of occupied seats adjacent to a given seat (one of the eight positions immediately up, down, left, right, or diagonal from the seat). The following rules are applied to every seat simultaneously:

    If a seat is empty (L) and there are no occupied seats adjacent to it, the seat becomes occupied.
    If a seat is occupied (#) and four or more seats adjacent to it are also occupied, the seat becomes empty.
    Otherwise, the seat's state does not change.

Floor (.) never changes; seats don't move, and nobody sits on the floor.

After one round of these rules, every seat in the example layout becomes occupied:
```
#.##.##.##
#######.##
#.#.#..#..
####.##.##
#.##.##.##
#.#####.##
..#.#.....
##########
#.######.#
#.#####.##
```
After a second round, the seats with four or more occupied adjacent seats become empty again:
```
#.LL.L#.##
#LLLLLL.L#
L.L.L..L..
#LLL.LL.L#
#.LL.LL.LL
#.LLLL#.##
..L.L.....
#LLLLLLLL#
#.LLLLLL.L
#.#LLLL.##
```
This process continues for three more rounds:
```
#.##.L#.##
#L###LL.L#
L.#.#..#..
#L##.##.L#
#.##.LL.LL
#.###L#.##
..#.#.....
#L######L#
#.LL###L.L
#.#L###.##

#.#L.L#.##
#LLL#LL.L#
L.L.L..#..
#LLL.##.L#
#.LL.LL.LL
#.LL#L#.##
..L.L.....
#L#LLLL#L#
#.LLLLLL.L
#.#L#L#.##

#.#L.L#.##
#LLL#LL.L#
L.#.L..#..
#L##.##.L#
#.#L.LL.LL
#.#L#L#.##
..L.L.....
#L#L##L#L#
#.LLLLLL.L
#.#L#L#.##
```
At this point, something interesting happens: the chaos stabilizes and further applications of these rules cause no seats to change state! Once people stop moving around, you count 37 occupied seats.

Simulate your seating area by applying the seating rules repeatedly until no seats change state. How many seats end up occupied?


```python
#with open('test1_seats_input.txt', 'r') as fid:
#with open('test2_seats_input.txt', 'r') as fid:
with open('seats_input.txt', 'r') as fid:
    seats = fid.read().splitlines()

# Convert tonumpy array so we can use 2D slicing syntax
# L - empty seat -> 0
# # - occupied seat -> 1
# . - floor -> 2
arr = list()
tr = {'L': 0, '#': 1, '.': 2}
for s in seats:
    arr.append(np.array([tr[x] for x in s], dtype = np.int8))
arr = np.array(arr)
#arr

# Get coordinate list for all grid positions
ri, ci = np.indices(arr.shape)

seatbool = arr.ravel() == 0
r_s = ri.ravel()[seatbool]
c_s = ci.ravel()[seatbool]
#arr_s = arr.ravel()[seatbool]
rc = np.array([r_s, c_s])
# Print out coordinates for every seat
#for r, c in rc.T:
#    print("({},{},{})".format(r, c, arr[r, c]), end=', ')

# Known seat position coordinates. We only ever need to check
# seat coordinates for occupancy/vacancy
seatcoords = [tuple(x) for x in np.array([r_s, c_s]).T]
```


```python
def get_window(index):
    # Get the 1-D window coordinates for spaces
    # that are adjacent to the supplied index
    if index == 0:
        index0 = 0
    else:
        index0 = index - 1
    index1 = index + 2
    return index0, index1
```


```python
# Test example
i = 2; j = 3
#i = 8; j = 8
#i = 9; j = 9
#i = 0; j = 0
arr[i, j]
arr[i-1:i+2, j-1:j+2]

i0, i1 = get_window(i)
j0, j1 = get_window(j)
arr[i0:i1, j0:j1]
```




    array([[0, 0, 0],
           [0, 0, 0],
           [0, 0, 0]], dtype=int8)




```python
def runseats(arr0, seatconfigs=set()):
    # Visit every array coordinate
    arr2 = arr0.copy()
    ri, ci = np.indices(arr0.shape)
    for r, c in np.nditer([ri, ci]):
        if 2 == arr0[r, c]:
            # Floor position, cannot occupy
            continue
        r0, r1 = get_window(r)
        c0, c1 = get_window(c)
        window = arr0[r0:r1, c0:c1]
        if 0 == arr0[r, c]:
            # Empty seat, check if it should get occupied
            if not np.any(window == 1):
                # No neighbors =1 and position should become occupied
                arr2[r, c] = 1
        elif 1 == arr0[r, c]:
            # Filled seat, check if it should get vacated
            #print(window)
            if 4 < np.sum(window == 1):
                # This seat plust at least 4 other neighbors are filled
                # Vacate
                arr2[r, c] = 0
        else:
            raise ValueError("Unhandled value: {}".format(arr0[r, c]))
    # Create a hashable representation of seating for set
    #print(arr2)
    seathash = ''.join([str(x) for x in arr2.ravel()])
    #print(seathash)
    if seathash in seatconfigs:
        # Done.. hit an existing stable config
        return arr2, seatconfigs, True
    seatconfigs.add(seathash)
    return arr2, seatconfigs, False
```


```python
#arr0 = runseats(arr)
#arr1 = runseats(arr0)
#arr1
```


```python
seatconfigs = set()
newarr = arr.copy()
done = False
trial = 1
print("Trials: ", end='')
while(not done):
    print(trial, end=', ')
    newarr, seatconfigs, done = runseats(newarr, seatconfigs)
    trial += 1
#len(seatconfigs)
occupied_seats = np.sum(newarr == 1)
```

    Trials: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 


```python
Markdown("The number of occupied seats is **{}**".format(occupied_seats))
```




The number of occupied seats is **2354**



## Part 2

As soon as people start to arrive, you realize your mistake. People don't just care about adjacent seats - they care about the first seat they can see in each of those eight directions!

Now, instead of considering just the eight immediately adjacent seats, consider the first seat in each of those eight directions. For example, the empty seat below would see eight occupied seats:
```
.......#.
...#.....
.#.......
.........
..#L....#
....#....
.........
#........
...#.....
```
The leftmost empty seat below would only see one empty seat, but cannot see any of the occupied ones:
```
.............
.L.L.#.#.#.#.
.............
```
The empty seat below would see no occupied seats:
```
.##.##.
#.#.#.#
##...##
...L...
##...##
#.#.#.#
.##.##.
```
Also, people seem to be more tolerant than you expected: it now takes five or more visible occupied seats for an occupied seat to become empty (rather than four or more from the previous rules). The other rules still apply: empty seats that see no occupied seats become occupied, seats matching no rule don't change, and floor never changes.

Given the same starting layout as above, these new rules cause the seating area to shift around as follows:
```
L.LL.LL.LL
LLLLLLL.LL
L.L.L..L..
LLLL.LL.LL
L.LL.LL.LL
L.LLLLL.LL
..L.L.....
LLLLLLLLLL
L.LLLLLL.L
L.LLLLL.LL

#.##.##.##
#######.##
#.#.#..#..
####.##.##
#.##.##.##
#.#####.##
..#.#.....
##########
#.######.#
#.#####.##

#.LL.LL.L#
#LLLLLL.LL
L.L.L..L..
LLLL.LL.LL
L.LL.LL.LL
L.LLLLL.LL
..L.L.....
LLLLLLLLL#
#.LLLLLL.L
#.LLLLL.L#

#.L#.##.L#
#L#####.LL
L.#.#..#..
##L#.##.##
#.##.#L.##
#.#####.#L
..#.#.....
LLL####LL#
#.L#####.L
#.L####.L#

#.L#.L#.L#
#LLLLLL.LL
L.L.L..#..
##LL.LL.L#
L.LL.LL.L#
#.LLLLL.LL
..L.L.....
LLLLLLLLL#
#.LLLLL#.L
#.L#LL#.L#

#.L#.L#.L#
#LLLLLL.LL
L.L.L..#..
##L#.#L.L#
L.L#.#L.L#
#.L####.LL
..#.#.....
LLL###LLL#
#.LLLLL#.L
#.L#LL#.L#

#.L#.L#.L#
#LLLLLL.LL
L.L.L..#..
##L#.#L.L#
L.L#.LL.L#
#.LLLL#.LL
..#.L.....
LLL###LLL#
#.LLLLL#.L
#.L#LL#.L#
```
Again, at this point, people stop shifting around and the seating area reaches equilibrium. Once this occurs, you count 26 occupied seats.

Given the new visibility method and the rule change for occupied seats becoming empty, once equilibrium is reached, how many seats end up occupied?


```python
# Define functions that will return the indices for
# "rays" headed out from all directions from any
# coordinate position in the array

def rplus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the coordinates from here along +r
    # Order from the coordinate forward
    ri = np.arange(r+1, arr.shape[0])
    ci = np.repeat(c, len(ri))
    return np.array([ri, ci])

def rminus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the coordinates from here along -r
    # Order from the coordinate back
    ri = np.arange(0, r)[::-1]
    ci = np.repeat(c, len(ri))
    return np.array([ri, ci])

def cplus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the coordinates from here along +c
    # Order from the coordinate forward
    ci = np.arange(c+1, arr.shape[1])
    ri = np.repeat(r, len(ci))
    return np.array([ri, ci])

def cminus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the coordinates from here along -c
    # Order from the coordinate back
    ci = np.arange(0, c)[::-1]
    ri = np.repeat(r, len(ci))
    return np.array([ri, ci])

def rcplusplus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the diagonal +r, +c coords
    # Order from the coordinate on
    ri = np.arange(r+1, arr.shape[0])
    ci = np.arange(c+1, arr.shape[1])
    return np.array([ri[:len(ci)], ci[:len(ri)]])

def rcplusminus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the diagonal +r, -c coords
    # Order from the coordinate on
    ri = np.arange(r+1, arr.shape[0])
    ci = np.arange(0, c)[::-1]
    return np.array([ri[:len(ci)], ci[:len(ri)]])

def rcminusplus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the diagonal -r, +c coords
    # Order from the coordinate on
    ri = np.arange(0, r)[::-1]
    ci = np.arange(c+1, arr.shape[1])
    return np.array([ri[:len(ci)], ci[:len(ri)]])

def rcminusminus(arr, r, c):
    # From coord r, c in arr, get and return
    # all of the diagonal -r, -c coords
    # Order from the coordinate on
    ri = np.arange(0, r)[::-1]
    ci = np.arange(0, c)[::-1]
    return np.array([ri[:len(ci)], ci[:len(ri)]])

```


```python
# Return the seat occupancy for a supplied ray
def get_occupancy_from_ray(ray):
    # A ray is a 1-D slice of the room visible
    # from the observer. The 0 position moves out
    # from the observer. The first time a seat is
    # encountered : 0 or 1 the ray stops and we
    # report what we've found
    for pos in ray:
        if 0 == pos:
            return 0
        if 1 == pos:
            return 1
    # No seats occupied or vacant encountered
    return None

# Accumulate the occupancy based on output
# from get_occupancy_from_ray
def accum_occ(occ, empty, filled):
    if occ is None:
        # Nothing encountered and nothing to do
        pass
    elif 0 == occ:
        # Empty seat encountered
        empty += 1
    elif 1 == occ:
        # Filled seat encountered
        filled += 1
    else:
        raise ValuError("Unhandled occupancy: {}".format(occ))   
    return empty, filled

# For a given r, c in arr, get state of all visible chairs
rayfns = [rplus, rminus, cplus, cminus,
          rcplusplus, rcplusminus,
          rcminusplus, rcminusminus]
def get_state(arr, r, c):
    empty = 0
    filled = 0
    for rfn in rayfns:
        raycoords = rfn(arr, r, c)
        if 0 == len(raycoords): continue # No rays
        r0, c0 = raycoords
        occ = get_occupancy_from_ray(arr[r0, c0])
        empty, filled = accum_occ(occ, empty, filled)
    return empty, filled

```


```python
# Known seat coordinates are in seatcoords
def runseats2(arr0, seatconfigs=set()):
    arr2 = arr0.copy()
    # Visit every seat coordinate
    for r, c in seatcoords:
        empty, filled = get_state(arr0, r, c)
        if 0 == arr0[r, c]:
            # Empty seat, check if it should get occupied
            if 0 == filled:
                # No visible filled positions. Seat should become occupied
                arr2[r, c] = 1
        elif 1 == arr0[r, c]:
            # Filled seat, check if it should get vacated
            if 4 < filled:
                # 5 or more other filled seats are visible... Vacate
                arr2[r, c] = 0
        else:
            raise ValueError("Unhandled value: {}".format(arr0[r, c]))
    # Create a hashable representation of seating for set
    #print(arr2)
    seathash = ''.join([str(x) for x in arr2.ravel()])
    #print(seathash)
    if seathash in seatconfigs:
        # Done.. hit an existing stable config
        return arr2, seatconfigs, True
    seatconfigs.add(seathash)
    return arr2, seatconfigs, False
```


```python
seatconfigs = set()
newarr = arr.copy()
done = False
trial = 1
print("Trials: ", end='')
while(not done):
    print(trial, end=', ')
    newarr, seatconfigs, done = runseats2(newarr, seatconfigs)
    trial += 1
#len(seatconfigs)
occupied_seats2 = np.sum(newarr == 1)
```

    Trials: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 


```python
Markdown("The number of occupied seats is **{}**".format(occupied_seats2))
```




The number of occupied seats is **2072**


