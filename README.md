# Computational_Theory
Module repository for all tasks and work from Computational Theory Module


Task: 1: 
task one basis itself off conversions made through bitwise operations as seen in page 62 section 2.9 of 'The C Programming langauge' 
each unsigned integer should be be matched to a 32 bit memory address using a bitwise `&` operator, once this is done, to move each integer
use the bitwise `<<` and `>>` operators which shift the bits to the left and right respectively, "filling vacated positions with zero".
as an example to how this would work, here is an example:
```c
int main(int x, int n) {
    n = 1;
    x = 1101; // binary number for 13 (4 bits)
    
    // to fufill the requirement we will split our number into two parts, a left and right section, these will be combined later
    int left_part = (x << n);
    printf(left_part); //RESULT: 1010 - number is shifted left 1 place, original 1 is lost from falling outside bounds
    
    /* now that we have rotated the number by 1, we have lost a vital 1 from the total binary, we need to retrive this, to do such we can 
    /* use the bitwise shift right operator, to get 1 on its own */
   int right_part = (x >> (4 - n)); // shifts x value to the right by the number of bits (4) subtracted by the number of spaces 
   printf(right_part); // RESULT: 0001 - number is shifted right 3 places, leaving us with the singular 1 we need
   
   // now we can combine the two numbers we have obtained using the bitwise OR/| operator
   int result = left_part | right_part // left_part(1010) + right_part(0001) = 1011
  }
```
The next key part in task 1 is the ch function, which acts as a [multiplexer](https://en.wikipedia.org/wiki/Multiplexer) for the bits, we take in three inputs `X`, `Z`, 
and `Y`, and process them through logic based on using the bitwise XOR `^` operator, we match and merge bits, this means if
before processing we had `X = 1100` and `Y = 1010`, and `Z = 0101` using the XOR operator we process the following operation:
- First bit: X=1, so take from Y (1)
- Second bit: X=1, so take from Y (0)
- Third bit: X=0, so take from Z (0)
- Fourth bit: X=0, so take from Z (1)

this gives us an output of `1001`, acting as a safe and secure way of matching two selected parts without any interference, it is worth noting the usage of bitwise NOT `~`
which gets inverse bits from X, meaning the bit will go through if it is 0