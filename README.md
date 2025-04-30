# Computational_Theory
Module repository for all tasks and work from Computational Theory Module


## Task 1: Binary Representations 
task one basis itself off conversions made through bitwise operations as seen in page 62 section 2.9 of 'The C Programming langauge' 
each unsigned integer should be be matched to a 32 bit memory address using a bitwise `&` operator, once this is done, to move each integer
use the bitwise `<<` and `>>` operators which shift the bits to the left and right respectively, "filling vacated positions with zero".

**Rotation Functions**

The `rotl` and `rotr` functions implement bit rotation on 32-bit unsigned integers. These operations are useful for:
- Memory address manipulation
- Cryptographic operations
- Preventing data loss during bit shifting operations

For example, when rotating the binary value `00...0101` (decimal 5):

- Left rotation by 1 converts it to `00...1010` (decimal 10)
- Right rotation by 1 wraps the rightmost bit to the beginning: `10...0010` (decimal 2147483650)

The implementation uses the following bitwise operations:
- `&` (AND): Masks values to ensure 32-bit bounds
- `<<` (Left shift): Moves bits to the left, filling vacated positions with zeros
- `>>` (Right shift): Moves bits to the right, filling vacated positions with zeros
- `|` (OR): Combines the shifted components to complete the rotation

**Choice Function (ch)** 

The `ch` function acts as a multiplexer, selecting bits from one of two inputs based on a control input:

- For each bit position where x is 1, select the corresponding bit from y
- For each bit position where x is 0, select the corresponding bit from z

For example, with:

- `X = 1100` (control)
- `Y = 1010` (first input)
- `Z = 0101` (second input)

The result is `1001`, where:

- First bit: X=1, so take from Y (1)
- Second bit: X=1, so take from Y (0)
- Third bit: X=0, so take from Z (0)
- Fourth bit: X=0, so take from Z (1)

**Majority Function (maj)**
The `maj` function performs a "majority vote" on each bit position:

- Output bit is 1 if at least two of the three input bits are 1
- Output bit is 0 otherwise

The implementation uses the formula: `(x & y) | (x & z) | (y & z)`

## Task 2: Hash Function Implementation
This task converts the classic hash function from Kernighan and Ritchie's "The C Programming Language" to Python. 
The hash function is a fundamental component in data structures like hash tables and is crucial for efficient data retrieval operations.

**Original C Implementation**

```c
Copyunsigned hash(char *s) {
    unsigned hashval;
    for (hashval = 0; *s != '\0'; s++)
        hashval = *s + 31 * hashval;
    return hashval % 101;
}
```
**Python Conversion Details**

The conversion process addresses several language differences:
- C uses null-terminated strings, while Python strings are objects with defined length
- C directly accesses character ASCII values with pointer dereferencing, Python uses ord()
- Python can handle arbitrarily large integers, while C is limited by the unsigned type

The resulting Python implementation:
```c
pythonCopydef hash(s: str) -> int:
    hashval = 0
    for c in s:
        if c == '\0': 
            break
        hashval = ord(c) + 31 * hashval
    return hashval % 101
```

**Mathematical Properties**

The function employs several important mathematical principles:

- Polynomial Hashing: The formula effectively computes:

```Copyh(s) = (s[0] × 31^(n-1) + s[1] × 31^(n-2) + ... + s[n-1]) mod 101```

This polynomial form distributes strings across the hash space with minimal collisions.
- Prime Multiplier (31): Using 31 as a multiplier has several advantages:

  - It's a prime number, which helps in minimizing collisions
  - It's one less than a power of 2 (32), allowing compiler optimization as (n << 5) - n
  - Empirical tests show 31 produces fewer collisions than many other small primes

- Prime Modulus (101): Taking modulo by a prime number:

  - Ensures even distribution across the hash table
  - Reduces clustering that would occur with composite moduli
  - 101 is large enough to provide reasonable distribution but small enough for efficiency


## Task 3: SHA256 Padding
This task implements the padding scheme used in the SHA256 cryptographic hash function, as specified in the NIST Federal Information 
Processing Standards (FIPS) Publication 180-4.

**Cryptographic Significance**

The padding mechanism is a critical component of SHA256's security properties:

- It ensures the input can be processed in complete 512-bit blocks
- It guarantees that different messages will have different padded forms
- It incorporates the message length, contributing to the avalanche effect (small changes in input create large changes in output)

**Padding Specification**

According to FIPS 180-4, the padding consists of:

- Append a '1' bit: Add a single '1' bit to the end of the message
- Add '0' bits: Append enough '0' bits so that the length of the padded message is congruent to 448 modulo 512
- Append length: Add the original message length as a 64-bit big-endian integer

**Implementation Details**

The function transforms this bit-oriented specification into byte operations:

```python
def calc_sha256_padding(file_path):
    with open(file_path, 'rb') as f:
        message = f.read()
    
    original_bit_length = len(message) * 8
    
    padding = bytearray()
    padding.append(0x80)
    
    zero_bytes_needed = 55 - (len(message) % 64)
    if zero_bytes_needed < 0:
        zero_bytes_needed += 64

    padding.extend(b'\x00' * zero_bytes_needed)

    for i in range(7, -1, -1):
        padding.append((original_bit_length >> (i * 8)) & 0xFF)
    
    return padding
```
**Mathematical Calculation of Zero Bytes**

The calculation `55 - (len(message) % 64)` deserves special explanation:

- We need the total padded length to be a multiple of 64 bytes (512 bits)
- The padding includes: 1 byte for the '1' bit + zero bytes + 8 bytes for the length
- This equation ensures that: original_length + 1 + zero_bytes + 8 is divisible by 64

For example, with the message "abc" (3 bytes):

- Zero bytes needed = 55 - (3 % 64) = 55 - 3 = 52
- Total padded length = 3 (original) + 1 (the '1' bit) + 52 (zeros) + 8 (length) = 64 bytes

The padding output for "abc" would be:
```
80 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 18
```
Where:

- `80` represents the appended '1' bit followed by seven '0' bits
- The `00` bytes are the padding zeros
- The final `18` (decimal 24) represents the length of the original message in bits (3 bytes × 8 = 24 bits)


## Task 4: Prime Number Calculation
This task implements and compares different algorithms for finding prime numbers, showcasing fundamental concepts in number theory and algorithm optimization.

**Algorithm 1: Sieve of Eratosthenes**
The Sieve of Eratosthenes is an ancient algorithm dating back to 240 BCE. It efficiently finds all prime numbers up to a given limit by systematically eliminating multiples of each prime.

**Implementation Details:**
```python
def sieve_of_eratosthenes(n: int) -> List[int]:
    is_prime = [True for i in range(n+1)]
    p = 2

    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n+1, p):
                is_prime[i] = False
        p += 1

    primes = [p for p in range(2, n+1) if is_prime[p]]
    return primes
```
**Algorithmic Complexity:**

- **Time Complexity**: O(n log log n)
- **Space Complexity**: O(n)

**Optimization Details:**

1. Starting the inner loop from p² rather than 2p (all smaller multiples have already been marked)
2. Only checking potential primes up to √n (any composite larger than √n must have a prime factor ≤ √n)

**Visual Example:**

For n = 30, the algorithm progresses through these steps:

1. Mark multiples of 2: 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30
2. Mark multiples of 3: 9, 15, 21, 27
3. Mark multiples of 5: 25
4. No need to check 7 (7² > 30)

Result: Primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

**Algorithm 2: Trial Division**

Trial division is the most intuitive primality testing method. It checks each candidate number for divisibility by smaller numbers.

**Implementation Details:**
```python
def trial_division(n: int) -> List[int]:
    primes = []
    num = 2
    
    while len(primes) < n:
        is_prime = True
        for prime in primes:
            if prime * prime > num:
                break
            if num % prime == 0:
                is_prime = False
                break
        
        if is_prime:
            primes.append(num)
        num += 1
    
    return primes
```
**Algorithmic Complexity:**

- **Time Complexity**: O(n² / log n) for the first n primes
- **Space Complexity**: O(n)

**Optimization Details:**

- Only testing divisibility by previously found primes (not all integers)
- Only checking divisibility up to √num
- Further optimized version skips even numbers after 2

**Theoretical Analysis:**

For large values of n, the Sieve of Eratosthenes (O(n log log n)) will significantly outperform Trial Division (O(n² / log n)). 
This difference grows more pronounced as n increases, demonstrating the importance of algorithm selection for computational efficiency.

**Prime Number Theorem Application:**

The benchmark uses the Prime Number Theorem to estimate how large a range we need to search to find the first n primes:

- The theorem states that the number of primes less than x is approximately x/ln(x)
- To invert this, we estimate that the nth prime is approximately n·ln(n)
- We add a safety factor to ensure we find enough primes

This task demonstrates fundamental concepts in number theory, algorithm analysis, and optimization techniques that are essential in time sensitive computing


## Task 5: Roots
In this task we extract the first 32 bits of the fractional part of the square roots of the first 100 prime numbers.

**Implementation**

The solution uses two main functions:

- **Bit Extraction Function**: Extracts the binary representation of a number's fractional part
```python 
def get_fractional_bits(x: float, num_bits: int = 32) -> str:
    # Extract fractional part
    fractional_part = x - math.floor(x)
    
    # Extract bits using binary multiplication method
    bits = ""
    for _ in range(num_bits):
        fractional_part *= 2
        if fractional_part >= 1:
            bits += "1"
            fractional_part -= 1
        else:
            bits += "0"
    
    return bits
```

- **Square Root Processing**: Calculates square roots and extracts the fractional bits
```python 
def calculate_prime_sqrt_bits(n_primes: int = 100, num_bits: int = 32):
    # Get prime numbers using the sieve function from Task 4
    primes = sieve_of_eratosthenes(upper_bound)[:n_primes]
    
    # Calculate square roots and extract bits
    results = []
    for prime in primes:
        sqrt_value = math.sqrt(prime)
        fractional_bits = get_fractional_bits(sqrt_value, num_bits)
        results.append((prime, fractional_bits))

```

The algorithm works by repeatedly multiplying the fractional part by 2:

- If the result is ≥ 1, the next bit is 1 (and subtract 1)
- If the result is < 1, the next bit is 0

This implementation relies on the sieve_of_eratosthenes function from Task 4 to generate the prime numbers.


## Task 6: Proof Of Work
This task identifies English words with the greatest number of leading zero bits in their SHA256 hash, similar to the concept of 
"proof of work" used in current blockchain technologies.

**Implementation**

The solution uses Pythons hashlib module to calculate SHA256 hashes and analyzes their binary representation:
```python
def get_sha256_hash(word: str) -> str:
    # Calculate the SHA256 hash of a given word
    hash_obj = hashlib.sha256(word.encode('utf-8'))
    return hash_obj.hexdigest()

def count_leading_zero_bits(hash_hex: str) -> int:
    # Count the number of leading zero bits in a hash 
    binary = bin(int(hash_hex, 16))[2:].zfill(256)
    
    for i, bit in enumerate(binary):
        if bit == '1':
            return i
    return 256  # All zeros (extremely unlikely)
    
```
**The implementation**:

- Loads a dictionary of English words
- Calculates the SHA256 hash for each word
- Counts the leading zero bits in each hash
- Identifies words with the most leading zero bits

This task demonstrates the fundamental concept behind cryptocurrency mining, where the goal is to find inputs that produce 
hashes with specific properties (such as leading zeros).


## Task 7: Turing Machines
This task implements a Turing machine that adds 1 to a binary number, demonstrating fundamental concepts of computation theory and automata.
Turing Machine Design

A Turing machine consists of:

- A tape divided into cells (each containing a symbol)
- A head that can read and write symbols on the tape and move left or right
- A state register that stores the state of the machine
- A finite table of instructions that tell the machine what to do based on its current state and the symbol it's reading

**Implementation**
The designed Turing machine uses the following states and transitions:
States:
- init: Initial state, moves to the rightmost bit (LSB)
- add_one: State for adding 1 and handling carry
- add_leading_one: State for adding a new leading 1 if necessary
- halt: Final state when addition is complete
The transition function is defined as:
```python
Format: {(current_state, current_symbol): (next_state, new_symbol, head_movement)}
transitions = {
    # Initialize: Move to the rightmost bit
    ('init', '0'): ('init', '0', 'R'),
    ('init', '1'): ('init', '1', 'R'),
    ('init', 'B'): ('add_one', 'B', 'L'),
    
    # Add 1
    ('add_one', '0'): ('halt', '1', 'N'),  # 0 -> 1, we're done
    ('add_one', '1'): ('add_one', '0', 'L'),  # 1 -> 0, carry the 1
    ('add_one', 'B'): ('add_leading_one', 'B', 'R'),
    
    # Add leading 1 if needed
    ('add_leading_one', 'B'): ('halt', '1', 'N')
}
```
**Algorithm Logic**

The algorithm works by:

- Moving to the rightmost bit (least significant bit)
- Changing 0 to 1 if no carry is needed
- Changing 1 to 0 and carrying the 1 to the left if needed
- If all bits were 1, adding a new leading 1

For example, with input 100111 (decimal 39):

1. The machine processes bits from right to left
2. It changes 1→0, 1→0, 1→0 (carrying the 1 each time)
3. When it reaches 0, it changes it to 1 and halts
4. The result is 101000 (decimal 40)

This implementation demonstrates how even a simple Turing machine can perform
arithmetic operations, providing insight into the foundations of computational theory.


## Task 8: Computational Complexity

This task implements the bubble sort algorithm with a counter to track the number of comparisons made during sorting. 
The implementation is then used to analyze sorting behavior across all permutations of a 5-element list.

**Bubble Sort Algorithm**

Bubble sort is a simple comparison-based sorting algorithm that works by repeatedly stepping through the list, 
comparing adjacent elements, and swapping them if they are in the wrong order. The algorithm gets its name because 
smaller elements "bubble" to the top of the list with each pass through the array.

**Implementation Details**

The implementation includes the following features:
- **Comparison Counter:** Tracks the number of element comparisons performed during sorting
- **Early Termination:** Optimises the algorithm by stopping when no swaps occur in pass
- **Permutation Analysis:** Tests all 120 possible permutations of the list [1, 2, 3, 4, 5]


**Mathematical Analysis**

For an array of length n, bubble sort has the following complexity characteristics:

- **Best-case scenario:** `O(n)` comparisons when the array is already sorted
- **Worst-case scenario:** `O(n^2)` comparison when the array is reverse-sorted
- **Average-case scenario:**`O(n^2)` comparisons

For our specific case with `n=5`, the theoretical bounds are:
- Minimum comparisons: `n-1 = 4` (already sorted array with early termination)
- Maximum comparisons: `n(n-1)/2 = 10` (reverse-sorted array)

**Results Analyis**
When running bubble sort on all `5! = 120` permutations of [1, 2, 3, 4, 5], we observe distinctive patterns:
1. **Frequency Distribution:** The number of comparisons varies across permutations, with clusters around certain values
2. **Best Case:** The already-sorted permutation [1, 2, 3, 4, 5] requires the minimum 4 comparisons
3. **Worst Case:** The reverse-sorted permutation [5, 4, 3, 2, 1] requires the maximum 10 comparisons
4. **Average Performance:** Across all permutations, the algorithm averages approximately 8-9 comparisons


This illustrates an important principle in algorithm analysis: the actual performance varies significantly based on the initial 
ordering of elements. The early termination optimization demonstrates how practical implementations can improve upon theoretical 
worst-case bounds for specific input patterns.

**Computational Complexity Implications**

The experiment confirms the theoretical analysis of bubble sort:

- The `O(n²)` worst-case complexity is demonstrated by the maximum 10 comparisons needed
- The optimization reduces comparisons for favorable input orderings
- The distribution of comparison counts across all permutations provides insight into the algorithm's average behavior

This task provides a practical demonstration of computational complexity concepts, 
showing how theoretical analysis translates to real-world performance variations.

**References**