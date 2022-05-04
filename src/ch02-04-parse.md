[cell]: https://github.com/strtok/marwood/blob/master/marwood/src/cell.rs
[parser]: https://github.com/strtok/marwood/blob/master/marwood/src/parse.rs
[lexer]: https://github.com/strtok/marwood/blob/master/marwood/src/lex.rs
[sexpr]: https://en.wikipedia.org/wiki/S-expression

# Parsing

### The Parse Function

Marwood's parser is a direct consumer of `Vec<Token>` returned by the lexer.

`parse::parse()` takes an iterator over `&Token`, the original source &str that was supplied to `lex::scan`, and returns a `Cell`.

```rust,noplayground
pub fn parse<'a, T: Iterator<Item = &'a Token>>(
    text: &str,
    cur: &mut Peekable<T>,
) -> Result<Cell, Error>
```

`parse` will only consume one expression from `cur`. If `cur` may contain multiple expressions, the cursor position that `parse` left off at can be used to determine the cursor position of the next expression.

For example, this scheme source contains five expressions and would require five separate calls to `parse`.

```scheme
1 2 3
(+ 1 2)(+ 3 4)
```

### Recursive Descent Parser




### Error::Incomplete

Similarly to scan, parse() will return Error::Incomplete if the token stream does not contain a complete expression.

Some examples of incomplete expressions:

```scheme
'(1 2 3
```

```scheme
'(1 2 3)'(4 5
```

```scheme
#(1 2 3)
```