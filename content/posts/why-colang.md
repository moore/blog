---
title: "Why Colang"
date: 2023-02-26T10:19:31-08:00
draft: false
---

# Why Colang

There is a gap in the current popular languages being used. Most services APIs are still written in high level languages from the 90's. These were designed based on lessons and assumptions about computing thirty yeas ago.

We are seeing lots of innovation in programing language design, Rust, Swift, Kotlin, and Go have see lots of growth and are supported by large corporations; others such Zig and Nim have popular fallowing and are growing. One thing all these languages have in common is that they were built largely as alternatives enterprises or systems languages: 

    - C++ -> Rust
    - ObjectiveC -> Swift
    - Java -> Kotlin
    - C -> Go
    
This is not to say each had no other goals, or that in all cases the replacement is one to one; but overall the current innovation is around languages in this domain.

Where systems and enterprise languages are important, much of the code written today is to build service APIs and written in high level languages such as JavaScript, Python, and PHP. The high level language category has seen some innovation with examples like TypeScript and Elm. Both TypeScript and Elm work to provide correctness and offer improvements in developer esperance, but both in practice are largely JavaScript replacements targeted at in browser applications.

For all the code that runs on servers and not written in a systems language, we have seen little innovation.

Code which runs on servers is important code. It costs organizations money to run, the time it takes to run effects latency for clients, and avoided computation can lead to a reduction in carbon emissions. For all these reasons I think it is worth trying to improve the performance and efficiency of the code we run on servers.

## Why are high level languages inefficient

Todays popular high level languages, PHP, Python, JavaScript, are all dynamically typed, garbage collected languages, which use heap allocations for most types. 

At the time they were designed most commercial code was written in C or Pascal, and their features made them much easer to learn, safer, and offered quicker to development. They were also all slow; but at the time, in late 80's early 90's, the web was starting to take off and the single threaded performance of computers was doubling every eighteen months. In this environment it was more effective to optimism for developer time rather than execution performance.

Today we have to live with there inefficiencies but are no longer getting a free ride from the work of CPU manufactures. Languages like JavaScript have made great strides in in performance but at the cost of complex multi stage optimizing JIT compilers; but even after this work JavaScript is still significantly slower that low level languages.

The source of the the inefficient execution comes down to two design choices. First the use of general purpose garbage collection, and second the use of dynamic typing a dispatch.

## Another Approach
In fallowing posts we will describe a language architecture we call Colang. Colang attempts to address both the inefficiencies of general purpose GC, as well as dynamicism.

The key design choices are:

1. The language runs in an event loop.
2. Initial allocations use a bump allocator.
3. Garbage collection only happens after exiting to the event loop.
3. All long lived allocations are owned by collections.
4. It will be high level but largely static language.

### Why Event Loops
The use of an event loop is driven by a few observations.

Event loops have been successful in languages such as javascript, and more generally event driven systems. 

Event loops compatible with async programming which is important for high request concurrency.

Lastly, and more importantly, when execution exits back to the event loop, the run time is provided clear and convent time to preform garbage collection which will never stall active work.

### Why a bump allocator
Bump allocators have been shown to be very efficient new generation allocators in generational garbage collectors such as the JVM. They require almost no book keeping, and most new allocations are ephemeral meaning in practice most memory never has to be moved.

### Why collections for long lived allocations
We observe that efficient collections (vector, list, map, etc) each implements their own memory management strategy. This is true even in garbage collected languages. We further assert, with out evidence, that the majority of long lived allocations are held in a collection. It is there for natural to manage log lived allocations using the memory management implemented by collection which owns it.

### Why Static
We hypotheses that slow performance in high level languages is due not to being highly expressive but due to being dynamic.

Dynamic dispatch require runtime checks and prevent the complier from preforming many optimizations. Advanced JITs, such as JavaScript runtimes found in browsers, try to avoid the penalty dynamicism by assuming that types don't change; but JavaScript engines must still include checks for when type guesses are wrong, and the checks fail they must recompile.

Dynamic types require checks on every use which impacts reliability as it results in runtime type errors, and like dynamic dispatch, dynamic types also prevent the use of common complier optimizations.

## Conclusion

Colang is a language architecture, and hopefully one day a language, which attempts to be a high level language for weighting services in. It's main goal is to provided a developer experience on par with languages like python, JavaScript, and PHP.

It is not meant to be a systems language, but strives give up little performance compared to a systems language.

In future posts we will explore in more detail the approach to execution and garbage collection, as well as consider the design of the type system.