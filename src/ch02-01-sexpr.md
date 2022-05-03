[sexpr]: https://en.wikipedia.org/wiki/S-expression

# S-Expressions

When designing a scheme parser, we first have to think a little bit about how to represent Scheme expressions in Rust's type system.

Fortunately, Scheme's predecessor Lisp set a precedent by designing the language so that data and code are interchangible and can be represented by a data structure called an S-expression or [sexpr].

A sexpr can be thought of as one of two things:

1. an atom, such as a booleans, numbers, and nil.
2. a pair in the form (*car* . *cdr*), where *car *and *cdr* are themselves S-expressions

The list `(1 2 3)` in scheme is sugar for a tree of pairs, which could be rewritten  `(1 . (2 . (3 . ())))`.

This diagram depicts the structure for this sexpr in a tree:

```mermaid
flowchart TD
    A(( )) --> 1((1))
    A --> B(( ))
    B --> 2((2))
    B --> C(( ))
    C --> 3((3))
    C --> nil((nil))
```
