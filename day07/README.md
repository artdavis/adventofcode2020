```python
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
from IPython.display import Image # Syntax: Image(filename='file', width=wd, height=ht)
```

# Day 7: Handy Haversacks

Reference: https://adventofcode.com/2020/day/7


```python
Image('recur_suitcase.jpg')
```




    
![jpeg](README_files/README_2_0.jpeg)
    



Image courtesy: [Critical-Function956](https://www.reddit.com/user/Critical-Function956/)
in [r/adventofcode](https://www.reddit.com/r/adventofcode/comments/k8ccpz/shiny_gold/)

## Part 1

**How many bag colors can eventually contain at least one shiny gold bag?**


```python
bagdict = dict()
with open('bagrules_input.txt', 'r') as fid:
#with open('test_bagrules_input.txt', 'r') as fid:
    for line in fid:
        k, bagstr = line.strip().split('contain')
        # baglist contains description of available bags
        baglist = [x.strip(' .') for x in bagstr.split(',')]
        # bagset is set of bags that can be held by the bag color
        bagset = set()
        if 'no other bags' not in baglist:
            for b in baglist:
                num, color = b.split(' ', 1)
                # Remove the 'bag(s)' from the color name
                bcolor = color.rsplit(' ', 1)[0]
                bagset.add(bcolor)
        else:
            pass
        # Remove the 'bag(s)' from the key name
        kcolor = k.strip().rsplit(' ', 1)[0]
        bagdict[kcolor] = bagset
#display(bagdict)
```


```python
# Top level gold
for k, v in bagdict.items():
    if 'shiny gold' in v:
        print(k, v)
```

    clear brown {'muted gray', 'shiny gold'}
    light lime {'shiny gold', 'posh salmon'}
    drab cyan {'posh blue', 'shiny gold', 'light cyan'}
    drab yellow {'shiny gold'}
    


```python
def hasgold(color, doeshave=False):
    for c in bagdict[color]:
        if doeshave:
            # Match already found, no need to continue
            return True
        if not bagdict[c]:
            # Empty set, nothing to do
            pass
        elif 'shiny gold' in c:
            # This color bag does hold shiny gold
            return True
        else:
            # Try the bags this bag holds
            doeshave = hasgold(c)
    return doeshave

#for bc in bagdict.keys():
#    display(Markdown("**{}**".format(bc)))
#    print(hasgold(bc))
```


```python
canhold = [hasgold(x) for x in bagdict.keys()]
```


```python
#Markdown("**{}** bag colors can eventually contain one shiny gold bag".format(sum(canhold)))
```

## Part 2

**How many individual bags are required inside your single shiny gold bag?**


```python
bagdict2 = dict()
with open('bagrules_input.txt', 'r') as fid:
#with open('test2_bagrules_input.txt', 'r') as fid:
#with open('test3_bagrules_input.txt', 'r') as fid:
    for line in fid:
        k, bagstr = line.strip().split('contain')
        # baglist contains description of available bags
        baglist = [x.strip(' .') for x in bagstr.split(',')]
        # baghold is dict of bags and number held by the bag color
        baghold = dict()
        if 'no other bags' not in baglist:
            for b in baglist:
                num, color = b.split(' ', 1)
                # Remove the 'bag(s)' from the color name
                bcolor = color.rsplit(' ', 1)[0]
                baghold[bcolor] = int(num)
        # Remove the 'bag(s)' from the key name
        kcolor = k.strip().rsplit(' ', 1)[0]
        bagdict2[kcolor] = baghold
#display(bagdict2)
```


```python
def holding(color, holdsum=0):
    for c, n in bagdict2[color].items():
        holdsum += n * (holding(c) + 1)
    return holdsum

#nbags = holding('gold')
nbags = holding('shiny gold')
```


```python
#Markdown("**{}** individual bags are required inside my single shiny gold bag"
#        .format(nbags))
```


```python

```
