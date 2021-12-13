```python
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 21: Allergen Assessment

Reference: https://adventofcode.com/2020/day/21

## Part 1

Determine which ingredients cannot possibly contain any of the allergens in your list. **How many times do any of those ingredients appear?**


```python
menu = list()
with open('foods_input.txt', 'r') as fid:
    for line in fid:
        ingreds, allergens = line.strip().rsplit('(contains ')
        ingreds = set(ingreds.split())
        allergens = set(allergens.rstrip(')').split(', '))
        menu.append((allergens, ingreds))
```

The `menu` list is now structured like:
```
[
    {allergen1, allergen2} {ingredient1, ingredient2, ...}
    {allergen3...} {ingredient3...}
    ...
]
```


```python
# Get all known allergens and ingredients
all_allergs = set()
all_ingreds = set()
for allers, ingreds in menu:
    all_allergs |= allers
    all_ingreds |= ingreds

print("All known allergens: ")
display(all_allergs)
```

    All known allergens: 
    


    {'dairy', 'eggs', 'fish', 'nuts', 'peanuts', 'sesame', 'shellfish', 'soy'}



```python
# Find the intersection of all ingredients per
# allergen to get the set of candidate ingredients
# to contain each allergen
aller_dict = dict()
for aller in all_allergs:
    final_ingreds = set()
    for allers, ingreds in menu:
        if aller in allers:
            if 0 == len(final_ingreds):
                # Prime our ingredients set
                final_ingreds = ingreds.copy()
            else:
                final_ingreds &= ingreds
    aller_dict[aller] = final_ingreds.copy()

print("Allergens with ingredients that may contain that allergen:")
display(aller_dict)
```

    Allergens with ingredients that may contain that allergen:
    


    {'peanuts': {'dvkbjh', 'mhnrqp'},
     'eggs': {'mfp', 'mgvfmvp', 'mhnrqp', 'nhdjth'},
     'sesame': {'dcvrf', 'hcdchl', 'mhnrqp'},
     'dairy': {'hcdchl', 'mfp'},
     'soy': {'mhnrqp'},
     'shellfish': {'bcjz', 'nhdjth'},
     'nuts': {'hcdchl', 'mhnrqp'},
     'fish': {'hcdchl', 'nhdjth'}}



```python
# Generate a set of ingredients with allergens
aller_ingreds = set()
for a in aller_dict.values():
    aller_ingreds |= a
print("Ingredients that contain allergens:")
display(aller_ingreds)
```

    Ingredients that contain allergens:
    


    {'bcjz', 'dcvrf', 'dvkbjh', 'hcdchl', 'mfp', 'mgvfmvp', 'mhnrqp', 'nhdjth'}



```python
# Find the non allergenic ingredients
non_allerg_ingreds = all_ingreds - aller_ingreds
#non_allerg_ingreds
```


```python
# Count how many times non-allergen ingredients appear in menu items
count = 0
for allers, ingreds in menu:
    count += len(non_allerg_ingreds & ingreds)
```


```python
#Markdown("Non-allergenic ingredients appear in the menu "
#         "**{}** times".format(count))
```

## Part 2

**What is your canonical dangerous ingredient list?**


```python
# From the known allergens and their candidate ingredients,
# wherever there is a unique ingredient per allergen we know
# that must be the ingredient for that allergen and it can be
# removed as a candidate from all other allergens.
aller_dict2 = aller_dict.copy()
aller_dict3 = dict()
while(0 < len(aller_dict2)):
    for k, v in aller_dict2.items():
        if 1 == len(v):
            #print(k, v)
            # Known allergen
            ingred = next(iter(v))
            aller_dict3[k] = ingred
            # Remove this ingredient from all other sets
            for k1, v1 in aller_dict2.items():
                if ingred in v1:
                    v1.discard(ingred)
            break
    # Remove this item from the dictionary
    aller_dict2.pop(k)

print("Allergens and the ingredients that contains them are:")
display(aller_dict3)
```

    Allergens and the ingredients that contains them are:
    


    {'soy': 'mhnrqp',
     'peanuts': 'dvkbjh',
     'nuts': 'hcdchl',
     'sesame': 'dcvrf',
     'dairy': 'mfp',
     'fish': 'nhdjth',
     'eggs': 'mgvfmvp',
     'shellfish': 'bcjz'}



```python
dangerous_ingreds = [aller_dict3[k] for k in sorted(aller_dict3)]
```


```python
#print('The dangerous ingredients sorted alphabetically by their allergen are:')
#display(dangerous_ingreds)
```


```python
#display(Markdown("The canonical dangerous ingredient list is:"))
#Markdown('**' + ','.join(dangerous_ingreds) + '**')
```


```python

```
