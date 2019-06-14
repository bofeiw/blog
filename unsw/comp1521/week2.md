# COMP1521 Week 2
This week is about data.

# `make` and `Makefile`
Make (build system) is a powerful tool.  
![dependencies and actions](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week02/Pics/compile/dependencies.png)  
```make
bm : bm.o Stack.o
	gcc -o bm bm.o Stack.o

bm.o : bm.c Stack.h
	gcc -c -Wall -Werror bm.c

Stack.o : Stack.c Stack.h
	gcc -c -Wall -Werror Stack.c
```
There are three components: target, source and action.
Words before `:` is the target, after it is the source, and below it is the action.  
`make` is smart as it compares the modification timestamp of sources to decide what actions to do (if it is newly modified).

# Memory
![Memory regions during C program execution](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week02/Pics/memory/regions.png)  
*Memory regions during C program execution*  
## Physical Memory
- called main memory (or RAM, or primary storage, ...)
- indexes are "memory addresses" (a.k.a. pointers)
- data can be fetched in chunks of 1,2,4,8 bytes
- cost of fetching any byte is same (ns)
- can be volatile or non-volatile
- When addressing objects in memory, any byte address can be used to fetch 1-byte object, and byte address for N-byte object must be divisible by N
## big-endian vs little-endian
![big-endian vs little-endian](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week02/Pics/memory/endian.png)

# Data Representation
Memory allows you to load bit-strings of sizes 1,2,4,8 bytes from N-byte boundary addresses into registers in the CPU. Data representation is to give these bytes meanings.

## Character Data
Character Data is about encodings of characters.  
### ASCII (ISO 646)
- 7-bit values, using lower 7-bits of a byte (top bit always zero)
- can encode roman alphabet, digits, punctuation, control chars  
- Uses values in the range 0x00 to 0x7F (0..127)
  - control characters (0..31) ... e.g. '\0', '\n'
  - punctuation chars (32..47,91..96,123..126)
  - digits (48..57) ... '0'..'9'
  - upper case alphabetic (65..90) ... 'A'..'Z'
  - lower case alphabetic (97..122) ... 'a'..'z'
### UTF-8 (Unicode)
- UTF-8 is one of the three standard character encodings used to represent Unicode as computer text (the others being UTF-16 and UTF-32) [reference](https://qr.ae/TWhYig)
- 8-bit values, with ability to extend to multi-byte values
- can encode all human languages plus other symbols
  - The 127 1-byte codes are compatible with ASCII
  - The 2048 2-byte codes include most Latin-script alphabets
  - The 65536 3-byte codes include most Asian languages
  - The 2097152 4-byte codes include symbols and emojis
## Numeric Data
Numeric data comes in two major forms: integer and floating point numbers
### Signed Integers
- signed magnitude : first bit is sign, rest are magnitude
- ones complement : form -N by inverting all bits in N
- twos complement : form -N by inverting N and adding 1, very useful for computer
### Floating Point Numbers
![single vs double](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week02/Pics/memory/float-rep.png)  
*single vs double*  
[IEEE 754](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week02/Pics/memory/float-rep.png) Standard for Floating-Point Arithmetic  
Double is more precise than single and occupies more memory.

# Fun stuffs
For week 2's lab, we are to create a `struct` for big number and write a big number adder. One of the challenge task and one of the most interesting task is to make the `struct` more compact. At first, we used `unsigned char` to represent a digit, and I was thinking of to use a data type that occupies only 4 bits (enough to represent 1 - 16). But after searching around, I found that `unsigned char` or  `uinit8_t` is the smallest data types in C, which only occupies 1 byte, which is atomic. 