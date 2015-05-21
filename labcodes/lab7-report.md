###lab7-report

#练习零

本次练习零简直是天地良心……直接粘贴lab6需要更新的代码就可以了，没有什么难度。

#练习一

首先执行make grade命令，得到如下的结果：
```
zyl@ubuntu:~/Documents/ucore_lab-master/labcodes/lab7$ make grade
badsegment:              (4.5s)
  -check result:                             OK
  -check output:                             OK
divzero:                 (4.5s)
  -check result:                             OK
  -check output:                             OK
softint:                 (4.2s)
  -check result:                             OK
  -check output:                             OK
faultread:               (3.1s)
  -check result:                             OK
  -check output:                             OK
faultreadkernel:         (3.0s)
  -check result:                             OK
  -check output:                             OK
hello:                   (3.9s)
  -check result:                             OK
  -check output:                             OK
testbss:                 (3.6s)
  -check result:                             OK
  -check output:                             OK
pgdir:                   (4.2s)
  -check result:                             OK
  -check output:                             OK
yield:                   (3.7s)
  -check result:                             OK
  -check output:                             OK
badarg:                  (4.2s)
  -check result:                             OK
  -check output:                             OK
exit:                    (3.6s)
  -check result:                             OK
  -check output:                             OK
spin:                    (3.6s)
  -check result:                             OK
  -check output:                             OK
waitkill:                (3.9s)
  -check result:                             OK
  -check output:                             OK
forktest:                (3.7s)
  -check result:                             OK
  -check output:                             OK
forktree:                (4.2s)
  -check result:                             OK
  -check output:                             OK
priority:                (22.9s)
  -check result:                             OK
  -check output:                             OK
sleep:                   (13.1s)
  -check result:                             OK
  -check output:                             OK
sleepkill:               (5.4s)
  -check result:                             OK
  -check output:                             OK
matrix:                  (90.7s)
  -check result:                             OK
  -check output:                             OK
Total Score: 190/190

```
都正确了

内核级信号量
```
typedef struct {
    int value;
    wait_queue_t wait_queue;
} semaphore_t;

void sem_init(semaphore_t *sem, int value);
void up(semaphore_t *sem);
void down(semaphore_t *sem);
bool try_down(semaphore_t *sem);
```
描述当时资源的值和带有一个等待的队列，这些函数当中有init构建初始化函数先进行执行，在而up和down函数是调用_up和_down两个函数，相当于PV操作，而trydown是一个尝试函数，在down当中进行判断
```
static __noinline void __up(semaphore_t *sem, uint32_t wait_state) {
    bool intr_flag;
    local_intr_save(intr_flag);
    {
        wait_t *wait;
        if ((wait = wait_queue_first(&(sem->wait_queue))) == NULL) {
            sem->value ++;
        }
        else {
            assert(wait->proc->wait_state == wait_state);
            wakeup_wait(&(sem->wait_queue), wait, wait_state, 1);
        }
    }
    local_intr_restore(intr_flag);
}
```
```
static __noinline uint32_t __down(semaphore_t *sem, uint32_t wait_state) {
    bool intr_flag;
    local_intr_save(intr_flag);
    if (sem->value > 0) {
        sem->value --;
        local_intr_restore(intr_flag);
        return 0;
    }
    wait_t __wait, *wait = &__wait;
    wait_current_set(&(sem->wait_queue), wait, wait_state);
    local_intr_restore(intr_flag);

    schedule();

    local_intr_save(intr_flag);
    wait_current_del(&(sem->wait_queue), wait);
    local_intr_restore(intr_flag);

    if (wait->wakeup_flags != wait_state) {
        return wait->wakeup_flags;
    }
    return 0;
}
```
在进行同步和互斥的操作的时候需要信号量实现这两个机制来保证os的正常运行，资源不会因为同时需要访问或者同步的原因发生错误。


用户态进程和线程信号量的实现可以依照内核进程和用户态进程的方法，在用户态使用一个虚拟的进程信号量，实际上内核维护一个对应的信号量，当需要进行操作的时候进行一次中断从用户态跳转到内核态，维护内核态对应的信号量，用内核态的信号量顶替用户态的信号量。


#练习二

monitor文件当中只需要把mt对应成为指向cvp的owner的指针，之后按照伪代码写下来就可以了，在此就不赘述了.

哲学家问题中在第一个需要添加代码的地方
```
state_condvar[i]=HUNGRY;
if ((state_condvar[LEFT]!=EATING)&&(state_condvar[RIGHT]!=EATING))
      {
         cprintf("phi_test_condvar: state_condvar[%d] will eating\n",i);
          state_condvar[i]=EATING;
      }
      else{
          cond_wait(&(mtp->cv[i]));
      }
```
哲学家是饥饿的，如果左右都没有在吃饭，那么他就可以开始吃了，如果有，那么他进入等待过程，按照注释实现代码即可。

第二个需要添加代码的地方
```
state_condvar[i] = THINKING;
phi_test_condvar(LEFT);
phi_test_condvar(RIGHT);
```
哲学家放下筷子进行思考，对于他左右两边的人进行检测是否满足可以吃饭的要求。

最终得到结果如下所示，make grade也正确
```
I am No.4 philosopher_condvar
Iter 1, No.4 philosopher_condvar is thinking
I am No.3 philosopher_condvar
Iter 1, No.3 philosopher_condvar is thinking
I am No.2 philosopher_condvar
Iter 1, No.2 philosopher_condvar is thinking
I am No.1 philosopher_condvar
Iter 1, No.1 philosopher_condvar is thinking
I am No.0 philosopher_condvar
Iter 1, No.0 philosopher_condvar is thinking
I am No.4 philosopher_sema
Iter 1, No.4 philosopher_sema is thinking
I am No.3 philosopher_sema
Iter 1, No.3 philosopher_sema is thinking
I am No.2 philosopher_sema
Iter 1, No.2 philosopher_sema is thinking
I am No.1 philosopher_sema
Iter 1, No.1 philosopher_sema is thinking
I am No.0 philosopher_sema
Iter 1, No.0 philosopher_sema is thinking
kernel_execve: pid = 2, name = "matrix".
fork ok.
pid 13 is running (1000 times)!.
pid 23 is running (37100 times)!.
pid 20 is running (37100 times)!.
pid 25 is running (23500 times)!.
Iter 1, No.0 philosopher_sema is eating
pid 17 is running (4600 times)!.
pid 24 is running (4600 times)!.
pid 28 is running (4600 times)!.
pid 26 is running (2600 times)!.
pid 30 is running (13100 times)!.
pid 19 is running (20600 times)!.
pid 14 is running (1000 times)!.
pid 15 is running (1100 times)!.
pid 32 is running (26600 times)!.
pid 33 is running (13100 times)!.
pid 27 is running (23500 times)!.
pid 21 is running (2600 times)!.
pid 18 is running (11000 times)!.
pid 22 is running (13100 times)!.
Iter 1, No.3 philosopher_sema is eating
pid 16 is running (1900 times)!.
phi_test_condvar: state_condvar[0] will eating
Iter 1, No.0 philosopher_condvar is eating
phi_test_condvar: state_condvar[2] will eating
Iter 1, No.2 philosopher_condvar is eating
cond_wait begin:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
cond_wait begin:  cvp c03a76d8, cvp->count 0, cvp->owner->next_count 0
pid 29 is running (33400 times)!.
pid 31 is running (2600 times)!.
cond_wait begin:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[3] will eating
phi_test_condvar: signal self_cv[3] 
cond_signal begin: cvp c03a76c4, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 1
Iter 1, No.3 philosopher_condvar is eating
Iter 2, No.3 philosopher_sema is thinking
Iter 1, No.2 philosopher_sema is eating
Iter 2, No.0 philosopher_sema is thinking
Iter 1, No.4 philosopher_sema is eating
Iter 2, No.4 philosopher_sema is thinking
Iter 2, No.2 philosopher_sema is thinking
Iter 2, No.3 philosopher_sema is eating
cond_signal end: cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
Iter 2, No.2 philosopher_condvar is thinking
phi_test_condvar: state_condvar[1] will eating
phi_test_condvar: signal self_cv[1] 
cond_signal begin: cvp c03a769c, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 1
Iter 1, No.1 philosopher_condvar is eating
cond_signal end: cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
Iter 2, No.0 philosopher_condvar is thinking
Iter 2, No.1 philosopher_condvar is thinking
phi_test_condvar: state_condvar[4] will eating
phi_test_condvar: signal self_cv[4] 
cond_signal begin: cvp c03a76d8, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a76d8, cvp->count 0, cvp->owner->next_count 1
Iter 1, No.4 philosopher_condvar is eating
pid 13 done!.
Iter 3, No.3 philosopher_sema is thinking
Iter 2, No.2 philosopher_sema is eating
Iter 2, No.0 philosopher_sema is eating
Iter 3, No.2 philosopher_sema is thinking
Iter 3, No.0 philosopher_sema is thinking
Iter 1, No.1 philosopher_sema is eating
Iter 2, No.4 philosopher_sema is eating
pid 15 done!.
Iter 2, No.1 philosopher_sema is thinking
Iter 2, No.1 philosopher_sema is eating
cond_signal end: cvp c03a76d8, cvp->count 0, cvp->owner->next_count 0
Iter 2, No.3 philosopher_condvar is thinking
Iter 2, No.4 philosopher_condvar is thinking
phi_test_condvar: state_condvar[2] will eating
Iter 2, No.2 philosopher_condvar is eating
Iter 3, No.1 philosopher_sema is thinking
Iter 3, No.1 philosopher_sema is eating
cond_wait begin:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[4] will eating
Iter 2, No.4 philosopher_condvar is eating
cond_wait begin:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[1] will eating
phi_test_condvar: signal self_cv[1] 
cond_signal begin: cvp c03a769c, cvp->count 1, cvp->owner->next_count 0
Iter 3, No.4 philosopher_sema is thinking
Iter 3, No.3 philosopher_sema is eating
Iter 4, No.3 philosopher_sema is thinking
cond_wait end:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 1
Iter 2, No.1 philosopher_condvar is eating
cond_signal end: cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
Iter 3, No.2 philosopher_condvar is thinking
cond_wait begin:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[3] will eating
phi_test_condvar: signal self_cv[3] 
cond_signal begin: cvp c03a76c4, cvp->count 1, cvp->owner->next_count 0
pid 14 done!.
Iter 4, No.1 philosopher_sema is thinking
Iter 3, No.2 philosopher_sema is eating
cond_wait end:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 1
Iter 2, No.3 philosopher_condvar is eating
cond_signal end: cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
Iter 3, No.4 philosopher_condvar is thinking
phi_test_condvar: state_condvar[0] will eating
phi_test_condvar: signal self_cv[0] 
cond_signal begin: cvp c03a7688, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 1
Iter 2, No.0 philosopher_condvar is eating
Iter 3, No.4 philosopher_sema is eating
Iter 4, No.2 philosopher_sema is thinking
Iter 4, No.1 philosopher_sema is eating
Iter 4, No.4 philosopher_sema is thinking
No.1 philosopher_sema quit
Iter 3, No.0 philosopher_sema is eating
Iter 4, No.3 philosopher_sema is eating
Iter 4, No.0 philosopher_sema is thinking
Iter 4, No.0 philosopher_sema is eating
cond_signal end: cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
Iter 3, No.1 philosopher_condvar is thinking
Iter 3, No.0 philosopher_condvar is thinking
cond_wait begin:  cvp c03a76b0, cvp->count 0, cvp->owner->next_count 0
pid 16 done!.
cond_wait begin:  cvp c03a76d8, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[2] will eating
phi_test_condvar: signal self_cv[2] 
cond_signal begin: cvp c03a76b0, cvp->count 1, cvp->owner->next_count 0
No.3 philosopher_sema quit
Iter 4, No.2 philosopher_sema is eating
cond_wait end:  cvp c03a76b0, cvp->count 0, cvp->owner->next_count 1
Iter 3, No.2 philosopher_condvar is eating
cond_signal end: cvp c03a76b0, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[4] will eating
phi_test_condvar: signal self_cv[4] 
cond_signal begin: cvp c03a76d8, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a76d8, cvp->count 0, cvp->owner->next_count 1
Iter 3, No.4 philosopher_condvar is eating
No.0 philosopher_sema quit
pid 21 done!.
Iter 4, No.4 philosopher_sema is eating
No.4 philosopher_sema quit
pid 26 done!.
No.2 philosopher_sema quit
cond_signal end: cvp c03a76d8, cvp->count 0, cvp->owner->next_count 0
Iter 3, No.3 philosopher_condvar is thinking
cond_wait begin:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
cond_wait begin:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
pid 31 done!.
phi_test_condvar: state_condvar[0] will eating
phi_test_condvar: signal self_cv[0] 
cond_signal begin: cvp c03a7688, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 1
Iter 3, No.0 philosopher_condvar is eating
cond_signal end: cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
Iter 4, No.4 philosopher_condvar is thinking
Iter 4, No.2 philosopher_condvar is thinking
phi_test_condvar: state_condvar[3] will eating
Iter 3, No.3 philosopher_condvar is eating
phi_test_condvar: state_condvar[1] will eating
phi_test_condvar: signal self_cv[1] 
cond_signal begin: cvp c03a769c, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a769c, cvp->count 0, cvp->owner->next_count 1
Iter 3, No.1 philosopher_condvar is eating
cond_signal end: cvp c03a769c, cvp->count 0, cvp->owner->next_count 0
Iter 4, No.0 philosopher_condvar is thinking
cond_wait begin:  cvp c03a76b0, cvp->count 0, cvp->owner->next_count 0
Iter 4, No.3 philosopher_condvar is thinking
phi_test_condvar: state_condvar[4] will eating
Iter 4, No.4 philosopher_condvar is eating
phi_test_condvar: state_condvar[2] will eating
phi_test_condvar: signal self_cv[2] 
cond_signal begin: cvp c03a76b0, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a76b0, cvp->count 0, cvp->owner->next_count 1
Iter 4, No.2 philosopher_condvar is eating
cond_signal end: cvp c03a76b0, cvp->count 0, cvp->owner->next_count 0
Iter 4, No.1 philosopher_condvar is thinking
pid 17 done!.
No.2 philosopher_condvar quit
cond_wait begin:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[1] will eating
Iter 4, No.1 philosopher_condvar is eating
pid 28 done!.
cond_wait begin:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
phi_test_condvar: state_condvar[3] will eating
phi_test_condvar: signal self_cv[3] 
cond_signal begin: cvp c03a76c4, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a76c4, cvp->count 0, cvp->owner->next_count 1
Iter 4, No.3 philosopher_condvar is eating
cond_signal end: cvp c03a76c4, cvp->count 0, cvp->owner->next_count 0
No.4 philosopher_condvar quit
phi_test_condvar: state_condvar[0] will eating
phi_test_condvar: signal self_cv[0] 
cond_signal begin: cvp c03a7688, cvp->count 1, cvp->owner->next_count 0
cond_wait end:  cvp c03a7688, cvp->count 0, cvp->owner->next_count 1
Iter 4, No.0 philosopher_condvar is eating
cond_signal end: cvp c03a7688, cvp->count 0, cvp->owner->next_count 0
No.1 philosopher_condvar quit
pid 24 done!.
No.3 philosopher_condvar quit
No.0 philosopher_condvar quit

```

内核级条件变量机制设计以
``
typedef struct monitor{
    semaphore_t mutex;      // the mutex lock for going into the routines in monitor, should be initialized to 1
    semaphore_t next;       // the next semaphore is used to down the signaling proc itself, and the other OR wakeuped waiting proc should wake up the sleeped signaling proc.
    int next_count;         // the number of of sleeped signaling proc
    condvar_t *cv;          // the condvars in monitor
} monitor_t;
```
为核心，每一个monitor有一个condvar，当一个monitor完成初始化之后，在cond_signal和cond_wait当中进行同步和互斥的操作，道理和PV相类似。
```
void 
cond_signal (condvar_t *cvp) {
   //LAB7 EXERCISE1: YOUR CODE
   cprintf("cond_signal begin: cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);  
  /*
   *      cond_signal(cv) {
   *          if(cv.count>0) {
   *             mt.next_count ++;
   *             signal(cv.sem);
   *             wait(mt.next);
   *             mt.next_count--;
   *          }
   *       }
   */
   struct monitor* mon = cvp->owner;
   if (cvp->count > 0){
          mon->next_count ++;
          up(&(cvp->sem));
          down(&(mon->next));
          mon->next_count --;
   }
   cprintf("cond_signal end: cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
}
```
```
void
cond_wait (condvar_t *cvp) {
    //LAB7 EXERCISE1: YOUR CODE
    cprintf("cond_wait begin:  cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
   /*
    *         cv.count ++;
    *         if(mt.next_count>0)
    *            signal(mt.next)
    *         else
    *            signal(mt.mutex);
    *         wait(cv.sem);
    *         cv.count --;
    */
    struct monitor* mon = cvp->owner;
    cvp->count ++;
      if (mon->next_count > 0){
          up(&(mon->next));
      }
      else{
          up(&(mon->mutex));
      }
      down(&(cvp->sem));
      cvp->count --;
    cprintf("cond_wait end:  cvp %x, cvp->count %d, cvp->owner->next_count %d\n", cvp, cvp->count, cvp->owner->next_count);
}

```

用户态线程的条件变量的实现也是使用一个虚拟的用户态条件变量，当需要进行条件变量的操作的时候进行中断转换到内核态对于一个构建的内核的条件变量进行操作，其实类似于上一个练习当中的问题，使用的是切换转换的方法。
