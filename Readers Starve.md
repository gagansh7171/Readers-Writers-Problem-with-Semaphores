Also called as **Second Readers-Writers problem**. It requires no writer, once added to the queue, shall be kept waiting longer than absolutely necessary. This is also called **writers-preference**.
This results in readers waiting and ultimately may lead to readers starving.

We use
* Semaphore **rw_mutex** initialized to 1 for mutual exclusion of reading or writing processes.
* Semaphore **rmutex** initialized to 1 for mutual exclusion between reading processes. It is used for a short duration when a reading process begins and ends,
 while updating read_count, not during the reading process itself.
* Semaphore **wmutex** initialized to 1 for mutual exclusion between writing processes. It is used for a short duration when a writing process begins and ends,
 while updating write_count, not during the writing process itself.
* Semaphore **readTry** initialized to 1 used by writing processes to lock out reading processes.
* Integer **read_count** initialized to 0 to count number of reading proceses.
* Integer **write_count** initialized to 0 to count number of writing processes.

Structure of Writer Process
```
do{
  wait(wmutex);
  if(++write_count == 1){
    wait(readTry);
  }
  signal(wmutex);
  
  wait(rw_mutex);
  ...
  /* writing is done*/
  ...
  signal(rw_mutex);
  
  wait(wmutex);
  if(--write_count == 0){
    signal(readTry);
  }
  signal(wmutex);
}while(true);
```
Structure of Reader Process
```
do{
  wait(readTry);
  wait(rmutex);
  if (++read_count == 1) 
    wait(rw_mutex); 
  signal(rmutex); 
  signal(readTry);
  
  ...
  /*Reading is done*/
  ...
  
  wait(rmutex);
  if (--read_count == 0) 
    signal(rw_mutex); 
  signal(rmutex);
}while(true);
```
