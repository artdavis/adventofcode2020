```python
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 15: Rambunctious Recitation

Reference: https://adventofcode.com/2020/day/15

# Part 1

**Given your starting numbers, what will be the 2020th number spoken?**

Your puzzle input was `20,0,1,11,6,3`.


```python
N0 = [20, 0, 1, 11, 6, 3]
#nturns = 2020
#nturns = 30000000

def elfgame(nturns, n0=N0):
    """
    Parameters
    ----------
    nturns: int
        Number or turns to report the result for (e.g. 2020)
    n0: list (default: N0)
        List of integer start numbers (e.g [0, 3, 6])
        
    Examples:
        n0 = [0, 3, 6] # 2020:1 3E7:175594
        n0 = [2, 1, 3] # 2020:10 3E7:3544142
        n0 = [1, 2, 3] # 2020:27 3E7:261214
        n0 = [2, 3, 1] # 2020:78 3E7:6895259
        n0 = [3, 2, 1] # 2020:438 3E7:18
        n0 = [3, 1, 2] # 2020:1836 3E7:362
        n0 = [20, 0, 1, 11, 6, 3] # 2020:421 3E7:436
    """
    # Initialize turn_indices with starting values
    turn_indices = [(0, 0) for x in range(nturns)]
    i = 1
    for n in n0[:-1]:
        turn_indices[n] = (i, i)
        i += 1
    spoken = n0[-1]
    turn_indices[spoken] = (0, i)

    for turn in range(len(n0) + 1, nturns + 1):
        #print("TURN: {}; PREVIOUS SPEAK: {}".format(turn, spoken))
        t0, t1 = turn_indices[spoken]
        #print("({}, {})".format(t0, t1))
        if 0 == t0:
            spoken = 0
        else:
            spoken = t1 - t0
        turn_indices[spoken] = (turn_indices[spoken][1], turn)
        #print("{}: {}".format(turn, spoken))

    #print("Turn: {}, Spoken: {}".format(turn, spoken))
    return {'turn': turn, 'spoken': spoken}
```


```python
#Markdown("The **{turn}th** spoken number is **{spoken}**".format(**elfgame(2020, N0)))
```

## Part Two

**Given your starting numbers, what will be the 30000000th number spoken?**


```python
#Markdown("The **{turn:,}th** spoken number is **{spoken}**"
#         .format(**elfgame(30000000, N0)))
```


```python

```
