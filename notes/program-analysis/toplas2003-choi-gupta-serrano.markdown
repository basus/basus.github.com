---
layout: notes
title: Stack allocation and synchronization optimizations for Java using escape analysis
author: Choi, Jong-Deok and Gupta, Manish and Serrano, Mauricio J. and Sreedhar, Vugranam C. and Midkiff, Samuel P.
publication: ACM Trans. Program. Lang. Syst.
---
# Abstract
## Escape analysis framework for Java to determine if an object
   1. is not reachable after the end of its creation method (allows stack allocation)
   2. is only reachable from a single thread (allows removing sync operations)
## Connection graph establishes reachability relationship between objects and references
   - can be summarized for each method so that the summary info can be used in different contexts without imprecision
   - interprocedural algorithm that efficiently computes the graph and identifies non-escaping objects
## Benefits
   - 10 benchmark programs
   - stack allocate max 70% median 19% dynamic objects
   - 11% to 92% mutex locks eliminated (median 51%)
   - execution time reduction from 2% to 23% (median 7%)
# Introduction
## VM implementation problems from performance point-of-view:
   - Object allocation on heap and deallocation only by garbage collection
   - Each object has a lock to ensure mutual exclusion when sync method is invoked on it
## Escape analysis determines the set of objects that escape a method invocation. Not local if they escape. Allows 2 optimizations
### Local objects can be allocated on the stack frame.
    - Stack allocation reduces GC overhead as stack storage is reclaimed on method return
    - Reduces synchronization done by heap allocator with other threads requiring memory
    - Aggressive code reordering in spite of precise exception semantics
    - Object access can be strength reduced -- object creation may be eliminated
### If an object is thread-local no other thread can access it
    - Eliminates low-level sync operations that ensure mutual exclusion
    - Thread local objects can be allocated to improve data locality
    - Some operations to flush the local memory can be eliminated
    - Aggressive code reordering optimizations (similar to above)
## Framework for escape analysis based on a connection graph
   - Connection graph captures connectivity relationship between heap-allocated objects and references
   - Perform reachability analysis on the graph to determine if an object is local to method/thread
   - Can be used in static/dynamic Java compiler, application extractor, bytecode optimizer
## Paper contributions
   1. New, simple interprocedural framework with flow sensitive and insensitive variations for escape analysis
   2. Important application of escape analysis: eliminating unnecessary lock operations to thread local objects
   3. Describe handling exceptions in the context of escape analysis with being too conservative
   4. Introduce the connection graph. Different from points-to graph. Uses phantom nodes which allow summarization of 
      effects of a callee procedure independent of calling context.
   5. Present experimental results from an implementation of escape analysis in a Java compiler

# Connection graph representation for escape analysis
## The connection graph
### Nodes of a connection graph are
    1. Objects
    2. References to objects
### Edges of connection graph are
    1. Reference to objects (points-to)
    2. Reference to reference, both pointing to same object (deferred)
    3. Object to reference, from object to field of that object (field)
### Each object is represented as a tree with the object as root and reference fields as children
### PointsTo(p) denotes object nodes immediately pointed to by p. Computed by traversing all outgoing paths, following deferred edges till an object is reached.
### ByPass(p) redirects incoming deferred edges to p to successor nodes and removes p's outgoing edges. 
    Improves efficiency by delaying and reducing graph updates. Reduces precision by allowing edges from mutually exclusive control flow paths if updates are delayed.

## Escape property of objects
### Object O escapes method M if the lifetime of O may exceed that of M
### Object O escapes thread T if O is visible to another thread T'
### M is a method in thread T that creates another thread T'. Set Escapes(O, M) true for all objects O(incl T') reachable from T' 
    since T' may continue executing after M returns. Ensures that a concrete object that is bounded by the lifetime of a method is 
    only accessed by a single thread.
### For static escape analysis, concrete object graph modeled by a call graph
    - For each CG node, an allocation site in the program can be identified where the corresponding object is created
    - Each CG node has an escape state associated with it.
    - EscapeSet = {NoEscape > ArgEscape > GlobalEscape} ArgEscape escapes method, but not thread
### Static fiels and thread objects are GlobalEscape. Objects reachable from a thread are GlobalEscape, objects created in it may not be
### Reachability analysis performed on CG to compute escape state
### Concrete objects allocated at a call site with escape state NoEscape are stack-allocatable in the creation method.
### All concrete objects at an allocation site marked NoEscape or ArgEscape are thread local.
# Intraprocedural analysis
  Assume all multi-level reference expressions are split into a sequence of two-level reference expressions
## Nontrivial transfer functions for new object creation and allocation (see text)
## Merge between two connection graphs is the union of the two graphs. If there are common nodes, the escape state of the node in the merged graph is the meet of the common nodes' escape state
## Loops are handled by iterating over the data flow solution until it converges. Upper limit of 10 iterations.
## Single CG for each method, updated incrementally.
### Flow-insensitive: updates made in place
### Flow-sensitive: kill local variables and add node at update of local variable in 1-limited manner. Merge local nodes at control flow join point to new node if needed. 
# Interprocedural analysis
