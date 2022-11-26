---
title: "Why Colang"
date: 2022-11-26T10:19:31-08:00
draft: true
---

# Why Colang

There is a gap in the current work being preformed in language design. Most services APIs are still written in high level languages from the 90's that were designed based on lessons and assumptions about computing thirty yeas ago.

Today we are seeing lots of innovation in programing language design, Rust, Swift, Kotlin, and Go have see lots of growth and are supported by large corporations; others such Zig have popular fallowing and are growing. One thing all these languages have in common is that they were built largely as alternatives enterprises or systems languages: 

    - C++ -> Rust
    - ObjectiveC -> Swift
    - Java -> Kotlin
    - C -> Go
    
This is not to say they had no other goals or that in all cases the replacement is one to one but overall the current innovation is around languages in this domain.

Where systems and enterprise languages are important much code is also written in high level languages such as JavaScript, Python, and PHP. This language category has seen some innovation such as TypeScript and Elm. Both of these languages work to provide correctness and offer improvements in developer esperance but both of these are  JavaScript replacements and mostly targeted at in browser applications.

For all the code that runs on servers that is not written in a systems language we have seen little innovation in year.

Code on servers is important code. This is the code that operators have to run so the cost of execution has real repercussions, the time it takes to run effects latency for clients, and avoidable computation can lead to a reduction in carbon emissions. For all these reasons I think it is worth trying to improve the performance and efficiency of the code we run on servers.

## Why are high level languages inefficient

Todays popular high level languages, PHP, Python, JavaScript, are all dynamically typed, garbage collected, languages, which use heap allocations for most types. At the time the were designed most code was still written in C or Pascal. This set of common features made them much easer to lear to use, made them safer, and quicker to develop in. It also made them slow, but at the time, late 80's early 90's, the web was starting to take off and the single threaded performance of computers was doubling every eighteen months. In this environment it was more effective to optimism for developer time rather than execution performance.

The downside of this approach today is that these languages are quite inefficient. I have measured python to dispatch both 30x more instructions and have 30x more cache misses then an equivalent C program. Languages like JavaScript with much of the performance back with complex multi stage optimizing compilers and JIT, but still end up meaningfully slower.

The source of the the inefficient execution comes down to two design choices. First the use of general purpose garbage collection, the second dynamicism.

## Another Approach
In fallowing posts we will describe a language architecture we call Colang which attempts to address both the inefficiencies of general purpose GC, as well as dynamicism.

The key design design are:

1. The language runs in an event loop.
2. There initial allocations use a bump allocator which frees in the main event loop.
3. All long lived allocations are owned by collections.
4. It will be high level but largely static language.

### Why Event Loops
The use of an event loop is driven both by two observations.

First the is that event loops has been successful in languages such as javascript and more generally event driven programs, and importantly are asynchronisms programming which is increasingly important to scale services.

Secondarily, and more importantly, when execution exits back to the event loop the run time is provided clear and convent time to preform garbage collection which will never stall active work.

### Why a bump allocator
Bump allocators have been shown to be very efficient new generation allocators in generational garbage collectors such as the JVM. They require almost no book keeping, and most new allocations are ephemeral. In addition the design calls for GC to be delayed until exit back to the event loop. This removes the possibility of use after free even when returning down the stack.

### Why collections for long lived allocations
Many programs benefit from persistent state, so to be an approachable language we must provide persistance of state between returns to the event loop. We also observe that efficient collections, vector, list, map, etc, each implements it's own collection strategy. This is true even in garbage collected languages. We further assert, with out evidence, that the majority of long lived allocations in most programs are held in a collection, and there for it is natural to manage their allocations using the memory management of implemented by collection implementations.

### Why Static
We hypotheses that slow performance in high level languages is due not to being highly expressive but due to being dynamic.

Dynamic dispatch require runtime checks and prevent the complier from preforming many optimizations. Advanced JITs like the JavaScript runtime found in browsers try to avoid some fo this penalty by assuming that types don't change, but must still include checks for when they are wrong which when triggered force recompilation.

Dynamic types require checks on every use which impacts reliability as this style of programing resists static analysis and results in runtime type errors. Like dynamic dispatch dynamic types also prevent the use of common compile time optimizations.

## Conclusion

Colang is a language architecture, and hopefully eventually a language, which attempts to be a high level language for weighting services in. Its main goal is to provided a developer experience on par with languages like python, JavaScript, and PHP while giving up little performance compared to systems language.

It is not meant to be a systems language and is not meant to be all things to all people.

In future posts we will explore in more detail the approach to execution and garbage collection, as well as consider the design of the type system.