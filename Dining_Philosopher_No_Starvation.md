There are 5 philosophers and 5 forks. We require following variables - 
```
NUM = 5
owner_of_fork[NUM] // the owner of fork i is owner_of_fork[i], initialized to -1
wish_to_eat[NUM] // if a philosopher wish to eat then he will make it 1 else 0. Initialized to 0
fork_mutex[NUM] // mutex of fork i for accessing owner_of_fork[i]
phil_cv[NUM] // conditional variables for philosophers
```
To avoid starvation we preassign forks.
```
for(int i = 0; i<NUM; i++){
  if( owner_of_fork[i] == -1 && owner_of_fork[(i+1)%NUM] == -1 ){
    owner_of_fork[i] = i; owner_of_fork[(i+1)%NUM] = i;
  }
}
```
Loop for philosopher i 
```
while(1){
  think();
  wish_to_eat[i] = 1;
  get_left_fork(i);
  get_right_fork(i);
  eat();
  wish_to_eat[i] = 0;
  put_left_fork(i);
  put_right_fork(i);
}
```
To avoid starvation while putting down forks we will assign the forks to respective neigbours
```
void get_left_fork(i){
  fork_mutex[i].lock();
  while( wish_to_eat[(NUM+i-1)%NUM] && owner_of_fork[i] == (NUM+i-1)%NUM ){
    cond_wait(phil_cv[i], fork_mutex[i]);
  }
  owner_of_fork[i] = i;
  fork_mutex[i].unlock();
}

void put_left_fork(i) {
  fork_mutex[i].lock();
  owner_of_fork[i] = (NUM+i-1)%NUM;
  cond_signal(phil_cv[(NUM+i-1)%NUM]);
  fork_mutex[i].unlock();
}
```
