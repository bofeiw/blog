# COMP1521 Week 1
The first week's lecture gives an introduction to the course and some recaps of the C programming language.

## What is this course about
Other introductory courses in UNSW gets us thinking like programmers, while this course, computer system fundamentals, gets us thinking like systems programmers. 

## Major themes 
- software components of modern computer systems  
- how C programs execute (at the machine level)  
- how to write (MIPS) assembly language  
- Unix/Linux system-level programming  
- how operating systems and networks are structured  
- introduction to concurrency, concurrent programming  

## Materials
No text books but has been drawn from:
- "Introduction to Computing Systems: 
  from bits and gates to C and beyond", 
  Patt and Patel
- "From NAND to Tetris: 
  Building a modern computer system from first principles", 
  Nisan and Schocken
- COMP2121 Course Web Site, Parameswaran and Guo

Recommended reference:
- Computer Systems:
  A Programmer's Perspective, 
  Bryant and O'Halloren

# Computer Systems
![Component view of typical modern computer system](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week01/Pics/system/components.png)  
*Component view of typical modern computer system*

## Processor
Modern processors provide:
- control, arithmetic, logic, bit operators   
- relatively small set of simple operations  
- with a range of ways to determine data locations  
- small amount of very fast storage (registers)  
- small number of control registers (e.g. PC)  
- fast fetch-decode-execute cycle (ns)  
- access to system bus to communicate with other components  
- all integrated on a single chip  

## Storage
![How typical C compiler uses the memory](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week01/Pics/memory/regions.png)  
*How typical C compiler uses the memory*  
Memory (main memory) consists of:
- very large random-addressable array of bytes  
- can fetch single bytes into CPU registers  
- can fetch multi-byte chunks into CPU (e.g. 4-byte int)  
- typically: access time 0.1 µs, size 64 GB  

Disk storage consist of:
- very very large block-oriented storage  
- often on spinning disk, fetching 512B-4KB per request  
- typically: access time 30 ms, size 8 TB  
- nowadays, SSD: access time 100 µs, size 512GB  

## Computer System Layers
![View of software layers in typical computer system](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week01/Pics/system/layers.png)    
*View of software layers in typical computer system*  


## C Program Life-cycle
![From source code to machine code](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week01/Pics/compile/compilation.png)    
*From source code to machine code* 

# C programming language recap

## Type Definitions
`typedef` is very handy. It is useful when we want some variables has a consistent type. For example, we can define `Integer` to `int`, and later if we found that `int` is not enough, we can just change the definition of `Integer` to `long long int` without modifying bunch of declarations of variables. It is also used in `struct`, where we would not need to write `struct` every time we declare a `struct` variable.  
Some examples of `typedef`:
```C
typedef int Integer;
typedef long long int BigInt;
typedef unsigned char Byte;
typedef struct { int x; int y; } Coord;
```

## Assignment as Expression
Assignments in C can be treated as expressions returning a value, so `x = y = z = 0;` can be understood equivalently as `x = (y = (z = 0));` recursively.  

This is useful in combination with loops. Instead of writing 
```C
ch = getchar();
while (ch != EOF) {
   // do something with ch
   ch = getchar();
}
```
We can write it in a more concise way:
```C
while ((ch = getchar()) != EOF) {
   // do something with ch
}
```
This reminds me of the recent [PEP 572 -- Assignment Expressions](https://www.python.org/dev/peps/pep-0572/), which is added in Python 3.8, a new `:=` operator, which works like the assignment expression in C. But it might be a little bit of redundant to add yet another expression. If I were to make the decision, I would just let the `=` operator in Python to return the R value instead of creating another operator.

# Stacks, Queues, PriorityQs recap
- Stack: LIFO (Last In First Out)
- Queue: FIFO (First In First Out)
- Priority Queue: highest-priority-out lists  

The only significant difference among these data structures is that the order of retrieval of data.   

# Bit manipulations
In a 32-bit computer, the basic data types in C occupies following spaces:
- char = 1 byte = 8 bits    ('a' is 01100001)  
- short = 2 bytes = 16 bits    (42 is 0000000000101010)  
- int = 4 bytes = 32 bits    (42 is 0000000000...0000101010)  
- double = 8 bytes = 64 bits  

But they [depends on the compiler](https://stackoverflow.com/a/11438838/9494810) and will equal to the value of `sizeof`.

## Bitwise AND `&`
Performs logical AND on each corresponding pair of bits
```
  00100111           AND | 0  1  
& 11100011           ----|-----  
  --------             0 | 0  0  
  00100011             1 | 0  1  
```
This can improve the efficiency of the function to determine if a number is odd. Instead of writing `(n % 2) == 1`, we can write `n & 1`, to check the last bit of the number `n`.  

## Bitwise OR `|`
Performs logical OR on each corresponding pair of bits
```
  00100111            OR | 0  1
| 11100011           ----|-----
  --------             0 | 0  1
  11100111             1 | 1  1
```

## Bitwise NEG `~` (unary)
Performs logical negation of each bit
```
~ 00100111           NEG | 0  1
  --------           ----|-----
  11011000               | 1  0
```


