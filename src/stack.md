# Stack

Marwood's stack is represented by a `Vec<VCell>`, and a stack pointer `sp`. Most of the VM's interactions with the stack are `push`, `pop` and relative access such as `get_offset`. The stack is generally used for arguments during procedure application, and to store meta data to represent call frames. This is described further in the procedure application section.

```rust,noplayground
pub struct Stack {
    /// Stack contents
    stack: Vec<VCell>,

    /// Stack Pointer. SP points to the top value to be pushed onto the stack,
    /// This value backs the SP register of the VM
    sp: usize,
}
```

>#### Stack Growth
>The `push` operation is backed directly by Vec's push function, which will grow the underlying vector automatically. The stack does not currently have any method of shrinking the stack vector.