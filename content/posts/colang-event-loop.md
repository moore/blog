---
title: "Colang Event Loop"
date: 2022-11-26T19:03:05-08:00
draft: false
---
# Colang Event Loop

The core design goal of the Colang event loop is to support low latency, efficient and safe memory management. 

## Why Event Loops
The use of an event loop is driven by a few observations.

Event loops have been successful in languages such as javascript, and more generally event driven systems. This offers a proof point the accessability of the approach to the development community as well of there ability to be deployed in our target domain of backend services.

Event loops are compatible with asynchronous programming models which are needed to support high request concurrency.

Lastly, when execution returns back to the event loop, the run time is provided clear and convent time to preform garbage collection which will never stall an active request.

## Allocation Strategy 

Our allocation strategy will leverage the properties of the event loop.


### Why generational allocator.
Generational allocators have been shown to support very low latency allocations, and support effechent managemnt 

### Why collections for long lived allocations
We observe that efficient collections (vector, list, map, etc) each implements their own memory management strategy. This is true even in garbage collected languages. We further assert, with out evidence, that the majority of long lived allocations are held in a collection. It is there for natural to manage log lived allocations using the memory management implemented by collection which owns it.

### Why Static
We hypotheses that slow performance in high level languages is due not to being highly expressive but due to being dynamic.

Dynamic dispatch require runtime checks and prevent the complier from preforming many optimizations. Advanced JITs, such as JavaScript runtimes found in browsers, try to avoid the penalty dynamicism by assuming that types don't change; but JavaScript engines must still include checks for when type guesses are wrong, and the checks fail they must recompile.

Dynamic types require checks on every use which impacts reliability as it results in runtime type errors, and like dynamic dispatch, dynamic types also prevent the use of common complier optimizations.

## Allocation and Reclamation
Memory management incurs latency because, both in kernel and in process, the finding, tracking, and reclaiming of memory regions involves non trivial data structures and algorithms.

For maximum resource efficiency allocation can be reclaimed as soon as they are no longer needed.
### Memory

### References

### Layout
A second performance challenge comes not directly from the cost of allocations but from lack of control over layout. Because casting to pointers and pointer arithmetic is disallowed it is more work for the a developer to control memory layout which can hurt efficiency, specifically locality is harder to achieve.  




![Memory Diagram](/memory-diagram.drawio.svg)


Each event on the event loop will be associated with a single 
