[cell]: https://github.com/strtok/marwood/blob/master/marwood/src/cell.rs
[parser]: https://github.com/strtok/marwood/blob/master/marwood/src/parse.rs
[lexer]: https://github.com/strtok/marwood/blob/master/marwood/src/lex.rs
[sexpr]: https://en.wikipedia.org/wiki/S-expression

# Parsing & Printing

Scheme, having an incredibly simple grammar, lends itself well to a hand-written lexer and recursive descent parser.

Marwood's parser is composed of three separate components:

* the `Cell` object, which represents a Scheme [sexpr] in Rust

* the `lexer` (or scanner or tokenizer), which is responsible for turning plain-text scheme into a list of tokens to be consumed by Marwood's `parser`.

* the `parser`, which is responsible for constructing an aggregate structure called `Cell` from the lexer output.

## Example

This example demonstrates the manual use of Marwood's lexer, parser, and Cell types. It also shows use of Marwood's printer. The alternative printing syntax `{:#}` provides additional escaping, and is the printer used for Scheme's `write` procedure.

```rust, noplayground
    let scheme_code = r#"
        (quote (#\x 20 puppies))
    "#;

    let tokens: Vec<Token> = lex::scan(scheme_code).unwrap();
    let cell = parse::parse(scheme_code, &mut tokens.iter().peekable()).unwrap();

    println!("{}", cell);
    println!("{:#}", cell);
```

outputs:

```scheme
'(x 20 puppies)
'(#\x 20 puppies)
```