###lab6-report


#练习零

从难度上本次练习零要比lab5的难度小很多，基本上来说只是需要复制lab5的代码，需要在lab5的基础之上修改两个地方：

第一个地方是trap的练习一的tick计数的地方代码，我们按照代码的注释提示重新写就可以了，进行一次tick的更新随后运行指定的函数：
```
ticks += 1;
run_timer_list();
```

第二个地方是在proc的练习四的地方进行初始化又多了几个,指针初始化为null，一般的数字变量初始化为0，而优先级的变量要按照指定的规则赋值：
```
memset(proc, 0, sizeof(struct proc_struct));
    proc->state = PROC_UNINIT;
    proc->pid = -1;
    proc->runs = 0;
    proc->kstack = 0;
    proc->need_resched = 0;
    proc->parent = NULL;
    proc->mm = NULL;
    memset(&(proc->context), 0, sizeof(struct context));
    proc->tf = NULL;
    proc->cr3 = boot_cr3;
    proc->flags = 0;
    set_proc_name(proc, "");
    list_init(&proc->list_link);
    list_init(&proc->hash_link);
     //LAB5 YOUR CODE : (update LAB4 steps)
    /*
     * below fields(add in LAB5) in proc_struct need to be initialized	
     *       uint32_t wait_state;                        // waiting state
     *       struct proc_struct *cptr, *yptr, *optr;     // relations between processes
	 */
    proc->wait_state = 0;
    proc->cptr = NULL;
    proc->yptr = NULL;
    proc->optr = NULL;
     //LAB6 YOUR CODE : (update LAB5 steps)
    /*
     * below fields(add in LAB6) in proc_struct need to be initialized
     *     struct run_queue *rq;                       // running queue contains Process
     *     list_entry_t run_link;                      // the entry linked in run queue
     *     int time_slice;                             // time slice for occupying the CPU
     *     skew_heap_entry_t lab6_run_pool;            // FOR LAB6 ONLY: the entry in the run pool
     *     uint32_t lab6_stride;                       // FOR LAB6 ONLY: the current stride of the process
     *     uint32_t lab6_priority;                     // FOR LAB6 ONLY: the priority of process, set by lab6_set_priority(uint32_t)
     */
    proc->rq = NULL;
    list_init(&proc->run_link);
    proc->time_slice = 0;
    proc->lab6_stride = 0;
    proc->lab6_priority = 1;
    }
```

#练习一


最终我们当完成上述的工作之后执行make qemu和lab5没有什么不同，当执行make grade的时候就会出现下面的情况：
```
K> zyl@ubuntu:~/Documents/ucore_lab-master/labcodes/lab6$ make grade
badsegment:              (3.3s)
  -check result:                             OK
  -check output:                             OK
divzero:                 (2.9s)
  -check result:                             OK
  -check output:                             OK
softint:                 (2.9s)
  -check result:                             OK
  -check output:                             OK
faultread:               (3.0s)
  -check result:                             OK
  -check output:                             OK
faultreadkernel:         (2.9s)
  -check result:                             OK
  -check output:                             OK
hello:                   (3.0s)
  -check result:                             OK
  -check output:                             OK
testbss:                 (3.5s)
  -check result:                             OK
  -check output:                             OK
pgdir:                   (3.0s)
  -check result:                             OK
  -check output:                             OK
yield:                   (2.9s)
  -check result:                             OK
  -check output:                             OK
badarg:                  (2.7s)
  -check result:                             OK
  -check output:                             OK
exit:                    (2.9s)
  -check result:                             OK
  -check output:                             OK
spin:                    (3.5s)
  -check result:                             OK
  -check output:                             OK
waitkill:                (5.4s)
  -check result:                             OK
  -check output:                             OK
forktest:                (3.1s)
  -check result:                             OK
  -check output:                             OK
forktree:                (3.2s)
  -check result:                             OK
  -check output:                             OK
matrix:                  (61.1s)
  -check result:                             OK
  -check output:                             OK
priority:                (24.2s)
  -check result:                             WRONG
   !! error: missing 'sched class: stride_scheduler'
   !! error: missing 'stride sched correct result: 1 2 3 4 5'

  -check output:                             OK
Total Score: 163/170
make: *** [grade] Error 1

```
在多了几个测试用例的情况下除了priority的测试用例都通过了


```
struct sched_class {
    // the name of sched_class
    const char *name;
    // Init the run queue
    void (*init)(struct run_queue *rq);
    // put the proc into runqueue, and this function must be called with rq_lock
    void (*enqueue)(struct run_queue *rq, struct proc_struct *proc);
    // get the proc out runqueue, and this function must be called with rq_lock
    void (*dequeue)(struct run_queue *rq, struct proc_struct *proc);
    // choose the next runnable task
    struct proc_struct *(*pick_next)(struct run_queue *rq);
    // dealer of the time-tick
    void (*proc_tick)(struct run_queue *rq, struct proc_struct *proc);
};
```
在sched_class当中，第一个指针指向了sched_class的名字，第二个指针是执行队列的初始化，第三个指针函数是把一个进程加入到当前的队列当中，第四个指针函数是把一个进程从进程的队列当中删除出去，最后一个函数指针是计算整个队列当中的proc的tick，进行更新。

ucore当中的RR算法是从进程的队列当中拿出队列头部的进程进行一定时间的运行，这里是通过前面的dequeue实现的，而当时间片消耗完就把进程加入到队列的尾部，这是通过enqueue实现的，如此循环执行，ucore的sched代码当中和timer相关的是表示进程执行时间相关的函数，用于计时等，其他函数在上面已经说明了具体的功能。

关于多级反馈队列算法的实现，我们按照ucore的实际情况，比如可以设置3个多级反馈队列，给这三个反馈队列不同的优先级，由高到低为1,2,3，这三个队列对应的时间片依次变为前一个的两倍，这样按照queue的结构定义三个队列，加入成员变量优先级，在proc运行的函数当中，一个新的proc一开始加入到优先级最高的队列当中去，而如果在高级的队列当中一个时间片没有完成这个进程，那么就把这个进程放入到低一级的队列当中去。对于timer的执行点我们对于ucore的高优先级队列进行判断，如果这个队列是空的话，那么就用低优先级的队列当中的proc对于高优先级进行抢占，这样的方法就大体实现了多级反馈队列的算法。


#练习二

在填写完代码完成调试之后得到了如下的结果，基本正确

```
kernel_execve: pid = 2, name = "priority".
main: fork ok,now need to wait pids.
child pid 7, acc 256000, time 2017
child pid 6, acc 204000, time 2028
child pid 5, acc 160000, time 2030
child pid 4, acc 112000, time 2036
child pid 3, acc 64000, time 2041
main: pid 3, acc 64000, time 2044
main: pid 4, acc 112000, time 2046
main: pid 5, acc 160000, time 2047
main: pid 6, acc 204000, time 2049
main: pid 7, acc 256000, time 2050
main: wait pids over
stride sched correct result: 1 2 3 3 4
all user-mode processes have quit.
init check memory pass.
kernel panic at kern/process/proc.c:480:
    initproc exit.

```

下面介绍填写的代码：

```
#define BIG_STRIDE 2048 /* you should give a value, and is ??? */
```
第一个需要确定big_stride的值，这个值需要是一个2的若干次方，值要稍微大一些，这里我填写了2048，从最终的结果看来这也是一个不错的选择。

```
static void
stride_init(struct run_queue *rq) {
     /* LAB6: YOUR CODE
      * (1) init the ready process list: rq->run_list
      * (2) init the run pool: rq->lab6_run_pool
      * (3) set number of process: rq->proc_num to 0
      */
    list_init(&rq->run_list);
    rq->lab6_run_pool=NULL;
    rq->proc_num=0;
}
```
这一部分代码主要按照注释填写即可，建立进程的list，对于run_pool进行初始化，把进程的计数器设置为0，对于max_slice的变量注释当中说不需要设置。

```
static void
stride_enqueue(struct run_queue *rq, struct proc_struct *proc) {
     /* LAB6: YOUR CODE
      * (1) insert the proc into rq correctly
      * NOTICE: you can use skew_heap or list. Important functions
      *         skew_heap_insert: insert a entry into skew_heap
      *         list_add_before: insert  a entry into the last of list
      * (2) recalculate proc->time_slice
      * (3) set proc->rq pointer to rq
      * (4) increase rq->proc_num
      */
    rq->lab6_run_pool=skew_heap_insert(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
    proc->time_slice=rq->max_time_slice;
    proc->rq=rq;
    rq->proc_num++;
}
```
这一段是入队列代码，按照注释当中使用提示的skew_heap_insert函数进行插入即可，对于进程的time_slice变量使用rq的max_time_slice变量重新计算，把proc->rq的值置为rq，最后增加进程总数即可。

```
static void
stride_dequeue(struct run_queue *rq, struct proc_struct *proc) {
     /* LAB6: YOUR CODE
      * (1) remove the proc from rq correctly
      * NOTICE: you can use skew_heap or list. Important functions
      *         skew_heap_remove: remove a entry from skew_heap
      *         list_del_init: remove a entry from the  list
      */
    rq->lab6_run_pool = skew_heap_remove(rq->lab6_run_pool, &(proc->lab6_run_pool), proc_stride_comp_f);
    rq->proc_num--;
}
```

这一段是出队列代码，按照提示，使用remove函数删除，随后减少进程总数即可。

```
static struct proc_struct *
stride_pick_next(struct run_queue *rq) {
     /* LAB6: YOUR CODE
      * (1) get a  proc_struct pointer p  with the minimum value of stride
             (1.1) If using skew_heap, we can use le2proc get the p from rq->lab6_run_poll
             (1.2) If using list, we have to search list to find the p with minimum stride value
      * (2) update p;s stride value: p->lab6_stride
      * (3) return p
      */
    struct proc_struct * p = NULL;
    if (rq->lab6_run_pool)
    {
        p=le2proc(rq->lab6_run_pool, lab6_run_pool);
        p->lab6_stride+=(BIG_STRIDE / p->lab6_priority);
    }
    return p;
}
```

这个函数是挑选下一个运行的进程，队列当中stride值最小的，随后更新stride值返回相应的进程。

```
static void
stride_proc_tick(struct run_queue *rq, struct proc_struct *proc) {
     /* LAB6: YOUR CODE */
    if (proc->time_slice>0)
    {
        proc->time_slice--;
    }
    if (proc->time_slice==0)
    {
        proc->need_resched=1;
    }
}
```

对于进程时间tick的处理，和doc当中的代码是一致的。
