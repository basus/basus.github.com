---
layout: notes
title: Using Static Analysis for IDE's for Dynamic Languages
author: Dolby, Julian
publication: Proceedings of the Eclipse Languages Symposium
---

# Abstract

+  Modern IDEs such exploit the static type system for languages like Java to provide shortcuts and hints to the programmer
+  In dynamic languages there is no static type system to exploit in a similar way
+  Source analysis of the program might provide similar information
+  Recommends a layered approach that uses the same analysis infrastructure for multiple languages

# Introduction
## Many modern IDE shortcuts (like autocompletion) are context sensitive.

1. Left hand side type information for . or -> can be used to derive possible right hand completions.
2. Examining method and class and inheritance in each class can provide information about when a method is being overriden.
3. List of possible called functions can be derived using type of receiver object.

## In dynamic language there is little or no static type information. Using JavaScript as an example:

1. It is dynamically typed with a prototype object system that allows dynamic inheritance.
2. Properties could be set and deleted anywhere.
3. Requires precise alias analysis to determine assigned properties for '.'
4. Deterimining overriding would also need alias analysis to determine inheritance for any object.

## One solution is to use alias analysis

1. Depends on the langauges and what assumptions are made about program completeness
2. But using inter-procedural alias analysis does not need any more assumptions than current type-based systems.
3. Implementing such static analyses is a mjaor undertaking

## Observation is that most popular scripting languages share a common core and approach

1. Managed runtimes
2. Common structures like loops
3. Object model, scoping and first class functions are not universal but share some basics

## Build an analysis framework with a shared core that understands the basic features with layers to adapt particular languages.
## Using an AST-base intermediate layer feeding into the DOMO infrastructure

1. Using Rhino parser and internal IBM parser for JavaScript
2. Substantial portion of Java
3. Planned work includes PHP

# Uses of program analysis
## Type approximation

Analysis for dynamic languages should substitute for static type information used by static language IDEs. Two ways to use this:

1. Approximate types of variables, substituting for type declarations in static languages.
2. Approximate concept of type for languages like JavaScript which have no notion of type

### Approximating Type declarations

1. Pointer analysis that distinguishes heap object on a per class basis
2. Concrete type is one that may actually be allocated by the program
3. One abstract object for all heap objects of one concrete type
4. Analysis for a variable will be the set of concrete types that the variable could hold at runtime

### Approximating types themselves

1. Languages like JavaScript have no declarative notion of types.
2. Synthesize a notion of type by statically differentiating runtime objects that have similar behavior.
3. In JavaScript, this means associating types with sets of possible objects that have same properties (especially functions).
4. Extend this to sub-typing where subtypes have all functional properties of supertypes and then some

## Approximating call targets
1. Java IDEs make use of two language features
 + Lack of first class functions
 + Declared type information
2. Set of possible callee methods are the methods with appropriate names and signatures in the receiver type and subtypes
3. Dynamic language have first class functions and lack static type declarations
4. Alias analysis can approximate needed information
5. Functions are objects and can be tracked by pointer analysis
6. Number of function allocation sites is typically limited
7. Aggressive pointer analysis can distinguish the functions created and track their flow to call sites

# Analysis infrastructure

+ Core program analysis infrastructure understanding objects, method calls, control flow etc. 
+ Adaptation for each language for two purposes
 1. Generate analysis internal forms from the source code
 2. Implement semantic quirks of the language in terms of analysis internal forms

## Analysis Adaptation Layer

+ Stylized AST for describing that statements supported by analysis IR
+ IR generation from AST has two steps:
 1. Take data structure from the front end (such as parse tree) and generate AST
 2. Convert AST to IR using common core for things like AST-to-CFG-to-SSA and language dependent module for details (like field access semantics)
 
## Core analysis engine
SSA-based IR with algorithms for call graph construction, pointer analysis etc. IR structure is extensible and extensions can be integrated with the algorithms.

### IR

+ Represents values in SSA form with instructions for general operations. 
+ Language dependent operations (field access) broken into an abstract class with subclasses for each language

### Analysis components

+ Core analysis components like call graph construction and pointer analysis work with extensible IR
+ Component is implemented using factory-constructed visitor across IR of the method and provide visitor implementation covering the core
+ Derived visitor also handles new IR forms for the language
