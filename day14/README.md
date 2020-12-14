```python
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 14: Docking Data

Reference: https://adventofcode.com/2020/day/14


## Part 1

As your ferry approaches the sea port, the captain asks for your help again. The computer system that runs this port isn't compatible with the docking program on the ferry, so the docking parameters aren't being correctly initialized in the docking program's memory.

After a brief inspection, you discover that the sea port's computer system uses a strange bitmask system in its initialization program. Although you don't have the correct decoder chip handy, you can emulate it in software!

The initialization program (your puzzle input) can either update the bitmask or write a value to memory. Values and memory addresses are both 36-bit unsigned integers. For example, ignoring bitmasks for a moment, a line like mem[8] = 11 would write the value 11 to memory address 8.

The bitmask is always given as a string of 36 bits, written with the most significant bit (representing 2^35) on the left and the least significant bit (2^0, that is, the 1s bit) on the right. The current bitmask is applied to values immediately before they are written to memory: a 0 or 1 overwrites the corresponding bit in the value, while an X leaves the bit in the value unchanged.

For example, consider the following program:
```
mask = XXXXXXXXXXXXXXXXXXXXXXXXXXXXX1XXXX0X
mem[8] = 11
mem[7] = 101
mem[8] = 0
```
This program starts by specifying a bitmask (mask = ....). The mask it specifies will overwrite two bits in every written value: the 2s bit is overwritten with 0, and the 64s bit is overwritten with 1.

The program then attempts to write the value 11 to memory address 8. By expanding everything out to individual bits, the mask is applied as follows:
```
value:  000000000000000000000000000000001011  (decimal 11)
mask:   XXXXXXXXXXXXXXXXXXXXXXXXXXXXX1XXXX0X
result: 000000000000000000000000000001001001  (decimal 73)
```
So, because of the mask, the value 73 is written to memory address 8 instead. Then, the program tries to write 101 to address 7:
```
value:  000000000000000000000000000001100101  (decimal 101)
mask:   XXXXXXXXXXXXXXXXXXXXXXXXXXXXX1XXXX0X
result: 000000000000000000000000000001100101  (decimal 101)
```
This time, the mask has no effect, as the bits it overwrote were already the values the mask tried to set. Finally, the program tries to write 0 to address 8:
```
value:  000000000000000000000000000000000000  (decimal 0)
mask:   XXXXXXXXXXXXXXXXXXXXXXXXXXXXX1XXXX0X
result: 000000000000000000000000000001000000  (decimal 64)
```
64 is written to address 8 instead, overwriting the value that was there previously.

To initialize your ferry's docking program, you need the sum of all values left in memory after the initialization program completes. (The entire 36-bit address space begins initialized to the value 0 at every address.) In the above example, only two values in memory are not zero - 101 (at address 7) and 64 (at address 8) - producing a sum of 165.

Execute the initialization program. What is the sum of all values left in memory after it completes?


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
Markdown("The sum of all values in memory is **{}**".format(bitcalc.memsum))
```




The sum of all values in memory is **6386593869035**



## Part 2

For some reason, the sea port's computer system still can't communicate with your ferry's docking program. It must be using version 2 of the decoder chip!

A version 2 decoder chip doesn't modify the values being written at all. Instead, it acts as a memory address decoder. Immediately before a value is written to memory, each bit in the bitmask modifies the corresponding bit of the destination memory address in the following way:

    If the bitmask bit is 0, the corresponding memory address bit is unchanged.
    If the bitmask bit is 1, the corresponding memory address bit is overwritten with 1.
    If the bitmask bit is X, the corresponding memory address bit is floating.

A floating bit is not connected to anything and instead fluctuates unpredictably. In practice, this means the floating bits will take on all possible values, potentially causing many memory addresses to be written all at once!

For example, consider the following program:
```
mask = 000000000000000000000000000000X1001X
mem[42] = 100
mask = 00000000000000000000000000000000X0XX
mem[26] = 1
```
When this program goes to write to memory address 42, it first applies the bitmask:
```
address: 000000000000000000000000000000101010  (decimal 42)
mask:    000000000000000000000000000000X1001X
result:  000000000000000000000000000000X1101X
```
After applying the mask, four bits are overwritten, three of which are different, and two of which are floating. Floating bits take on every possible combination of values; with two floating bits, four actual memory addresses are written:
```
000000000000000000000000000000011010  (decimal 26)
000000000000000000000000000000011011  (decimal 27)
000000000000000000000000000000111010  (decimal 58)
000000000000000000000000000000111011  (decimal 59)
```
Next, the program is about to write to memory address 26 with a different bitmask:
```
address: 000000000000000000000000000000011010  (decimal 26)
mask:    00000000000000000000000000000000X0XX
result:  00000000000000000000000000000001X0XX
```
This results in an address with three floating bits, causing writes to eight memory addresses:
```
000000000000000000000000000000010000  (decimal 16)
000000000000000000000000000000010001  (decimal 17)
000000000000000000000000000000010010  (decimal 18)
000000000000000000000000000000010011  (decimal 19)
000000000000000000000000000000011000  (decimal 24)
000000000000000000000000000000011001  (decimal 25)
000000000000000000000000000000011010  (decimal 26)
000000000000000000000000000000011011  (decimal 27)
```
The entire 36-bit address space still begins initialized to the value 0 at every address, and you still need the sum of all values left in memory at the end of the program. In this example, the sum is 208.

Execute the initialization program using an emulator for a version 2 decoder chip. What is the sum of all values left in memory after it completes?


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
Markdown("The sum of all values in memory is **{}**".format(bitcalc.memsum))
```




The sum of all values in memory is **4288986482164**


