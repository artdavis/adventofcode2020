```python
from IPython.display import Markdown
from IPython.core.debugger import set_trace
```

# Day 25: Combo Breaker

Reference: https://adventofcode.com/2020/day/25

## Part 1

**What encryption key is the handshake trying to establish?**


```python
with open('pubk_input.txt', 'r') as fid:
    pubk1 = int(fid.readline().strip())
    pubk2 = int(fid.readline().strip())
```


```python
def get_val(loopsize, subjectnum=7, val=1):
    for i in range(loopsize):
        val = (val * subjectnum) % 20201227
    return val

def seek_loop(pubk, subjectnum=7, val=1):
    loopsize = 0
    while(val != pubk) :
        loopsize += 1
        val = (val * subjectnum) % 20201227
    return val, loopsize

```


```python
pubk1_verify, loop1 = seek_loop(pubk1)
assert pubk1 == pubk1_verify
Markdown("Loop size for the card was found to be {}".format(loop1))
```




Loop size for the card was found to be 13207740




```python
pubk2_verify, loop2 = seek_loop(pubk2)
assert pubk2 == pubk2_verify
Markdown("Loop size for the door was found to be {}".format(loop2))
```




Loop size for the door was found to be 8229037




```python
secretk1 = get_val(loop1, subjectnum=pubk2)
secretk2 = get_val(loop2, subjectnum=pubk1)
assert secretk1 == secretk2
```


```python
#Markdown("The shared secret key using the card loop size "
#         "and the door public key is **{}**".format(secretk1))
```


```python
#Markdown("The shared secret key using the door loop size "
#         "and the card public key is **{}**".format(secretk2))
```

## Part Two

Looks like you only needed 49 stars after all.


```python

```
