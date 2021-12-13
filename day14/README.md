```python
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 14: Docking Data

Reference: https://adventofcode.com/2020/day/14


## Part 1

**Execute the initialization program. What is the sum of all values left in memory after it completes?**


```python
# Read the whole program in to a list
with open('dock_input.txt', 'r') as fid:
    cmdlist = fid.read().splitlines()
```


```python
class BitCalc(object):
    """
    Data structure for "decoder chip v1"
    """
    def __init__(self):
        # Initialize mask with no maskvalues
        self._mask = 'X' * 36
        self.mem = dict()
    
    @staticmethod
    def get_binstr(val):
        # Get the 36bit binary string representation of the
        # supplied values
        s = bin(val)[2:]
        if 36 < len(s):
            raise ValueError('Value too large for 36 bits')
        return '0'*(36 - len(s)) + s

    @property
    def mask(self):
        return self._mask
    @mask.setter
    def mask(self, val):
        self._mask = val
        # Compute the mask indices
        self._calc_mask_indices()
        
    def _calc_mask_indices(self):
        self._izeros, self._iones = list(), list()
        for i, c in enumerate(self.mask):
            if '0' == c:
                self._izeros.append(i)
            elif '1' == c:
                self._iones.append(i)   

    def apply_mask_to_val(self, val):
        vstr = list(self.get_binstr(val))
        for i in self._izeros:
            vstr[i] = '0'
        for i in self._iones:
            vstr[i] = '1'
        return ''.join(vstr)

    def get_masked_val(self, val):
        # Return the integer value from applying
        # mask to val
        return int(self.apply_mask_to_val(val), 2)
    
    def assign_val(self, mem, val):
        self.mem.update({mem: self.get_masked_val(val)})
        
    @property
    def memsum(self):
        return sum(self.mem.values())
```


```python
#mask0 = '00X10101X110010011XX0X011X100000X010'
```


```python
# Get an instance of our decoder
bitcalc = BitCalc()

# Run through the program
for cmd in cmdlist:
    op, val = cmd.split(' = ')
    if 'mask' == op:
        bitcalc.mask = val
    elif op.startswith('mem'):
        addr = int(op.lstrip('mem[').rstrip(']'))
        bitcalc.assign_val(addr, int(val))
    else:
        raise ValueError("Unhandled op: {}".format(op))
```


```python
#Markdown("The sum of all values in memory is **{}**".format(bitcalc.memsum))
```

## Part 2

Execute the initialization program using an emulator for a version 2 decoder chip. **What is the sum of all values left in memory after it completes?**


```python
class BitCalc2(object):
    """
    Data structure for "decoder chip v2" (memory address decoder)
    """
    def __init__(self):
        # Initialize mask as None. Must be assigned to be valid
        self._mask = None
        self.mem = dict()
    
    @staticmethod
    def get_binstr(val, bitlen=36):
        # Get the bitlen binary string representation of the
        # supplied value
        s = bin(val)[2:]
        if bitlen < len(s):
            raise ValueError('Value too large for {} bits'.format(bitlen))
        return '0'*(bitlen - len(s)) + s
    
    @property
    def mask(self):
        return self._mask
    @mask.setter
    def mask(self, val):
        self._mask = val
        # Compute the mask indices
        self._calc_mask_indices()
        
    def _calc_mask_indices(self):
        self._izeros, self._iones = list(), list()
        self._ix = list()
        for i, c in enumerate(self.mask):
            if '0' == c:
                # Zeros do not change anything when applied
                # May be able to just drop this
                self._izeros.append(i)
            elif '1' == c:
                self._iones.append(i)   
            elif 'X' == c:
                self._ix.append(i)
            else:
                raise ValueError("Unsupported bit value: {}".format(c))

    def apply_mask_to_addr(self, addr):
        # Applying the mask to the address should
        # return a list of address indices to vary
        combos = 2 ** len(self._ix)
        bincombos = [self.get_binstr(x, len(self._ix))
                     for x in range(combos)]
        icombos = list()
        for ib in bincombos:
            icombos.append([self._ix[i]
                            for i, j in enumerate(ib) if j == '1'])
        return icombos

    def get_masked_addrs(self, addr):
        # Return list of addresses for addr with mask applied
        binaddr = list(self.get_binstr(addr))
        # Apply self._iones to the binaddr
        for i in self._iones:
            binaddr[i] = '1'
        ixs = self.apply_mask_to_addr(addr)
        addrlist = list()
        for ix in ixs:
            b = binaddr.copy()
            # Set zero at all self._ix locations
            for i in self._ix:
                b[i] = '0'
            if ix:
                # ix is not an empty list
                for i in ix:
                    b[i] = '1'
            addrlist.append(int(''.join(b), 2))
        return addrlist
    
    def assign_val(self, mem, val):
        self.mem.update({addr: val for addr in self.get_masked_addrs(mem)})
        
    @property
    def memsum(self):
        return sum(self.mem.values())
```


```python
# Testing
#m0 = '000000000000000000000000000000X1001X'
#m1 = '00000000000000000000000000000000X0XX'
#b = BitCalc2()
#b.mask = m0
#b.assign_val(42, 100)
#b.mask = m1
#b.assign_val(26, 1)
#b.memsum
#b.apply_mask_to_addr(42)
#b.apply_mask_to_addr(26)
#b.get_masked_addrs(42)
#b.get_masked_addrs(26)
#icombos = list()
#for ib in bincombos:
#    icombos.append([xs[i] for i, j in enumerate(ib) if j == '1'])
#icombos
```


```python
# Get an instance of our v2 decoder
bitcalc = BitCalc2()

# Run through the program
for cmd in cmdlist:
    op, val = cmd.split(' = ')
    if 'mask' == op:
        bitcalc.mask = val
    elif op.startswith('mem'):
        addr = int(op.lstrip('mem[').rstrip(']'))
        bitcalc.assign_val(addr, int(val))
    else:
        raise ValueError("Unhandled op: {}".format(op))
```


```python
#Markdown("The sum of all values in memory is **{}**".format(bitcalc.memsum))
```


```python

```
