## BIT Manipulation

1. BitMask Construction
Compute 1-bits bitmask of length l: = ``` (1 << l) - 1;```

2. Unset the least significant bit 
``` n&(n-1)```

3. Operations on nth Bit
- Test if nth bit is set
``` num&(1<<n)```
- Similarly nth bit can be set -> ``` num|=(1<<n)```
- To unset nth bit -> ```num&=~(1<<n) ```
- Toggle nth bit -> ``` num^=(1<<n)```

4. Unset all the bits except for lsb (rightmost bit) or isolating the rightmost bit
``` num&(-num) ```
