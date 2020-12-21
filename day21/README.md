```python
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 21: Allergen Assessment

Reference: https://adventofcode.com/2020/day/21

## Part 1

You reach the train's last stop and the closest you can get to your vacation island without getting wet. There aren't even any boats here, but nothing can stop you now: you build a raft. You just need a few days' worth of food for your journey.

You don't speak the local language, so you can't read any ingredients lists. However, sometimes, allergens are listed in a language you do understand. You should be able to use this information to determine which ingredient contains which allergen and work out which foods are safe to take with you on your trip.

You start by compiling a list of foods (your puzzle input), one food per line. Each line includes that food's ingredients list followed by some or all of the allergens the food contains.

Each allergen is found in exactly one ingredient. Each ingredient contains zero or one allergen. Allergens aren't always marked; when they're listed (as in (contains nuts, shellfish) after an ingredients list), the ingredient that contains each listed allergen will be somewhere in the corresponding ingredients list. However, even if an allergen isn't listed, the ingredient that contains that allergen could still be present: maybe they forgot to label it, or maybe it was labeled in a language you don't know.

For example, consider the following list of foods:
```
mxmxvkd kfcds sqjhc nhms (contains dairy, fish)
trh fvjkl sbzzf mxmxvkd (contains dairy)
sqjhc fvjkl (contains soy)
sqjhc mxmxvkd sbzzf (contains fish)
```
The first food in the list has four ingredients (written in a language you don't understand): mxmxvkd, kfcds, sqjhc, and nhms. While the food might contain other allergens, a few allergens the food definitely contains are listed afterward: dairy and fish.

The first step is to determine which ingredients can't possibly contain any of the allergens in any food in your list. In the above example, none of the ingredients kfcds, nhms, sbzzf, or trh can contain an allergen. Counting the number of times any of these ingredients appear in any ingredients list produces 5: they all appear once each except sbzzf, which appears twice.

Determine which ingredients cannot possibly contain any of the allergens in your list. How many times do any of those ingredients appear?


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
Markdown("Non-allergenic ingredients appear in the menu "
         "**{}** times".format(count))
```




Non-allergenic ingredients appear in the menu **2412** times



## Part 2

Now that you've isolated the inert ingredients, you should have enough information to figure out which ingredient contains which allergen.

In the above example:

    mxmxvkd contains dairy.
    sqjhc contains fish.
    fvjkl contains soy.

Arrange the ingredients alphabetically by their allergen and separate them by commas to produce your canonical dangerous ingredient list. (There should not be any spaces in your canonical dangerous ingredient list.) In the above example, this would be mxmxvkd,sqjhc,fvjkl.

Time to stock your raft with supplies. What is your canonical dangerous ingredient list?


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
print('The dangerous ingredients sorted alphabetically by their allergen are:')
display(dangerous_ingreds)
```

    The dangerous ingredients sorted alphabetically by their allergen are:
    


    ['mfp', 'mgvfmvp', 'nhdjth', 'hcdchl', 'dvkbjh', 'dcvrf', 'bcjz', 'mhnrqp']



```python
display(Markdown("The canonical dangerous ingredient list is:"))
Markdown('**' + ','.join(dangerous_ingreds) + '**')
```


The canonical dangerous ingredient list is:





**mfp,mgvfmvp,nhdjth,hcdchl,dvkbjh,dcvrf,bcjz,mhnrqp**


