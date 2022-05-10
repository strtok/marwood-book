[heap]: https://github.com/strtok/marwood/blob/master/marwood/src/vm/heap.rs
[environment]: https://github.com/strtok/marwood/blob/master/marwood/src/vm/environment.rs
[stack]: https://github.com/strtok/marwood/blob/master/marwood/src/vm/stack.rs
[virtual machine]: https://github.com/strtok/marwood/blob/master/marwood/src/vm/mod.rs

# Virtual Machine

Marwood has a stack based virtual machine represented by the `Vm` object. Each Vm object represents a scheme runtime environment, each with a stack, heap and global environment.

```rust,noplayground
#[derive(Debug)]
pub struct Vm {
    /// The heap and global environment
    heap: Heap,
    globenv: GlobalEnvironment,

    /// The current program stack
    stack: Stack,

    /// Registers
    acc: VCell,
    ep: usize,
    ip: (usize, usize),
    bp: usize,

    /// System Interface (display, write, etc).
    sys: Box<dyn SystemInterface>,
}
```

The `Vm` object provides an eval interface for evaluating scheme expressions, and is the main interface for the console and web repl:

```rust,noplayground
fn eval(&mut self, cell: &Cell) -> Result<Cell, Error>
```

## Components

The `Vm` is composed of these high-level components, which are explored later in this chapter:

* the [stack]
* a garbage collected [heap]
* the global [environment]
* registers

## Registers

The Vm has a handful of registers that are used to maintain currently running program state:

| Register | Description         |
|----------|---------------------|
| %acc     | Accumulator         |
| %ip      | Instruction Pointer |
| %bp      | Frame Base Pointer  |
| %sp      | Stack Pointer       |
| %ep      | Environment Pointer |

The %acc (or accumulator) register contains the result of any Vm instruction that produces a result The %acc register is also used by procedures to store the result of applying a procedure. For example, the procedure `(+ 10 10)` would result in %acc containing the result `20`.

The %ip register contains a tuple that points to the currently running block of bytecode, and the next instruction to execute.

The %bp, %sp and %ep registers maintain state of Marwood's stack and call frame state.

## OpCodes

Marwood's instruction set is built specifically for executing scheme. It's not a general purpose virtual machine. 

The MOV instruction is used to move values between Marwood's registers, stack, global and lexical environments.

The CALL, CLOSURE, ENTER, RET, TCALL, and VARARG instructions support procedure application.

The JMP and JNT provide branching support for the compiler to support the scheme `if` primitive.

The HALT instruction stops the virtual machine, returning the contents of the %acc register as the result to `eval`.

| Opcode                       | Description                                                                                                           |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| CALL %acc                    | Given the arguments for the procedure in %acc have been pushed on the stack, jump to the procedure in %acc.           |
| CLOSURE %acc                 | Create a lexical environment as a result of evaluating a (lambda ...) expression.                                     |
| ENTER                        | Setup the currently executing procedure's stack frame                                                                 |
| HALT                         | Halt program, returning the result contained within ACC                                                               |
| JNT &lt;OFFSET&gt;           | Set %ip to OFFSET if %acc is #f                                                                                       |
| JMP &lt;OFFSET&gt;           | Set %ip to OFFSET                                                                                                     |
| MOV &lt;SRC&gt; &lt;DEST&gt; | Move the value from SRC into DEST                                                                                     |
| PUSH                         | Push the value in ACC on to the stack                                                                                 |
| RET                          | Return from a procedure entered via CALL                                                                              |
| TCALL %acc                   | Identical to a CALL instruction, except that a tail optimizing CALL is performed.                                     |
