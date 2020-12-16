```python
import pandas as pd
pd.set_option('display.notebook_repr_html', False)
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 16: Ticket Translation

Reference: https://adventofcode.com/2020/day/16

## Part 1

As you're walking to yet another connecting flight, you realize that one of the legs of your re-routed trip coming up is on a high-speed train. However, the train ticket you were given is in a language you don't understand. You should probably figure out what it says before you get to the train station after the next flight.

Unfortunately, you can't actually read the words on the ticket. You can, however, read the numbers, and so you figure out the fields these tickets must have and the valid ranges for values in those fields.

You collect the rules for ticket fields, the numbers on your ticket, and the numbers on other nearby tickets for the same train service (via the airport security cameras) together into a single document you can reference (your puzzle input).

The rules for ticket fields specify a list of fields that exist somewhere on the ticket and the valid ranges of values for each field. For example, a rule like class: 1-3 or 5-7 means that one of the fields in every ticket is named class and can be any value in the ranges 1-3 or 5-7 (inclusive, such that 3 and 5 are both valid in this field, but 4 is not).

Each ticket is represented by a single line of comma-separated values. The values are the numbers on the ticket in the order they appear; every ticket has the same format. For example, consider this ticket:
```
.--------------------------------------------------------.
| ????: 101    ?????: 102   ??????????: 103     ???: 104 |
|                                                        |
| ??: 301  ??: 302             ???????: 303      ??????? |
| ??: 401  ??: 402           ???? ????: 403    ????????? |
'--------------------------------------------------------'
```
Here, ? represents text in a language you don't understand. This ticket might be represented as 101,102,103,104,301,302,303,401,402,403; of course, the actual train tickets you're looking at are much more complicated. In any case, you've extracted just the numbers in such a way that the first number is always the same specific field, the second number is always a different specific field, and so on - you just don't know what each position actually means!

Start by determining which tickets are completely invalid; these are tickets that contain values which aren't valid for any field. Ignore your ticket for now.

For example, suppose you have the following notes:
```
class: 1-3 or 5-7
row: 6-11 or 33-44
seat: 13-40 or 45-50

your ticket:
7,1,14

nearby tickets:
7,3,47
40,4,50
55,2,20
38,6,12
```
It doesn't matter which position corresponds to which field; you can identify invalid nearby tickets by considering only whether tickets contain values that are not valid for any field. In this example, the values on the first nearby ticket are all valid for at least one field. This is not true of the other three nearby tickets: the values 4, 55, and 12 are are not valid for any field. Adding together all of the invalid values produces your ticket scanning error rate: 4 + 55 + 12 = 71.

Consider the validity of the nearby tickets you scanned. What is your ticket scanning error rate?


```python
notemap = {'departure location': 'locdep',
           'departure station': 'stadep',
           'departure platform': 'platdep',
           'departure track': 'trackdep',
           'departure date': 'datedep',
           'departure time': 'timedep',
           'arrival location': 'locar',
           'arrival station': 'staar',
           'arrival platform': 'platar',
           'arrival track': 'trackar',
          }
readmode = 'notes'
notes = dict()
nearby = list()

def get_rangeset(rangestr):
    # Take a range string like "1-10 or 20-30"
    # and return a set of numbers it corresponds to
    r1, r2 = rangestr.split(' or ')
    r1_0, r1_1 = [int(x) for x in r1.split('-')]
    r2_0, r2_1 = [int(x) for x in r2.split('-')]
    s1 = set(range(r1_0, r1_1 + 1))
    s2 = set(range(r2_0, r2_1 + 1))
    return s1 | s2

with open('ticket_input.txt', 'r') as fid:
    for line in fid:
        if line.strip().startswith('your ticket'):
            readmode = 'ticket'
            continue
        if line.strip().startswith('nearby tickets'):
            readmode = 'nearby'
            continue
        if not line.strip():
            # Empty Line
            continue
        if 'notes' == readmode:
            k, v = line.strip().split(': ')
            rangeset = get_rangeset(v)
            if k in notemap:
                notes[notemap[k]] = rangeset
            else:
                notes[k] = rangeset
        elif 'ticket' == readmode:
            ticket = [int(x) for x in line.strip().split(',')]
        elif 'nearby' == readmode:
            nearby.append([int(x) for x in line.strip().split(',')])
        else:
            raise ValueError("Unhandled readmode: {}".format(readmode))
```


```python
# Get comprehensive set of all valid values
allvals = set()
for vals in notes.values():
    allvals |= vals

invalids = list()
for tick in nearby:
    # Put any invalid numbers we find into invalids list
    invalids.extend(set(tick) - allvals)
```


```python
Markdown("The ticket scanning error rate (sum of invalid values) "
         "is **{}**".format(sum(invalids)))
```




The ticket scanning error rate (sum of invalid values) is **29759**



## Part Two

Now that you've identified which tickets contain invalid values, discard those tickets entirely. Use the remaining valid tickets to determine which field is which.

Using the valid ranges for each field, determine what order the fields appear on the tickets. The order is consistent between all tickets: if seat is the third field, it is the third field on every ticket, including your ticket.

For example, suppose you have the following notes:
```
class: 0-1 or 4-19
row: 0-5 or 8-19
seat: 0-13 or 16-19

your ticket:
11,12,13

nearby tickets:
3,9,18
15,1,5
5,14,9
```
Based on the nearby tickets in the above example, the first position must be row, the second position must be class, and the third position must be seat; you can conclude that in your ticket, class is 12, row is 11, and seat is 13.

Once you work out which field is which, look for the six fields on your ticket that start with the word departure. What do you get if you multiply those six values together?


```python
# First off, toss out any invalid tickets
goodticks = list()
for tick in nearby:
    if (set(tick) - allvals):
        # This ticket is invalid
        continue
    goodticks.append(tick)

# Convert to DataFrame for easier handling
df = pd.DataFrame(goodticks)
df.columns.name = 'field'
df.index.name = 'ticknum'
df
```




    field      0    1    2    3    4    5    6    7    8    9   10   11   12   13  \
    ticknum                                                                         
    0        390  125  294  296  621  356  716  135  845  790  433  348  710  927   
    1        819  227  432  784  840  691  760  608  352  759   85  712  578  575   
    2        455  784  136  934  493  390  140   53  397  355  802  100  420  126   
    3         71  303  390  394   68  796  372  829  153  656  769  103  827  588   
    4        494  323  586  945  847   75  839  606  586  457  355  840  114  376   
    ...      ...  ...  ...  ...  ...  ...  ...  ...  ...  ...  ...  ...  ...  ...   
    185      489  764  761   51  500  267  858  869  786  626  499  522  753  624   
    186      793  791  577  870  411  814  336  712  816  874  367   60  521  135   
    187      259  102  136  423  310  752  486  486  229  456  371  246  725  585   
    188      602  391  715  479  116  762  148  269  403  151  768  263  857  184   
    189      303  337  657   69  622  839  603   95  294  675  803  787  628  424   
    
    field     14   15   16   17   18   19  
    ticknum                                
    0        863  136  834  139  115  323  
    1        901  151  440  494  283  274  
    2        902  870  588  498   60  607  
    3        873  595  619  149  235  785  
    4        753  207  205  823  273  840  
    ...      ...  ...  ...  ...  ...  ...  
    185       73  584  281  196  237  147  
    186       66  298  855  616  582  558  
    187      869  406  430  703  456  549  
    188      762  749   95  231  934  629  
    189       55  459  421  766  323  399  
    
    [190 rows x 20 columns]




```python
# Construct a truth table where if all fields for a given field number
# fall within the range of a field name then it is True; otherwise False
candidates = dict()
for fieldname in notes.keys():
    truthtable = list()
    for field, vals in df.iteritems():
        truthtable.append(vals.apply(lambda x: x in notes[fieldname]).all())
    candidates[fieldname] = truthtable

df2 = pd.DataFrame(candidates, index=df.columns)
df2.index.name = 'field'
df2.columns.name = 'fieldname'
df2
```




    fieldname  locdep  stadep  platdep  trackdep  datedep  timedep  locar  staar  \
    field                                                                          
    0           False   False    False     False    False    False  False  False   
    1           False   False     True      True    False    False  False  False   
    2            True    True     True      True     True    False  False  False   
    3            True    True     True      True     True     True   True   True   
    4            True    True     True      True     True     True  False   True   
    5            True    True     True      True     True     True  False  False   
    6            True    True     True      True     True     True  False  False   
    7            True    True     True      True     True     True  False   True   
    8            True    True     True      True     True     True   True   True   
    9           False   False    False     False    False    False  False  False   
    10           True    True     True      True     True     True  False  False   
    11           True    True     True      True     True     True  False  False   
    12          False   False    False     False    False    False  False  False   
    13          False   False    False     False    False    False  False  False   
    14          False    True     True      True    False    False  False  False   
    15           True    True     True      True     True     True   True   True   
    16          False   False     True     False    False    False  False  False   
    17          False    True     True      True     True    False  False  False   
    18           True    True     True      True     True     True   True   True   
    19           True    True     True      True     True     True  False  False   
    
    fieldname  platar  trackar  class  duration  price  route    row   seat  \
    field                                                                     
    0           False    False   True     False  False   True  False  False   
    1           False    False   True     False  False   True   True  False   
    2           False    False   True     False  False   True   True  False   
    3            True    False   True      True   True   True   True   True   
    4           False    False   True      True  False   True   True  False   
    5           False    False   True     False  False   True   True  False   
    6           False    False   True     False  False   True   True  False   
    7           False    False   True      True   True   True   True  False   
    8           False    False   True      True   True   True   True   True   
    9           False    False   True     False  False   True   True  False   
    10          False    False   True     False  False   True   True  False   
    11          False    False   True     False  False   True   True  False   
    12          False    False  False     False  False  False  False  False   
    13          False    False  False     False  False   True  False  False   
    14          False    False   True     False  False   True   True  False   
    15           True     True   True      True   True   True   True   True   
    16          False    False   True     False  False   True   True  False   
    17          False    False   True     False  False   True   True  False   
    18          False    False   True      True   True   True   True  False   
    19          False    False   True      True  False   True   True  False   
    
    fieldname  train   type  wagon   zone  
    field                                  
    0          False  False   True  False  
    1          False  False   True  False  
    2          False  False   True  False  
    3           True   True   True   True  
    4           True   True   True   True  
    5           True   True   True   True  
    6          False  False   True  False  
    7           True   True   True   True  
    8           True   True   True   True  
    9          False  False   True  False  
    10         False   True   True  False  
    11          True   True   True  False  
    12         False  False   True  False  
    13         False  False   True  False  
    14         False  False   True  False  
    15          True   True   True   True  
    16         False  False   True  False  
    17         False  False   True  False  
    18          True   True   True   True  
    19          True   True   True   True  




```python
# Where a column sums to 1 this identifies a field that this column
# name must be associated with since there's no other field it is True for.
df2.sum() == 1
```




    fieldname
    locdep      False
    stadep      False
    platdep     False
    trackdep    False
    datedep     False
    timedep     False
    locar       False
    staar       False
    platar      False
    trackar      True
    class       False
    duration    False
    price       False
    route       False
    row         False
    seat        False
    train       False
    type        False
    wagon       False
    zone        False
    dtype: bool




```python
# In the first case then only 'trackar' is uniquely identified
df2.columns[df2.sum() == 1][0]
```




    'trackar'




```python
# And the field index for trackar must be 15
df2.index[df2['trackar']][0]
```




    15




```python
# Iteratively perform this process eliminating field
# names and field indices as you go until all field names
# have been associeated with a field index
assocs = dict()
df3 = df2.copy()
while(0 < df3.size):
    # Keep processing df3 to build the associations
    # until it's all gone
    colname = df3.columns[df3.sum() == 1][0]
    ii = df3.index[df3[colname]][0]
    assocs[ii] = colname
    # Drop this column and row
    df3.drop(columns=colname, inplace=True)
    df3.drop(index=ii, inplace=True)

assocs
```




    {15: 'trackar',
     3: 'platar',
     8: 'seat',
     18: 'locar',
     7: 'price',
     4: 'staar',
     19: 'duration',
     5: 'zone',
     11: 'train',
     10: 'type',
     6: 'timedep',
     2: 'locdep',
     17: 'datedep',
     14: 'stadep',
     1: 'trackdep',
     16: 'platdep',
     9: 'row',
     0: 'class',
     13: 'route',
     12: 'wagon'}




```python
# Use the associations to parse our ticket
tickparse = dict()
for k, v in assocs.items():
    tickparse[v] = ticket[k]
tickparse
```




    {'trackar': 59,
     'platar': 127,
     'seat': 73,
     'locar': 107,
     'price': 53,
     'staar': 83,
     'duration': 79,
     'zone': 61,
     'train': 113,
     'type': 131,
     'timedep': 89,
     'locdep': 139,
     'datedep': 97,
     'stadep': 71,
     'trackdep': 149,
     'platdep': 103,
     'row': 67,
     'class': 137,
     'route': 101,
     'wagon': 109}




```python
soln_fields = ['timedep', 'locdep', 'datedep', 'stadep', 'trackdep', 'platdep']
soln_factors = [tickparse[k] for k in soln_fields]
soln_factors
```




    [89, 139, 97, 71, 149, 103]




```python
mx = 1
for factor in soln_factors:
    mx *= factor

Markdown("For our ticket, the product of all the fields that start "
         "with 'departure' is **{}**".format(mx))
```




For our ticket, the product of all the fields that start with 'departure' is **1307550234719**


