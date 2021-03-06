[sexpr]: https://en.wikipedia.org/wiki/S-expression

# S-Expressions

When designing a scheme parser, we first have to think a little bit about how to represent Scheme expressions in Rust's type system.

Fortunately Scheme's source code is already representable in a data structure called an S-expression (or [sexpr]). This is one of the fundamental properties of a lisp: source code is data.

A sexpr can be thought of as an aggregate type that can represent one of two things:

1. an atom, such as a booleans, numbers, symbols, strings, nil, etc.
2. a pair in the form (*car* . *cdr*), where *car *and *cdr* are themselves S-expressions

The expression `(+ 2 3)` in scheme is sugar for a tree of pairs, which could be rewritten  `(+ . (2 . (3 . ())))`.

This diagram depicts the above expression as a tree:

```mermaid
flowchart TD
    A(( )) -->|car| +((+))
    A -->|cdr| B(( ))
    B -->|car| 2((2))
    B -->|cdr| C(( ))
    C -->|car| 3((3))
    C -->|cdr| nil((nil))
```

## Improper Lists

The expression `(1 2 . 3)` is an example of an *improper list, a list not terminated with '(). It relies on special use of a dot preceding the very last element in a list. This list could also be written as `(1 . (2 . 3))`.

```mermaid
flowchart TD
    A(( )) -->|car| +((+))
    A -->|cdr| B(( ))
    B -->|car| 2((2))
    B -->|cdr| 3((3))
```

Improper lists are less common in Scheme, but there are a few parts of R7Rs, such as variable argument support, that rely on improper lists.

Examples of improper lists are:

```scheme
'(1 2 3 4 5 6 7 8 9 . 10)
'((10 10) (20 20) . (/ 30 30))
```

This example is invalid scheme because there are two expressions after the dot:

```scheme
'(1 2 . 3 4)
```