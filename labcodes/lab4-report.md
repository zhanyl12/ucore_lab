###lab4-report

#练习零

这次的填充代码还是有一些问题，我们在完成所有的lab123的代码填充成功之后并不会check_swap成功而认为是建立进程失败，这时候不要产生怀疑继续往下做lab4就好，当你完成好proc的函数的时候所有的函数都会运行成功的，似乎代码把这个swap看做了是一个新的进程，千万不要以为是自己merge错误了而不敢往下做。

#练习一

本练习主要就是进行一些初始化操作，把state赋值成为还未建立，指针类型为null，其他类型一般为0或-1，对于parent和cr3进行一些单独的处理就可以了，cr3为boot_cr3，而parent是当前的进程。

struct context当中保存的是八个寄存器的值
```
struct context {
    uint32_t eip;
    uint32_t esp;
    uint32_t ebx;
    uint32_t ecx;
    uint32_t edx;
    uint32_t esi;
    uint32_t edi;
    uint32_t ebp;
};
```
属于我们术语当中说的“上下文”，当进行进程切换的时候我们保存context结构，当再次返回这个进程的时候仍然可以继续执行而不出问题，保证八个寄存器的值符合我们的需求。

tf是中断时候的帧的指针
```
struct trapframe {
    struct pushregs tf_regs;
    uint16_t tf_gs;
    uint16_t tf_padding0;
    uint16_t tf_fs;
    uint16_t tf_padding1;
    uint16_t tf_es;
    uint16_t tf_padding2;
    uint16_t tf_ds;
    uint16_t tf_padding3;
    uint32_t tf_trapno;
    /* below here defined by x86 hardware */
    uint32_t tf_err;
    uintptr_t tf_eip;
    uint16_t tf_cs;
    uint16_t tf_padding4;
    uint32_t tf_eflags;
    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding5;
} 
```
这个tf变量记录了发生中断之前堆栈的状态，保证当中断完成恢复之后，栈当中的内容不会发生变化，而进程切换其实也是可以看做是一种中断，tf和context配合，一个保证寄存器当中的值不变，而另一个保证堆栈当中的内容不变，他们一起保证了进程发生切换的时候进
程的数据不会有丢失。


#练习二

这段代码基本按照注释当中的写就可以轻松完成
```
proc=alloc_proc();
setup_kstack(proc);
copy_mm(clone_flags, proc);
copy_thread(proc, stack, tf);
proc->pid=get_pid();
hash_proc(proc);
list_add_before(&proc_list, &(proc->list_link));
wakeup_proc(proc);
ret=proc->pid;
```
首先使用alloc_proc分配空间，随后使用setup_kstack分配堆栈，之后进行copy赋值内存管理信息和上下文，使用get_pid把进程id得到然后通过hash将这个进程增加到进程列表当中去，唤醒进程使之运行随后函数返回进程号码。


ucore为新的fork的线程分配id的函数是get_pid，看看这个函数就可以知道这个问题的答案
```
static int
get_pid(void) {
    static_assert(MAX_PID > MAX_PROCESS);
    struct proc_struct *proc;
    list_entry_t *list = &proc_list, *le;
    static int next_safe = MAX_PID, last_pid = MAX_PID;
    if (++ last_pid >= MAX_PID) {
        last_pid = 1;
        goto inside;
    }
    if (last_pid >= next_safe) {
    inside:
        next_safe = MAX_PID;
    repeat:
        le = list;
        while ((le = list_next(le)) != list) {
            proc = le2proc(le, list_link);
            if (proc->pid == last_pid) {
                if (++ last_pid >= next_safe) {
                    if (last_pid >= MAX_PID) {
                        last_pid = 1;
                    }
                    next_safe = MAX_PID;
                    goto repeat;
                }
            }
            else if (proc->pid > last_pid && next_safe > proc->pid) {
                next_safe = proc->pid;
            }
        }
    }
    return last_pid;
}
```

整个有0-4095编号，这里的last_pid是最终的编号，可以看出它一直在递增，如果这些进程号码被装满的话，这里的repeat结构会起到作用，它会一直监听直到一个进程被释放掉，last_pid就会返回这个进程号码，因而有了这一套机制，ucore是一定可以保证进程可以被分配唯一的id。

最终我们得到如下的输出结果：
```
ebp:0xc0123f38 eip:0xc01009e7 0x00010094 0x00000000 0xc0123f68 0xc01000d3     kern/debug/kdebug.c:309: print_stackframe+21
ebp:0xc0123f48 eip:0xc0100cc7 0x00000000 0x00000000 0x00000000 0xc0123fb8     kern/debug/kmonitor.c:129: mon_backtrace+10
ebp:0xc0123f68 eip:0xc01000d3 0x00000000 0xc0123f90 0xffff0000 0xc0123f94     kern/init/init.c:58: grade_backtrace2+33
ebp:0xc0123f88 eip:0xc01000fc 0x00000000 0xffff0000 0xc0123fb4 0x0000002a     kern/init/init.c:63: grade_backtrace1+38
ebp:0xc0123fa8 eip:0xc010011a 0x00000000 0xc010002a 0xffff0000 0x0000001d     kern/init/init.c:68: grade_backtrace0+23
ebp:0xc0123fc8 eip:0xc010013f 0xc0109cbc 0xc0109ca0 0x00003188 0x00000000     kern/init/init.c:73: grade_backtrace+34
ebp:0xc0123ff8 eip:0xc010007f 0x00000000 0x00000000 0x0000ffff 0x40cf9a00     kern/init/init.c:33: kern_init+84
memory management: default_pmm_manager
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07efe000, [00100000, 07ffdfff], type = 1.
  memory: 00002000, [07ffe000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
check_alloc_page() succeeded!
check_pgdir() succeeded!
check_boot_pgdir() succeeded!
-------------------- BEGIN --------------------
PDE(0e0) c0000000-f8000000 38000000 urw
  |-- PTE(38000) c0000000-f8000000 38000000 -rw
PDE(001) fac00000-fb000000 00400000 -rw
  |-- PTE(000e0) faf00000-fafe0000 000e0000 urw
  |-- PTE(00001) fafeb000-fafec000 00001000 -rw
--------------------- END ---------------------
use SLOB allocator
kmalloc_init() succeeded!
check_vma_struct() succeeded!
page fault at 0x00000100: K/W [no page found].
check_pgfault() succeeded!
check_vmm() succeeded.
ide 0:      10000(sectors), 'QEMU HARDDISK'.
ide 1:     262144(sectors), 'QEMU HARDDISK'.
SWAP: manager = fifo swap manager
BEGIN check_swap: count 1, total 31986
setup Page Table for vaddr 0X1000, so alloc a page
setup Page Table vaddr 0~4MB OVER!
set up init env for check_swap begin!
page fault at 0x00001000: K/W [no page found].
page fault at 0x00002000: K/W [no page found].
page fault at 0x00003000: K/W [no page found].
page fault at 0x00004000: K/W [no page found].
set up init env for check_swap over!
write Virt Page c in fifo_check_swap
write Virt Page a in fifo_check_swap
write Virt Page d in fifo_check_swap
write Virt Page b in fifo_check_swap
write Virt Page e in fifo_check_swap
page fault at 0x00005000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x1000 to disk swap entry 2
write Virt Page b in fifo_check_swap
write Virt Page a in fifo_check_swap
page fault at 0x00001000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x2000 to disk swap entry 3
swap_in: load disk swap entry 2 with swap_page in vadr 0x1000
write Virt Page b in fifo_check_swap
page fault at 0x00002000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x3000 to disk swap entry 4
swap_in: load disk swap entry 3 with swap_page in vadr 0x2000
write Virt Page c in fifo_check_swap
page fault at 0x00003000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x4000 to disk swap entry 5
swap_in: load disk swap entry 4 with swap_page in vadr 0x3000
write Virt Page d in fifo_check_swap
page fault at 0x00004000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x5000 to disk swap entry 6
swap_in: load disk swap entry 5 with swap_page in vadr 0x4000
count is 0, total is 5
check_swap() succeeded!
++ setup timer interrupts
this initproc, pid = 1, name = "init"
To U: "Hello world!!".
To U: "en.., Bye, Bye. :)"
kernel panic at kern/process/proc.c:334:
    process exit!!.

```

#练习三

proc_run函数的代码如下
```
void 
proc_run(struct proc_struct *proc) {
    
if (proc != current) 
{
        
bool intr_flag;
        
struct proc_struct *prev = current, *next = proc;
        
local_intr_save(intr_flag);
        
{
            
current = proc;
            
load_esp0(next->kstack + KSTACKSIZE);
            
lcr3(next->cr3);
            
switch_to(&(prev->context), &(next->context));
       
 }
        
local_intr_restore(intr_flag);
   
 }

}
```

这个函数的执行过程首先是判断当前的进程是不是要切换到的进程，如果是……就省事了，如果不是就进行如下的操作：
第一个是要进制中断，随后把进程更新为要切换到的进程，之后修改esp0和cr3，使esp0指向新的进程的堆栈，使cr3指向新的进程的第一个页表的地址，随后在switch当中进行context八个寄存器的保存更换，最后回复中断。

在整个实验的过程当中，实际上是创建了两个内核进程，第一个proc_init当中的id为0的进程，即idleproc。而第二个内核进程是id为1的initproc.

这两个函数的作用是禁止中断和恢复中断。
