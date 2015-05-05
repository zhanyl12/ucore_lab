###lab5 reprot

#练习零

练习零算是本次一个比较有难度的练习，练习一和二相比于练习零简单的太多。首先看到doc当中的要求说会对实验1,2,3,4的代码有修改就蒙了，不过还好在实际复制代码的时候如果需要有更改的时候会有注释提示哪里需要修改，这样相对难度就简化了一些：

第一个修改点是在lab1的练习二的地方，我们在设置门的时候需要设置一次系统调用的门，只需要在原来的代码的基础之上加一个门：
```
extern uintptr_t __vectors[];
    int i;
    for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i++){
         SETGATE(idt[i], 0 ,GD_KTEXT ,__vectors[i], DPL_KERNEL);
    }
     /* LAB5 YOUR CODE */ 
     //you should update your lab1 code (just add ONE or TWO lines of code), let user app to use syscall to get the service of ucore
     //so you should setup the syscall interrupt gate in here
     SETGATE(idt[T_SYSCALL], 1, GD_KTEXT, __vectors[T_SYSCALL], DPL_USER);
     lidt(&idt_pd);
```

第二个修改点是在lab1的练习三的地方，对于tick_num设置一次current的need_resched为1即可，设置需要重新调用或者释放cpu：
```
current->need_resched = 1;
```

第三个修改点是lab4的练习一，新增了变量wait_state表示进程当前的状态是不是wait，以及变量proc_struct *cptr, *yptr, *optr表示进程和进程之间的关系，我们在初始化的时候也要对这些变量进行初始化，指针置为null，wait_state根据要求置为0：
```
proc->wait_state = 0;
proc->cptr = NULL;
proc->yptr = NULL;
proc->optr = NULL;
```
第四个修改点是lab4的练习二，这里需要设置相关，不过注释并没有包括所有的设置情况，所以在第一次完成所有的代码编写之后我的make grade结果是这样的，有一个错误：
```
forktest:                (2.9s)
  -check result:                             WRONG
   !! error: missing 'I am child 19'
   !! error: missing 'I am child 13'
   !! error: missing 'I am child 0'

```
经过查资料，参考result，我发现do_fork函数的当中还需要排除一些错误的情况，这样才能最终达到make grade的要求，于是经过修改，整理之后的代码如下：
```
    proc = alloc_proc();
    if (proc == NULL){
        goto fork_out;
    }
    proc->parent = current;
    if (setup_kstack(proc)){
        goto bad_fork_cleanup_proc;
    }
    if (copy_mm(clone_flags, proc)){
        goto bad_fork_cleanup_kstack;
    }
    copy_thread(proc, stack, tf);
    bool intr_flag;
    local_intr_save(intr_flag);
    proc->pid = get_pid();
    hash_proc(proc);
    set_links(proc);
    local_intr_restore(intr_flag);

    wakeup_proc(proc);
    ret = proc->pid;
```
至此，本lab最具有难度的部分就结束了。


#练习二

本练习当中的内容是对于中断现场进行设置，trapframe当中的内容基本上就是寄存器和标志，使用赋值语句按照注释当中的要求设置即可，把cs，ds，es，ss设置成为USER_DS,对于esp寄存器存储stack信息，eip记录代码入口，flags设置为FL_IF：
```
    tf->tf_cs = USER_CS;
    tf->tf_ds = USER_DS;
    tf->tf_es = USER_DS;
    tf->tf_ss = USER_DS;
    tf->tf_esp = USTACKTOP;
    tf->tf_eip = elf->e_entry;
    tf->tf_eflags |= FL_IF;
```

当加载了应用程序之后首先为当前继承新开一份存储区域，随后设置新的页表，对于堆栈进行相应的处理，对于页表首地址cr3等进行处理，随后设置上下文对于寄存器进行正确的赋值为执行elf文件做好准备，当进程被cpu选择进行running的时候程序的指令被执行。


#练习三

练习三完成对于资源的赋值，需要填充的代码实际上很简单，按照注释的提示设置对应的地址并进行拷贝就结束了：
```
	uintptr_t src_kvaddr = page2kva(page);
        uintptr_t dst_kvaddr = page2kva(npage);
        memcpy(dst_kvaddr, src_kvaddr, PGSIZE);
        ret = page_insert(to, npage, start, perm);
``` 当完成此函数的编写后我们已经可以得到正确结果了：

```
zyl@ubuntu:~/Documents/ucore_lab-master/labcodes/lab5$ make grade
badsegment:              (2.5s)
  -check result:                             OK
  -check output:                             OK
divzero:                 (2.5s)
  -check result:                             OK
  -check output:                             OK
softint:                 (2.5s)
  -check result:                             OK
  -check output:                             OK
faultread:               (2.6s)
  -check result:                             OK
  -check output:                             OK
faultreadkernel:         (2.7s)
  -check result:                             OK
  -check output:                             OK
hello:                   (2.4s)
  -check result:                             OK
  -check output:                             OK
testbss:                 (2.8s)
  -check result:                             OK
  -check output:                             OK
pgdir:                   (2.6s)
  -check result:                             OK
  -check output:                             OK
yield:                   (2.5s)
  -check result:                             OK
  -check output:                             OK
badarg:                  (2.6s)
  -check result:                             OK
  -check output:                             OK
exit:                    (2.6s)
  -check result:                             OK
  -check output:                             OK
spin:                    (5.5s)
  -check result:                             OK
  -check output:                             OK
waitkill:                (14.5s)
  -check result:                             OK
  -check output:                             OK
forktest:                (2.7s)
  -check result:                             OK
  -check output:                             OK
forktree:                (2.6s)
  -check result:                             OK
  -check output:                             OK
Total Score: 150/150

```


copy-on-write机制的实现：对于一个操作看究竟是否改变了内存的当中的内容，如果改变了是内存的当中的操作，使用一个结构体，首先是建立一个新的内存空间和页表，这个结构体存储的是对应的关系，一个用户对应一个存储的页表的首地址，这样可以保证一个使用者只能找到它对应的资源并进行使用，而其他的使用者并不能看到这一段资源。


#练习三

fork:fork函数实现的是进程的创建过程，它从父进程进行赋值生成了一个子进程

exec:exe函数实现的是函数是把进程进行运行的函数，它通过load_icode等可以进行把进程设置为运行的过程

wait:wait函数可以把进程设置为等待状态

exit:进程退出或者被杀死

进程一开始被创建，创建之后如果进程被调入就绪队列，进程的状态就会变为就绪，就绪状态下，当进程根据一些法则或者是被调度的时候进程就会运行，如果进程运行结束，那么进程就会退出，而如果发生了资源上面的确实或者是一些其他的等待事件，那么进程就会进入等待状态，而如果事件发生了，那么进程会重新从等待状态进入就绪队列，而如果因为时间片用完或者是抢占等因素的调度原因，进程也会从运行状态重新进入就绪队列。
