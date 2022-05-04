# Crafting Marwood

*by Erik Bremen*

This book is built with [mdBook](https://github.com/rust-lang/mdBook) and [Mermaid-JS](https://mermaid-js.github.io). Its source may be found at [marwood-book](https://github.com/strtok/marwood-book).

# Introduction

Crafting Marwood is a book describing the process behind researching and building Marwood, a Scheme compiler and runtime written in Rust. Marwood may be interacted with on the web at [repl.marwood.io](https://repl.marwood.io), and its source is MIT Licensed and may be found on [github](https://github.com/strtok/marwood).

## What is Scheme?

Scheme is a programming language invented in 1975 by Guy Steel and Gerald Jay Sussman at MIT. It's the first dialect of lisp to use lexical scope, and its simplicity makes it a great choice for hobby programming language implementers.

Here's a sample of scheme's syntax, demonstrating a recursive implementation of factorial:

```scheme
(define (factorial n)
    (if (= n 0) 
        1
        (* n (factorial (- n 1)))))
```

## Why Scheme?

Scheme has very simple grammar, and small number of core language features that once implemented provide the building blocks for the rest of the implementation.

[R7Small](https://small.r7rs.org), the latest standard at the time of writing, was published in 2013 and is just under 90 pages of text including full library specification and appendix.

The simplicity of scheme allows the implementer to spend more time exploring complex branches of programming language implementations, such as garbage collection, macro expansion, optimizations, etc.

## How is Marwood different?

When researching scheme implementation material I came across two types of sources:

1. Academic material dating back to the 1970s. This material is quite abstract and a majority of these papers and books describe implementing a metacircular evaluator. There are very few materials on implementing scheme in a language like C or Rust.
   
2. Modern scheme codebases such as Chez Scheme, Guile, Chicken, Chibi. These implementations have the best performance, but are composed of millions of lines of code over years of implementation from dozens of developers. 

Marwood's goal was to provide an example of an implemenation of scheme that fits somewhere between #1 and #2.

## Who is this book for?

This book is walkthrough through Marwood's source code. It is aimed at an audience with some experience with the Rust programming language, parsers, and compilers.

For a detailed tutorial series on writing an interpreter, I would recommend [Crafting Interpreters](https://craftinginterpreters.com).

## Resources and Influences

These resources were found useful when researching scheme implementations:

- [Scheme Bibliography Github](https://github.com/schemedoc/bibliography)
- [Three Implementation Models for Scheme - Kent Dybvig](https://dl.acm.org/doi/10.5555/37555)
- [An Incremental Approach to Compiler Construction](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf)
- [Crafting Interpreters](https://craftinginterpreters.com)
- [LISP System Implementation - Nils Holm](http://t3x.org/lsi/index.html)