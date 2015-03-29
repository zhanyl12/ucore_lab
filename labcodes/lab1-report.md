#lab1-report

###练习一

1、首先通过学习man make，我在一定程度上熟悉了make在linux当中的使用方法，在这个练习题目当中我们在lab1的环境下面输入make –n的指令就能够达到题目的要求，在手册当中-n Print the commands that would be executed, but do not execute them.意思是执行命令，但不是真正的执行。常用来看，自己写的makefile变量展开后是什么，我们通过这条指令就可以看到ucore.img是如何一步一步地生成的具体的代码如下，在代码当中我会插入对这一段代码的解释：
```
echo + cc kern/init/init.c
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
echo + cc kern/libs/readline.c
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/readline.c -o obj/kern/libs/readline.o
echo + cc kern/libs/stdio.c
gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/stdio.c -o obj/kern/libs/stdio.o
echo + cc kern/debug/kdebug.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o
echo + cc kern/debug/kmonitor.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kmonitor.c -o obj/kern/debug/kmonitor.o
echo + cc kern/debug/panic.c
gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/panic.c -o obj/kern/debug/panic.o
echo + cc kern/driver/clock.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/clock.c -o obj/kern/driver/clock.o
echo + cc kern/driver/console.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/console.c -o obj/kern/driver/console.o
echo + cc kern/driver/intr.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/intr.c -o obj/kern/driver/intr.o
echo + cc kern/driver/picirq.c
gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/picirq.c -o obj/kern/driver/picirq.o
echo + cc kern/trap/trap.c
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trap.c -o obj/kern/trap/trap.o
echo + cc kern/trap/trapentry.S
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trapentry.S -o obj/kern/trap/trapentry.o
echo + cc kern/trap/vectors.S
gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/vectors.S -o obj/kern/trap/vectors.o
echo + cc kern/mm/1.c
gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/1.c -o obj/kern/mm/1.o
echo + cc kern/mm/pmm.c
gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/pmm.c -o obj/kern/mm/pmm.o
echo + cc libs/printfmt.c
gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/printfmt.c -o obj/libs/printfmt.o

```

其实以上这些代码都是差不多的，下面我们以下面三行的代码为示例进行
echo + cc libs/string.c
echo是dos批处理指令当中的一条子指令，这条指令的功能有很多种，这里就不一一赘述了，echo这里的功能似乎输出提示信息，输出了+cc libs/string.c。gcc是我们都熟悉的指令，而-Ilibs，-Ikern等则是表示程序需要链接的一些函数库，就如同说是要链接kern和libs等文件夹里面的代码函数。

而-fno-builtin表示是不接受不是两个下划线开头的内建函数，-Wall 是打开警告开关，-O代表默认优化，可选：-O0不优化，-O1低级优化，-O2中级优化，-O3高级优化，-Os代码空间优化。这里的wall指令是开启了编译器很多的常用的警告。-ggdb的意思是要生成较多的gdb调试信息，-m32意味着生成32位的代码，-gstabs是要以stabs的格式声明调试信息而不再是gdb的调试信息。

-nostdinc表示不在标准系统的目录中寻找头文件，只是搜索以“-I”为选项的目录，-fno-stack-protector表示禁用栈溢出保护，-c表示预处理编译汇编等工作生成-o文件，使用-o指出生成的文件的名称。

gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/string.c -o obj/libs/string.o
mkdir -p bin 
这个指令的意思是新建一个名称为bin的文件夹
echo + ld bin/kernel
echo打印提示的信息
ld -m  表示链接代码  elf_i386 生成的代码是elf-i386格式的代码-nostdlib -T tools/kernel.ld 通过-T的指令来制定要链接到的函数代码文件 -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/1.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o于是我们把上面生成的-o文件链接起来达到了我们的目的

objdump -S bin/kernel > obj/kernel.asm通过objdump –s的指令进行反汇编生成了asm文件
objdump -t bin/kernel | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > obj/kernel.sym 通过objdump -t的指令找到了文件符号表的入口，后面显示了编译的符号表。
echo + cc boot/bootasm.S 打印调试信息，编译bootasm.s
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o生成bootasm.o文件
echo + cc boot/bootmain.c编译bootmain.c
gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o生成bootasm.o文件
echo + cc tools/sign.c编译sign.c文件
gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o生成sign.o文件
gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign通过sign.o生成可执行文件，以上的详细代码基本都在前面有过叙述。

echo + ld bin/bootblock
ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o除了之前提过的elf_i386格式的代码之外，这里是对于bootasm.o文件和bootmain.o文件联合编译生成bootblock.o文件，这里的-e start的指令指示了进程入口的位置
objdump -S obj/bootblock.o > obj/bootblock.asm将bootblock.o文件反汇编生成bootblock.asm文件
objcopy -S -O binary obj/bootblock.o obj/bootblock.out这里吧bootblock.o文件转化通过拷贝成为二进制生成bootblock.out文件
bin/sign obj/bootblock.out bin/bootblock将sign文件和bootblock.out文件结合生成了硬盘引导扇区
dd if=/dev/zero of=bin/ucore.img count=10000  if表示输入文件，of表示输出文件，这里要输出10000个约定的东西，生成了一个512count的空白文件，每次输出要跳过512个字节，若seek为0就要在起始位置输出
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc


2、我们在上一个小问题当中是使用了sign.c文件配合其他文件生成符合规范的硬盘主引导扇区，其特征我们可以在sign.c的文件当中进行提取，首先我们看它的数组大小，这个硬盘的主引导扇区的大小必须是512个字节，而通过buf[510] = 0x55;buf[511] = 0xAA;两行代码我们可以看出这个主引导扇区的最后两个字节必须是0x55和0xAA。

小结：在第一个实验当中我们可以初步地掌握ucore代码的一些架构，整个ucore系统中的代码是怎样编译运行的，对于gcc，make以及-I等windows一系列批处理指令有了一定的了解，为以后使用或者阅读这些指令打下了基础。另外我们对于操作系统的硬盘的主引导扇区有了新的了解，这是第一次认识到符合规范的硬盘主引导扇区的特征，通过阅读一些windows批处理代码和ucore的代码对于lab1的实验有了初步的了解。

###练习二

1、CPU加电之后执行的第一条指令表示了qemu启动的时候虚拟机的状态，当生成ucore.img的镜像文件之后，使用命令行进入lab1的可执行文件的文件夹当中去，首先，我们打开qemu通过执行“qemu –S –s –hda ucore.img –monitor stdio”指令，但我们发现这会出现异常情况，qemu打不开，我们通过返回上一目录使用make debug的指令，之后再通过在一个新的teminal中执行“gdb”打开gdb调试，根据gitbook上面的提示知识，再输入“target remote:1234”，此时qemu 会进入停止状态,听从 gdb 的命令此时我们可以开始使用gdb进行调试了，通过gdb的单步调试，我们可以完成这道练习题。随后，我们在gdb中进行单步跟踪，执行“x/i $pc” 即可查看当前CPU执行的指令，这就是CPU加电之后执行的第一条指令。我们可以通过这种方式不断地获取如“=> 0x100000 <kern_init>:        push   %ebp”的指令内容

2、首先我们通过gdb设置调试断点，当设置断点成功以后，我们使用c即continue命令让qemu继续执行直到断点处，发现qemu显示着正在加载bootloader，结合gdb和qemu的显示结果可以判定断点设置成功。以下是gdb的调试信息：
Breakpoint 1, kern_init () at kern/init/init.c:17
(gdb) break *0x7c00
Breakpoint 2 at 0x7c00
(gdb) c
Continuing.
Breakpoint 2, 0x00007c00 in ?? ()

3、这一道练习题应在上一题目的基础上进行，通过x/?i $pc的指令，其中？为查看的指令数字，这样我们可以得到存在CPU里面的汇编代码：
```
0x00007c00:  cli    
0x00007c01:  cld    
0x00007c02:  xor    %ax,%ax
0x00007c04:  mov    %ax,%ds
0x00007c06:  mov    %ax,%es
0x00007c08:  mov    %ax,%ss
0x00007c0a:  in     $0x64,%al
0x00007c0c:  test   $0x2,%al
0x00007c0e:  jne    0x7c0a
0x00007c10:  mov    $0xd1,%al
0x00007c12:  out    %al,$0x64
0x00007c14:  in     $0x64,%al
0x00007c16:  test   $0x2,%al
0x00007c18:  jne    0x7c14
0x00007c1a:  mov    $0xdf,%al
0x00007c1c:  out    %al,$0x60
0x00007c1e:  mov    $0xdf,%al
0x00007c20:  out    %al,$0x60
0x00007c22:  lgdtw  0x7c70
0x00007c27:  mov    %cr0,%eax
0x00007c2a:  or     $0x1,%eax
0x00007c2e:  mov    %eax,%cr0
0x00007c31:  ljmp   $0x8,$0x7c36
0x00007c36:  mov    $0xd88e0010,%eax
0x00007c3c:  mov    %ax,%es
0x00007c3e:  mov    %ax,%fs
0x00007c40:  mov    %ax,%gs
0x00007c42:  mov    %ax,%ss
0x00007c44:  mov    $0x0,%bp
0x00007c47:  add    %al,(%bx,%si)
0x00007c49:  mov    $0x7c00,%sp
0x00007c4c:  add    %al,(%bx,%si)
0x00007c4e:  call   0x7d0f
0x00007c51:  add    %al,(%bx,%si)
0x00007c53:  jmp    0x7c53
0x00007c55:  lea    0x0(%bp),%si
0x00007c58:  add    %al,(%bx,%si)
0x00007c5a:  add    %al,(%bx,%si)
0x00007c5c:  add    %al,(%bx,%si)
0x00007c5e:  add    %al,(%bx,%si)

```

这和bootasm.S和bootblock.asm的代码是相同的，当然这并不难理解，.o文件实际上是由这两个文件链接之后编译得到的，那么最终得到的.o文件反汇编之后和那两个文件的汇编代码也自然是相同的。


4、以bootblock.o文件的readseg函数为例，我们按照第二题的方法以同样的方式启动QEMU，
	file obj/bootblock.o
	break readseg
	c
	在qemu输入命令：x/？i $pc
	gdb输出结果为：
```
0x00007c78:  push   %ebp
	0x00007c79:  add    %eax,%edx
	0x00007c7b:  mov    %esp,%ebp
	0x00007c7d:  push   %edi
	0x00007c7e:  push   %esi
	0x00007c7f:  mov    $0x1f7,%esi
	0x00007c84:  push   %ebx
	0x00007c85:  sub    $0x8,%esp
	0x00007c88:  mov    %edx,-0x14(%ebp)
	0x00007c8b:  mov    %ecx,%edx

```

这一段通过反汇编得到的指令和bootblock.asm的文件当中是相同的，这也符合我们的预期猜测。

小结：本阶段的练习可能和操作系统的知识点并没有太大的关系，主要是让我们熟悉了qemu以及gdb的使用方法，打开qemu，使用gdb设置断点，通过编译以及反汇编-o文件验证了练习一当中的makefile文件的执行过程，通过这一练习，我更加熟悉了gdb和qemu调试程序的方法，在今后的实验中会非常常用。

###练习三

想要分析bootloader如何从实模式进入保护模式，我们实际上结合bootasm.s的代码进行分析即可知道答案，下面我们细致地分析bootasm的代码，在分析过程当中回答gitbook上面的问题：
include <asm.h>
Start the CPU: switch to 32-bit protected mode, jump into C.
The BIOS loads this code from the first sector of the hard disk into
memory at physical address 0x7c00 and starts executing in real mode
with %cs=0 %ip=7c00.

.set PROT_MODE_CSEG,        0x8                     # kernel code segment selector
.set PROT_MODE_DSEG,        0x10                    # kernel data segment selector
.set CR0_PE_ON,             0x1                     # protected mode enable flag
以上一段是代码的头文件以及代码的标志和注释


start address should be 0:7c00, in real mode, the beginning address of the running bootloader
真正的起始地址应当是0x7c00，这是bootloader开始运行的地址
.globl start
start:
.code16                                             # Assemble for 16-bit mode
这是16位的模式
    cli                                             # Disable interrupts
    cld                                             # String operations increment

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment
这一段其实算是整个代码的一个初始化阶段，它首先是禁止了中断服务（利用cli指令），然后它将ax寄存器置为0，通过一系列操作最终实现了把DS，ES，SS寄存器都置零的目的，为下面的工作做好了准备

# Enable A20:
    #  For backwards compatibility with the earliest PCs, physical
    #  address line 20 is tied low, so that addresses higher than
    #  1MB wrap around to zero by default. This code undoes this.
seta20.1:
inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
从端口0x64读取数据，通过下面的一个判断，看看这个端口到底是不是繁忙的，如果这个端口是busy的，就持续下去这种不断的动作，继续看这个端口是不是繁忙的。
    testb $0x2, %al
    jnz seta20.1

    movb $0xd1, %al                                 # 0xd1 -> port 0x64
outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port
如果这个端口并不是繁忙的而是空闲的，程序就把al寄存器当中的数据写入0x64端口当中去。

seta20.2:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.2

    movb $0xdf, %al                                 # 0xdf -> port 0x60
outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
这一段代码和上一段其实是类似的，只不过有一些小的区别，如果0x64端口繁忙就继续监听下去，如果0x64端口是空闲的，那么就把数据写入0x60端口，注意这里其实把P2的A20的一个bit置为1，A20开启。

# Switch from real to protected mode, using a bootstrap GDT
    # and segment translation that makes virtual addresses
    # identical to physical addresses, so that the
    # effective memory map does not change during the switch.
lgdt gdtdesc
加载了全局符号描述表
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
movl %eax, %cr0
通过一系列操作吧cr0寄存器的值变为CR0_PE_ON，从此bootloader进入了保护模式

    # Jump to next instruction, but in 32-bit code segment.
    # Switches processor into 32-bit mode.
ljmp $PROT_MODE_CSEG, $protcseg
这里使用jump指令到32位模式下的protcseg执行指令，以下的代码都是使用了32位的模式

.code32                                             # Assemble for 32-bit mode
protcseg:
    # Set up the protected-mode data segment registers
    movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
    movw %ax, %ds                                   # -> DS: Data Segment
    movw %ax, %es                                   # -> ES: Extra Segment
    movw %ax, %fs                                   # -> FS
    movw %ax, %gs                                   # -> GS
    movw %ax, %ss                                   # -> SS: Stack Segment
这段代码首先把ax寄存器当中的值赋值为0x10，随后把ax寄存器当中的值依次赋给保护模式当中的各种寄存器，如DS,ES,FS,GS,SS寄存器，这些寄存器的值现在都是0x10,。
    # Set up the stack pointer and call into C. The stack region is from 0--start(0x7c00)
    movl $0x0, %ebp
    movl $start, %esp
call bootmain
将ebp寄存器置零，随后把start的地址0x7c00给esp地址寄存器就可以调函数bootmain，开始加载操作系统，完成bootloader的任务

# If bootmain returns (it shouldn't), loop.
如果这个bootmain有返回值就重复上面的步骤确保操作系统可以成功被加载。

spin:
    jmp spin

# Bootstrap GDT
.p2align 2                                          # force 4 byte alignment
gdt:
    SEG_NULLASM                                     # null seg
    SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)           # code seg for bootloader and kernel
    SEG_ASM(STA_W, 0x0, 0xffffffff)                 # data seg for bootloader and kernel
GDT表进行初始化，对于数据变量等进行赋值操作，为加载操作系统做好准备。
gdtdesc:
    .word 0x17                                      # sizeof(gdt) - 1
.long gdt                                       # address gdt

小结：这算是我在ucore大实验当中第一次比较细致地读某一部分的代码，结合后面的英文注释以及上网查找资料，发现只要投入足够的时间，读懂代码还是非常有可能的，这也增长了我对后面实验的信心。

###练习四

首先我们查阅gitbook的准备知识3.2.3节，读取一个扇区的流程有四步：
1.	等待磁盘准备好
2.	发出读取扇区的命令
3.	等待磁盘准备好
4.	把磁盘扇区数据读到制定的内存地方去
在上一个练习当中，我们已经call bootmain函数，下面我们在bootmain函数当中去找代码实际上是如何读取一个扇区的，事实上这一段流程主要是由readsect函数完成的：

readsect(void *dst, uint32_t secno) {
    // wait for disk to be ready
    waitdisk();
这是等待磁盘做好准备，waitdisk(void) {
    while ((inb(0x1F7) & 0xC0) != 0x40)
        /* do nothing */;
}可以看出waitdisk实际上只是等待磁盘的空闲，并没有做任何事情
    outb(0x1F2, 1);                         // count = 1
    outb(0x1F3, secno & 0xFF);
    outb(0x1F4, (secno >> 8) & 0xFF);
    outb(0x1F5, (secno >> 16) & 0xFF);
    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);
    outb(0x1F7, 0x20);                      // cmd 0x20 - read sectors
发出读取扇区的命令，实际上这是通过给这些IO地址寄存器进行赋值来完成这些发出指令这一工作的
    // wait for disk to be ready
    waitdisk();
继续等待磁盘准备好
    // read a sector
insl(0x1F0, dst, SECTSIZE / 4);
读取磁盘相应的扇区数据到指定的位置上面去
}

接着对于实验的第二个任务我们查阅相关资料，发现gitbook上面的指示并不明确，感觉核心是elfheader上面包含了很多的信息，我们需要通过这个elf的头进行判断，对接下来的事情进行指导，于是继续看bootmain.c的代码，其中的readseg的函数就是加载elf文件的函数：main函数的当中的代码：// read the 1st page off disk
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);
	从硬盘上读取elf文件的第一页
    // is this a valid ELF?
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
}
这里通过elf_header里面的e_magic选项，看是否等于ELF_MAGIC从而判断这个elf文件究竟是不是一个合法的elf文件，

readseg(uintptr_t va, uint32_t count, uint32_t offset) {
    uintptr_t end_va = va + count;

    // round down to sector boundary
    va -= offset % SECTSIZE;

    // translate from bytes to sectors; kernel starts at sector 1
    uint32_t secno = (offset / SECTSIZE) + 1;

    // If this is too slow, we could read lots of sectors at a time.
    // We'd write more to memory than asked, but it doesn't matter --
    // we load in increasing order.
    for (; va < end_va; va += SECTSIZE, secno ++) {
        readsect((void *)va, secno);
    }
}
如果这个elf文件是不合法的就执行goto bad地方的代码，如果这个elf文件是合法的：
// load each program segment (ignores ph flags)
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
    }
这里的elf文件头的位置是ph，ph+elf文件大小=eph，eph-1其实是这个elf文件结束的位置，当判定这个elf文件是一个合法的文件的时候，我们就通过for循环调用readseg函数把这个elf文件从硬盘中读入内存当中
    // call the entry point from the ELF header
    // note: does not return
((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

小结：通过练习四我对于如何读取指定的硬盘扇区以及bootloader如何加载elf文件有了一定的了解，这次看代码结合了比较丰富的理论知识，感觉收获良多。

###练习五

这个练习需要修改dern/debug当中的kdebug.c文件当中的堆栈函数，我们根据需要填充代码地方的注释步骤填写即可，以下按照注释解释一下我填充的代码：
uint32_t ebp = read_ebp();
uint32_t eip = read_eip();
第一个步骤，使用两个read函数，把ebp和eip的值得出来
    int depth=0;
    while(ebp > 0&&depth<STACKFRAME_DEPTH){
	int i;
	depth++;
        cprintf("ebp:0x%08x eip:0x%08x ",ebp,eip);
        uint32_t *args=(uint32_t *)ebp+2;
打印ebp和eip的值，对于args的数组进行赋值，注意要有栈帧深度的限制，另外c语言不能在for循环内部的变量使用的时候才进行声明，必须要在for循环开始之前就进行声明。
	for (i= 0;i<=3;i++) 
	{
            cprintf("0x%08x ", args[i]);
        }
        print_debuginfo(eip - 1);
调用print_debuginfo函数，把eip-1当作参数传入
        eip = ((uint32_t *)ebp)[1];
        ebp = ((uint32_t *)ebp)[0];
}
处理notice当中的内容


最终得到了如下的输出信息：(THU.CST) os is loading ...

Special kernel symbols:
  entry  0x00100000 (phys)
  etext  0x001032c3 (phys)
  edata  0x0010ea16 (phys)
  end    0x0010fd20 (phys)
Kernel executable memory footprint: 64KB
ebp:0x00007b08 eip:0x001009a6 args:0x00010094 0x00000000 0x00007b38 0x00100092 
    kern/debug/kdebug.c:321: print_stackframe+21
ebp:0x00007b18 eip:0x00100c95 args:0x00000000 0x00000000 0x00000000 0x00007b88 
    kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b38 eip:0x00100092 args:0x00000000 0x00007b60 0xffff0000 0x00007b64 
    kern/init/init.c:48: grade_backtrace2+33
ebp:0x00007b58 eip:0x001000bb args:0x00000000 0xffff0000 0x00007b84 0x00000029 
    kern/init/init.c:53: grade_backtrace1+38
ebp:0x00007b78 eip:0x001000d9 args:0x00000000 0x00100000 0xffff0000 0x0000001d 
    kern/init/init.c:58: grade_backtrace0+23
ebp:0x00007b98 eip:0x001000fe args:0x001032fc 0x001032e0 0x0000130a 0x00000000 
    kern/init/init.c:63: grade_backtrace+34
ebp:0x00007bc8 eip:0x00100055 args:0x00000000 0x00000000 0x00000000 0x00010094 
    kern/init/init.c:28: kern_init+84
ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8 
    <unknow>: -- 0x00007d67 --
++ setup timer interrupts


和使用answer_code当中的答案输出的信息是一样的，和gitbook上面的输出基本一致
最后一行代码，其实ebp已经回到了调用栈的最底一层，值为0，此处显示了unknow，eip实际上指向了当函数返回结束之后的下一条指令的地址，在这个例子当中为0x00007d68,0x00007d67为返回地址值

###练习六

1、在mmu.h的代码中我们可以看到
struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
};
	不难看出这个struct结构其实是64个bit，也就是8个Bit，2个字节，中断入口的几位应当是前16位和后16位合成一个地址。

2、按照代码注释步骤进行实行，结合mmu.h当中的函数，#define SETGATE(gate, istrap, sel, off, dpl)确定需要传入的参数究竟是什么，首先使用extern对于uintptr_t __vectors[]进行定义，随后即可使用，然后根据i究竟是系统调用还是异常，还是什么都不是去选择是否调用trap，setgate函数是在用户态下进行还是核心态下进行，最后使用lidt函数lidt使CPU获取idt地址，完成函数编写。


3、	这……略简单，不说了，直接对于ticks按要求进行操作，达到时间的时候调用print函数，不过感觉题目要求应该改为对于dispatch进行修改，当成功make之后，命令行会输出ticks并可以捕捉到按键的中断，以下是完整的输出结果：
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0x00100000 (phys)
  etext  0x00103585 (phys)
  edata  0x0010ea16 (phys)
  end    0x0010fd20 (phys)
Kernel executable memory footprint: 64KB
ebp:0x00007b08 eip:0x001009a6 0x00010094 0x00000000 0x00007b38 0x00100092     kern/debug/kdebug.c:306: print_stackframe+21
ebp:0x00007b18 eip:0x00100c86 0x00000000 0x00000000 0x00000000 0x00007b88     kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b38 eip:0x00100092 0x00000000 0x00007b60 0xffff0000 0x00007b64     kern/init/init.c:48: grade_backtrace2+33
ebp:0x00007b58 eip:0x001000bb 0x00000000 0xffff0000 0x00007b84 0x00000029     kern/init/init.c:53: grade_backtrace1+38
ebp:0x00007b78 eip:0x001000d9 0x00000000 0x00100000 0xffff0000 0x0000001d     kern/init/init.c:58: grade_backtrace0+23
ebp:0x00007b98 eip:0x001000fe 0x001035bc 0x001035a0 0x0000130a 0x00000000     kern/init/init.c:63: grade_backtrace+34
ebp:0x00007bc8 eip:0x00100055 0x00000000 0x00000000 0x00000000 0x00010094     kern/init/init.c:28: kern_init+84
ebp:0x00007bf8 eip:0x00007d68 0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8     <unknow>: -- 0x00007d67 --
++ setup timer interrupts
100 ticks
100 ticks
100 ticks
100 ticks
100 ticks
100 ticks
kbd [010] 

kbd [000] 
100 ticks
100 ticks
kbd [100] d
kbd [000] 
100 ticks
kbd [115] s
kbd [000] 
100 ticks
100 ticks

###challenge1

这个challenge任务是要完成用户态和内核态的切换，首先在trap文件当中的init_idt函数当中要加入状态切换的TOK判断，不能只是系统调用和非系统调用。随后更改init文件当中的switch函数，使用c联合汇编指令，进行状态的切换：asm volatile ("int %0\n" :: "i" (T_SWITCH_TOK));最后需要更改trap文件当中的dispatch函数，对于两个状态的切换进行细致函数描述：

case T_SWITCH_TOU:
这是从其他状态到用户态的切换
        if (tf->tf_cs == USER_CS) 
            break;如果当前的状态已经是用户态，函数就可以结束了
        else
        {
            struct trapframe to_user;
            to_user= *tf;
			使用临时struct变量将tf拷贝下来，将需要赋值的参数赋值给临时变量。
            to_user.tf_cs = USER_CS;
            to_user.tf_ds = USER_DS;
            to_user.tf_es = USER_DS;
            to_user.tf_ss = USER_DS;
            to_user.tf_esp = (uint32_t)tf+sizeof(struct trapframe)-8;
这里-8的意思是根据trapframe的结构定义，/* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;
    uint16_t tf_ss;
    uint16_t tf_padding5;这三个变量是需要不计算在内的。
            to_user.tf_eflags |= FL_IOPL_MASK;IO可以进行
            *((uint32_t*)tf-1) = (uint32_t)&to_user;
            break;
        }


切换到内核态
case T_SWITCH_TOK:
         if (tf->tf_cs == KERNEL_CS) 
            break;若是内核态也无须进行下面的函数
        else
        {
            struct trapframe* to_kernel;
            tf->tf_cs = KERNEL_CS;
            tf->tf_ds = KERNEL_DS;
            tf->tf_es = KERNEL_DS;
            tf->tf_eflags &= ~FL_IOPL_MASK;这里需要将中断位置保存下来进行处理并设置不能进行IO操作，当剔除掉无用的tf部分之后将其memmove得到kernel结果随后拷贝回去。
            to_kernel = (struct trapframe*)(tf->tf_esp-(sizeof(struct trapframe)-8));
            memmove(to_kernel, tf, sizeof(struct trapframe)-8);
            *((uint32_t*)tf-1) = (uint32_t)to_kernel;
            break;
        }
根据答案提示，需要tf->tf_eflags |= 0x3000;把IO的权限调低。
当此时我make grade的时候发现后面的地址数字都是正确的，但是round不能正确显示，而显示的是两个“！”，通过调试我发现是在init函数里面没有把调用switch的注释删除掉，当删除之后问题得到了解决。


###challenge2

本题目需要修改trap.c文件当中dispatch函数的case IRQ_OFFSET + IRQ_KBD，由于是硬中断不涉及challenge1当中的esp，ss等的操作，因而只需要几乎是将challenge1当中的函数照着抄过来，把局部临时变量等东西去掉就好了，参照题目的要求，不使用lab1_print_cur_status()的函数，而是使用print_trapframe(tf)函数即可：

```
if(c=='3')
        {
				tf->tf_eflags|=0x3000;
				if(tf->tf_cs==USER_CS)
					break;
				else
				{
					tf->tf_cs=USER_CS;
					//tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
					tf->tf_ds=USER_DS;
					tf->tf_es=USER_DS;
					tf->tf_ss=USER_DS;
					tf->tf_eflags|=FL_IOPL_MASK;
				}
				print_trapframe(tf);
		  }
		  else if(c=='0')
		  {
				if (tf->tf_cs==KERNEL_CS) 
            	break;
        		else
        		{
	            tf->tf_cs=KERNEL_CS;
	            tf->tf_ds=KERNEL_DS;
	            tf->tf_es=KERNEL_DS;
	            tf->tf_eflags&=~FL_IOPL_MASK;
        		}
        		print_trapframe(tf);
		  }

```
