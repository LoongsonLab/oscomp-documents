# 调试StarrOS

此部分主要介绍调试kernel的一些方法与技巧。

### 产生调试目标ELF文件

调试kernel需要产生对应的可执行文件，目前LoongArch支持两类调试。

1. 带有调试信息的kernel ELF文件

```shell
make MODE=debug ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp
# 此时产生的可执行文件带有调试信息
file apps/oscomp/oscomp_loongarch64-qemu-virt.elf

## 显示如下：
# apps/oscomp/oscomp_loongarch64-qemu-virt.elf: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), statically linked, with debug_info, not stripped

# 带有符合信息 with debug_info

```

此种生成的ELF文件，可以对照这rust源码，单步调试，也可以设置断点，查看执行状态

*注意*： 此步骤对于前期刚开始接触Starry内核时，比较友好。但是缺点也比较明显，就是执行
比较缓慢。

2. 优化后的不带调试信息的kernel ELF文件
```shell
make ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp
# 此时产生的可执行文件不带调试信息
file apps/oscomp/oscomp_loongarch64-qemu-virt.elf

## 显示如下：
# apps/oscomp/oscomp_loongarch64-qemu-virt.elf: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), statically linked, not stripped

# 没有符合信息，但是可以调试。

```
此方式下kernel执行效率较高，推荐此方式。


### QEMU模拟器

Qemu是一个开源的托管虚拟机，通过纯软件来实现虚拟化模拟器，几乎可以模拟任何硬件设备。

QEMU有两种主要运作模式：
* User mode模拟模式，亦即是用户模式。QEMU能启动那些为不同中央处理器编译的Linux程序。
* System mode模拟模式，亦即是系统模式。QEMU能模拟整个电脑系统，包括中央处理器及其他周边设备。它使得为跨平台编写的程序进行测试及除错工作变得容易。其亦能用来在一部主机上虚拟数部不同虚拟电脑。

我们使用QEMU的System mode模拟模式。

下面介绍一些QEMU的常用参数：

#### 1.设备类型(-machine/-M)
在qemu中，不同的指令集的模拟器会编译成不同的可执行文件，诸如：qemu-system-x86_64/qemu-system-aarch64/qemu-system-loongarch64/qemu-system-riscv64等。使用参数-machine help运行模拟器可以查看当前模拟器支持的设备信息，例如：

```shell
$ qemu-system-loongarch64 -M ?

Supported machines are:
none                 empty machine
virt                 Loongson-3A5000 LS7A1000 machine (default)

```
#### 2.内存大小(-m)
参数-m 1G就是指定虚拟机内部的内存大小为1GB，帮助说明如下：
```shell
$ qemu-system-loongarch64 -m ?

qemu-system-loongarch64: -m ?: Parameter 'size' expects a non-negative number below 2^64
Optional suffix k, M, G, T, P or E means kilo-, mega-, giga-, tera-, peta-
and exabytes, respectively.
```

#### 3.核心数(-smp)
现代cpu往往是对称多核心的，因此通过指定-smp 8可以指定虚拟机核心数。
```shell
$ qemu-system-loongarch64 -smp ?

smp-opts options:
  books=<num>
  clusters=<num>
  cores=<num>
  cpus=<num>
  dies=<num>
  drawers=<num>
  maxcpus=<num>
  sockets=<num>
  threads=<num>
```

#### 4.-nographic
disable graphical output and redirect serial I/Os to console.

#### 5.调试参数(-s -S)
此参数选项用于启动gdb服务，启动后qemu不立即运行guest，而是等待主机gdb发起连接。

#### 6.指定kernel镜像
此参数指定传入qemu的内核的镜像文件，一般是ELF文化，也可以是uImage等。

其他的可参考张老师的仓库[qemu-loongarch-runenv](https://github.com/LoongsonLab/qemu-loongarch-runenv)

### 使用QEMU调试

按照上面步骤，产生相应的kernel ELF文件。

``` shell
make debug ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp

```

此方式会调用qemu，相当于如下：

```shell
qemu-system-loongarch64 -m 16G -smp 1 -machine virt -kernel apps/oscomp/ \
oscomp_loongarch64-qemu-virt.elf -nographic -vga none -s -S

```

新打开一个终端，输入:
```shell
loongarch64-linux-gnu-gdb apps/oscomp/oscomp_loongarch64-qemu-virt.elf \
  -ex 'target remote localhost:1234' \
  -ex 'b rust_entry' \
  -ex 'continue' \
  -ex 'disp /16i $pc'
```

其中`target remote localhost:1234`通过端口1234链接到qemu，`b rust_entry`是在rust_entry处设置断点。
`disp /16i $pc`为显示当前pc位置的16条汇编指令。


如果配置正常的话，会显示如下：

```shell
Breakpoint 1, axhal::platform::loongarch64_qemu_virt::rust_entry (cpu_id=0, _dtb=0) at modules/axhal/src/platform/loongarch64_qemu_virt/mod.rs:25
25	    crate::mem::clear_bss();
1: x/16i $pc
=> 0x90000000921c6cf4 <axhal::platform::loongarch64_qemu_virt::rust_entry+20>:	
    bl          	3852(0xf0c)	# 0x90000000921c7c00 <_ZN5axhal3mem9clear_bss17h90acbf5a70f3d7b1E>
   0x90000000921c6cf8 <axhal::platform::loongarch64_qemu_virt::rust_entry+24>:	ld.d        	$a0, $sp, 0
   0x90000000921c6cfc <axhal::platform::loongarch64_qemu_virt::rust_entry+28>:	
--Type <RET> for more, q to quit, c to continue without paging--
    bl          	-12156(0xfffd084)	# 0x90000000921c3d80 <_ZN5axhal3cpu12init_primary17hed83e7a768a9c739E>
   0x90000000921c6d00 <axhal::platform::loongarch64_qemu_virt::rust_entry+32>:	pcalau12i   	$a0, -7(0xffff9)
   0x90000000921c6d04 <axhal::platform::loongarch64_qemu_virt::rust_entry+36>:	addi.d      	$a0, $a0, 0
   0x90000000921c6d08 <axhal::platform::loongarch64_qemu_virt::rust_entry+40>:	
    bl          	-584(0xffffdb8)	# 0x90000000921c6ac0 <_ZN5axhal4arch11loongarch6420set_trap_vector_base17hdb527d2259c8c6cdE>
   0x90000000921c6d0c <axhal::platform::loongarch64_qemu_virt::rust_entry+44>:	ori         	$a0, $zero, 0x200
   0x90000000921c6d10 <axhal::platform::loongarch64_qemu_virt::rust_entry+48>:	st.d        	$a0, $sp, 32(0x20)
   0x90000000921c6d14 <axhal::platform::loongarch64_qemu_virt::rust_entry+52>:	pcalau12i   	$a0, 154(0x9a)
   0x90000000921c6d18 <axhal::platform::loongarch64_qemu_virt::rust_entry+56>:	addi.d      	$a0, $a0, 0
   0x90000000921c6d1c <axhal::platform::loongarch64_qemu_virt::rust_entry+60>:	st.d        	$a0, $sp, 24(0x18)
   0x90000000921c6d20 <axhal::platform::loongarch64_qemu_virt::rust_entry+64>:	pcalau12i   	$a1, -6(0xffffa)
   0x90000000921c6d24 <axhal::platform::loongarch64_qemu_virt::rust_entry+68>:	addi.d      	$a1, $a1, 0
   0x90000000921c6d28 <axhal::platform::loongarch64_qemu_virt::rust_entry+72>:	
    bl          	-520(0xffffdf8)	# 0x90000000921c6b20 <_ZN5axhal4arch11loongarch648tlb_init17h97116086c310791dE>
   0x90000000921c6d2c <axhal::platform::loongarch64_qemu_virt::rust_entry+76>:	ld.d        	$a0, $sp, 0
   0x90000000921c6d30 <axhal::platform::loongarch64_qemu_virt::rust_entry+80>:	move        	$a1, $zero
(gdb) 

```

#### 1.设置断点
设置断点的命令为b(break)，分带有符合调试信息与不带符合调试信息。如下:

带符合调试信息的ELF文件

```shell
b platform/loongarch64_qemu_virt/boot.rs:30 # 在文件platform/loongarch64_qemu_virt/boot.rs的30行，设置断点。

# 上面等同于：
break platform/loongarch64_qemu_virt/boot.rs:30

```
对于没有调试信息的ELF文件，上面方式无法打断点。我们使用另外的方式打断点。

``` shell 
# 1. 获得需要设置断点的位置
loongarch64-linux-gnu-objdump -ald apps/oscomp/oscomp_loongarch64-qemu-virt.elf > elf.S

# 2. 比如想查看一下 trap_vector_base

loongarch64-linux-gnu-nm oscomp_loongarch64-qemu-virt.elf | grep trap_vector_base
# 显示 90000000921bf000 T trap_vector_base

# 3. 设置断点
b *0x90000000921bf000

# 4. 继续执行
c # 或者 continue

```

#### 2.单步调试

```shell
si  # 或者 step into
```

#### 3.查看寄存器的值

```shell 
i r #或者 info register

r0             0x0                 0
r1             0x0                 0x0 <axruntime::init_interrupt::__PERCPU_NEXT_DEADLINE>
r2             0x0                 0x0 <axruntime::init_interrupt::__PERCPU_NEXT_DEADLINE>
r3             0x90000000949c1fd0  0x90000000949c1fd0 <axhal::platform::loongarch64_qemu_virt::boot::BOOT_STACK+262096>
r4             0x0                 0
r5             0x0                 0
r6             0x0                 0
r7             0x0                 0
r8             0x0                 0
r9             0x0                 0
r10            0x0                 0
r11            0x0                 0
r12            0x90000000921c6ce0  10376293543912959200

```

查看某一个寄存器的值:

```shell
i r r1    #查看ra的值
i r r2    #查看tp的值
i r r3    #查看sp的值
i r $pc   #查看当前pc的值
```

#### 4.查看内存的值

查看内存0x120000000的值，以16进制显示。

```shell
p /x *0x120000000

```

#### 5.查看堆栈信息

- backtrace，也可简写为 bt，用于查看当前调用堆栈。
- frame 堆栈编号，也可简写为 f 堆栈编号，用于切换到其他堆栈处。

```shell
bt :
#0  axhal::platform::loongarch64_qemu_virt::rust_entry (cpu_id=0, _dtb=0)
    at modules/axhal/src/platform/loongarch64_qemu_virt/mod.rs:25
#1  0x0000000000000000 in ?? ()
Backtrace stopped: frame did not save the PC

f 0 :
#0  axhal::platform::loongarch64_qemu_virt::rust_entry (cpu_id=0, _dtb=0)
    at modules/axhal/src/platform/loongarch64_qemu_virt/mod.rs:25
25	    crate::mem::clear_bss();

```

其他的调试参数与gdb查看相关的文档。

[Qemu 官方文档](https://www.qemu.org/docs/master/)
[GDB 官方文档](https://www.gnu.org/software/gdb/documentation/)


