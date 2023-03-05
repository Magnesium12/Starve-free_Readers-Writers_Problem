# A starve-free solution to Readers-Writers problem.
## Description
Readers-Writers problem is a classical problem in the field of Operating Systems in which some processes may read, and some may write in a shared memory segment, with the constraint that no process can access the shared memory while another process is writing in it.

Many kinds of solutions exist for this problem, some give priority to Reader and some give priority to Writer. But both these solutions result in Starvation, writers starve in first, and readers starve in second. Here, I will propose a possible starve-free solution to Readers-Writers problem.
## Starve-free solution (Pseudocode)
### Global variables:
```cpp
// Counting variable
int readcount = 0;            // current number of reading processes

// Binary semaphores
semaphore shared = 1;         // for access to reading/writing shared memory
semaphore rcmutex = 1;        // for access to updating `readcount`
semaphore fifo_queue = 1;     // for avoiding starvation by maintaining FIFO order
```
### Reader Process:
```cpp
Reader() {
  .
  .
  .
  wait(fifo_queue);           // wait in FIFO queue for it's turn
  wait(rcmutex);              // Request and Lock access to readcount to update it
  readcount++;                // update number of reading processes
  if(readcount == 1){         // if current process is the first reader
    wait(shared);             // Lock the shared resource for all readers
  }
  signal(fifo_queue);         // Exit from FIFO queue
  signal(rcmutex);            // Release access to readcount
    
  // Critical Section
  // Reading done here

  wait(rcmutex);               // Request and Lock access to readcount to update it
  readcount--;                 // update number of reading processes
  if(readcount == 0){          // if there are no readers left
    signal(shared);            // Release the shared resource
  }
  signal(rcmutex);             // Release access to readcount
  .
  .
  .
}
```
### Writer Process:
```cpp
Writer() {
  .
  .
  .
  wait(fifo_queue);           // wait in FIFO queue for it's turn
  wait(shared);               // Request and Lock the shared resource
  signal(fifo_queue);         // Exit from FIFO queue
    
  // Critical Section
  // Writing is done here
  
  signal(shared);             // Release the shared resource
  .
  .
  .
}
```
### Explanation:
All readers and writers will first have to wait for their turn in the FIFO queue through the semaphore `fifo_queue`. The queue grants them access to the shared resource through the semaphore `shared`. Now, since Readers should be able to read simultaneously, so we use `readcount` to maintain the number of active readers, and when this is 1 (meaning our first reader), then only we request access to shared memory, because when this is not 1, then it means multiple readers are reading AND they have already recieved access to shared memory. Also to avoid race condition in updating `readcount`, we use another semaphore `rcmutex`. For writers, simultaneous writing is not allowed, so we use `fifo_queue` to lock the shared resource and release it only after writing.
### Highlights of this solution:
#### Mutual Exclusion:
The solution achieves Mutual exclusion w.r.t writing, as multiple writers cannot write simultaneously and reading cannot be performed with writing (semaphore `shared` is locked before writing and relased only after writing).
#### Progress:
The solution achieves Progress, as initially all semaphores are set to 1, so a reader or writer can perform and after the execution of a reader or writer, all semaphores are reset, thus this chain of reading/writing continues.
#### Starvation:
The solution is Starvation free. Each process is executed on the basis of it's time of entry in the FIFO queue only, thus no process can starve.
#### Deadlock:
The solution is Deadlock free. Deadlock occurs when a process is waiting for another waiting process to release resources. Here, if a writer is waiting for readers to release the shared memory, then it means that `readcount` will not increase now (as all new readers are waiting in FIFO queue behind the writer), so it will become zero in finite time (when all active readers finish reading), and our writer will have access to shared memory. Also any process waiting for a writer to release the shared memory will have it in finite time (after writer finishes writing).
#### Bounded Wait:
The solution has Bounded waiting time for each process. As explained above, any active reader or writer will be able to finish in finite time, thus allowing others to execute.
#### Fairness:
The solution achieves Fairness, It is fair because each process is executed in the order of their entry in the FIFO queue, thus giving equal opportunities to all processes.
