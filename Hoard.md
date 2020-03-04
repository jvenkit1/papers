# Title:
Hoard: A scalable Memory allocator for Multithreaded applications

# Description:
For scalable applications such as web servers, database managers etc, memory allocation is often a bottleneck. Allocators suffer from problems such as poor performance, false sharing in heap organizations and in some cases, dramatic increase in memory consumption when observing continuous memory allocation and deallocation.

Hoard solves this by combining one global heap with other per-processor heaps, using techniques having low synchronization costs. It averages low average fragmentation, and improves overall program performance by a factor of 18 over the next best allocator.

# Terms:
1. False Sharing: If two processors share a cache line, despite never modifying the data accessed by each other, cache coherence and synchronization will kick in whenever either processor performs a write operation on that cache line.<br/>
Thus data is falsely shared among the 2 processors even though there is no actual inter-usage.<br/>
