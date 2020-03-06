# Title:
Hoard: A scalable Memory allocator for Multithreaded applications

# Description:
For scalable applications such as web servers, database managers etc, memory allocation is often a bottleneck. Allocators suffer from problems such as poor performance, false sharing in heap organizations and in some cases, dramatic increase in memory consumption when observing continuous memory allocation and deallocation.

Hoard solves this by combining one global heap with other per-processor heaps, using techniques having low synchronization costs. It averages low average fragmentation, and improves overall program performance by a factor of 18 over the next best allocator.

# Introduction:
Requirements from an efficient allocator:
1. Speed
2. Scalability
3. Avoid False sharing
4. Low fragmentation

### False Sharing:

If two processors share a cache line, despite never modifying the data accessed by each other, cache coherence and synchronization will kick in whenever either processor performs a write operation on that cache line.<br/>
Thus data is falsely shared among the 2 processors even though there is no actual inter-usage.<br/>

### Blowup:
Increase in memory consumption caused when a concurrent allocator reclaims memory freed by the program, but fails to use it to satisfy future memory requests.<br/>
Ratio of maximum amount of memory allocated by a given allocator and maximum amount of memory allocated by an ideal uniprocessor allocator.<br/>
In simple terms, it is expected that an efficient memory allocator will allocate memory that has been freed already for future memory allocation requests, instead of making system level calls for memory allocation (read mmap). ***Blowup*** is the failure of the allocator to grant(reuse) already freed memory to future threads/processes. Instead, for systems affected by high blowup, frequent memory allocation and freeing requests are sent by the OS, which are extremely taxing on the system.

# Hoard:

Hoard allocates 2 levels of heap to allocate memory. It maintains a global heap and a per processor heap. Thus if there are 'n' processors, there will be 'n' local per processor heaps.<br/>
Each heap is composed of Superblocks and each superblock is made up of multiple blocks.<br/>

### Allocation:
A thread, when requiring memory, will request for memory from the local heap for the processor. If there exists memory available in the local heap, Hoard locks the heap, gets a block of a superblock with free space(if present on the heap).<br/>
If there exists such a superblock, it is marked as allocated and a pointer to that block is returned, and if there doesn't exist one, Hoard allocates a new superblock and inserts it into the local heap.<br/>

### Deallocation:
Each superblock has an owner i.e the processor whose heap the current superblock is on, and hence when a particular memory location is freed, the corresponding block in the superblock is marked as deallocated.<br/>
If for a particular local heap, the amount of free memory exceeds a threshold, the memory is recycled, i.e superblocks are removed from the local heap and placed onto the global heap.<br/>

### Note:
* Hence, in case of allocation, availability is first checked in the local heap. If there does not exist a superblock in the local heap, a check is made on the global heap. If this check also fails, the Operating System must provision new superblocks. However if at any of the two steps, the check succeeds, a block is simply allocated from the corresponding superblock.

# Terms:
1. **Fragmentation**: Ratio of the maximum amount of memory allocated by the operating system, and the maximum amount of memory required by the application. Excessive fragmentation causes poor data locality, leading to paging.<br/>
