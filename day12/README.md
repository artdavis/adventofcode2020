```python
import math
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 12: Rain Risk

Reference: https://adventofcode.com/2020/day/12

## Part 1

**What is the Manhattan distance between that location and the ship's starting position?**


```python
#with open('test1_ferry_input.txt', 'r') as fid:
with open('ferry_input.txt', 'r') as fid:
    instructs = fid.read().splitlines()
```


```python
ang_dict = {0: 'E', 90: 'N', 180: 'W', 270: 'S'}
orient_dict = {v: k for k, v in ang_dict.items()}

def parse_instruct(instr, e, n, heading):
    """
    Parse the supplie instruction string and
    return the ferry's new position/orientation
    
    Parameters
    ----------
    instr: str
        Instruction like <str><val> where
    """
    op, val = instr[0], int(instr[1:])
    if op in 'NESWF':
        mvdict = {'N': lambda x: (e, n + x, heading),
                  'S': lambda x: (e, n - x, heading),
                  'E': lambda x: (e + x, n, heading),
                  'W': lambda x: (e - x, n, heading)}
        if 'F' == op:
            return mvdict[heading](val)
        else:
            return mvdict[op](val)
    if op in 'RL':
        if 'R' == op:
            direction = -1
        else:
            direction = 1
        new_heading = (orient_dict[heading] + direction * val) % 360
        return (e, n, ang_dict[new_heading])
    raise ValueError("Unhandled instruction: {}".format(instr))
```


```python
e = 0
n = 0
heading = 'E'
coords = list()
for instr in instructs:
    e, n, heading = parse_instruct(instr, e, n, heading)
    coords.append((e, n, heading))
    
pos = coords[-1]
#pos
manhat_dist = sum([abs(x) for x in pos[:2]])
```


```python
#Markdown("The Manhattan distance of the ship from its start position is **{}**"
#         .format(manhat_dist))
```

## Part 2

**What is the Manhattan distance between that location and the ship's starting position?**


```python
# Use a class data structure this time
class Ferry(object):
    def __init__(self, e=0, n=0, eway=10, nway=1):
        # Ship coordinates: East, North
        self.e = e
        self.n = n
        # Waypoint coordinates: East, North
        # relative to the ferry
        self.eway = eway
        self.nway = nway

    def rotate(self, ang):
        c = int(math.cos(math.radians(ang)))
        s = int(math.sin(math.radians(ang)))
        e2 = self.eway * c - self.nway * s
        n2 = self.eway * s + self.nway * c
        self.eway, self.nway = e2, n2
    
    def parse_instruct(self, instr):
        op, val = instr[0], int(instr[1:])
        if op in 'NESW':
            # Move the waypoint
            if 'N' == op: self.nway += val
            if 'S' == op: self.nway -= val
            if 'E' == op: self.eway += val
            if 'W' == op: self.eway -= val
        elif op in 'LR':
            if 'L' == op:
                self.rotate(val)
            else:
                self.rotate(-val)
        elif 'F' == op:
            # Move the ship
            self.e += self.eway * val
            self.n += self.nway * val
        else:
            raise ValueError("Unhandled instruction: {}".format(instr))
        return
    
    @property
    def manhattan_dist(self):
        return abs(self.e) + abs(self.n)
        
    def __str__(self):
        # What to return for string context representation
        outstr = "Ship coords: ({}, {})\n".format(self.e, self.n)
        outstr += "Waypoint coords: ({}, {})\n".format(self.eway, self.nway)
        outstr += "Manhattan distance: {}".format(self.manhattan_dist)
        return outstr
    
    def __repr__(self):
        # Return string representation
        return self.__str__()
    
    def __call__(self, instr):
        # If instance is called directly, parse_instruct
        self.parse_instruct(instr)

```


```python
# Starting position
ferry = Ferry()
ferry
```




    Ship coords: (0, 0)
    Waypoint coords: (10, 1)
    Manhattan distance: 0




```python
# Follow instructions
for instr in instructs:
    ferry(instr)
ferry
```




    Ship coords: (17379, 6581)
    Waypoint coords: (-7, -7)
    Manhattan distance: 23960




```python
#Markdown("The Manhattan distance of the ship from its start position is **{}**"
#         .format(ferry.manhattan_dist))
```


```python

```
