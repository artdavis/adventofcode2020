```python
import numpy as np
import itertools
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 17: Conway Cubes

Reference: https://adventofcode.com/2020/day/17

## Part 1

**How many cubes are left in the active state after the sixth cycle?**


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
#Markdown("The number of cubes left in the active state is "
#         "**{}**.".format(np.sum(arr1)))
```

## Part 2

**How many cubes are left in the active state after the sixth cycle?**


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
#Markdown("The number of cubes left in the active state is "
#         "**{}**.".format(np.sum(arr11)))
```


```python

```
