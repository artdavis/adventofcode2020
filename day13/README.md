```python
import numpy as np
import re
import pandas as pd
from functools import reduce
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 13: Shuttle Search

Reference: https://adventofcode.com/2020/day/13
        
## Part 1

Your ferry can make it safely to a nearby port, but it won't get much further. When you call to book another ship, you discover that no ships embark from that port to your vacation island. You'll need to get from the port to the nearest airport.

Fortunately, a shuttle bus service is available to bring you from the sea port to the airport! Each bus has an ID number that also indicates how often the bus leaves for the airport.

Bus schedules are defined based on a timestamp that measures the number of minutes since some fixed reference point in the past. At timestamp 0, every bus simultaneously departed from the sea port. After that, each bus travels to the airport, then various other locations, and finally returns to the sea port to repeat its journey forever.

The time this loop takes a particular bus is also its ID number: the bus with ID 5 departs from the sea port at timestamps 0, 5, 10, 15, and so on. The bus with ID 11 departs at 0, 11, 22, 33, and so on. If you are there when the bus departs, you can ride that bus to the airport!

Your notes (your puzzle input) consist of two lines. The first line is your estimate of the earliest timestamp you could depart on a bus. The second line lists the bus IDs that are in service according to the shuttle company; entries that show x must be out of service, so you decide to ignore them.

To save time once you arrive, your goal is to figure out the earliest bus you can take to the airport. (There will be exactly one such bus.)

For example, suppose you have the following notes:
```
939
7,13,x,x,59,x,31,19
```
Here, the earliest timestamp you could depart is 939, and the bus IDs in service are 7, 13, 59, 31, and 19. Near timestamp 939, these bus IDs depart at the times marked D:
```
time   bus 7   bus 13  bus 59  bus 31  bus 19
929      .       .       .       .       .
930      .       .       .       D       .
931      D       .       .       .       D
932      .       .       .       .       .
933      .       .       .       .       .
934      .       .       .       .       .
935      .       .       .       .       .
936      .       D       .       .       .
937      .       .       .       .       .
938      D       .       .       .       .
939      .       .       .       .       .
940      .       .       .       .       .
941      .       .       .       .       .
942      .       .       .       .       .
943      .       .       .       .       .
944      .       .       D       .       .
945      D       .       .       .       .
946      .       .       .       .       .
947      .       .       .       .       .
948      .       .       .       .       .
949      .       D       .       .       .
```
The earliest bus you could take is bus ID 59. It doesn't depart until timestamp 944, so you would need to wait 944 - 939 = 5 minutes before it departs. Multiplying the bus ID by the number of minutes you'd need to wait gives 295.

What is the ID of the earliest bus you can take to the airport multiplied by the number of minutes you'll need to wait for that bus?


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
Markdown("We can catch a bust at {} minutes".format(t))
```




We can catch a bust at 1008175 minutes




```python
wait_time = t - t0
Markdown("We'll have to wait {} minutes".format(wait_time))
```




We'll have to wait 6 minutes




```python
id_wait_product = bustimes[t % bustimes == 0][0] * wait_time
Markdown("The product of the ID of the earliest bus we can take "
         "and the wait time is: **{}**".format(id_wait_product))
```




The product of the ID of the earliest bus we can take and the wait time is: **4938**



## Part 2

The shuttle company is running a contest: one gold coin for anyone that can find the earliest timestamp such that the first bus ID departs at that time and each subsequent listed bus ID departs at that subsequent minute. (The first line in your input is no longer relevant.)

For example, suppose you have the same list of bus IDs as above:
```
7,13,x,x,59,x,31,19
```
An x in the schedule means there are no constraints on what bus IDs must depart at that time.

This means you are looking for the earliest timestamp (called t) such that:

- `Bus ID 7` departs at timestamp t.
- `Bus ID 13` departs one minute after timestamp t.
- There are no requirements or restrictions on departures at two or three minutes after timestamp t.
- `Bus ID 59` departs four minutes after timestamp t.
- There are no requirements or restrictions on departures at five minutes after timestamp t.
- `Bus ID 31` departs six minutes after timestamp t.
- `Bus ID 19` departs seven minutes after timestamp t.

The only bus departures that matter are the listed bus IDs at their specific offsets from t. Those bus IDs can depart at other times, and other bus IDs can depart at those times. For example, in the list above, because bus ID 19 must depart seven minutes after the timestamp at which bus ID 7 departs, bus ID 7 will always also be departing with bus ID 19 at seven minutes after timestamp t.

In this example, the earliest timestamp at which this occurs is 1068781:
```
time     bus 7   bus 13  bus 59  bus 31  bus 19
1068773    .       .       .       .       .
1068774    D       .       .       .       .
1068775    .       .       .       .       .
1068776    .       .       .       .       .
1068777    .       .       .       .       .
1068778    .       .       .       .       .
1068779    .       .       .       .       .
1068780    .       .       .       .       .
1068781    D       .       .       .       .
1068782    .       D       .       .       .
1068783    .       .       .       .       .
1068784    .       .       .       .       .
1068785    .       .       D       .       .
1068786    .       .       .       .       .
1068787    .       .       .       D       .
1068788    D       .       .       .       D
1068789    .       .       .       .       .
1068790    .       .       .       .       .
1068791    .       .       .       .       .
1068792    .       .       .       .       .
1068793    .       .       .       .       .
1068794    .       .       .       .       .
1068795    D       D       .       .       .
1068796    .       .       .       .       .
1068797    .       .       .       .       .
```
In the above example, bus ID 7 departs at timestamp 1068788 (seven minutes after t). This is fine; the only requirement on that minute is that bus ID 19 departs then, and it does.

Here are some other examples:
```
    The earliest timestamp that matches the list 17,x,13,19 is 3417.
    67,7,59,61 first occurs at timestamp 754018.
    67,x,7,59,61 first occurs at timestamp 779210.
    67,7,x,59,61 first occurs at timestamp 1261476.
    1789,37,47,1889 first occurs at timestamp 1202161486.
```
However, with so many bus IDs in your list, surely the actual earliest timestamp will be larger than 100000000000000!


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




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>busid</th>
      <th>7</th>
      <th>13</th>
      <th>59</th>
      <th>31</th>
      <th>19</th>
    </tr>
    <tr>
      <th>t</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>8</td>
      <td>11</td>
      <td>13</td>
      <td>14</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>15</td>
      <td>18</td>
      <td>20</td>
      <td>21</td>
    </tr>
    <tr>
      <th>21</th>
      <td>21</td>
      <td>22</td>
      <td>25</td>
      <td>27</td>
      <td>28</td>
    </tr>
    <tr>
      <th>28</th>
      <td>28</td>
      <td>29</td>
      <td>32</td>
      <td>34</td>
      <td>35</td>
    </tr>
    <tr>
      <th>35</th>
      <td>35</td>
      <td>36</td>
      <td>39</td>
      <td>41</td>
      <td>42</td>
    </tr>
    <tr>
      <th>42</th>
      <td>42</td>
      <td>43</td>
      <td>46</td>
      <td>48</td>
      <td>49</td>
    </tr>
    <tr>
      <th>49</th>
      <td>49</td>
      <td>50</td>
      <td>53</td>
      <td>55</td>
      <td>56</td>
    </tr>
    <tr>
      <th>56</th>
      <td>56</td>
      <td>57</td>
      <td>60</td>
      <td>62</td>
      <td>63</td>
    </tr>
    <tr>
      <th>63</th>
      <td>63</td>
      <td>64</td>
      <td>67</td>
      <td>69</td>
      <td>70</td>
    </tr>
    <tr>
      <th>70</th>
      <td>70</td>
      <td>71</td>
      <td>74</td>
      <td>76</td>
      <td>77</td>
    </tr>
    <tr>
      <th>77</th>
      <td>77</td>
      <td>78</td>
      <td>81</td>
      <td>83</td>
      <td>84</td>
    </tr>
    <tr>
      <th>84</th>
      <td>84</td>
      <td>85</td>
      <td>88</td>
      <td>90</td>
      <td>91</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Where a timecode is integer divisible by it's
# bus ID (modulo equals 0), that is a possible
# solution
df % buses.values
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>busid</th>
      <th>7</th>
      <th>13</th>
      <th>59</th>
      <th>31</th>
      <th>19</th>
    </tr>
    <tr>
      <th>t</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>0</td>
      <td>8</td>
      <td>11</td>
      <td>13</td>
      <td>14</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0</td>
      <td>2</td>
      <td>18</td>
      <td>20</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>0</td>
      <td>9</td>
      <td>25</td>
      <td>27</td>
      <td>9</td>
    </tr>
    <tr>
      <th>28</th>
      <td>0</td>
      <td>3</td>
      <td>32</td>
      <td>3</td>
      <td>16</td>
    </tr>
    <tr>
      <th>35</th>
      <td>0</td>
      <td>10</td>
      <td>39</td>
      <td>10</td>
      <td>4</td>
    </tr>
    <tr>
      <th>42</th>
      <td>0</td>
      <td>4</td>
      <td>46</td>
      <td>17</td>
      <td>11</td>
    </tr>
    <tr>
      <th>49</th>
      <td>0</td>
      <td>11</td>
      <td>53</td>
      <td>24</td>
      <td>18</td>
    </tr>
    <tr>
      <th>56</th>
      <td>0</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>6</td>
    </tr>
    <tr>
      <th>63</th>
      <td>0</td>
      <td>12</td>
      <td>8</td>
      <td>7</td>
      <td>13</td>
    </tr>
    <tr>
      <th>70</th>
      <td>0</td>
      <td>6</td>
      <td>15</td>
      <td>14</td>
      <td>1</td>
    </tr>
    <tr>
      <th>77</th>
      <td>0</td>
      <td>0</td>
      <td>22</td>
      <td>21</td>
      <td>8</td>
    </tr>
    <tr>
      <th>84</th>
      <td>0</td>
      <td>7</td>
      <td>29</td>
      <td>28</td>
      <td>15</td>
    </tr>
  </tbody>
</table>
</div>




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




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>busid</th>
      <th>7</th>
      <th>13</th>
      <th>59</th>
      <th>31</th>
      <th>19</th>
    </tr>
    <tr>
      <th>t</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>77</th>
      <td>77</td>
      <td>78</td>
      <td>81</td>
      <td>83</td>
      <td>84</td>
    </tr>
    <tr>
      <th>168</th>
      <td>168</td>
      <td>169</td>
      <td>172</td>
      <td>174</td>
      <td>175</td>
    </tr>
    <tr>
      <th>259</th>
      <td>259</td>
      <td>260</td>
      <td>263</td>
      <td>265</td>
      <td>266</td>
    </tr>
    <tr>
      <th>350</th>
      <td>350</td>
      <td>351</td>
      <td>354</td>
      <td>356</td>
      <td>357</td>
    </tr>
    <tr>
      <th>441</th>
      <td>441</td>
      <td>442</td>
      <td>445</td>
      <td>447</td>
      <td>448</td>
    </tr>
    <tr>
      <th>532</th>
      <td>532</td>
      <td>533</td>
      <td>536</td>
      <td>538</td>
      <td>539</td>
    </tr>
    <tr>
      <th>623</th>
      <td>623</td>
      <td>624</td>
      <td>627</td>
      <td>629</td>
      <td>630</td>
    </tr>
    <tr>
      <th>714</th>
      <td>714</td>
      <td>715</td>
      <td>718</td>
      <td>720</td>
      <td>721</td>
    </tr>
    <tr>
      <th>805</th>
      <td>805</td>
      <td>806</td>
      <td>809</td>
      <td>811</td>
      <td>812</td>
    </tr>
    <tr>
      <th>896</th>
      <td>896</td>
      <td>897</td>
      <td>900</td>
      <td>902</td>
      <td>903</td>
    </tr>
    <tr>
      <th>987</th>
      <td>987</td>
      <td>988</td>
      <td>991</td>
      <td>993</td>
      <td>994</td>
    </tr>
    <tr>
      <th>1078</th>
      <td>1078</td>
      <td>1079</td>
      <td>1082</td>
      <td>1084</td>
      <td>1085</td>
    </tr>
    <tr>
      <th>1169</th>
      <td>1169</td>
      <td>1170</td>
      <td>1173</td>
      <td>1175</td>
      <td>1176</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Modulo the timecodes with their bus IDs
df2 % buses.values
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>busid</th>
      <th>7</th>
      <th>13</th>
      <th>59</th>
      <th>31</th>
      <th>19</th>
    </tr>
    <tr>
      <th>t</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>77</th>
      <td>0</td>
      <td>0</td>
      <td>22</td>
      <td>21</td>
      <td>8</td>
    </tr>
    <tr>
      <th>168</th>
      <td>0</td>
      <td>0</td>
      <td>54</td>
      <td>19</td>
      <td>4</td>
    </tr>
    <tr>
      <th>259</th>
      <td>0</td>
      <td>0</td>
      <td>27</td>
      <td>17</td>
      <td>0</td>
    </tr>
    <tr>
      <th>350</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>15</td>
      <td>15</td>
    </tr>
    <tr>
      <th>441</th>
      <td>0</td>
      <td>0</td>
      <td>32</td>
      <td>13</td>
      <td>11</td>
    </tr>
    <tr>
      <th>532</th>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>11</td>
      <td>7</td>
    </tr>
    <tr>
      <th>623</th>
      <td>0</td>
      <td>0</td>
      <td>37</td>
      <td>9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>714</th>
      <td>0</td>
      <td>0</td>
      <td>10</td>
      <td>7</td>
      <td>18</td>
    </tr>
    <tr>
      <th>805</th>
      <td>0</td>
      <td>0</td>
      <td>42</td>
      <td>5</td>
      <td>14</td>
    </tr>
    <tr>
      <th>896</th>
      <td>0</td>
      <td>0</td>
      <td>15</td>
      <td>3</td>
      <td>10</td>
    </tr>
    <tr>
      <th>987</th>
      <td>0</td>
      <td>0</td>
      <td>47</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1078</th>
      <td>0</td>
      <td>0</td>
      <td>20</td>
      <td>30</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1169</th>
      <td>0</td>
      <td>0</td>
      <td>52</td>
      <td>28</td>
      <td>17</td>
    </tr>
  </tbody>
</table>
</div>




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
Markdown("The earliset timestamp such that all bus IDs depart "
         "at offsets matching their positions in the list is: "
         "**{}**".format(t))
```




The earliset timestamp such that all bus IDs depart at offsets matching their positions in the list is: **230903629977901**




```python

```
