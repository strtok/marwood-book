# Heap

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