
Also known as <b>First</b> Readers-Writers problem. It requires that no reader be kept waiting unless a writer has already obtained
permission to use the shared object.

We use 
<ul>
  <li>Semaphore <b>rw_mutex</b> initialized to 1 for mutual exclusion of reading or writing processes.</li>
  <li>Semaphore <b>mutex</b> initialized to 1 for mutual exclusion between reading processes. It is used for a short duration when a reading process begins and ends, while updating read_count, <b>not</b> during the reading process itself.</li>
  <li>Integer <b>read_count</b> initialized to 0 to keep track of number reading processes.</li>
 </ul>

Structure of Writer Process
```
do {
  wait(rw_mutex); 
               ...     
               /* writing is performed */ 
               ... 
  signal(rw_mutex); 
} while (true);

```

Structure of Reader Process
```
do {
  wait(mutex);
  if (++read_count == 1) 
    wait(rw_mutex); 
  signal(mutex); 
  ...
  /* reading is performed */ 
  ... 
  wait(mutex);
  if (--read_count == 0) 
    signal(rw_mutex); 
  signal(mutex); 
} while (true);
```

Reader Process : 
* First lock mutex to update number of reading processes(read_count). If there's any reader process then rw_mutex is locked. This puts writers on <b>wait</b>. On updating number of reading processes mutex is released.
*  Reading is performed.
*  Lock mutex to update read_count. If no reading process left release rw_mutex else dont. If rw_mutex is not released due to continuous request from reader process(i.e. read_count is not 0) then it may lead **Writers Starvation** . On updating read_count release mutex.

Writer Process:
* Lock rw_mutex.
* Do the writing.
* Release rw_mutex.

