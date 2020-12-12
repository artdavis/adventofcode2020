```python
import math
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 12: Rain Risk

Reference: https://adventofcode.com/2020/day/12

## Part 1

Your ferry made decent progress toward the island, but the storm came in faster than anyone expected. The ferry needs to take evasive actions!

Unfortunately, the ship's navigation computer seems to be malfunctioning; rather than giving a route directly to safety, it produced extremely circuitous instructions. When the captain uses the PA system to ask if anyone can help, you quickly volunteer.

The navigation instructions (your puzzle input) consists of a sequence of single-character actions paired with integer input values. After staring at them for a few minutes, you work out what they probably mean:

    Action N means to move north by the given value.
    Action S means to move south by the given value.
    Action E means to move east by the given value.
    Action W means to move west by the given value.
    Action L means to turn left the given number of degrees.
    Action R means to turn right the given number of degrees.
    Action F means to move forward by the given value in the direction the ship is currently facing.

The ship starts by facing east. Only the L and R actions change the direction the ship is facing. (That is, if the ship is facing east and the next instruction is N10, the ship would move north 10 units, but would still move east if the following action were F.)

For example:
```
F10
N3
F7
R90
F11
```
These instructions would be handled as follows:

    F10 would move the ship 10 units east (because the ship starts by facing east) to east 10, north 0.
    N3 would move the ship 3 units north to east 10, north 3.
    F7 would move the ship another 7 units east (because the ship is still facing east) to east 17, north 3.
    R90 would cause the ship to turn right by 90 degrees and face south; it remains at east 17, north 3.
    F11 would move the ship 11 units south to east 17, south 8.

At the end of these instructions, the ship's Manhattan distance (sum of the absolute values of its east/west position and its north/south position) from its starting position is 17 + 8 = 25.

Figure out where the navigation instructions lead. What is the Manhattan distance between that location and the ship's starting position?


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
Markdown("The Manhattan distance of the ship from its start position is **{}**"
         .format(manhat_dist))
```




The Manhattan distance of the ship from its start position is **1589**



## Part 2

Before you can give the destination to the captain, you realize that the actual action meanings were printed on the back of the instructions the whole time.

Almost all of the actions indicate how to move a waypoint which is relative to the ship's position:

    Action N means to move the waypoint north by the given value.
    Action S means to move the waypoint south by the given value.
    Action E means to move the waypoint east by the given value.
    Action W means to move the waypoint west by the given value.
    Action L means to rotate the waypoint around the ship left (counter-clockwise) the given number of degrees.
    Action R means to rotate the waypoint around the ship right (clockwise) the given number of degrees.
    Action F means to move forward to the waypoint a number of times equal to the given value.

The waypoint starts 10 units east and 1 unit north relative to the ship. The waypoint is relative to the ship; that is, if the ship moves, the waypoint moves with it.

For example, using the same instructions as above:

    F10 moves the ship to the waypoint 10 times (a total of 100 units east and 10 units north), leaving the ship at east 100, north 10. The waypoint stays 10 units east and 1 unit north of the ship.
    N3 moves the waypoint 3 units north to 10 units east and 4 units north of the ship. The ship remains at east 100, north 10.
    F7 moves the ship to the waypoint 7 times (a total of 70 units east and 28 units north), leaving the ship at east 170, north 38. The waypoint stays 10 units east and 4 units north of the ship.
    R90 rotates the waypoint around the ship clockwise 90 degrees, moving it to 4 units east and 10 units south of the ship. The ship remains at east 170, north 38.
    F11 moves the ship to the waypoint 11 times (a total of 44 units east and 110 units south), leaving the ship at east 214, south 72. The waypoint stays 4 units east and 10 units south of the ship.

After these operations, the ship's Manhattan distance from its starting position is 214 + 72 = 286.

Figure out where the navigation instructions actually lead. What is the Manhattan distance between that location and the ship's starting position?


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
Markdown("The Manhattan distance of the ship from its start position is **{}**"
         .format(ferry.manhattan_dist))
```




The Manhattan distance of the ship from its start position is **23960**




```python

```
