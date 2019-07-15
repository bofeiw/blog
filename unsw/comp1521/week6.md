# COMP1521 Week 6 files

This week is all about files.

## File Systems

![file tree](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week06/Pics/opsys/unix-fs.png)
*Unix/Linux file tree*

(We say it's "tree structured", but symlinks actually make it into a graph)

- File systems manage stored data (e.g. on disk, SSD)
- File = named sequence of bytes, stored on device
  - file system maps name to location on device
  - file system maintains meta-data (e.g. access rights)
- Directory = file containing references to other files

![stream](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week06/Pics/filesys/streams.png)

- System calls provide low-level API to manipulate files  (byte streams)
- Libraries provide higher-level API to manipulate files  (text streams)

## File path

- absolute path (full path from root)  
  eg. /usr/include/stdio.h
- reletive path (path starts from CWD(Current Working Directory))  
  eg. ../../another/path/prog.c ./a.out

**Why `./` before `a.out`?**

It does not lead to redundency, but it is safer and more readable. If you have a runnable file named `ls` which deletes all important files, then when you run actual ls in your terminal, catastrophe happens. Luckily you have to type `./ls` to run your dangerous program but `ls` to run the programs in PATH.

## File Descriptor

In Unix, a file descriptor is a none negative integer, which maps to files in the Unix process. The total amount of descriptors has limitation.

**Closing files after use matters**  
Each opened file occupies a file descriptor, and if you do not close the file after use, you will end up with cannot open more files when you open many files. It is essential to close a file when dealing with large amount of files.
