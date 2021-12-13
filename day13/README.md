```python
import numpy as np
import re
import pandas as pd
pd.set_option('display.notebook_repr_html', False)
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 13: Shuttle Search

Reference: https://adventofcode.com/2020/day/13
        
## Part 1

**What is the ID of the earliest bus you can take to the airport multiplied by the number of minutes you'll need to wait for that bus?**


```python
#with open('test1_bus_input.txt', 'r') as fid:
with open('bus_input.txt', 'r') as fid:
    t0 = int(fid.readline().strip())
    bustimes = fid.readline().strip()
    
bustimes
```




    '29,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,41,x,x,x,37,x,x,x,x,x,653,x,x,x,x,x,x,x,x,x,x,x,x,13,x,x,x,17,x,x,x,x,x,23,x,x,x,x,x,x,x,823,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,x,19'




```python
r1 = re.compile('(,x)+')
bustimes = np.array([int(x) for x in r1.sub('', bustimes).split(',')])
bustimes
```




    array([ 29,  41,  37, 653,  13,  17,  23, 823,  19])




```python
t = t0
while True:
    if len(np.where(t % bustimes == 0)[0]) > 0:
        # Stop as soon as we can catch a bus
        break
    t += 1
Markdown("We can catch a bus at {} minutes".format(t))
```




We can catch a bus at 1008175 minutes




```python
wait_time = t - t0
Markdown("We'll have to wait {} minutes".format(wait_time))
```




We'll have to wait 6 minutes




```python
id_wait_product = bustimes[t % bustimes == 0][0] * wait_time
```


```python
#Markdown("The product of the ID of the earliest bus we can take "
#         "and the wait time is: **{}**".format(id_wait_product))
```

## Part 2

**What is the earliest timestamp such that all of the listed bus IDs depart at offsets matching their positions in the list?**


```python
with open('test1_bus_input.txt', 'r') as fid:
#with open('bus_input.txt', 'r') as fid:
    t0 = int(fid.readline().strip())
    bustimes = fid.readline().strip()
    
#bustimes

ibus, busid = list(), list()
for i, b in enumerate(bustimes.split(',')):
    if 'x' != b:
        ibus.append(i)
        busid.append(int(b))
ibus = np.array(ibus)
busid = np.array(busid)
```


```python
# Make a series of bus IDs indexed on time offsets
buses = pd.Series(data=busid, index=ibus, name='busid', dtype=np.uint64)
buses.index.name = 'toff'
buses
```




    toff
    0     7
    1    13
    4    59
    6    31
    7    19
    Name: busid, dtype: uint64




```python
# Also make a series that indexes time offsets on bus IDs
toffs = pd.Series(data=ibus, index=busid, name='toff', dtype=np.uint64)
toffs.index.name = 'busid'
toffs
```




    busid
    7     0
    13    1
    59    4
    31    6
    19    7
    Name: toff, dtype: uint64




```python
# Make some helper functions for easily checking bus IDs
# time values and solutions

def get_tvals(t):
    # From supplied t value, compute and return requesite
    # solution t values
    return np.array(t + buses.index, dtype=np.uint64)

def get_busid_tvals(t, busid):
    # From supplied t value of supplied busid get solution t values
    return np.array(t - toffs[busid] + buses.index, dtype=np.uint64)

def check_solution(tvals):
    # Return True if supplied tvals satisfies solution
    #return (tvals % buses == 0).all()
    return (tvals % buses).sum() == 0 # slightly faster

def check_busid_solution(t, busid):
    # From supplied t value of supplied busid check if a solution
    return check_solution(get_busid_tvals(t, busid))
```


```python
# In the example data we have busid 7 at the 0 time offset.
# So we know we can increment by AT LEAST this busid since
# the solution must be integer divisible by this busid.
# Have a look at the first dozen timecodes at this step
# increment and what the subsequent time codes per bus ID
# works out to be:
tarr = np.arange(buses[0], 12*buses[0]+1, buses[0], dtype=np.uint64)
tvals_list = [get_tvals(x) for x in tarr]
df = pd.DataFrame(tvals_list, index=tarr, columns=buses)
df.index.name = 't'
df
```




    busid  7   13  59  31  19
    t                        
    7       7   8  11  13  14
    14     14  15  18  20  21
    21     21  22  25  27  28
    28     28  29  32  34  35
    35     35  36  39  41  42
    42     42  43  46  48  49
    49     49  50  53  55  56
    56     56  57  60  62  63
    63     63  64  67  69  70
    70     70  71  74  76  77
    77     77  78  81  83  84
    84     84  85  88  90  91




```python
# Where a timecode is integer divisible by it's
# bus ID (modulo equals 0), that is a possible
# solution
df % buses.values
```




    busid  7   13  59  31  19
    t                        
    7       0   8  11  13  14
    14      0   2  18  20   2
    21      0   9  25  27   9
    28      0   3  32   3  16
    35      0  10  39  10   4
    42      0   4  46  17  11
    49      0  11  53  24  18
    56      0   5   1   0   6
    63      0  12   8   7  13
    70      0   6  15  14   1
    77      0   0  22  21   8
    84      0   7  29  28  15




```python
# Of course all of the time codes for bus ID 7 are
# possible solutions (modulo 7 all equals zero)
# since we incremented the time codes by 7.
# Now look what has happened at t=77. We have two
# adjacent possible solutions since one minute later
# t=78 modulo bus ID 13 = 0. We know this must hold
# for our solution in order for all timecodes modulo
# their bus IDs evaluate to zero. This means we may
# now increment the time codes by 7 * 13 = 91 since
# incrementing by any less will not allow bus IDs
# 7 & 13 to "line up". Let's check. Construct a new
# DataFrame starting at 77 with our new time code
# increment of 91.
t2 = 77
incr = 7 * 13
tarr2 = np.arange(t2, t2 + 12*incr+1, incr, dtype=np.uint64)
tvals2_list = [get_tvals(x) for x in tarr2]
df2 = pd.DataFrame(tvals2_list, index=tarr2, columns=buses)
df2.index.name = 't'
df2
```




    busid    7     13    59    31    19
    t                                  
    77       77    78    81    83    84
    168     168   169   172   174   175
    259     259   260   263   265   266
    350     350   351   354   356   357
    441     441   442   445   447   448
    532     532   533   536   538   539
    623     623   624   627   629   630
    714     714   715   718   720   721
    805     805   806   809   811   812
    896     896   897   900   902   903
    987     987   988   991   993   994
    1078   1078  1079  1082  1084  1085
    1169   1169  1170  1173  1175  1176




```python
# Modulo the timecodes with their bus IDs
df2 % buses.values
```




    busid  7   13  59  31  19
    t                        
    77      0   0  22  21   8
    168     0   0  54  19   4
    259     0   0  27  17   0
    350     0   0   0  15  15
    441     0   0  32  13  11
    532     0   0   5  11   7
    623     0   0  37   9   3
    714     0   0  10   7  18
    805     0   0  42   5  14
    896     0   0  15   3  10
    987     0   0  47   1   6
    1078    0   0  20  30   2
    1169    0   0  52  28  17




```python
# Buses 7 & 13 line up every time!
# Now we just need to keep going until bus
# 59 lines up. From that point forward we
# could increase our time step increment to
# 7 * 13 * 59 = 5369. This increases our
# search speed dramatically. Proceed with this
# process updating the time step increment
# accordingly until ultimately reaching the solution!
```


```python
#with open('test1_bus_input.txt', 'r') as fid:
with open('bus_input.txt', 'r') as fid:
    t0 = int(fid.readline().strip())
    bustimes = fid.readline().strip()
    
#bustimes

ibus, busid = list(), list()
for i, b in enumerate(bustimes.split(',')):
    if 'x' != b:
        ibus.append(i)
        busid.append(int(b))
ibus = np.array(ibus)
busid = np.array(busid)

# Make a series of bus IDs indexed on time offsets
buses = pd.Series(data=busid, index=ibus, name='busid', dtype=np.uint64)
buses.index.name = 'toff'

# Also make a series that indexes time offsets on bus IDs
toffs = pd.Series(data=ibus, index=busid, name='toff', dtype=np.uint64)
toffs.index.name = 'busid'
```


```python
t = buses[0]
i = 1
incr = buses[0]
while True:
    # Check this time
    tvals = get_tvals(t)
    tvals_eval = tvals % buses == 0
    if tvals_eval.all():
        # This is a solution and we are done!
        #print("FOUND SOLUTION! t: {}".format(t), tvals)
        break
    if (tvals_eval)[:i].all():
        #print("t: {}; Possible solution:".format(t), tvals)
        # Up to the i'th tval is a solution. Calculate
        # the new increment
        incr = np.prod(buses[:i])
        #print("   {} -- New increment: {}".format(i, incr))
        i += 1
    t += incr
```


```python
#Markdown("The earliset timestamp such that all bus IDs depart "
#         "at offsets matching their positions in the list is: "
#         "**{}**".format(t))
```


```python

```
