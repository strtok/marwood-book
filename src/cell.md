[cell]: https://github.com/strtok/marwood/blob/master/marwood/src/cell.rs
[parser]: https://github.com/strtok/marwood/blob/master/marwood/src/parse.rs
[lexer]: https://github.com/strtok/marwood/blob/master/marwood/src/lex.rs
[sexpr]: https://en.wikipedia.org/wiki/S-expression

# Cell

If Scheme can be represented by an aggregate type called a sexpr, then we just need to create a type in Rust that represents an S-expression: a type that can be either an atom, or a pair. Fortunately Rust's enum type allows exactly this: the creation of a type that represents one of several possible variants.

Marwood's [Cell] is the enum type that represents Scheme data and code in Marwood. It is the main input to Marwood's compiler, macro transformers, and printer, and is the output of Marwood's VM as a result of evaluating scheme.

Most Cell's variants represent *atom* types in Scheme, such as booleans, numbers, symbols, characters, and strings:

```rust,noplayground
pub enum Cell {
    Bool(bool),
    Char(char),
    Number(Number),
    Nil,
    String(String),
    Symbol(String),
    ...
}
```

As an example, parsing the scheme expression `#f` would result in Marwood's parser outputting the value `Cell::Bool(false)`.

## Pairs

Cell also contains an aggregate type called *Pair*, which represents a sexpr pair:

```rust,noplayground
pub enum Cell {
    ...
    Pair(Box<Cell>, Box<Cell>),
    ...
}
```

`Pair` is a pair of Cells representing the `car` and `cdr` parts of a cons pair. This structure can be used to represent pairs in scheme, which in turn may be used to represent lists. This is the main building block for representing Scheme code's tree structure in Rust.

## Vectors

`Vector` represents a literal scheme vector, such as `#(10 20 30)`.

```rust,noplayground
pub enum Cell {
    ...
    Vector(Vec<Cell>),
    ...
}
```

## Special Variants

The Cell enum also contains a few variants that can only be produced as output of Marwood's evaluator. For example, the `Cell::Void` variant is produced by some Marwood procedures that evaluate to `#<void>` (e.g. define, set!). It is not possible to produce these variants via Marwood's parser.

```rust,noplayground
#[derive(Debug, Eq, PartialEq, Hash, Clone)]
pub enum Cell {
    ...
    Continuation,
    Macro,
    Procedure(Option<String>),
    Undefined,
    Void,
}

```