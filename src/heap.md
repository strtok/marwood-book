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

The `put` function puts the supplied vcell on the next available slot on the heap returned by `alloc`. If vcell was already a pointer type, then it's returned instead of being double boxed by the heap. This removes the need for a lot of boxing code in Marwood to check if the value it's boxing is already boxed.

```rust,noplayground
    pub fn put<T: Into<VCell> + Clone>(&mut self, vcell: T) -> VCell {
        let vcell = vcell.into();
        match &vcell {
            VCell::Ptr(_) => vcell,
            VCell::Symbol(sym) => match self.symbol_table.get(sym.deref()) {
                Some(ptr) => VCell::ptr(*ptr),
                None => {
                    let ptr = self.alloc();
                    *self.heap.get_mut(ptr)
                        .expect("heap index is out of bounds") = vcell.clone();
                    self.symbol_table.insert(sym.deref().into(), ptr);
                    VCell::ptr(ptr)
                }
            },
            vcell => {
                let ptr = self.alloc();
                *self.heap.get_mut(ptr)
                    .expect("heap index is out of bounds") = vcell.clone();
                VCell::Ptr(ptr)
            }
        }
    }
```

> #### Symbol Interning
> The heap provides symbol interning by use of Heap::symbol_table. On put of a symbol, the heap will check if the symbol already exists in the symbol table and will return the stored heap location if the symbol already exists.
> This results in the same symbols always having the same heap location in Marwood.
> Why is this useful? This allows various parts of Marwood to refer to symbols by their heap reference instead of the string.

The `maybe_put` function acts as a wrapper around `put`, and will only put a VCell value if the value is one that must be boxed on a heap. Atom values such as numbers, bool, nil, etc are immutable values that may not always need to be boxed.

```rust,noplayground
    pub fn maybe_put<T: Into<VCell> + Clone>(&mut self, vcell: T) -> VCell
```

# Put & Get Cell

The `put_cell` and `maybe_put_cell` are versions of `put` and `maybe_put` that recursively place a `Cell` structure on the heap. Aggregate structures such as pairs and vectors may end up allocating multiple heap slots to represent the structure in the heap.

```rust,noplayground
    pub fn put_cell(&mut self, ast: &cell::Cell) -> VCell
    pub fn maybe_put_cell(&mut self, ast: &cell::Cell) -> VCell
```

The following call to put_cell ends up allocating 10 slots on the heap to create the list structure and storage for values:

```rust,noplayground
    let mut heap = Heap::new(8192);
    heap.put_cell(&parse!("(puppies 42.0 cats puppies #t)"));
```

The chart below contains the resulting heap allocations for the storage of this list in the heap. Note that because puppies is an interned symbol at position $00 in the heap, both pairs that reference the value puppy point to the same heap position of $00.

| Slot | Value                   |
|------|-------------------------|
| 0    | Symbol(puppies)         |
| 1    | Float(42.0)             |
| 2    | Symbol(cats)            |
| 3    | #t                      |
| 4    | ()                      |
| 5    | ($03 . $04)             |
| 6    | ($00 . $05)             |
| 7    | ($02 . $06)             |
| 8    | ($01 . $07)             |
| 9    | ($00 . $08)             |
