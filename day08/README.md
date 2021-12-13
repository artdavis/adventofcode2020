# Day 8: Handheld Halting

Reference: https://adventofcode.com/2020/day/8

## Part 1

Immediately before any instruction is executed a second time, **what value is in the accumulator?**


```python
from IPython.display import Markdown
#from IPython.core.debugger import set_trace as breakpt
```


```python
with open('code_input.txt', 'r') as fid:
#with open('test1_code_input.txt', 'r') as fid:
    stack = fid.read().splitlines()

#stack
```


```python
def parse_cmd(cmdstr, ptr):
    cmd, val = cmdstr.split(' ')
    val = int(val)
    if 'acc' == cmd:
        accumval = val
        ptr += 1
    elif 'jmp' == cmd:
        accumval = 0
        ptr += val
    elif 'nop' == cmd:
        accumval = 0
        ptr += 1
    else:
        raise ValueError("Unhandled cmd '{}''".format(cmd))
    return (ptr, accumval)

accum = 0
ptr = 0
ptrs_seen = set()
ptr = 0
while(True):
    #print(ptr, accum, stack[ptr])
    ptr, accumval = parse_cmd(stack[ptr], ptr)
    if ptr in ptrs_seen:
        #print("Already seen ptr: {}".format(ptr))
        #print("Accumulator value: {}".format(accum))
        break
    if accumval:
        accum += accumval
    ptrs_seen.add(ptr)
    
```


```python
#Markdown("Accumulator before program hang: **{}**".format(accum))
```

## Part 2

**What is the value of the accumulator after the program terminates?**


```python
# Convert stack to 2 lists of ops and vals
stack_ops, stack_params = list(zip(*[x.split(' ') for x in stack]))
stack_ops = list(stack_ops)
stack_vals = [int(x) for x in stack_params]
```


```python
def parse_op(cmd, val, ptr):
    if 'acc' == cmd:
        accumval = val
        ptr += 1
    elif 'jmp' == cmd:
        accumval = 0
        ptr += val
    elif 'nop' == cmd:
        accumval = 0
        ptr += 1
    else:
        raise ValueError("Unhandled cmd '{}''".format(cmd))
    return (ptr, accumval)

def test_hang(stack_ops, stack_vals):
    # Program succeeds if it gets to the last
    # instruction at ptr_end in ehich case return the
    # accumulator values. Otherwise return None
    ptr_end = len(stack_ops) - 1
    accum = 0
    ptr = 0
    ptrs_seen = set()
    ptr = 0
    while(True):
        #print(ptr, accum, stack[ptr])
        ptr, accumval = parse_op(stack_ops[ptr], stack_vals[ptr], ptr)
        if ptr in ptrs_seen:
            #print("Already seen ptr: {}".format(ptr))
            #print("Accumulator value: {}".format(accum))
            # Program hangs
            #return [None, ptr, accum]
            return None
        if accumval:
            accum += accumval
        if ptr == ptr_end:
            # Got to the end. Success!
            return accum
        ptrs_seen.add(ptr)


#test_hang(stack_ops, stack_vals)
```


```python
# Process stack_ops each time swapping a jmp for a nop or vice versa
accum = None
for i, op in enumerate(stack_ops):
    if 'nop' == op:
        so = stack_ops.copy()
        so[i] = 'jmp'
    elif 'jmp' == op:
        so = stack_ops.copy()
        so[i] = 'nop'
    else:
        # Nothing to try changing
        continue
    accum = test_hang(so, stack_vals)
    if accum is not None:
        print("SUCCESS!")
        break
```

    SUCCESS!
    


```python
#Markdown("Accumulator for successful program run: **{}**".format(accum))
```
