---
title: "Colang Event Loop"
date: 2022-11-26T19:03:05-08:00
draft: true
---
# Colang Event Loop

The core design goal of the Colang event loop is to support low latency, efficient and safe memory management. 

The key challenge in memory management is knowing when some allocation is no longer in use and can be reclaimed. Once some memory is reclaimed it will either be returned to the operating system or recycled in the process to satisfy a subsequent dynamic memory allocation.

There are many approaches to memory management today. All approaches must ballance between:

1. Efficiency
2. Latency
3. Throughput
4. Safety

## Allocation and Reclamation
Memory management incurs latency because, both in kernel and in process, the finding, tracking, and reclaiming of memory regions involves non trivial data structures and algorithms.

For maximum resource efficiency allocation can be reclaimed as soon as they are no longer needed.
### Memory

### References

### Layout
A second performance challenge comes not directly from the cost of allocations but from lack of control over layout. Because casting to pointers and pointer arithmetic is disallowed it is more work for the a developer to control memory layout which can hurt efficiency, specifically locality is harder to achieve.  




![Memory Diagram](/memory-diagram.drawio.svg)


Each event on the event loop will be associated with a single 
