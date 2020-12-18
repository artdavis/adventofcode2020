```python
import pyparsing as pp
from pyparsing import pyparsing_common as ppc
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```

# Day 18: Operation Order

Reference: https://adventofcode.com/2020/day/18

## Part 1

As you look out the window and notice a heavily-forested continent slowly appear over the horizon, you are interrupted by the child sitting next to you. They're curious if you could help them with their math homework.

Unfortunately, it seems like this "math" follows different rules than you remember.

The homework (your puzzle input) consists of a series of expressions that consist of addition (+), multiplication (*), and parentheses ((...)). Just like normal math, parentheses indicate that the expression inside must be evaluated before it can be used by the surrounding expression. Addition still finds the sum of the numbers on both sides of the operator, and multiplication still finds the product.

However, the rules of operator precedence have changed. Rather than evaluating multiplication before addition, the operators have the same precedence, and are evaluated left-to-right regardless of the order in which they appear.

For example, the steps to evaluate the expression 1 + 2 * 3 + 4 * 5 + 6 are as follows:
```
1 + 2 * 3 + 4 * 5 + 6
  3   * 3 + 4 * 5 + 6
      9   + 4 * 5 + 6
         13   * 5 + 6
             65   + 6
                 71
```
Parentheses can override this order; for example, here is what happens if parentheses are added to form 1 + (2 * 3) + (4 * (5 + 6)):
```
1 + (2 * 3) + (4 * (5 + 6))
1 +    6    + (4 * (5 + 6))
     7      + (4 * (5 + 6))
     7      + (4 *   11   )
     7      +     44
            51
```

Here are a few more examples:
- `2 * 3 + (4 * 5)` becomes `26`.
- `5 + (8 * 3 + 9 + 3 * 4 * 3)` becomes `437`.
- `5 * 9 * (7 * 3 * 3 + 9 * 3 + (8 + 6 * 4))` becomes `12240`.
- `((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2` becomes `13632`.

Before you can help with the homework, you need to understand it yourself. Evaluate the expression on each line of the homework; what is the sum of the resulting values?


```python
# Test equations
eq1 = '1 + (2 * 3) + (4 * (5 + 6))' # 51
eq2 = '((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2' # 13632
```


```python
def applytoks(s, loc, toks):
    print('s:', s)
    print('loc:', loc)
    print('toks:', toks)
    if '+' == toks[1]:
        return int(toks[0]) + int(toks[2])
    if '*' == toks[1]:
        return int(toks[0]) * int(toks[2])
    raise ValueError("Unhandled token: {}".format(toks[1]))
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
Markdown("The sum of all the homework answers is "
         "**{}**".format(sum(answers)))
```




The sum of all the homework answers is **510009915468**



## Part 2

You manage to answer the child's questions and they finish part 1 of their homework, but get stuck when they reach the next section: advanced math.

Now, addition and multiplication have different precedence levels, but they're not the ones you're familiar with. Instead, addition is evaluated before multiplication.

For example, the steps to evaluate the expression 1 + 2 * 3 + 4 * 5 + 6 are now as follows:
```
1 + 2 * 3 + 4 * 5 + 6
  3   * 3 + 4 * 5 + 6
  3   *   7   * 5 + 6
  3   *   7   *  11
     21       *  11
         231
```
Here are the other examples from above:

- `1 + (2 * 3) + (4 * (5 + 6))` still becomes `51`.
- `2 * 3 + (4 * 5)` becomes `46`.
- `5 + (8 * 3 + 9 + 3 * 4 * 3)` becomes `1445`.
- `5 * 9 * (7 * 3 * 3 + 9 * 3 + (8 + 6 * 4))` becomes `669060`.
- `((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2` becomes `23340`.

What do you get if you add up the results of evaluating the homework problems using these new rules?


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
Markdown("The sum of all the 'advanced math' homework answers is "
         "**{}**".format(sum(answers2)))
```




The sum of all the 'advanced math' homework answers is **321176691637769**


