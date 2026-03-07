# Hash Hash Hash
A serial hash table implementation and two concurrent variants, which are made thread-safe by the use of mutexes.

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
The hash table has a single mutex. In the `hash_table_v1_add_entry` function, the mutex is used to guard both the "read" (obtaining the head of the linked list that will contain the key of interest, and traversing this list to determine whether the key already exists), as well as the "store" (updating the corresponding node, or else creating a new one if none exists). This prevents two threads reading from the same list before either finishes their store, which would create a race condition. 

### Performance
```shell
> ./hash-table-tester 
Generation: 25645 usec
Hash table base: 44068 usec
  - 0 missing
Hash table v1: 140642 usec
  - 0 missing
```
Version 1 is slower than the base version, as it introduces overhead related to thread management but doesn't actually leverage the threads it creates to gain any efficiency. In particular, the use of a single mutex serializes the add_entry function, so only one thread at a time can modify the hash table. Thus, we gain none of the benefits of parallelism, but all of the costs, including the overhead of creating and joining threads, context switches, and locking/unlocking mutexes. 

## Second Implementation
Rather than the entire hash table using a single mutex, every hash table **bucket** has its own mutex. In the function `hash_table_v2_add_entry`, we identify the target bucket and then lock only the mutex for that particular bucket. This ensures that concurrent updates to the same bucket are serialized, but threads working on different buckets can proceed in parallel. 

### Performance
Version 2 is faster than the base version. This is because it allows multiple threads to add entries to the hash table simultaneously, provided that the entries hash to different buckets. 

For instance:
```shell
> ./hash-table-tester
Generation: 27475 usec
Hash table base: 43878 usec
  - 0 missing
Hash table v1: 134315 usec
  - 0 missing
Hash table v2: 31767 usec
  - 0 missing
```

Using the default values (4 threads that each add 25000 entries), we achieve a speedup of 43878 usec / 31767 usec = 1.38x speedup.

The speedup becomes more prominent with more threads and entries per thread:
```shell
>  ./hash-table-tester -t 8 -s 50000
Generation: 93543 usec
Hash table base: 1236082 usec
  - 0 missing
Hash table v1: 6102570 usec
  - 0 missing
Hash table v2: 249803 usec
  - 0 missing
```

With 8 threads adding 50000 entries each, we achieve a speedup of 1236082 usec / 249803 usec = 4.95x speedup.

## Cleaning up
```shell
make clean
```