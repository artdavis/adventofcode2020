# Day 6: Custom Customs

Reference: https://adventofcode.com/2020/day/6

## Part 1

For each group, count the number of questions to which anyone answered "yes". **What is the sum of those counts?**


```python
from functools import reduce
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```


```python
# Read in the customs data
with open('customs_input.txt', 'r') as fid:
    clist = [s.strip().split('\n') for s in fid.read().split('\n\n')]
```


```python
any_yes = list()
for glist in clist:
    any_yes.append(len(reduce(lambda x, y: set(x) | set(y), glist)))
```


```python
#Markdown("The sum of the number of ANYONE answering yes per group is **{}**"
#         .format(sum(any_yes)))
```

## Part 2

For each group, count the number of questions to which everyone answered "yes". **What is the sum of those counts?**


```python
every_yes = list()
for glist in clist:
    every_yes.append(len(reduce(lambda x, y: set(x) & set(y), glist)))
```


```python
#Markdown("The sum of the number of EVERYONE answering yes per group is **{}**"
#         .format(sum(every_yes)))
```


```python

```
