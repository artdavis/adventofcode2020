# Day 2: Password Philosophy

Reference: https://adventofcode.com/2020/day/2

## Part 1

**How many passwords are valid according to their policies?**


```python
import re
from IPython.display import Markdown
```


```python
i = 0
with open('passwd_input.txt', 'r') as fid:
    for line in fid:
        nrange, char, pw = line.split()
        nchars = re.findall(char[0], pw)
        if nchars:
            lo, hi = nrange.split('-')
            if int(lo) <= len(nchars) and int(hi) >= len(nchars):
                i+=1
```


```python
# Markdown("There were **{}** valid passwords.".format(i))
```

## Part 2

**How many passwords are valid according to the new interpretation of the policies?**


```python
i = 0
with open('passwd_input.txt', 'r') as fid:
    for line in fid:
        nrange, char, pw = line.split()
        if char[0] in pw:
            lo, hi = list(map(int, nrange.split('-')))
            # Use bitwise xor (^) to ensure char in exactly one required position
            if (char[0] == pw[lo - 1]) ^ (char[0] == pw[hi - 1]):
                i += 1
```


```python
# Markdown("There were **{}** valid passwords.".format(i))
```


```python

```
