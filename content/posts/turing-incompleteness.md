---
title: "Decidability"
date: 2023-09-10T11:53:42-07:00
draft: true
---

# Turing Incompleteness and Avoiding Undecidability

We often ask if a langue is Turing Complete. If it is, we know that the language can solve any
computable problem; but is completeness actually important? How much expressiveness is required to solve the problems that engineers regularly encounter and what is the price we pay when we get completeness?

The cost of completeness is the undecidability of the halting problem. That is a arbitrary function in a turning complete language can not be show to return.
 

------

We are taught about Truing Completeness as a core concept, and where it is clearly 
a important result how much value dose it have outside of theory? With Turing Completeness 
comes the undesirability for the Halting Problem. Is this good? Don't we consider 
it a bug if a function loops forever or recurses infinitely?

If a language is
Turing Complete the Halting Problem is undecidable. This is not true for all classes of
languages.