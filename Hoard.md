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

False Sharing occurs when multiple processors share/store words in the same cache line, without actually sharing data. That is an allocator can divide a cache line into number of smaller objects that distinct processors then write to.<br/>


# Terms:
1. **False Sharing**: If two processors share a cache line, despite never modifying the data accessed by each other, cache coherence and synchronization will kick in whenever either processor performs a write operation on that cache line.<br/>
Thus data is falsely shared among the 2 processors even though there is no actual inter-usage.<br/>

2. **Fragmentation**: Ratio of the maximum amount of memory allocated by the operating system, and the maximum amount of memory required by the application. Excessive fragmentation causes poor data locality, leading to paging.<br/>

3. **Blowup**: Increase in memory consumption caused when a concurrent allocator reclaims memory freed by the program, but fails to use it to satisfy future memory requests.<br/>
Ratio of maximum amount of memory allocated by a given allocator and maximum amount of memory allocated by an ideal uniprocessor allocator.<br/>
