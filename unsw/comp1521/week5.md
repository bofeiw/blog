# COMP1521 Week 5

This week dives much deeper into MIPS and system calls.

## Structs in MIPS

![Structs in MIPS](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week05/Pics/processor/struct.png)

Struct members are accessed by offset.

```c
// new type called "struct _student"
struct _student {...};
// new type called Student
typedef struct _student Student;
```

```assembly
                  // sizeof(Student) == 56
stu1:             Student stu1;
   .space 56
stu2:             Student stu2;
   .space 56
stu:
   .space 4       Student *stu;
```

To define a struct in MIPS, `space` is used  and `sizeof` is used to determine the actual space used.

```assembly
stu1: .space 56       # Student stu1;
stu2: .space 56       # Student stu2;
# stu is $s1          # Student *stu;

li  $t0  5012345
sw  $t0, stu1+0       # stu1.id = 5012345;
li  $t0, 3778
sw  $t0, stu1+44      # stu1.program = 3778;

la  $s1, stu2         # stu = &stu2;
li  $t0, 3707
sw  $t0, 44($s1)      # stu->program = 3707;
li  $t0, 5034567
sw  $t0, 0($s1)       # stu->id = 5034567;1
```

```assembly
# Student stu; ...
# // set values in stu struct
# showStudent(stu);

   .data
stu: .space 56
   .text
   ...
   la   $t0, stu
   addi $sp, $sp, -56    # push Student object onto stack
   lw   $t1, 0($t0)      # allocate space and copy all
   sw   $t1, 0($sp)      # values in Student object
   lw   $t1, 4($t0)      # onto stack
   sw   $t1, 4($sp)
   ...
   lw   $t1, 52($t0)     # and once whole object copied
   sw   $t1, 52($sp)
   jal  showStudent      # invoke showStudent()
   ...
```

![pass struct by value](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week05/Pics/processor/pass-struct-by-value.png)

To pass whole structure to functions, push the structure onto stack.

## Computer Systems Architecture

![architecture](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week05/Pics/opsys/layers.png)

Operating systems provide an abstraction layer on top of hardware. Some characteristics of OSs:

- have privileged access to the raw machine
- manage use of machine resources (CPU, disk, memory, etc.)
- provide uniform interface to access machine-level operations
- arrange for controlled execution of user programs
- provide multi-tasking and (pseudo) parallelism

There are different types of OSs:

- batch  (e.g. Eniac, early IBM OSs)  
  computational jobs run one-at-a-time via a queue
- multi-user  (e.g. Multics, Unix/Linux, OSX, Windows)  
  mulitple jobs (appear to) run in parallel
- embedded  (e.g. Android, iOS, ...)  
  small(ish), cut-down OS embedded in a device
- real-time  (e.g. RTLinux, DuinOS, ...)  
  specialised OS with time guarantees on job completion

## Fun stuffs

There is a system call `open` (`man 2 open`) defined in `<fcntl.h>`. There is also a command called `open` in macOS. I aws confused about these two commands and thought typing `open` would trigger the system call. They are really confusing because their names are same. After searching around, I found that macOS does not provide a direct syscall like `open`, rather it just tries to open the file using installed application.
