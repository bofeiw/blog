# COMP1521 Week 3 CPU & MIPS
This week we touched on CPU architecture and MIPS Assembly language. And the lab last week is about floating points, which is very interesting.

## CPU Architecture
A typical modern CPU has: (configuration differs)
- a set of data registers
- a set of control registers (incl PC)
- an arithmetic-logic unit (ALU)
- access to memory (RAM)
- a set of simple instructions
  - transfer data between memory and registers
  - push values through the ALU to compute results
  - make tests and transfer control of execution

### Fetch-Execute Cycle
The CPU register keeps track of the program counter, and halts when reaches a halt signal. Some instructions may modify program counter, such as `JUMP`.   
![Example instruction encodings (not from a real machine)](https://www.cse.unsw.edu.au/~cs1521/19t2/lectures/week03/Pics/processor/instructions.png)  
*Example instruction encodings (not from a real machine)*  
Each instruction involves:
- determine what the operator is
- determine which registers, if any, are involved
- determine which memory location, if any, is involved
- carry out the operation with the relevant operands
- store result, if any, in appropriate register

## MIPS Architecture
MIPS is a well-known and relatively simple architecture
- qtspim: provides a GUI front-end, useful for debugging
- [spim](http://spimsimulator.sourceforge.net): command-line based version, useful for testing
- xspim: GUI front-end, useful for debugging, only in CSE labs

### MIPS CPU
- 32 × 32-bit general purpose registers
- 16 × 64-bit double-precision registers
- PC: 32-bit register (always aligned on 4-byte boundary)
- HI,LO: for storing results of multiplication and division

### Registers
- $0: always 0, cannot be overwritten
- $1: $at assembler temporary; reserved for assembler use
- $2: ($v0) value from expression evaluation or function return
- $3: ($v1) value from expression evaluation or function return
- $4: ($a0) first argument to a function/subroutine, if needed
- $5: ($a1) second argument to a function/subroutine, if needed
- $6: ($a2) third argument to a function/subroutine, if needed
- $7: ($a3) fourth argument to a function/subroutine, if needed
- $8..$15: ($t0..$t7) temporary; must be saved by caller to subroutine;subroutine can overwrite
- $16..$23: ($s0..$s7) safe function variable;must not be overwritten by called subroutine
- $24..$25: ($t8..$t9) temporary; must be saved by caller to subroutine;subroutine can overwrite
- $26..$27: ($k0..$k1) for kernel use; may change unexpectedly
- $28: ($gp) global pointer
- $29: ($sp) stack pointer
- $30: ($fp) frame pointer
- $31: ($ra) return address of most recent caller

### Floating point registers
- $f0..$f2: hold floating-point function results
- $f4..$f10: temporary registers; not preserved across function calls
- $f12..$f14: used for first two double-precision function arguments
- $f16..$f18: temporary registers; used for expression evaluation
- $f20..$f30: saved registers; value is preserved across function calls

### MIPS Assembly Language
```assembly
# hello.s ... print "Hello, MIPS"

       .data          # the data segment
msg:   .asciiz "Hello, MIPS\n"

       .text          # the code segment
       .globl main
main:
        la $a0, msg   # load the argument string
        li $v0, 4     # load the system call (print)
        syscall       # print the string
        jr $ra        # return to caller (__start)
```
*hello world*
```assembly
    .data
a:  .word 42            # int a = 42;
b:  .space 4            # int b;
    .text
    .globl main
main:   
    lw   $t0, a         # reg[t0] = a
    li   $t1, 8         # reg[t1] = 8
    add  $t0, $t0, $t1  # reg[t0] = reg[t0]+reg[t1]
    li   $t2, 666       # reg[t2] = 666
    mult $t0, $t2       # (Lo,Hi) = reg[t0]*reg[t2]
    mflo $t0            # reg[t0] = Lo
    sw   $t0, b         # b = reg[t0]
```  
*42 + 4*  

#### MIPS instructions
![MIPS instructions](https://www.cse.unsw.edu.au/~cs1521/19t2/lectures/week03/Pics/processor/mips-instr.png)  
*32 bits long*  

| Region |Address | Notes |
| ----: | :---- | :---- |
| text | 0x00400000 | contains only instructions; read-only; cannot expand |
| data | 0x10000000 | data objects; readable/writable; can be expanded |
| stack | 0x7fffefff | grows down from that address; readable/writable |
| k_text | 0x80000000 | kernel code; read-only; only accessible kernel mode |
| k_data | 0x90000000 | kernel data; read/write; only accessible kernel mode |

- syscall: 1 prints a number,  4 prints a string
- .word: allocates 4 bytes in memory and initialises it

# Fun stuffs
I was confused about the difference between `asciiz` and `ascii`. Doing some shallow searches, I know that they are both used to print ascii characters, but `asciiz` keeps printing until it meets a `\0` whereas `ascii` automatically adds a `\0` after your string. But I did not see differences by doing experiments. Doing deeper searches, I found that I used wrong method to experiment them. Initially I created one string and tried to see the different outputs between them. But later I found that there need to be something in after (e.g. another string) to let the differences appear. Because if there are only two string, using `asciiz` will stop printing anyway. But with another string, it will stop only when all strings are printed out. So `asciiz` should be used for most times to avoid unexpected behaviours. 