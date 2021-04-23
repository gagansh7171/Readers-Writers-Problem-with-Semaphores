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
Reader Process:
* Check with **readTry** if reading is allowed. If there is any writer process readTry is already locked and read process is halted here only. If not released this may lead to **Reader Starvation**.
* Reader process proceed by locking readTry if not already occupied by a writer process. Lock **rmutex** to update read_count. If it is first reader then lock **rw_mutex** so that writer process can not access the resource now. Subsequently rmutex and readTry are released.
* Reading is done.
* Lock **rmutex**. Update read_count. If no reading process left then release rw_mutex. Release rmutex at the end.

Writer Process:
*


