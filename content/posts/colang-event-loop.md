---
title: "Colang Event Loop"
date: 2022-11-26T19:03:05-08:00
draft: true
---
# Colang Event Loop

The core design goal of the Colang event loop is to support low latency, efficient and safe memory management. 

The key challenge in most memory management solutions is knowing when some allocation is no longer in use and can be reclaimed. Once some memory is reclaimed it will either be returned to the operating system or recycled in the process to satisfy a subsequent dynamic allocation.

There are many approaches to memory management today. All approaches must ballance between:

1. Memory Efficiency
2. Latency
3. Throughput
4. Safety

## Allocation and Reclamation

### Memory

### References


### Static Allocation
The simplest approach is to use only static allocations. In this strategy do dynamic allocations are used and memory layout is decided at compile time. This is most often used in embedded and safety critical systems, because there is zero runtime cost and no concern about allocations failing due to running out of capacity.

The down side is that the developer must know ahead of time how much memory every data structure needs. For systems that interact with outside data sources this is often not possible, and my lead to application errors if assumptions about load are violated. 

**Memory Efficiency:** This depends the application so no clear cut advantage or disadvantage. 

**Latency:** There is zero dynamic allocation so the latency is effectively zero because choices were made at compile time.

**Throughput:** This is not applicable as there is no allocations.

**Safety:** Clearly there can not be a use after free, or double free bug if you never free so there are safety advantages.


### Manual Memory Management
In the C language memory management is entirely manual; the programmer must explicitly ask memory to be allocated, and manually free it once it is no longer in use. This approach allows for complete control. The advantage is that the engineer can pick the trade off between resource efficiency and performance that fits the needs of there application.

For maximum performance all memory allocations can preformed prior to performance critical code, and all reclamation can be deferred until after the performance critical section. This allows carefully written code minimize latency. 

Memory management incurs latency because, both in kernel and in process, the finding, tracking, and reclaiming of memory regions involves non trivial data structures and algorithms.

For maximum resource efficiency allocation can be reclaimed as soon as they are no longer needed.

The down side of manual memory management is that evidence is that it is practically impossible to get right. Use after free, double free, and other such bugs are common reasons for CVEs and many lead to remote code execution. Given it's difficulty it also imposes a high cost in the development cycle where much time must be spent verifying correctness, and reworking code when issues are found (often after release).

**Memory Efficiency:** If the developers are willing to put the effort in this approach  allows very efficient use of available memory. Some issues with memory fragmentation that leaves pages unused or partially filled can happen, but excellent results are available.

**Latency:** Over all allocates in manually managed languages tend to be low latency, but they do have a cost, additionally in languages like C one can pick the allocator they use, so it is possible to select one the meets the needs of the application.

**Throughput:** Over all allocates in manually managed languages tend have good throughput, but they do have a cost, additionally in languages like C one can pick the allocator they use, so it is possible to select one the meets the needs of the application.

**Safety:** Safety depends on developers. Many tools and technics have been developed such as static analyzers and fuzzers have been developed but they only partially mitigate the the inherent unsafety of the approach.

### (Automatic) Reference Counting
The use of reference counting was popularized in the 90's by languages like TCL, Perl, and Python. It has seen recent resurgence in languages like ObjectiveC and Swift. 

It works by keeping a counter for each allocation which is incremented each a new reference is created and decremented each time a reference is retired. When the reference count of an allocation becomes zero it is considered to be no longer needed and is freed.

The main advantage of reference counting is that it takes the responsibility of freeing memory from the developer, greatly increasing safety. 

Reclamation is immediate and incremental keeping some of the best properties of manual memory management.

Languages that use reference counting usually do not allow pointer arithmetic or casting to pointers. This is necessary as this could result in uncounted references leading to use after free bugs.

One common issue with reference counting it that if the program ever creates a cycle, where directly or transitively, an allocation holds a reference to itself the count will never go to zero and the memory will never be reclaimed even when it is no longer in use.

In practice the greater issue with reference counting is the performance cost. In code that takes many short lived references to allocations the counter is constantly incremented and decremented. Where each of these operations are fast, over all the effect on whole program performance can be significant.

**Memory Efficiency:** Because casting to pointers and pointer arithmetic is disallowed it is more work for the a developer to control memory layout which can hurt efficiency, specifically locality is harder to achieve.  

**Latency:** Over all allocates used with reference counting tend to be low latency, but they do have a cost. This cost will be similar to allocations in manually managed languages; but unlike them one can not usually switch out the allocator.

**Throughput:** This approach takes a hit in throughput due to the cost of managing the counter. It should be noted that this cost is not payed during allocation and reclamation but during use of the allocated memory and so effects overall application performance.

**Safety:** Over all safety is good removing most of the errors that lead to unsafety in memory management. One caveat is that because circular reverences prevent collection memory may leek over time which leeds to memory exhaustion and a reduction in availability.

### Mark and Sweep
Mark and sweep works by first clearing a visited bit stored for each allocation. Then continues by finding all the pointers on the stack which point to heap allocations; these are marked as visited. The garbage collector then finds pointers in those heap allocations to other heap allocations and repeats the process. If the collector fines a pointer to an allocation but it already has it's visited bit set it is not revisited. Once no new unvisited allocations are found in a iteration, any allocations which do not have there visited bit set are unreachable and can be reclaimed.

The approach will still collect allocations with cyclic reference and avoids the maintenance cost of the counter.

As with reference counting, languages that use reference counting usually do not allow pointer arithmetic or casting to pointers. This is necessary as this could result in uncounted references leading to use after free bugs.

One complication with mark and sweep collectors if new allocations are made concurrently with collection they may not be marked correctly and incorrectly reclaimed while they are still reachable from the stack. 

The simplest approach to this is to pause the application while garbage collection is running. There are implementations called concurrent mark and sweep which do not require the main program to be paused during collection but they do impose additional complexity and throughput costs.

A second complexity of a mark and sweep garbage collector is that memory is not reclaimed until the garbage collector is run. This might be long after the memory is no longer in use. This can have surprising effects languages which support object destructors as one can not predict when or if they will ever be called.

**Memory Efficiency:** Because casting to pointers and pointer arithmetic is disallowed it is more work for the a developer to control memory layout which can hurt efficiency, specifically locality is harder to achieve.  

**Latency:** Over all allocates used with reference counting tend to be low latency, but they do have a cost. This cost will be similar to allocations in manually managed languages; but unlike them one can not usually switch out the allocator. 

If a using a stop the world collector the pauses add an additional unpredictable latency the the application.

**Throughput:** There is a wide verity of implementations of mark and sweep collection but over all collectors which stop the world tend to have higher throughput at the cost of latency, and those that have current collectors get lower latency but at the cost of throughput. 

**Safety:** Over all safety is good removing most of the errors that lead to unsafety in memory management.


### Compacting Garbage Collection

Compacting garbage collectors try to increase memory efficiency by moving allocations in to contiguous memory during garbage collection. This will reduce fragmentation. Compaction may result if fewer regions to track decreasing memory overhead reducing allocation latency.

The cost of compaction is that memory must be copied and pointers must be updated to reference the new memory. 

**Memory Efficiency:** Compacting can have positive impacts on efficiency by removing fragmentation.  

**Latency:** Compacting can reduce allocation latency but pauses dur to garbage collection may get longer.

**Throughput:** The time spent moving memory during compaction negatively effects throughput.

**Safety:** Compaction dose not have a positive or negative effect on safety.

### Generational Garbage Collection


### Lifetime Analysis

# Data Structures

![Memory Diagram](/memory-diagram.drawio.svg)


Each event on the event loop will be associated with a single 
