# COMP1521 Week 7 Memory & Process Management

This week is a bit hard and need more time to fully absorb.

## Memory Manamagement

![memory](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week07/Pics/opsys/multi-process.png)

In modern machines, multiple processes can be loaded in memory. To calculate their address, each process's base address.

### Add a new process

![split](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week07/Pics/opsys/multi-process4.png)

Adding a new process where there is no one continous piece for the process, the process need to be splited into several regions on physical memory.

- each chunk of process address space has its own base
- each chunk of process address space has its own size
- each chunk of process address space has its own memory location

![table](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week07/Pics/opsys/proc-table.png)

```c
Address processToPhysical(pid, addr)
{
   Chunk chunks[] = getChunkInfo(pid);
   for (int i = 0; i < nChunks(pid); i++) {
      Chunk *c = &chunks[i];
      if (addr >= c->base && addr < c->base+c->size)
         break;
   }
   uint offset = addr - c->base;
   return c->mem + offset;
}
```

*Mapping virtual address to physical address*

The mapping **must** be done in hardware to be efficient.

If each chunck are of the same size, the mapping and mapping table would be simpler.

![table2](https://www.cse.unsw.edu.au/~cs1521/19T2/lectures/week07/Pics/opsys/proc-table2.png)

```c
Address processToPhysical(pid, Vaddr)
{
   PageInfo pages[] = getPageInfo(pid);
   uint pageno = Vaddr / PageSize;  // int div
   uint offset = Vaddr % PageSize;
   return pages[pageno].mem + offset;
}
```

*Mapping from virtual address to physical address* 

Computation of pageno,offset is efficient if `Pagesize == 2^n`

### Virtual memory

We dont need to load all of the process's pages up-front, and just need to load new process address pages into memory as needed. The strategy of dividing process memory space into fixed-size pages and on-demand loading of process pages into physical memory is called virtual memory.

Pages/frames are typically 512B .. 8KB in size.  
Each frame can hold one page of process address space.

```c
typedef struct {char status, uint frameNo, ...} PageData;

PageData *AllPageTables[maxProc];
    // one entry for each process

Address processToPhysical(pid, Vaddr)
{
   PageData *PageTable = AllPageTables[pid];
   uint pageno = PageNumberFrom(Vaddr);
   uint offset = OffsetFrom(Vaddr);
   if (PageTable[pageno].status != Loaded) {
      // load page into free frame
      // set PageTable[pageno]
   }
   uint frame = PageTable[pageno].frameNo;
   return frame * P + offset;
}
```

## Process Management

