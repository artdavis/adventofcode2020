```python
import numpy as np
import itertools
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 17: Conway Cubes

Reference: https://adventofcode.com/2020/day/17

## Part 1

As your flight slowly drifts through the sky, the Elves at the Mythical Information Bureau at the North Pole contact you. They'd like some help debugging a malfunctioning experimental energy source aboard one of their super-secret imaging satellites.

The experimental energy source is based on cutting-edge technology: a set of Conway Cubes contained in a pocket dimension! When you hear it's having problems, you can't help but agree to take a look.

The pocket dimension contains an infinite 3-dimensional grid. At every integer 3-dimensional coordinate (x,y,z), there exists a single cube which is either active or inactive.

In the initial state of the pocket dimension, almost all cubes start inactive. The only exception to this is a small flat region of cubes (your puzzle input); the cubes in this region start in the specified active (#) or inactive (.) state.

The energy source then proceeds to boot up by executing six cycles.

Each cube only ever considers its neighbors: any of the 26 other cubes where any of their coordinates differ by at most 1. For example, given the cube at x=1,y=2,z=3, its neighbors include the cube at x=2,y=2,z=2, the cube at x=0,y=2,z=3, and so on.

During a cycle, all cubes simultaneously change their state according to the following rules:

- If a cube is active and exactly 2 or 3 of its neighbors are also active, the cube remains active. Otherwise, the cube becomes inactive.
- If a cube is inactive but exactly 3 of its neighbors are active, the cube becomes active. Otherwise, the cube remains inactive.

The engineers responsible for this experimental energy source would like you to simulate the pocket dimension and determine what the configuration of cubes should be at the end of the six-cycle boot process.

For example, consider the following initial state:
```
.#.
..#
###
```
Even though the pocket dimension is 3-dimensional, this initial state represents a small 2-dimensional slice of it. (In particular, this initial state defines a 3x3x1 region of the 3-dimensional space.)

Simulating a few cycles from this initial state produces the following configurations, where the result of each cycle is shown layer-by-layer at each given z coordinate (and the frame of view follows the active cells in each cycle):
```
Before any cycles:

z=0
.#.
..#
###


After 1 cycle:

z=-1
#..
..#
.#.

z=0
#.#
.##
.#.

z=1
#..
..#
.#.


After 2 cycles:

z=-2
.....
.....
..#..
.....
.....

z=-1
..#..
.#..#
....#
.#...
.....

z=0
##...
##...
#....
....#
.###.

z=1
..#..
.#..#
....#
.#...
.....

z=2
.....
.....
..#..
.....
.....


After 3 cycles:

z=-2
.......
.......
..##...
..###..
.......
.......
.......

z=-1
..#....
...#...
#......
.....##
.#...#.
..#.#..
...#...

z=0
...#...
.......
#......
.......
.....##
.##.#..
...#...

z=1
..#....
...#...
#......
.....##
.#...#.
..#.#..
...#...

z=2
.......
.......
..##...
..###..
.......
.......
.......
```
After the full six-cycle boot process completes, 112 cubes are left in the active state.

Starting with your given initial configuration, simulate six cycles. How many cubes are left in the active state after the sixth cycle?


```python
trdict = {'.': np.uint8(0), '#': np.uint8(1)}
arr0 = list()
with open('cubes_input.txt', 'r') as fid:
#with open('test1_cubes_input.txt', 'r') as fid:
    for line in fid:
        arr0.append(np.array([trdict[x] for x in line.strip()], dtype=np.uint8))
        
arr0 = np.array(arr0, dtype=np.uint8)

# Reshape the array to give it a depth dimension
arr0_3d = arr0.reshape(1, *arr0.shape)
```


```python
# Lets work in array notation coordinates instead of x,y,z.
# In array coordinates indices are specified as (depth, row, column)
# which we'll call (d, r, c)

# Use np.ndenumerate to get indices with array value.
# At this index get the indices of the 26 neighbors.
# Use those indices into the array to check the values of
# the neighbors and apply rules

ineigh_3d = itertools.product([-1, 0, 1], [-1, 0, 1], [-1, 0, 1])
neigh_offsets_3d = np.array([i for i in ineigh_3d if i != (0, 0, 0)])

Markdown("Number of neighbors per 3D coordinate point will be "
         "{}.".format(len(neigh_offsets_3d)))
```




Number of neighbors per 3D coordinate point will be 26.




```python
def get_neighbors3d(ind, arr):
    # For supplied ind triplet, return a list of
    # 26 neighbor index triplets
    neigh_coords = neigh_offsets_3d + np.array(ind)
    # Keep only usable coordinates
    usable_coords = list()
    for coord in neigh_coords:
        if np.any(0 > coord):
            # No good. Under bound
            continue
        if arr.shape[0] <= coord[0]:
            # No good. Over depth bound
            continue
        if arr.shape[1] <= coord[1]:
            # No good. Over row bound
            continue
        if arr.shape[2] <= coord[2]:
            # No good. Over col bound
            continue
        usable_coords.append(coord)
    return np.array(usable_coords)
```


```python
ncycles = 6
arr1 = arr0_3d.copy()
for cycle in range(ncycles):
    # Pad the kernel array with zeros
    arr = np.pad(arr1, 1)
    # Make a copy of kernel array for next iteration
    arr1 = arr.copy()
    #ii = np.ndenumerate(arr0) # For testing
    #coord, v = next(ii) # For testing
    for coord, v in np.ndenumerate(arr):
        neighs = get_neighbors3d(coord, arr)
        neighvals = [arr[tuple(n.tolist())] for n in neighs]
        active_neighbors = np.sum(neighvals)
        if 0 == v:
            # Cube starts inactive
            if 3 == active_neighbors:
                # Exactly 3 neighbors are active
                # Cube becomes active
                v1 = 1
            else:
                # Cube stays inactive
                v1 = 0
        elif v == 1:
            # Cube starts active
            if 1 < active_neighbors < 4:
                # Exactly 2 or 3 active neighbors
                # Cube stays active
                v1 = 1
            else:
                # Cube deactivates
                v1 = 0
        arr1[coord] = v1
```


```python
Markdown("The number of cubes left in the active state is "
         "**{}**.".format(np.sum(arr1)))
```




The number of cubes left in the active state is **255**.



## Part 2

For some reason, your simulated results don't match what the experimental energy source engineers expected. Apparently, the pocket dimension actually has four spatial dimensions, not three.

The pocket dimension contains an infinite 4-dimensional grid. At every integer 4-dimensional coordinate (x,y,z,w), there exists a single cube (really, a hypercube) which is still either active or inactive.

Each cube only ever considers its neighbors: any of the 80 other cubes where any of their coordinates differ by at most 1. For example, given the cube at x=1,y=2,z=3,w=4, its neighbors include the cube at x=2,y=2,z=3,w=3, the cube at x=0,y=2,z=3,w=4, and so on.

The initial state of the pocket dimension still consists of a small flat region of cubes. Furthermore, the same rules for cycle updating still apply: during each cycle, consider the number of active neighbors of each cube.

For example, consider the same initial state as in the example above. Even though the pocket dimension is 4-dimensional, this initial state represents a small 2-dimensional slice of it. (In particular, this initial state defines a 3x3x1x1 region of the 4-dimensional space.)

Simulating a few cycles from this initial state produces the following configurations, where the result of each cycle is shown layer-by-layer at each given z and w coordinate:

Before any cycles:
```
z=0, w=0
.#.
..#
###


After 1 cycle:

z=-1, w=-1
#..
..#
.#.

z=0, w=-1
#..
..#
.#.

z=1, w=-1
#..
..#
.#.

z=-1, w=0
#..
..#
.#.

z=0, w=0
#.#
.##
.#.

z=1, w=0
#..
..#
.#.

z=-1, w=1
#..
..#
.#.

z=0, w=1
#..
..#
.#.

z=1, w=1
#..
..#
.#.


After 2 cycles:

z=-2, w=-2
.....
.....
..#..
.....
.....

z=-1, w=-2
.....
.....
.....
.....
.....

z=0, w=-2
###..
##.##
#...#
.#..#
.###.

z=1, w=-2
.....
.....
.....
.....
.....

z=2, w=-2
.....
.....
..#..
.....
.....

z=-2, w=-1
.....
.....
.....
.....
.....

z=-1, w=-1
.....
.....
.....
.....
.....

z=0, w=-1
.....
.....
.....
.....
.....

z=1, w=-1
.....
.....
.....
.....
.....

z=2, w=-1
.....
.....
.....
.....
.....

z=-2, w=0
###..
##.##
#...#
.#..#
.###.

z=-1, w=0
.....
.....
.....
.....
.....

z=0, w=0
.....
.....
.....
.....
.....

z=1, w=0
.....
.....
.....
.....
.....

z=2, w=0
###..
##.##
#...#
.#..#
.###.

z=-2, w=1
.....
.....
.....
.....
.....

z=-1, w=1
.....
.....
.....
.....
.....

z=0, w=1
.....
.....
.....
.....
.....

z=1, w=1
.....
.....
.....
.....
.....

z=2, w=1
.....
.....
.....
.....
.....

z=-2, w=2
.....
.....
..#..
.....
.....

z=-1, w=2
.....
.....
.....
.....
.....

z=0, w=2
###..
##.##
#...#
.#..#
.###.

z=1, w=2
.....
.....
.....
.....
.....

z=2, w=2
.....
.....
..#..
.....
.....
```
After the full six-cycle boot process completes, 848 cubes are left in the active state.

Starting with your given initial configuration, simulate six cycles in a 4-dimensional space. How many cubes are left in the active state after the sixth cycle?


```python
trdict = {'.': np.uint8(0), '#': np.uint8(1)}
arr00 = list()
with open('cubes_input.txt', 'r') as fid:
#with open('test1_cubes_input.txt', 'r') as fid:
    for line in fid:
        arr00.append(np.array([trdict[x] for x in line.strip()], dtype=np.uint8))
        
arr00 = np.array(arr00, dtype=np.uint8)

# Reshape the array to give it depth and hyper-depth dimension
arr00 = arr00.reshape(1, 1, *arr00.shape)
```


```python
# Use np.ndenumerate to get indices with array value.
# At this index get the indices of the 80 hyper-neighbors.
# Use those indices into the array to check the values of
# the neighbors and apply rules

ineigh_4d = itertools.product([-1, 0, 1], [-1, 0, 1], [-1, 0, 1], [-1, 0, 1])
neigh_offsets_4d = np.array([i for i in ineigh_4d if i != (0, 0, 0, 0)])

Markdown("Number of neighbors per 4D coordinate point will be "
         "{}.".format(len(neigh_offsets_4d)))
```




Number of neighbors per 4D coordinate point will be 80.




```python
def get_neighbors4d(ind, arr):
    # For supplied ind triplet, return a list of
    # 80 neighbor index triplets
    neigh_coords = neigh_offsets_4d + np.array(ind)
    # Keep only usable coordinates
    usable_coords = list()
    for coord in neigh_coords:
        if np.any(0 > coord):
            # No good. Under bound
            continue
        if arr.shape[0] <= coord[0]:
            # No good. Over hyper-depth bound
            continue
        if arr.shape[1] <= coord[1]:
            # No good. Over depth bound
            continue
        if arr.shape[2] <= coord[2]:
            # No good. Over row bound
            continue
        if arr.shape[3] <= coord[3]:
            # No good. Over col bound
            continue
        usable_coords.append(coord)
    return np.array(usable_coords)
```


```python
ncycles = 6
arr11 = arr00.copy()
for cycle in range(ncycles):
    print("CYCLE: {}".format(cycle + 1))
    # Pad the kernel array with zeros
    arr = np.pad(arr11, 1)
    # Make a copy of kernel array for next iteration
    arr11 = arr.copy()
    #ii = np.ndenumerate(arr0) # For testing
    #coord, v = next(ii) # For testing
    for coord, v in np.ndenumerate(arr):
        neighs = get_neighbors4d(coord, arr)
        neighvals = [arr[tuple(n.tolist())] for n in neighs]
        active_neighbors = np.sum(neighvals)
        if 0 == v:
            # Hypercube starts inactive
            if 3 == active_neighbors:
                # Exactly 3 neighbors are active
                # Hypercube becomes active
                v1 = 1
            else:
                # Hypercube stays inactive
                v1 = 0
        elif v == 1:
            # Hypercube starts active
            if 1 < active_neighbors < 4:
                # Exactly 2 or 3 active neighbors
                # Hypercube stays active
                v1 = 1
            else:
                # Hypercube deactivates
                v1 = 0
        arr11[coord] = v1
```

    CYCLE: 1
    CYCLE: 2
    CYCLE: 3
    CYCLE: 4
    CYCLE: 5
    CYCLE: 6
    


```python
Markdown("The number of cubes left in the active state is "
         "**{}**.".format(np.sum(arr11)))
```




The number of cubes left in the active state is **2340**.




```python

```
