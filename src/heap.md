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

# Heap Structure

Like the stack, Marwood's heap is composed of a `Vec<VCell>`, along with various supporting structures used to track free slots, intern symbols, and garbage collection.

```rust,noplayground
pub struct Heap {
    chunk_size: usize,
    free_list: Vec<usize>,
    heap: Vec<VCell>,
    heap_map: gc::Map,
    symbol_table: HashMap<String, usize>,
}
```

`chunk_size` refers to the number of heap slots allocated when the heap needs to grow. On initialization, the Heap will allocate chunk_size slots initialized to VCell::Undefined, and add the index of each unused heap slot to `free_list`. This free list contains indexes on the heap of unused slots ready to be allocated.

```rust,noplayground
    pub fn new(chunk_size: usize) -> Heap {
        Heap {
            chunk_size,
            heap: vec![VCell::undefined(); chunk_size],
            free_list: (0..chunk_size).rev().into_iter().collect()
        }
    }
```

>#### Heap References
>References into Marwood's heap are of the type `type HeapRef = usize;` and are direct indexes into the heap. These heap references can be found in various places in Marwood's VCell and VM structures:
>* VCell::Ptr(HeapRef)
>* VCell::Pair(HeapRef, HeapRef)
>* VCell::Closure(HeapRef, HeapRef)

# Alloc & Free

The heap's core API is composed of alloc, free and get operations on the heap.

```rust,noplayground
    pub fn grow(&mut self)
    pub fn alloc(&mut self) -> usize
    pub fn free(&mut self, ptr: usize)

    pub fn get_at_index(&self, ptr: usize) -> &VCell
    pub fn get_at_index_mut(&mut self, ptr: usize) -> &mut VCell
    pub fn get<'a, T: Into<Cow<'a, VCell>>>(&self, vcell: T) -> VCell
```

`alloc` returns the next available slot on the `free_list`, calling `grow` to grow the heap by `chunk_size` elements if the `free_list` is empty. The disposition of the slot is also marked as State::Allocated in the heap_map, which is used by Marwood's garbage collector and discussed in more detail in later sections.

```rust,noplayground
    pub fn alloc(&mut self) -> usize {
        match self.free_list.pop() {
            None => {
                self.grow();
                self.alloc()
            }
            Some(ptr) => {
                self.heap_map.set(ptr, State::Allocated);
                ptr
            }
        }
    }
```

`free` returns a heap slot back to the `free_list`. The contents is set back to `VCell::Undefined`, which may help catch any bugs in Marwood where a heap reference is used when it no longer should be. 

The slot is also marked State::Free in the heap_map, a data structure used by Marwood's garbage collector.

```rust,noplayground
    pub fn free(&mut self, ptr: usize) {
        self.heap_map.set(ptr, State::Free);
        if let Some(VCell::Symbol(sym)) = self.heap.get(ptr) {
            self.symbol_table.remove(&**sym);
        }
        *self.heap.get_mut(ptr).unwrap() = VCell::Undefined;
        self.free_list.push(ptr);
    }
```

The core get operations on the heap are not notable, except that the get functions will panic if given an index that is not valid for the current heap. 

The function `get` acts as a wrapper around `get_at_index` and only performs a heap lookup if the supplied vcell is a pointer type. This allows code in Marwood to access values on the stack or environment slots without having to check of the value is a reference first.

```rust,noplayground
    pub fn get_at_index(&self, ptr: usize) -> &VCell {
        self.heap.get(ptr).expect("heap index out of bounds")
    }

    pub fn get_at_index_mut(&mut self, ptr: usize) -> &mut VCell {
        self.heap.get_mut(ptr).expect("heap index out of bounds")
    }

    pub fn get<'a, T: Into<Cow<'a, VCell>>>(&self, vcell: T) -> VCell {
        match vcell.into() {
            Cow::Borrowed(vcell) => match vcell {
                VCell::Ptr(ptr) => self.get_at_index(*ptr).clone(),
                vcell => vcell.clone(),
            },
            Cow::Owned(vcell) => match vcell {
                VCell::Ptr(ptr) => self.get_at_index(ptr).clone(),
                vcell => vcell,
            },
        }
    }
```

# Put



# Symbol Interning