## UID: 105817312
(IMPORTANT: Only replace the above numbers with your true UID, do not modify spacing and newlines, otherwise your tarfile might not be created correctly)

# Hash Hash Hash

2 hash table implementations safe to use concurrently.

## Building

To build the program, run 

    make

## Running

  > ./hash-table-tester -t 8 -s 50000

  Generation: 68,319 usec
  Hash table base: 1,203,667 usec
    - 0 missing
  Hash table v1: 1,469,327 usec
    - 0 missing
  Hash table v2: 396,060 usec
    - 0 missing

## First Implementation

The first implementation uses a single mutex. It is a simple, course-grained locking strategy where only one thread at a time can add to the hash table. The mutex  locks the section of the hash_table_v1_add_entry function where a thread gets a hash table entry, gets a list entry from the hash table entry, and modifies the list (lines 74-92). This prevents any other thread from modifying or reading from the hash table if another thread is currently doing so. It is similar to the hash table base implementation because adding to the hash table is performed sequentially amongst all threads. This strategy focuses only on correctness because it is like there is only one thread running in the program. It is safe to use concurrently and produces the correct resuts, but the first implementation becomes slower than the base implementation as you use more threads. 

### Performance

Low number of threads: 

  > ./hash-table-tester -t 1 -s 100000

  Generation: 17,262 usec
  Hash table base: 49,747 usec
    - 0 missing
  Hash table v1: 53,087 usec
    - 0 missing

High number of threads:   

  > ./hash-table-tester -t 4 -s 100000

  Generation: 68,158 usec
  Hash table base: 1,238,591 usec
    - 0 missing
  Hash table v1: 2,237,661 usec
    - 0 missing

Version 1 performed 1.07x slower than base implementation with a low number of threads.

Version 1 performed 1.81x slower than base implementaion with a high number of threads. 

Version 1 performed closely to the base implementation when using a low number of threads because its implementation performs operations sequentially like the base implemenation. Using 1 thread is the same as running the base implementation, but Version 1 runs a little slower due to the small overhead from locking and unlocking the single mutex in the code. Version 1 performed drastically slower than the base implementation when using a high number of threads because there is large overhead for multiple threads to try and acquire the lock, the lock being held by another thread, and then waiting in queue for the lock to be freed.  

## Second Implementation

The second implementation uses multiple mutexes to achieve concurrency. Instead of having a single lock for the entire hash table structure like the first implementation, it uses a lock per hash bucket. Each hash table entry contains a linked list struct and a single mutex (lines 17-20). It initializes the mutex in the hash_table_v1_create function for every hash table entry created. It is unsafe for 2 threads to simultaneously read and modify a linked list, so both of these functions will be locked using the same mutex. Now multiple threads can run at the same time if they are accessing different hash table entries, optimizing performance. It achieves correctness by preventing another thread from reading or modifying the same linked list in a hash table entry by locking the hash table entry's mutex. The hash table entry's mutex is locked whenever a thread is inside the get_list_entry function and a portion of the code in the hash_table_v2_add_entry function that actually modifies the linked list (lines 84-96).

### Performance

  > ./hash-table-tester -t 4 -s 100000

  Generation: 69,011 usec
  Hash table base: 1,072,267 usec
    - 0 missing
  Hash table v2: 337,341 usec
    - 0 missing

Version 2 is 3.18x faster than the base implementation.

There is a performance increase in Version 2 because there are many concurrent operations taking place. The base implementation performs operations sequentially, where each thread must wait until the previous thread finishes accessing the hash table data structure as a whole. The second implementation uses a lock per hash table entry, so multiple threads at a time can use the hash table struct so long as they are accessing different entries. 

## Cleaning up

To clean up all binary files, run

    make clean
