###lab8-report

#练习零

没有需要更改的代码，只是单纯的复制粘贴即可

#练习一

实验的第一个任务是完成sys_inode.c中的sys_io_nolock函数，这个函数的内容实际上是完成SFS层面上面对于文件的读写，以下为实现代码：
```
blkoff = offset % SFS_BLKSIZE;
        if (blkoff != 0){
            size = ( nblks != 0) ? (SFS_BLKSIZE - blkoff) : (endpos - offset);
            if (ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino))
                goto out;
            if (ret = sfs_buf_op(sfs, buf, size, ino, blkoff))
                goto out;
            alen += size;
            if (!nblks)
                goto out;
            buf += size;
            blkno++;
            nblks--;
        }
        size = SFS_BLKSIZE;
        while (nblks){
            if (ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino))
                goto out;
            if (ret = sfs_block_op(sfs, buf, ino, 1))
                goto out;
            alen += size;
            buf += size;
            blkno++;
            nblks--;
        }
        if (size = endpos % SFS_BLKSIZE){
            if (ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino))
                goto out;
            if (ret = sfs_buf_op(sfs, buf, size, ino, 0))
                goto out;
            alen += size;
        }

    out:
        *alenp = alen;
        if (offset + alen > sin->din->size) {
            sin->din->size = offset + alen;
            sin->dirty = 1;
        }
        return ret;
```

这一段代码是通过bmp_load_nolock和buf_op以及block_op三个函数实现的，把逻辑单元的内容映射称为磁盘物理单元的内容，把文件内容读取到buf当中或者是把文件按照单元的大小进行读取，在代码当中读写文件当中的内容是分为从开始到这一块结束，中间块以及最后一块内容到文件结束，分别调用不同的函数去完成这两种读取的方式，写也是类似的。

Unix的PIPE机制是指前一个程序的输出可以用作后一个程序的输入，其实是一种管道通信的机制，这种机制可以通过信号量实现，每完成一个输出就send一个信号到另一个程序当中去，也可以是一个程序完成输出，输出到一个文件当中，而另外一个程序去打开它，完成输入操作。


#练习二

练习二实际上需要修改三个部分的代码，第一个部分代码量很小，对于do_fork函数进行一些修改，加上copy_files即可，一开始在完成练习二之后代码崩掉了在mksfs文件当中的assert出现错误，我使用result进行对拍发现其实是自己的lab6当中的算法并不够强大，虽然可以过掉当时的测试，但是在这里弊端就暴露了出来。
第二个部分是之前的init阶段需要加上一个变量的初始化。
练习二第三个需要修改的地方是load_icode函数，基本需要重写：
```
 assert(argc >= 0 && argc <= EXEC_MAX_ARG_NUM);

    if (current->mm != NULL) {
        panic("load_icode: current->mm must be empty.\n");
    }

    int ret = -E_NO_MEM;
    struct mm_struct *mm;
    if ((mm = mm_create()) == NULL) {
        goto bad_mm;
    }
    if (setup_pgdir(mm) != 0) {
        goto bad_pgdir_cleanup_mm;
    }

    struct Page *page;

    struct elfhdr __elf, *elf = &__elf;
    if ((ret = load_icode_read(fd, elf, sizeof(struct elfhdr), 0)) != 0) {
        goto bad_elf_cleanup_pgdir;
    }

    if (elf->e_magic != ELF_MAGIC) {
        ret = -E_INVAL_ELF;
        goto bad_elf_cleanup_pgdir;
    }

    struct proghdr __ph, *ph = &__ph;
    uint32_t vm_flags, perm, phnum;
    for (phnum = 0; phnum < elf->e_phnum; phnum ++) {
        off_t phoff = elf->e_phoff + sizeof(struct proghdr) * phnum;
        if ((ret = load_icode_read(fd, ph, sizeof(struct proghdr), phoff)) != 0) {
            goto bad_cleanup_mmap;
        }
        if (ph->p_type != ELF_PT_LOAD) {
            continue ;
        }
        if (ph->p_filesz > ph->p_memsz) {
            ret = -E_INVAL_ELF;
            goto bad_cleanup_mmap;
        }
        if (ph->p_filesz == 0) {
            continue ;
        }
        vm_flags = 0, perm = PTE_U;
        if (ph->p_flags & ELF_PF_X) vm_flags |= VM_EXEC;
        if (ph->p_flags & ELF_PF_W) vm_flags |= VM_WRITE;
        if (ph->p_flags & ELF_PF_R) vm_flags |= VM_READ;
        if (vm_flags & VM_WRITE) perm |= PTE_W;
        if ((ret = mm_map(mm, ph->p_va, ph->p_memsz, vm_flags, NULL)) != 0) {
            goto bad_cleanup_mmap;
        }
        off_t offset = ph->p_offset;
        size_t off, size;
        uintptr_t start = ph->p_va, end, la = ROUNDDOWN(start, PGSIZE);

        ret = -E_NO_MEM;

        end = ph->p_va + ph->p_filesz;
        while (start < end) {
            if ((page = pgdir_alloc_page(mm->pgdir, la, perm)) == NULL) {
                ret = -E_NO_MEM;
                goto bad_cleanup_mmap;
            }
            off = start - la, size = PGSIZE - off, la += PGSIZE;
            if (end < la) {
                size -= la - end;
            }
            if ((ret = load_icode_read(fd, page2kva(page) + off, size, offset)) != 0) {
                goto bad_cleanup_mmap;
            }
            start += size, offset += size;
        }
        end = ph->p_va + ph->p_memsz;

        if (start < la) {
            /* ph->p_memsz == ph->p_filesz */
            if (start == end) {
                continue ;
            }
            off = start + PGSIZE - la, size = PGSIZE - off;
            if (end < la) {
                size -= la - end;
            }
            memset(page2kva(page) + off, 0, size);
            start += size;
            assert((end < la && start == end) || (end >= la && start == la));
        }
        while (start < end) {
            if ((page = pgdir_alloc_page(mm->pgdir, la, perm)) == NULL) {
                ret = -E_NO_MEM;
                goto bad_cleanup_mmap;
            }
            off = start - la, size = PGSIZE - off, la += PGSIZE;
            if (end < la) {
                size -= la - end;
            }
            memset(page2kva(page) + off, 0, size);
            start += size;
        }
    }
    sysfile_close(fd);

    vm_flags = VM_READ | VM_WRITE | VM_STACK;
    if ((ret = mm_map(mm, USTACKTOP - USTACKSIZE, USTACKSIZE, vm_flags, NULL)) != 0) {
        goto bad_cleanup_mmap;
    }
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-2*PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-3*PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-4*PGSIZE , PTE_USER) != NULL);
    
    mm_count_inc(mm);
    current->mm = mm;
    current->cr3 = PADDR(mm->pgdir);
    lcr3(PADDR(mm->pgdir));

    //setup argc, argv
    uint32_t argv_size=0, i;
    for (i = 0; i < argc; i ++) {
        argv_size += strnlen(kargv[i],EXEC_MAX_ARG_LEN + 1)+1;
    }

    uintptr_t stacktop = USTACKTOP - (argv_size/sizeof(long)+1)*sizeof(long);
    char** uargv=(char **)(stacktop  - argc * sizeof(char *));
    
    argv_size = 0;
    for (i = 0; i < argc; i ++) {
        uargv[i] = strcpy((char *)(stacktop + argv_size ), kargv[i]);
        argv_size +=  strnlen(kargv[i],EXEC_MAX_ARG_LEN + 1)+1;
    }
    
    stacktop = (uintptr_t)uargv - sizeof(int);
    *(int *)stacktop = argc;
    
    struct trapframe *tf = current->tf;
    memset(tf, 0, sizeof(struct trapframe));
    tf->tf_cs = USER_CS;
    tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
    tf->tf_esp = stacktop;
    tf->tf_eip = elf->e_entry;
    tf->tf_eflags = FL_IF;
    ret = 0;
```
其主要的执行步骤是创建mm和PDT，随后加载elf文件，对于每一个程序加载，分配内存，对于堆栈进行分配操作，对于tf要进行必要的设置。


回答问题首先要搞清楚什么是硬链接和软链接，硬链接是一个指针，fs并不会分配inode，而软链接算是一个新的文件，有inode。因而有需要创建硬链接的时候只是需要创建一个新的指针即可完成硬链接的要求，而如果需要创建软链接就可以把之前的文件的信息拷贝下来，重新创建一个新的文件，完成软链接。

下面来看看最终的运行结果，以ls和hello为例：
```
Iter 4, No.4 philosopher_condvar is eating
cond_signal end: cvp c0356a30, cvp->count 0, cvp->owner->next_count 0
No.3 philosopher_condvar quit
No.1 philosopher_condvar quit
No.4 philosopher_condvar quit
ls
 @ is  [directory] 2(hlinks) 23(blocks) 5888(bytes) : @'.'
   [d]   2(h)       23(b)     5888(s)   .
   [d]   2(h)       23(b)     5888(s)   ..
   [-]   1(h)       10(b)    40448(s)   waitkill
   [-]   1(h)       10(b)    40313(s)   hello
   [-]   1(h)       10(b)    40336(s)   sleep
   [-]   1(h)       11(b)    44572(s)   ls
   [-]   1(h)       10(b)    40336(s)   divzero
   [-]   1(h)       10(b)    40318(s)   badsegment
   [-]   1(h)       10(b)    40323(s)   faultreadkernel
   [-]   1(h)       11(b)    44626(s)   sh
   [-]   1(h)       10(b)    40317(s)   sleepkill
   [-]   1(h)       10(b)    40317(s)   faultread
   [-]   1(h)       11(b)    44503(s)   priority
   [-]   1(h)       10(b)    40312(s)   spin
   [-]   1(h)       10(b)    40315(s)   softint
   [-]   1(h)       11(b)    44516(s)   matrix
   [-]   1(h)       10(b)    40313(s)   yield
   [-]   1(h)       10(b)    40340(s)   testbss
   [-]   1(h)       10(b)    40338(s)   exit
   [-]   1(h)       10(b)    40342(s)   forktest
   [-]   1(h)       10(b)    40367(s)   forktree
   [-]   1(h)       10(b)    40313(s)   pgdir
   [-]   1(h)       10(b)    40314(s)   badarg
lsdir: step 4
$ hello
Hello world!!.
I am process 14.
hello pass.

```

感谢老师和助教的耐心评测！
