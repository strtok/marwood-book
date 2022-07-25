# Stack

Marwood's stack is represented by an upward growing stack of VCell values, and a stack pointer `%sp` that represents the current top of the stack.

The stack's main purpose is to support the application of procedures, which include the pushing of arguments and meta data required for call frame setup.

Here's an example stack during evaluation of the expression `(+ 10 5)`, with the stack pointer pointing at slot 7, and the arguments to `+` having been pushed in slots 5 and 6.

| Slot     | Value                   | SP |
|----------|-------------------------|----|
| **7**    | **%argc[2]**            | ←  |
| **6**    | **Fixnum(5)**           |    |
| **5**    | **Fixnum(10)**          |    |
| 4        | %bp[$00]                |    |
| 3        | %ip[$223][$06]          |    |
| 2        | %ep[$ffffffffffffffff]  |    |
| 1        | %argc[0]                |    |
| 0        | Undefined               |    |


# Stack Structure

Marwood's stack is backed by a Rust `Vec<VCell>`, and the stack pointer `sp` which contains the index of the current top of the stack in the stack vector. Operations such as push and pop are backed by the rust Vector's implementation of push and pop.

```rust,noplayground
pub struct Stack {
    /// Stack contents
    stack: Vec<VCell>,

    /// Stack Pointer. SP points to the top value to be pushed onto the stack,
    /// This value backs the SP register of the VM
    sp: usize,
}
```

Stack manipulation is provided by push and pop functions:

```rust,noplayground
    pub fn push<T: Into<VCell> + Display>(&mut self, vcell: T)
    pub fn pop(&mut self) -> Result<&VCell, Error>

```

Access to values values on the stack are provided by either direct index into the stack, or offset relative to the current stack pointer:

```rust, noplayground

    pub fn get(&self, index: usize) -> Result<&VCell, Error>
    pub fn get_mut(&mut self, index: usize) -> Result<&mut VCell, Error>
    pub fn get_offset(&self, offset: i64) -> Result<&VCell, Error> {
    pub fn get_offset_mut(&mut self, offset: i64) -> Result<&mut VCell, Error>
```

>#### Stack Growth
>The `push` operation is backed directly by Vec's push function, which will grow the underlying vector automatically. The stack does not currently have any method of shrinking the stack vector.


# Stack API Example

```rust,noplayground
    let mut stack = Stack::new();

    // Push
    stack.push(VCell::number(1));
    stack.push(VCell::number(2));

    // Access relative to %sp
    assert_eq!(stack.get_offset(0), Ok(&VCell::number(2)));
    assert_eq!(stack.get_offset(-1), Ok(&VCell::number(1)));
    assert_eq!(stack.get_offset(-2), Ok(&VCell::Undefined));

    // Pop
    assert_eq!(stack.pop(), Ok(&VCell::number(2)));
    assert_eq!(stack.pop(), Ok(&VCell::number(1)));
    assert_eq!(stack.pop(), Err(InvalidStackIndex(0)));
```