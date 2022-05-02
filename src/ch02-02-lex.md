[cell]: https://github.com/strtok/marwood/blob/master/marwood/src/cell.rs
[parser]: https://github.com/strtok/marwood/blob/master/marwood/src/parse.rs
[lexer]: https://github.com/strtok/marwood/blob/master/marwood/src/lex.rs
[sexpr]: https://en.wikipedia.org/wiki/S-expression


# Tokenizing

Marwood's [lexer] is in the form:

```rust,noplayground 
fn scan(text: &str) -> Result<Vec<Token>, Error>
```

Given scheme code as input, the scan function returns a vector of tokens according to the Scheme grammar Each token contains a span, which may be used to extract the token's text from the source text, and the token type (e.g. LeftParen).

```rust,noplayground
pub struct Token {
    /// (start, end) index of original span in the source &str
    pub span: (usize, usize),
    /// The type output by the scanner
    pub token_type: TokenType,
}

pub enum TokenType {
    Char,
    Dot,
    False,
    LeftParen,
    Number,
    NumberPrefix,
    RightParen,
    SingleQuote,
    String,
    Symbol,
    True,
    WhiteSpace,
    HashParen,
}
```

### Error::Incomplete

The special error `Error::Incomplete` is returned in certain circumstances where the input looks incomplete, and may be hint to the REPL interface that the user intends to enter a multi-line expression.

An example that may cause an `Error::Incomplete` error in Marwood's scanner is a mismatched double quote: `"hello world`.

### Scanner Example

Given this code:

```'(10 20 "puppies") ; heterogeneous```

The tokenizer will produce the following tokens:

```
[
    Token {
        span: (0, 1),
        token_type: SingleQuote,
    },
    Token {
        span: (1, 2),
        token_type: LeftParen,
    },
    Token {
        span: (2,4),
        token_type: Number,
    },
    Token {
        span: (5,7),
        token_type: Number,
    },
    Token {
        span: (8,17),
        token_type: String,
    },
    Token {
        span: (17,18),
        token_type: RightParen,
    }
]
```

