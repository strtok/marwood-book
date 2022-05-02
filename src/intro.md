# Introduction

Crafting Marwood is a book describing the process behind building Marwood, a Scheme R7 compiler and runtime written in Rust.

## What is Scheme?

Scheme is a programming language invented in 1975 by Guy Steel and Gerald Jay Sussman at MIT. It's the first dialect of lisp to use lexical scope, and its simplicity makes it a great choice for hobby programming language implementers.

## Why Scheme?

Scheme has very simple grammar, and small number of core language features that once implemented provide the building blocks for the rest of the implementation.

[R7Small](https://small.r7rs.org), the latest standard at the time of writing, was published in 2013 and is just under 90 pages of text including full library specification and appendix.

The simplicity of scheme allows the implementer to spend more time exploring complex branches of programming language implementations, such as garbage collection, macro expansion, optimizations, etc.

## How is Marwood different?

When researching scheme implementation material I came across two types of sources:

1. Academic material dating back to the 1970s. This material is quite abstract and a majority of these papers and books describe implementing a metacircular evaluator. There are very few materials on implementing scheme in a language like C or Rust.
   
2. Modern scheme codebases such as Chez Scheme, Guile, Chicken, Chibi. These implementations have the best performance, but are composed of millions of lines of code over years of implementation from dozens of developers. 

Marwood's goal was to provide an example of an implemenation of scheme that fits somewhere between #1 and #2.

## Resources

These resources were found useful when researching scheme implementations:

- [Scheme Bibliography Github](https://github.com/schemedoc/bibliography)
- [Three Implementation Models for Scheme - Kent Dybvig](https://dl.acm.org/doi/10.5555/37555)
- [An Incremental Approach to Compiler Construction](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf)
- [Crafting Interpreters](https://craftinginterpreters.com)
- [LISP System Implementation - Nils Holm](http://t3x.org/lsi/index.html)