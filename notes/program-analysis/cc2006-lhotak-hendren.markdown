---
layout: notes
title: "Context-sensitive points-to analysis: Is it worth it?"
author: Lhotak, Ondrej and Hendren, Laurie
publication: Compiler Construction, 15th International Conference, volume 3923 of LNCS
---
# Abstract
## Empirical study evaluating the precision of subset-based point-to-analysis with context sensitivity variations
## Compare use of call site strings as context abstraction, object sensitivity and BDD-based context sensitive algorithm
## Measure characteristics of points-to sets as well as effects on precision of client analyses
## Includes analyses that specialize pointers and those that specialize heap abstraction
## Object sensitive analyses scale better and more predictably than other approaches and are precise
## Specialization of hep abstraction improves precision more than extending context string length
## Cycles in java call graphs severely reduce precision of analyses that forsake context sensitivity in cyclic regions
# Introduction
## Suggested that context sensitivity significantly improves precision of interprocedural analysis of OOP
## Answers the following questions:
### Which context-sensitive points-to analysis improves precision the most?
### Which are effective for particular client analysis or specific code patterns?
### Which variations have scalable implementations?
## Implemented 3 different families of context-sensitive points-t analysis:
   1. Use of call sites as context abstraction
   2. Object sensitivity
   3. ZCWL algorithm
## Evaluate effect of different lengths of context strings and specializing the heap abstraction
## Determine how many contexts each variation of context sensitivity actually generates and how it relates to the precision
## Effect on precision depends on client analysis. Benefits of context sensitivity are significant for some
## Object sensitivity consistently improves precision most and modeling heap objects with context also improves precision
# Background
## Context sensitive points-to analysis requires abstraction over pointer targets, pointers and method invocations.
### Pointer target is always a dynamically allocated object. Program statement at which object was allocated is used as abstraction
### Pointers to local variables are abstracted by the local variable. Pointeres to fields are the field sensitive abstractio [O(o), f]
### A context is a static abstraction of a method invocation, Abstraction is by call site or receiver object.
### Context abstraction can be made finer by using a string of contexts
### Each context string can be limited to a fixed height, or bounded by the number of acyclic paths in the call graph (ZCWL)
## Objects are modeled context-sensitively by the allocation sites abstraction
## Shows information from context-insensitive, object-sensitive and call-site sensitive abstractions
## Call graph is constructed on-the-fly during the points-to analysis, except for ZCWL.
