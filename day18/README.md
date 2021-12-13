```python
import pyparsing as pp
from pyparsing import pyparsing_common as ppc
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 18: Operation Order

Reference: https://adventofcode.com/2020/day/18

## Part 1

Before you can help with the homework, you need to understand it yourself. Evaluate the expression on each line of the homework; **what is the sum of the resulting values?**


```python
# Test equations
eq1 = '1 + (2 * 3) + (4 * (5 + 6))' # 51
eq2 = '((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2' # 13632
```


```python
# Addition '+' and multiplication '*' have the same precedence
op = pp.oneOf('+ *')
expr = pp.infixNotation(ppc.integer, [(op, 2, pp.opAssoc.LEFT)])
```


```python
res1 = expr.parseString(eq1)
print("Eq:", eq1)
print("Parses as:")
print(res1.asList()[0])
print('=-'*20)
res2 = expr.parseString(eq2)
print("Eq:", eq2)
print("Parses as:")
print(res2.asList()[0])
```

    Eq: 1 + (2 * 3) + (4 * (5 + 6))
    Parses as:
    [1, '+', [2, '*', 3], '+', [4, '*', [5, '+', 6]]]
    =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
    Eq: ((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2
    Parses as:
    [[[2, '+', 4, '*', 9], '*', [6, '+', 9, '*', 8, '+', 6], '+', 6], '+', 2, '+', 4, '*', 2]
    


```python
# Recursively apply operations to the supplied entry list.
def recur_list(entry, v0=None):
    for element in entry:
        if type(element) is list:
            # Recurse on it
            element = recur_list(element)
        if v0 is None:
            v0 = element
        elif type(element) is int:
            if '+' == opcode:
                v0 += element
            elif '*' == opcode:
                v0 *= element
            else:
                raise ValueError("Unhandled opcode: {}".format(opcode))

        elif element in '+*':
            opcode = element
        else:
            raise ValueError("Unhandled element: {}".format(element))
    #print(v0, entry)
    return v0
```


```python
print("Soln to eq1:")
print(recur_list(res1.asList()[0]))
print('=-'*20)
print("Soln to eq2:")
print(recur_list(res2.asList()[0]))
```

    Soln to eq1:
    51
    =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
    Soln to eq2:
    13632
    


```python
# Get all of the homework equations
with open('operation_input.txt', 'r') as fid:
    eqs = fid.read().splitlines()
```


```python
# Find the answers
parsed = [expr.parseString(eq).asList()[0] for eq in eqs]
answers = [recur_list(e) for e in parsed]
```


```python
#Markdown("The sum of all the homework answers is "
#         "**{}**".format(sum(answers)))
```

## Part 2

**What do you get if you add up the results of evaluating the homework problems using these new rules?**


```python
# Now Addition '+' has precedence over Multiplication '*'
# Addition appears first in the opList:
expr2 = pp.infixNotation(ppc.integer,
                         [('+', 2, pp.opAssoc.LEFT),
                          ('*', 2, pp.opAssoc.LEFT)])

```


```python
res1_2 = expr2.parseString(eq1)
print("Eq:", eq1)
print("Parses as:")
print(res1_2.asList()[0])
print('=-'*20)
res2_2 = expr2.parseString(eq2)
print("Eq:", eq2)
print("Parses as:")
print(res2_2.asList()[0])
```

    Eq: 1 + (2 * 3) + (4 * (5 + 6))
    Parses as:
    [1, '+', [2, '*', 3], '+', [4, '*', [5, '+', 6]]]
    =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
    Eq: ((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2
    Parses as:
    [[[[[2, '+', 4], '*', 9], '*', [[[6, '+', 9], '*', [8, '+', 6]], '+', 6]], '+', 2, '+', 4], '*', 2]
    


```python
print("Soln to eq1:")
print(recur_list(res1_2.asList()[0]))
print('=-'*20)
print("Soln to eq2:")
print(recur_list(res2_2.asList()[0]))
```

    Soln to eq1:
    51
    =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
    Soln to eq2:
    23340
    


```python
# Find the answers using the "advanced math" rules
parsed2 = [expr2.parseString(eq).asList()[0] for eq in eqs]
answers2 = [recur_list(e) for e in parsed2]
```


```python
#Markdown("The sum of all the 'advanced math' homework answers is "
#         "**{}**".format(sum(answers2)))
```


```python

```
