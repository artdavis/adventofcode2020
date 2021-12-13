```python
from collections import deque
import numpy as np
import pandas as pd
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 23: Crab Cups

Reference: https://adventofcode.com/2020/day/23

## Part 1

**What are the labels on the cups after cup 1?**

Your puzzle input was `219347865`.


```python
# Helper functions while for round iterations

def minusone(n, elems):
    """
    From the supplied list elements, return the value
    for n - 1. If n - 1 is not in elems, return n - 2
    if n - 2 not in elemes, return n -3 and so on.
    Wrap around to max value of list if necessay
    """
    n1 = n - 1
    while True:
        if n1 in elems:
            return n1
        if n1 < min(elems):
            n1 = max(elems)
        else:
            n1 -= 1

def next1(n, elems):
    """
    From the supplied list elements, return
    the next 1 after n, wrapping around if
    necessary
    """
    i = elems.index(n) + 1
    return elems[i % len(elems)]

def next3(n, elems):
    """
    From the supplied list elements, return
    the next 3 after n, wrapping around if
    necessary
    """
    i = elems.index(n) + 1
    return [elems[x % len(elems)] for x in range(i, i+3)]
```


```python
#cups = [int(x) for x in '389125467'] # Test input
cups = [int(x) for x in '219347865']
nrounds = 100
current_cup = cups[0]
for i in range(nrounds):
    ##print('-- move {} --'.format(i + 1))
    ##print("cups: ", cups)
    pick = next3(current_cup, cups)
    ##print("pick up: ", pick)
    pick_stack = list()
    for p in pick:
        pick_stack.append(p)
        cups.remove(p)
    #print("pick stack: ", pick_stack)
    #print("cups: ", cups)
    destination = minusone(current_cup, cups)
    ##print("destination: ", destination)
    idest = cups.index(destination) + 1
    #print("idest: ", idest)
    for c in pick_stack[::-1]:
        cups.insert(idest, c)
    #print("cups: ", cups)
    current_cup = next1(current_cup, cups)
```


```python
# Assemble string from 1 and working around
soln_list = [cups[x % len(cups)] for x in 
             range(cups.index(1) + 1, cups.index(1) + len(cups))]
soln_list
```




    [3, 6, 4, 7, 2, 5, 9, 8]




```python
soln_labels = ''.join([str(x) for x in soln_list])
```


```python
#Markdown("The labels after cups 1 are **{}**".format(soln_labels))
```

## Part Two

Determine which two cups will end up immediately clockwise of cup 1. **What do you get if you multiply their labels together?**

Your puzzle input was `219347865`.

### Failed Strategy

Of course expanding the number of cups to 1 million and iterating 10 million is
going to take way to long for the orignal process that involved list
manitpulations.

By using collections.deque for the cups and a second deque for
indexing the cups we can speed up the cycle and remove the expensive
need for doing value look ups. I was never able to make this fast
enough either.


```python
# =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
# FAILED CODE...
# TOO SLOW & STILL NEEDS DBUGGING
# =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

# Get the destination cup based on the label
# supplied, the known length of cups array and
# labels that should be excluded from consideration.
def get_dest(n, elems_len, exclude):
    n1 = n - 1
    while True:
        if 0 == n1:
            n1 = elems_len
        if n1 not in exclude:
            return n1
        n1 -= 1

s0 = '389125467' # Test input
#s0 = '219347865'
cups = deque([int(x) for x in s0]) # Test input
#cups.extend(range(max(cups) + 1, 1000001))
stack = deque()
# Keep an independent list that indexes cup value to it's
# position in cups (hacky linked list)
iq = deque(range(len(cups) + 1))
# Update the referencs in iq
for i, c in enumerate(s0):
    iq[int(c)] = i
# Now the index of iq holds that value at the address in the list
# so for example Say the 5th element of iq is 8: iq[5] = 8
# That means 5 can be found in cups at index 8: cups[8] = 5
# Keep iq updated as cups is changed so we'll never have to do
# an expensive lookup by value
cupslen = len(cups)

#nrounds = 10000000
nrounds = 10
#nrounds = 100
#nrounds = 3

for i in range(nrounds):
    #if i % 100 == 0:
    #    print(i, end='.')
    print('-- move {} --'.format(i + 1))
    print("cups:  ", list(cups))
    #print("iq: ", iq)
    # Rotate cups queue so current cup is the last element
    cups.rotate(-(i + 1)) # ROTATION
    # The last element is the current_cup
    current_cup = cups[-1]
    print("current cup: ", current_cup)
    # Pop-left the next 3 cups onto the stack
    for _ in range(3):
        stack.append(cups.popleft())
    print("pick up: ", stack)
    # Find index of destination
    dest = get_dest(current_cup, cupslen, stack)
    print("destination: ", dest)
    # Subtract 3 + i from the destination since 3 values have been popped off
    # and the queue has been rotated -(i + 1)
    ii = (iq[dest] - (3 + i + 1)) % cupslen
    # The iq values from here to ii need to be decremented by 3 for the
    # cups that have been removed
    for j in range(ii + 1):
        iq[cups[j]] = (iq[cups[j]]- 3) % cupslen # 350us TOO SLOW!
    #print('ii: ', ii)
    # Rotate cups to where the stack should be inserted
    cups.rotate(-(ii + 1)) # ROTATION
    for j in range(3):
        v = stack.pop()
        iq[v] = (2 - j + (ii + 1 + i + 1)) % cupslen
        cups.appendleft(v)
    # Advance the cups to the ground state
    cups.rotate(ii + 1 + i + 1)
```

### Successful Strategy

Use a Pandas.Series with the index as the cup labels
and the corresponding value as the next cup label to
the right as a linked list. Then all operations can
be reduced to look-ups or assignments of values in
the Series instance.


```python
# Helper functions for the rounds

def get_seq(current_cup, cuplist=[], seqlen=3):
    global cups
    # Use recursion to get the next sequence length of cups
    if seqlen <= len(cuplist):
        #print(cuplist)
        return cuplist
    nextcup = cups[current_cup]
    cuplist.append(nextcup)
    get_seq(nextcup, cuplist=cuplist, seqlen=seqlen)
    return cuplist

def get_pick(current_cup):
    global cups
    # Get the tuple from the next 3
    # after current cup
    c1 = cups[current_cup]
    c2 = cups[c1]
    c3 = cups[c2]
    return (c1, c2, c3)

# Get the destination cup based on the label
# supplied, the known length of cups array and
# labels that should be excluded from consideration.
def get_dest(n, elems_len, exclude):
    n1 = n - 1
    while True:
        if 0 == n1:
            n1 = elems_len
        if n1 not in exclude:
            return n1
        n1 -= 1
```


```python
# Use a pandas series to link cup numbers to it's
# immediate neighbor on the right
#s0 = '389125467' # Test input
s0 = '219347865'
#cups = pd.Series(index=range(1, len(s0) + 1), dtype=np.uint32)
#cups = pd.Series(range(2, 22), index=range(1, 21), dtype=np.uint32)
cups = pd.Series(range(2, 1000002), index=range(1, 1000001), dtype=np.uint32)
cupslen = len(cups)

# Assign neighbors
for i, v in enumerate(s0):
    ii = (i + 1) % len(s0)
    cups[int(v)] = int(s0[ii])
# Now for the extended cups set the neighbor for the last
# input cup to the first in the extension
cups[int(s0[-1])] = len(s0) + 1
# And set the last cup to be the neighbor of the first cup
cups[cupslen] = int(s0[0])

nrounds = 10000000
#nrounds = 10
#nrounds = 100
#nrounds = 3

current_cup = int(s0[0])

for i in range(nrounds):
    if i % 10000 == 0:
        print(i, end='.')
    #print('-- move {} --'.format(i + 1))
    #print("cups:  ", get_seq(current_cup, cuplist=[], seqlen=cupslen))
    #print("current cup: ", current_cup)
    pick_up = get_pick(current_cup)
    #print("pick up: ", pick_up)
    # The cup after the current_cup now becomes
    # whatever WAS after the last cup picked up
    cups[current_cup] = cups[pick_up[2]]
    # The picked up cups will be placed after
    # current_cup - 1 excluding any of the cups
    # that were picked
    dest = get_dest(current_cup, cupslen, pick_up)
    #print("destination: ", dest)
    # The cup after the last of the cups picked up
    # will be whaterver WAS after the dest cup
    cups[pick_up[2]] = cups[dest]
    # The NEW cup after the dest cup will be the
    # first of the picked up cups
    cups[dest] = pick_up[0]
    # The next current cup will be the cup after current cup
    current_cup = cups[current_cup]
```


```python
# For testing...
#final_cups = get_seq(1, cuplist=[], seqlen=cupslen)
#final_cups
# Assemble string from 1 and working around
#soln_list = final_cups[:-1]
#soln_list
```


```python
# After cup 1 the next two labels are:
label1 = cups[1]
label2 = cups[label1]
```


```python
label_product = np.uint64(label1) * np.uint64(label2)
```


```python
#Markdown("The product of the 2 cup labels immediately after 1 "
#         "is **{}**".format(label_product))
```


```python

```
