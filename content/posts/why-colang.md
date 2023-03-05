---
title: "Why Colang"
date: 2023-02-26T10:19:31-08:00
draft: false
---

# Why Colang

There is a gap in the current popular progrmming languages. Most services are still written in high level languages from the 90's. These languages were designed based on lessons and assumptions about computing thirty yeas ago.

There is currently lots of innovation in programming language design, Rust, Swift, Kotlin, and Go have see lots of growth and are supported by large corporations. One thing all these languages have in common is that they were built largely as alternatives enterprises or systems languages: 

    - C++ -> Rust
    - ObjectiveC -> Swift
    - Java -> Kotlin
    - C -> Go
    
This is not to say each has no goals other than to be an improvement over there predecessors, or that in all cases the replacement is one to one; but overall the current innovation is in this domain of systems  programming languages.

Where this domain important, much of the code written today is to build services and written in high level languages such as JavaScript, Python, and PHP. The high level language category has seen some innovation with examples like TypeScript and Elm but these tend to focuses on correctness and developer ergonomics rather the runtime model.

For all the code that runs on servers and not written in a systems language, we have seen little innovation.

Code which runs on servers is important code; it costs organizations money to run, it effects latency experienced by clients, and avoided computation can lead to a reduction in carbon emissions. For all these reasons I think it is worth trying to improve the performance and efficiency of the code we write services in.

## Why are high level languages inefficient

Todays popular high level languages, PHP, Python, JavaScript, are all dynamically typed, garbage collected languages, which use heap allocations for most values. 

At the time they were designed most commercial code was written in C or Pascal. The features of these high level langues made them much easer to learn, safer, and development time. They were also all slow; but at the time, in late 80's early 90's, the web was starting to take off and the single threaded performance of computers was doubling every eighteen months. In this environment it was more effective to optimism for developer time rather than execution performance.

Today we have inherited the costs but are no longer getting a free ride from the work of CPU manufactures. Languages like JavaScript have made great strides in in performance at the cost of complex multi stage optimizing JIT compilers; but even after this work JavaScript is still significantly slower systems languages for most domains.

The source of the  inefficient execution comes down to two design choices. First the use of general purpose garbage collection, and second the use of dynamic typing and dispatch.

## Another Approach
In fallowing posts we will describe a language architecture we call Colang. Colang attempts to address both the inefficiencies of general purpose GC, as well as dynamicism.

 It's main goal is to provided a developer experience on par with languages like python, JavaScript, and PHP.

It is not meant to be a systems language, but strives give up little performance compared to a systems language.

In future posts we will explore in more detail the approach to execution and garbage collection, as well as consider the design of the type system.