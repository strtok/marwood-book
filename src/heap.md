# Heap

This small snippset of scheme is useful for reasoning about why scheme might need a heap:

```scheme
(define x 10)
(define y (* x x ))
(define animals '(cats otters puppies))
(define sub-animals (cdr animals))
```

Both the value for x and y are bound to numbers. Mutating `x` and `y` only require changing what value the symbol is bound to in the environment.

But, animals and sub-animals are a little more nuanced. animals is a list composed of `pairs` and `symbols`, and more importantly sub-animals refers to tail of the same list containing the values `'(otters puppies)`. That is, both `animals` and `sub-animals` refer to parts of the same data structure. Not a copy.

If sub-animals were modified with `set-car!` or `set-cdr!`, the mutation must also be realized on the list that `animals` is bound to.

One way of supporting this type of funcitonality is to store these shared values on a heap. `animals` and `sub-animals` can then contain references into the same data structure on a heap.

# Heap Implementation

Like the stack, Marwood's heap is composed of a `Vec<VCell>`. The initial heap size is `chunk_size`, and any growth of the heap will be of additional `chunk_size` blocks of VCells.

```rust,noplayground
#[derive(Debug)]
pub struct Heap {
    chunk_size: usize,
    free_list: Vec<usize>,
    heap: Vec<VCell>,
    heap_map: gc::Map,
    symbol_table: HashMap<String, usize>,
}
```