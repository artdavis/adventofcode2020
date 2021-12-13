# Day 4: Passport Processing

Reference: https://adventofcode.com/2020/day/4

## Part 1

**In your batch file, how many passports are valid?**


```python
import re
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```


```python
# Read in the passport data and do 
# some preliminary parsing
rlist = list()
record = dict()
with open('passport_input.txt', 'r') as fid:
    for line in fid:
        line = line.strip()
        for field in line.split():
            k, v = field.split(':')
            record[k] = v
        if not line and record:
            rlist.append(record.copy())
            record = dict()
```


```python
# Create a list of valid records for entries that
# contain all requisite fields
reqkeys = {'byr', 'iyr', 'eyr', 'hgt', 'hcl', 'ecl', 'pid'}
vlist = list()
for rec in rlist:
    if not(reqkeys - set(rec.keys())):
        # If an empty set, all reqkeys were present
        # and record was valid
        #print("{} Valid: {}".format(i, ', '.join(list(rec.keys()))))
        vlist.append(rec)
```


```python
#Markdown("Found **{}** valid passport records".format(len(vlist)))
```

## Part 2

**In your batch file, how many passports are valid?**


```python
# Set up some useful variables for validation
rht = re.compile('(\d+)(cm|in)')
rhcl = re.compile('#[0-9a-f]{6}')
eyecolors = {'amb', 'blu', 'brn', 'gry', 'grn', 'hzl', 'oth'}
rpid = re.compile('^[0-9]{9}$')

# Keep a list of validated fields for debugging purposes
lbyr, liyr, leyr, lhgt = [list()] * 4
lhgtnum, lhgtunits, lhcl = [list()] * 3
lecl, lpid = [list()] * 2

# Slog through all the records. If any field does
# not verify, skip it and move on. If all the fields
# verify by the end of the loop, log it as verified.
verified = 0
for rec in vlist:
    # Check byr
    byr = int(rec['byr'])
    if not(1920 <= byr and 2002 >= byr):
        continue
    # Check iyr
    iyr = int(rec['iyr'])
    if not(2010 <= iyr and 2020 >= iyr):
        continue
    # Check eyr
    eyr = int(rec['eyr'])
    if not(2020 <= eyr and 2030 >= eyr):
        continue
    # Check hgt
    htm = rht.match(rec['hgt'])
    if htm is None:
        # No match means invalid height spec
        continue
    ht = int(htm[1])
    units = htm[2]
    if 'cm' == units:
        if not(150 <= ht and 193 >= ht):
            continue
    elif 'in' == units:
        if not(59 <= ht and 76 >= ht):
            continue
    else:
        # Invalid height units
        continue
    # Check hcl
    hclm = rhcl.match(rec['hcl'])
    if hclm is None:
        # Invalid hair color
        continue
    # Check ecl
    if not(rec['ecl'] in eyecolors):
        continue
    # Check pid
    pidm = rpid.match(rec['pid'])
    if pidm is None:
        # Invalid passport id
        continue
    # Still here? Information was verified
    #print("byr: {}, iyr: {}, eyr: {}, hgt: {}, hcl: {}, ecl: {}, pid: {}"
    #      .format(byr, iyr, eyr, rec['hgt'], rec['hcl'], rec['ecl'], rec['pid']))
    verified += 1
    lbyr.append(byr)
    liyr.append(iyr)
    leyr.append(eyr)
    lhgt.append(rec['hgt'])
    lhgtnum.append(ht)
    lhgtunits.append(units)
    lhcl.append(rec['hcl'])
    lecl.append(rec['ecl'])
    lpid.append(rec['pid'])
```


```python
# For debugging, make a DataFrame of the validated records
#import pandas as pd
#df = pd.DataFrame({'byr': lbyr, 'iyr': liyr, 'eyr': leyr,
#                   'hgt': lhgt, 'htnum': lhgtnum, 'htun': lhgtunits,
#                   'hcl': lhcl, 'ecl': lecl, 'pid': lpid})

# Show the range of validated 'cm' heights
#ght = df.groupby('htun')
#ght.get_group('cm')['htnum'].agg([np.min, np.max])
# Show the range of validated 'in' heights
#ght.get_group('in')['htnum'].agg([np.min, np.max])
```


```python
#Markdown("Verified **{}** passport records".format(verified))
```
