Also called as **Third Readers-Writers** problem. It requires that no thread shall be allowed to starve; that is, 
the operation of obtaining a lock on the shared data will always terminate in a bounded amount of time.
Here I present two solutions for the problem. One is the classical solution and the other is faster solution. The faster solution is 
fast as in it requires less number of semaphore lockings.

## Classical Solution

We use
* Semaphore **in** initialized to 1 to check which process is going to enter critical section.
* Semaphore **mutex** initialized to 1 for mutual exculsion in reading processes while updating read_count.
* Semaphore **rw_mutex** initialized to 1 for mutual exclusion between writing processes.
* Integer **read_count** initialized to 0 to count number of reading processes.

Structure of writer process
```
do{
  wait(in);
  wait(rw_mutex);
  ...
  /* Writing is done*/
  ...
  signal(rw_mutex);
  signal(in);
}while(true);
```
Structure of reader process
```
do{
  wait(in);
  wait(mutex);
  if(++read_count==1){
    wait(rw_mutex);
  }
  signal(mutex);
  signal(in);
  ...
  /* reading is done*/
  ...
  wait(mutex);
  if(--read_count==0){
    signal(rw_mutex);
  }
  signal(mutex);
}while(true);
```

**Writer Process** : Lock **in** semaphore to prevent reader process from reading.
Lock **rw_mutex** to prevent other writing processes from writing. Do the writing then release the semaphores.
Readers can block writers by locking rw_mutex if readers obtained permission to read first.<br/>
**Reader Process** : Wait if there's any writer using **in**. Lock **mutex** to update read_count and if its the first reader lock **rw_mutex** to 
prevent writers from writing. Release **in** and **mutex**. Do the reading.<br/>
Again lock **mutex** to update read_count and if its last reading process release **rw_mutex** to allow writers to write.

## Faster Solution

Faster Solution | Classical Solution
----------------|-------------------
Reader:<ul> <li>1 mutex for entering</li><li>1 mutex for leaving</li></ul>| Reader:<ul> <li>2 mutex for entering</li><li>1 mutex for leaving</li></ul>

We use
* Semaphore **in** initialized to 1 for a process entering.
* Semaphore **out** initialized to 1 for a proccess exiting.
* Semaphore **rw_mutex** initialized to 0 for exclusion of writing and reading processes.
* Integer **in_count** initialized to 0 number of entered reading processes.
* Integer **out_count** initialized to 0 number of exited reading processes.
* Boolean **wait** initialized to false to check if a writer is waiting.

Structure of Writer process
```
do{
  wait(in);
  wait(out);
  if(in_count==out_count){
    signal(out);
  }
  else{
    wait=true;
    signal(out);
    wait(rw_mutex);
    wait=false;
  }
  ...
  /* Writing is done*/
  ...
  signal(in);
}while(true);
```
Structure of Reader proces
```
do{
  wait(in);
  in_count++;
  signal(in);
  ...
  /*Reading is done*/
  ...
  wait(out);
  out_count++;
  if(wait && in_count==out_count){
    signal(rw_mutex);
  }
  signal(out);
}while(true);
```

**Writer process** : Lock **in** to prevent any new reader from entering. If number of entered reading process equal to exited reading process then just continue with writing process. Else mark **wait** equal to true to let readers know that writer processes are waiting. Lock **rw_mutex** when readers allow it to prevent other readers and writers from accessing the resource. When rw_mutex is acquired wait is marked false and writing is done. Release **in** in the end.
**Reader process** : 
