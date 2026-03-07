# Hash Hash Hash
A serial hash table implementation and two concurrent variants, made thread-safe by the use of mutexes.

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t <num_threads> -s <hash_table_entries_per_thread>
```

For instance, you can run
```shell
./hash-table-tester -t 8 -s 50000
```
Both arguments are optional. By default, num_threads is 4 and hash_table_entries_per_thread is 25000.

## First Implementation
In the `hash_table_v1_add_entry` function, I added a single mutex that guards both the "read" (obtaining the head of the linked list that will contain the key of interest, and traversing this list to determine whether the key already exists), as well as the "store" (updating the corresponding node, or else creating a new one if none exists). This prevents two threads from reading the same data before either finishes their store. 

### Performance
```shell
> ./hash-table-tester 
Generation: 25645 usec
Hash table base: 44068 usec
  - 0 missing
Hash table v1: 140642 usec
  - 0 missing
```
Version 1 is slower than the base version, as it introduces overhead related to thread management but doesn't actually leverage these threads to gain any efficiency. In particular, the use of a single mutex serializes the add_entry function, so only one thread at a time can modify the hash table. Not only do we lose the benefits of parallelism, we've also added (for no reason!) the overhead of creating and joining threads, context switches, and locking/unlocking mutexes. 

## Second Implementation
In the `hash_table_v2_add_entry` function, I TODO

### Performance
```shell
TODO how to run and results
```

TODO more results, speedup measurement, and analysis on v2

## Cleaning up
```shell
make clean
```