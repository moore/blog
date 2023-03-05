---
title: "Memory Management"
date: 2023-02-26T14:15:43-08:00
draft: false
---
# Memory Management

Colang intends to achieve high throughput and low latency using two strategies: static typing and dispatch, and efficient memory management. In this post we will review common approaches to memory management. The post is background for a later post describing memory management Colang.

## Balance
The key challenge in memory management is knowing when some allocation is no longer in use and can be reclaimed. Once some memory is reclaimed it will either be returned to the operating system or recycled in the process to satisfy a subsequent dynamic memory allocation.

There are many approaches to memory management today. All approaches must ballance between:

1. Efficiency
2. Latency
3. Throughput
4. Safety

### Efficiency
A memory management approach is can be said to be efficient if the majority of memory provided by the hardware can be put to use serving the functional goals of the software. Common causes of inefficient use of memory are overhead, fragmentation, and stale allocations. 

Overhead is caused by additional memory utilization for the sole purpose of the memory management.

Fragmentation occurs when there are ranges of memory which can not be allocated for functional use either because they are too small or due to inefficiency in the allocation strategy.

Stale allocation occur when an allocation is no longer needed for the function of the application but is not available to service new allocation.

### Latency
Latency is the number of cycles spent servicing an allocation request.

### Throughput
Is the total allocations per unit time an allocator can service. It is not only dependent on latency but also on time spent on reclaim and copying in the case of moving memory manager.

### Safety
The property of safety inf a memory manger is how it protects the developer form misusing allocations in a way the leads to functional defects. Used after free, and double free bugs are examples of unsafety in C language memory managers.

## Approaches

In the next section we will compare many existing approaches to memory management used today. Each will be described and examined though the lense of efficiency, latency, throughput, and safety.

### Static Allocation
The simplest approach is to use only static allocations. In this strategy no dynamic allocations are used and memory layout is decided at compile time. This is most often used in embedded and safety critical systems, there is zero runtime cost and no concern about allocations failing due to lack of capacity at runtime.

The down side is that the developer must know ahead of time how much memory every data structure needs. For systems that interact with outside data sources this is often not possible, and my lead to application errors if assumptions about load are incorrect. 

**Memory Efficiency:** This depends the application but it is possible to reach very high efficiency as there is no overhead from tracking allocation.

**Latency:** There is zero dynamic allocation so the latency is effectively zero.

**Throughput:** This is not applicable as there is no allocations.

**Safety:** Clearly there can not be a use after free, or double free bug if you never free so there are safety advantages; but to reach higher efficiency it may be that one region might be used for more then one purpose which could lead to serious errors. 

### Bump Allocation

A Bump Allocator uses a very simple strategy where new allocations can be serviced dynamically. The work by tracking a single pointer in to available memory and incrementing the pointer to service each allocation.  This leads to a very low latency and high thought allocator but allocations can never be returned except by decrementing the free pointer. In practice most implementations of bump allocator only support resetting the free pointer to its initial state reclaiming all allocations service as a single operation. 

**Memory Efficiency:** Bump allocators have very low overhead but often lead to a large number of stale allocations leading to low efficiency.

**Latency:** Latency is very low as allocations can be serviced by a single increment operation.

**Throughput:** Bump allocators achieve very high throughput.

**Safety:** The lack of reclaim in bump allocators removes most allocator related unsafety issues. 

### Manual Memory Management
In the C language memory management is entirely manual; the programmer must explicitly ask memory to be allocated, and manually free it once it is no longer in use. This approach allows for complete control. The advantage is that the engineer can pick the trade off between resource efficiency and performance that fits the needs of there application.

For maximum performance all memory allocations can preformed prior to performance critical code, and all reclamation can be deferred until after a performance critical section. This allows carefully written code minimize latency. 

The down side of manual memory management is that it is practically impossible to do safely. Use after free, double free, and other such bugs are common reasons for security critical defects, as well as crashes and memory leeks which reduce availability. Given it's difficulty manual memory management also imposes a high cost in the development cycle where  time must be spent verifying correctness, and reworking code when issues are found (often after release).

**Memory Efficiency:** This approach allows very efficient use of available memory. Some issues with memory fragmentation but can occur and some overhead exists for tracking free pages suitable for allocation. Over with manual management excellent efficiency is possible.

**Latency:** Allocates in manually managed languages tend to be low latency, but they do have a measurable cost. An advantage is that in languages like C, one can pick the allocator used. Because of this it is possible to select an allocator the meets the specific needs of the application.

**Throughput:** Over all allocates in manually managed languages tend have good throughput, but have a measurable cost. An advantage is that in languages like C, one can pick the allocator used. Because of this it is possible to select an allocator the meets the specific needs of the application.

**Safety:** Safety depends on developers. Many tools and technics have been developed to aid developers, such as static analyzers and fuzzers, but they only partially mitigate the the inherent unsafety of manual memory management.

### (Automatic) Reference Counting
The use of reference counting was popularized in the 90's by languages like TCL, Perl, and Python. It has seen recent resurgence in languages like ObjectiveC and Swift. 

It works by keeping a counter for each allocation, incrementing the counter when a new reference to an allocations is created and decremented each time a reference is retired. When the reference count of an allocation becomes zero it is considered unused and is reclaimed.

The main advantage of reference counting is that it takes the responsibility of reclaiming memory from the developer, greatly increasing safety. 

Reclamation is immediate and incremental keeping some of the best properties of manual memory management.

Languages that use reference counting usually do not allow pointer arithmetic or casting to pointers. This is necessary as this could result in uncounted references leading to use after free bugs.

One common issue with reference counting it that if the program ever creates a reference cycle, where directly or transitively, an allocation holds a reference to itself, the reference count may never go to zero and the memory will never be reclaimed.

In practice the greater issue with reference counting is the performance cost. In code that takes many short lived references the reference counter is constantly incremented and decremented. Where each of these operations are fast, over all the effect on whole program performance can be significant.


**Memory Efficiency:** Reference counting memory allocators will have similar efficiency to manually managed memory with the difference being the in the need to increase allocation sizes to allow for the counter.

**Latency:** Reference counting tend to be low latency. This cost will be similar to allocations in manually managed languages; but unlike them one can not usually switch out the allocator.

**Throughput:** This approach takes a hit in throughput due to the cost of managing the counter. It should be noted that this cost is not payed during allocation and reclamation but during use of the allocated memory and so effects overall application performance.

**Safety:** Over all safety is good; removing most of the errors that lead to unsafety in memory management. One caveat is that because circular reverences prevent collection, memory may leek over time leading to to memory exhaustion.

### Mark and Sweep
Mark and sweep is a reclaim strategy which works by starting with know live allocations, referred to as roots, and scanning each allocation for reference to other allocations. This process is repeated until no new allocation are found. At this point any outstanding allocations which were not visited during the process must be unused and can be reclaimed.

In contrast to reference counting, mark and sweep will collect allocations with cyclic references and avoids the maintenance cost of the counter.

As with reference counting, languages that use reference counting usually do not allow pointer arithmetic or casting to pointers. This is necessary as this could result in uncounted references leading to use after free bugs.

One complication with mark and sweep collectors is that in there simple form, new allocations can not be made concurrently with collection. If this were avowed the new allocations may not be visited and reclaimed while they are still in use.

Because of this many mark and sweep collectors pause the application while garbage collection is running. There are implementations called concurrent mark and sweep which do not require the main program to be paused during collection, but they impose additional complexity and throughput costs.

A second artifact of a mark and sweep garbage collector is that memory is not reclaimed until the garbage collection is run. This might be long after a allocation is no longer in use. Because of this delay surprising effects in languages which support object destructors as it can not predicted when or if a destructor will ever be called.

**Memory Efficiency:** Mark and sweep memory allocator will have similar efficiency to manually managed memory.

**Latency:** Over all allocates used with mark and sweep tend to be low latency, but they do have a cost. This cost will be similar to allocations in manually managed languages; but unlike manually managed allocation one can not usually switch out the allocator. 

When a using a stop the world collector, pauses add an additional unpredictable latency the the application.

**Throughput:** There is a wide verity of implementations of mark and sweep collection but over all collectors which stop the world tend to have higher throughput at the cost of latency, and those that have current collectors achieve lower latency at the cost of throughput. 

**Safety:** Safety is good removing most of the errors that lead to unsafety in memory management.


### Compacting Garbage Collection

Compacting garbage collectors try to increase memory efficiency by moving allocations into contiguous memory ranges during garbage collection, reducing fragmentation. 

The cost of compaction is that memory must be copied and pointers must be updated to reference new memory locations. 

**Memory Efficiency:** Compacting can have positive impacts on efficiency by removing fragmentation.  

**Latency:** Compacting can reduce allocation latency but pauses due to garbage collection may get longer.

**Throughput:** The time spent moving memory during compaction negatively effects throughput.

**Safety:** Compaction dose not have a positive or negative effect on safety.

### Generational allocation

Generational allocation is a variation of a compacting garbage collector. They are based on the observation that most allocations have short life times. Using this idea it divides the allocations in to new and old types. All allocations start as new. The new allocator is optimized for latency and throughput at the cost of efficiency. When a collection cycle occurs, all in use new allocations are moved to the old generation. THe old generation allocator uses an strategy which makes more efficient use of memory at some cost to latency and throughput.

**Memory Efficiency:** This approach keeps much of the efficiency gains of compaction, and should have similar advantages.

**Latency:** Allocation latency in generational allocation is very low; but may be paused if a collection cycle is occurring.

**Throughput:** The time spent moving memory during compaction negatively effects throughput, but new generation allocation is very high thruput; resulting is good throughput for bursty allocation patters.

**Safety:** Compaction dose not have a positive or negative effect on safety.

### Static Lifetime Analysis

An approach that has gained reascent popularity is static life time analysis. This approach is most like manually memory management, but instead of the developer being responsible for deciding when an allocation is not needed and manually freeing memory, the compiler uses static analysis to automatically insert calls to free the memory. This approach was brought to public attention by the Rust programming language but also exists C++ as RAII, and is also implemented by linear types in langues like AST.

**Memory Efficiency:** This approach like manual memory management puts the developer in control allowing for very high efficiency.

**Latency:** Latency will match that of manual memory management and will be dependent on the underlying allocator.

**Throughput:** Like Latency throughput will match that of manual memory management and will be dependent on the underlying allocator.

**Safety:** The use of static analysis can completely remove the safety issues associated with manual memory management making this approach very safe.

# Collections Data Structures

An potentially surprising entry in the list of memory management approaches are data structures. The type of structures we are talking about are ones that hold many values such as list, maps, and arrays. High performance implementations of these data structures must carefully arrange stored records to minimize overhead and maximize performance.

A simple example of this is automatically resized arrays. These are vectors of values which support indexing and iteration. A common way to implement this structure to have a underlying fixed size array of liner memory. THe capacity of the array is dabbled anytime its capacity is exceeded. This doubling is preformed by allocating a new array and then copying the existing values in to the new array. The original backing array is then reclaimed.

While the data structure is built on top of a second allocator it also acts as one. 

Collections implementations must know is when it is safe to free any old backing memory. In the case where only small values are stored it may be posable to alway resolve reads by value, but if it is necessary to return references to records or use the collection in concurrently the collection must implement reference counting, life time analysis, etc must be used.

**Memory Efficiency:** The reason that collections implement custom allocation strategies is increase efficiency so they are usually quite efficient for their use case.

**Latency:** Latency will also often be very good if the appropriate data structure is picked for the workload.

**Throughput:** Likewise throughput will often be good if the data structure chosen matches the workload.

**Safety:** The safety of this approach based on the API provided which is often influenced by the underlying memory management approach. Overall there is not a single answer to the question of safety for collections and it must be answered for each collection independently.

## Conclusion

We have described many different approaches to memory management, each with different goals, costs, and benefits. When we detail the Colang design it will be a mixture of these approaches optimized for the Colang execution model.