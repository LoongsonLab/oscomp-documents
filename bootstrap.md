
# 启动流程

本章节主要说明程序执行的一些关键步骤，我们可能会涉及部分的module，具体module的功能与实现说明
参考文档在线文档详见：[Starry (azure-stars.github.io)](https://azure-stars.github.io/Starry/)。

下面的例子主要以Qemu-virt平台为主，2k1000的类似。

## kernel的入口
LoongArch的reset复位入口为0x1c000000，此处的代码我们称为BIOS，初始化设备后会将执行权交给
kernel。而StarryOS的入口地址为0x9000000092000000。对应到物理地址为0x92000000，符号为_start，
实现如下。

> Qemu Virt平台的物理内存根据传入参数的不同，大小也不一样。具体的可参考[平台支持](/platforms.md)中Qemu virt平台设备树的描述。
> 2K1000平台可参考[龙芯2k1000LA处理器](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E9%BE%99%E8%8A%AF2K1000LA%E5%A4%84%E7%90%86%E5%99%A8%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_V1.0.pdf)中第6部分**地址空间分配**章节的描述。需要注意的是2k1000的处理器还要参考具体的开发板的型号已经支持的最大内存等等。
> 为了统一，减少平台差异，我们选择0x92000000作为Qemu Virt和2K1000平台的内核入口地址。

具体如何设置，可参考文件platforms/loongarch64-2k1000.toml和platforms/loongarch64-qemu-virt.toml配置文件。

代码位置：modules/axhal/src/platform/loongarch64_qemu_virt/boot.rs
```rust
#[naked]
#[no_mangle]
#[link_section = ".text.boot"]
unsafe extern "C" fn _start() -> ! {
    core::arch::asm!("
        ori         $t0, $zero, 0x1     # CSR_DMW1_PLV0
        lu52i.d     $t0, $t0, -2048     # UC, PLV0, 0x8000 xxxx xxxx xxxx
        csrwr       $t0, 0x180          # LOONGARCH_CSR_DMWIN0
        ori         $t0, $zero, 0x11    # CSR_DMW1_MAT | CSR_DMW1_PLV0
        lu52i.d     $t0, $t0, -1792     # CA, PLV0, 0x9000 xxxx xxxx xxxx
        csrwr       $t0, 0x181          # LOONGARCH_CSR_DMWIN1
        # Enable PG 
        li.w		$t0, 0xb0		# PLV=0, IE=0, PG=1
        csrwr		$t0, 0x0        # LOONGARCH_CSR_CRMD
        li.w		$t0, 0x00		# PLV=0, PIE=0, PWE=0
        csrwr		$t0, 0x1        # LOONGARCH_CSR_PRMD
        li.w		$t0, 0x00		# FPE=0, SXE=0, ASXE=0, BTE=0
        csrwr		$t0, 0x2        # LOONGARCH_CSR_EUEN
        la.global   $sp, {boot_stack}
        li.d        $t0, {boot_stack_size}
        add.d       $sp, $sp, $t0       # setup boot stack
        csrrd       $a0, 0x20           # cpuid
        la.global $t0, {entry}
        jirl $zero,$t0,0
        ",
        boot_stack_size = const TASK_STACK_SIZE,
        boot_stack = sym BOOT_STACK,
        entry = sym super::rust_entry,
        options(noreturn),
    )
}

```

首先设置DMW映射窗（可参看文档[龙芯架构参考手册卷一](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E9%BE%99%E8%8A%AF%E6%9E%B6%E6%9E%84%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%E5%8D%B7%E4%B8%80.pdf)第五章节），然后关闭中断，关闭浮点等相关功能。打开页表，设置栈指针sp，然后跳转到rust的环境中执行，此时执行权交给了rust。


```rust
#[no_mangle]
unsafe extern "C" fn rust_entry(cpu_id: usize, _dtb: usize) {
    crate::mem::clear_bss();
    crate::cpu::init_primary(cpu_id);
    crate::arch::set_trap_vector_base(trap_vector_base as usize);
    crate::arch::tlb_init(boot::KERNEL_PAGE_TABLE.as_ptr() as usize, handle_tlb_refill as usize);
    rust_main(cpu_id, 0);
}
```

做一些初始化工作，然后进入rust_main,此函数位于modules/axruntime/src/lib.rs。

```rust
pub extern "C" fn rust_main(cpu_id: usize, dtb: usize) -> ! {
    ax_println!("{}", LOGO);
    ax_println!(
        "\
        arch = {}\n\
        platform = {}\n\
        target = {}\n\
        smp = {}\n\
        build_mode = {}\n\
        log_level = {}\n\
        ",
        option_env!("AX_ARCH").unwrap_or(""),
        option_env!("AX_PLATFORM").unwrap_or(""),
        option_env!("AX_TARGET").unwrap_or(""),
        option_env!("AX_SMP").unwrap_or(""),
        option_env!("AX_MODE").unwrap_or(""),
        option_env!("AX_LOG").unwrap_or(""),
    );

    axlog::init();
    axlog::set_max_level(option_env!("AX_LOG").unwrap_or("")); // no effect if set `log-level-*` features
    info!("Logging is enabled.");
    info!("Primary CPU {} started, dtb = {:#x}.", cpu_id, dtb);

    info!("Found physcial memory regions:");
    for r in axhal::mem::memory_regions() {
        info!(
            "  [{:x?}, {:x?}) {} ({:?})",
            r.paddr,
            r.paddr + r.size,
            r.name,
            r.flags
        );
    }

    #[cfg(feature = "alloc")]
    init_allocator();

    #[cfg(all(feature = "paging", not(target_arch = "loongarch64")))]
    {
        info!("Initialize kernel page table...");
        remap_kernel_memory().expect("remap kernel memoy failed");
    }

    #[cfg(all(feature = "paging", target_arch = "loongarch64"))]
    {
        extern "C" { fn img_start();}
        trace!("IMG: {:p}", img_start as *const());
    }

    info!("Initialize platform devices...");
    axhal::platform_init();

    cfg_if::cfg_if! {
        if #[cfg(feature = "monolithic")] {
            info!("Initialize Kernel Process");
            axprocess::init_kernel_process();
        }
        else {
            #[cfg(feature = "multitask")]
            axtask::init_scheduler();
        }
    }
    #[cfg(any(feature = "fs", feature = "net", feature = "display"))]
    {
        #[allow(unused_variables)]
        let all_devices = axdriver::init_drivers();

        #[cfg(feature = "fs")]
        axfs::init_filesystems(all_devices.block);

        #[cfg(feature = "net")]
        axnet::init_network(all_devices.net);

        #[cfg(feature = "display")]
        axdisplay::init_display(all_devices.display);
    }

    #[cfg(feature = "smp")]
    self::mp::start_secondary_cpus(cpu_id);

    #[cfg(feature = "irq")]
    {
        info!("Initialize interrupt handlers...");
        init_interrupt();
    }

    #[cfg(all(feature = "tls", not(feature = "multitask")))]
    {
        info!("Initialize thread local storage...");
        init_tls();
    }

    info!("Primary CPU {} init OK.", cpu_id);
    INITED_CPUS.fetch_add(1, Ordering::Relaxed);

    while !is_init_ok() {
        core::hint::spin_loop();
    }

    unsafe { main() };

    #[cfg(feature = "multitask")]
    axtask::exit(0);
    #[cfg(not(feature = "multitask"))]
    {
        debug!("main task exited: exit_code={}", 0);
        axhal::misc::terminate();
    }
}

```

rust_main执行，执行各个module，完成初始化工作后，将执行权交给main函数。此时的main函数和执行的应用相关。

比如我执行的是apps/oscomp，则执行：
```rust
use syscall_entry::run_testcases;

#[no_mangle]
fn main() {
    run_testcases("sdcard");
}
```

接着交给了run_testcases。

```rust
pub fn run_testcases(case: &'static str) {
	...
loop {
        let mut ans = None;
        if let Some(command_line) = test_iter.next() {
            let args = get_args(command_line.as_bytes());
            let testcase = args.clone();
            let main_task = axprocess::Process::init(args).unwrap();
            let now_process_id = main_task.get_process_id() as isize;
            TESTRESULT.lock().load(&(testcase));
            let mut exit_code = 0;
            ans = loop {
                if wait_pid(now_process_id, &mut exit_code as *mut i32).is_ok() {
                    break Some(exit_code);
                }

                yield_now_task();
            };
        }
        TaskId::clear();
        
        #[cfg(not(target_arch = "loongarch64"))]
        {
            write_page_table_root(KERNEL_PAGE_TABLE.root_paddr());
        };
        
        flush_tlb(None);

        EXITED_TASKS.lock().clear();
        if let Some(exit_code) = ans {
            let kernel_process = Arc::clone(PID2PC.lock().get(&KERNEL_PROCESS_ID).unwrap());
            kernel_process
                .children
                .lock()
                .retain(|x| x.pid() == KERNEL_PROCESS_ID);
            // 去除指针引用，此时process_id对应的进程已经被释放
            // 释放所有非内核进程
            finish_one_test(exit_code);
        } else {
            // 已经测试完所有的测例
            TESTRESULT.lock().show_result();
            break;
        }
        // chdir会改变当前目录，需要重新设置
        init_current_dir();
    }
}

```

根据传入的命令，创建进程，执行程序，等待程序执行结束，返回结果。
```rust
axprocess::Process::init(args);
...
wait_pid(now_process_id, &mut exit_code as *mut i32).is_ok();
```


## 进程创建

具体代码在：modules/axprocess/src/process.rs

```rust
pub fn init(args: Vec<String>) -> AxResult<AxTaskRef> {
        let path = args[0].clone();
        let mut memory_set = MemorySet::new_with_kernel_mapped();
        let page_table_token = memory_set.page_table_token();
        if page_table_token != 0 {
            {
                write_page_table_root(page_table_token.into());
                let task = current();
                info!("Cur Task: {}", task.id_name());
                warn!("0x{:x}",page_table_token);
                // riscv::register::sstatus::set_sum();
            };
        }
        // 运行gcc程序时需要预先加载的环境变量
        let envs:Vec<String> = vec![
            "SHLVL=1".into(),
            "PATH=/usr/sbin:/usr/bin:/sbin:/bin".into(),
            "PWD=/".into(),
            "GCC_EXEC_PREFIX=/riscv64-linux-musl-native/bin/../lib/gcc/".into(),
            "COLLECT_GCC=./riscv64-linux-musl-native/bin/riscv64-linux-musl-gcc".into(),
            "COLLECT_LTO_WRAPPER=/riscv64-linux-musl-native/bin/../libexec/gcc/riscv64-linux-musl/11.2.1/lto-wrapper".into(),
            "COLLECT_GCC_OPTIONS='-march=rv64gc' '-mabi=lp64d' '-march=rv64imafdc' '-dumpdir' 'a.'".into(),
            "LIBRARY_PATH=/lib/".into(),
            "LD_LIBRARY_PATH=/lib/".into(),
        ];
        let (entry, user_stack_bottom, heap_bottom) =
            if let Ok(ans) = load_app(path.clone(), args, envs, &mut memory_set) {
                ans
            } else {
                error!("Failed to load app {}", path);
                return Err(AxError::NotFound);
            };
        let new_process = Arc::new(Self::new(
            TaskId::new().as_u64(),
            KERNEL_PROCESS_ID,
            Arc::new(Mutex::new(memory_set)),
            heap_bottom.as_usize() as u64,
            vec![
                // 标准输入
                Some(Arc::new(Stdin {
                    flags: Mutex::new(OpenFlags::empty()),
                })),
                // 标准输出
                Some(Arc::new(Stdout {
                    flags: Mutex::new(OpenFlags::empty()),
                })),
                // 标准错误
                Some(Arc::new(Stderr {
                    flags: Mutex::new(OpenFlags::empty()),
                })),
            ],
        ));
        let new_task = TaskInner::new(
            || {},
            path,
            axconfig::TASK_STACK_SIZE,
            new_process.pid(),
            page_table_token,
            #[cfg(feature = "signal")]
            false,
        );
        TID2TASK
            .lock()
            .insert(new_task.id().as_u64(), Arc::clone(&new_task));
        new_task.set_leader(true);

        info!("Process entry{:?}, stack{:?}", entry, user_stack_bottom);
        let new_trap_frame =
            TrapFrame::app_init_context(entry.as_usize(), user_stack_bottom.as_usize());
        new_task.set_trap_context(new_trap_frame);
        // 需要将完整内容写入到内核栈上，first_into_user并不会复制到内核栈上
        new_task.set_trap_in_kernel_stack();
        new_process.tasks.lock().push(Arc::clone(&new_task));
        #[cfg(feature = "signal")]
        new_process
            .signal_modules
            .lock()
            .insert(new_task.id().as_u64(), SignalModule::init_signal(None));
        new_process
            .robust_list
            .lock()
            .insert(new_task.id().as_u64(), FutexRobustList::default());
        PID2PC
            .lock()
            .insert(new_process.pid(), Arc::clone(&new_process));
        // 将其作为内核进程的子进程
        match PID2PC.lock().get(&KERNEL_PROCESS_ID) {
            Some(kernel_process) => {
                kernel_process.children.lock().push(new_process);
            }
            None => {
                return Err(AxError::NotFound);
            }
        }
        RUN_QUEUE.lock().add_task(Arc::clone(&new_task));
        Ok(new_task)
    }
}
```
创建一些进程所需要的资源，比如加载elf文件，进行解析，创建对应的Task，执行elf文件等等。

```rust
TrapFrame::app_init_context(entry.as_usize(), user_stack_bottom.as_usize());

impl TrapFrame {
    fn set_user_sp(&mut self, user_sp: usize) {
        self.regs[3] = user_sp;
    }
    /// 用于第一次进入应用程序时的初始化
    pub fn app_init_context(app_entry: usize, user_sp: usize) -> Self {
        let mut trap_frame = TrapFrame::default();
        trap_frame.set_user_sp(user_sp);
        trap_frame.era = app_entry;
        trap_frame.prmd = 3 | 1<<2; // user and enable int
        trap_frame
    }
}

```

将应用程序elf的入口地址设置到era中，并且设置好sp指针。

将进程创建好后，会将Task加入到队列，然后等待调度执行。


## 用户态与内核态的交互

异常的入口函数为trap_vector_base:

```assembly
trap_vector_base:
    csrwr   $t0, KSAVE_T0
    csrrd   $t0, 0x1
    andi    $t0, $t0, 0x3
    bnez    $t0, .Lfrom_userspace 

.Lfrom_kernel:
    move    $t0, $sp  
    addi.d  $sp, $sp, -{trapframe_size} // allocate space
    // save kernel sp
    st.d    $t0, $sp, 3*8
    b .Lcommon 

.Lfrom_userspace:       
    csrwr   $sp, KSAVE_USP                   // save user sp into SAVE1 CSR
    csrrd   $sp, KSAVE_KSP                   // restore kernel sp
    addi.d  $sp, $sp, -{trapframe_size}      // allocate space
    // switch tp and r21
    st.d    $tp,  $sp, 2*8
    ld.d    $tp,  $sp, 36*8
    st.d    $r21, $sp, 21*8
    ld.d    $r21, $sp, 37*8
    // save user sp
    csrrd   $t0, KSAVE_USP
    st.d    $t0, $sp, 3*8 // sp
     
.Lcommon:
    // save the registers.
    SAVE_REGS
    csrrd	$t2, 0x1
    st.d	$t2, $sp, 8*32  // prmd
    csrrd   $t1, 0x6        
    st.d    $t1, $sp, 8*33  // era
    csrrd   $t1, 0x7   
    st.d    $t1, $sp, 8*34  // badv  
    csrrd   $t1, 0x0   
    st.d    $t1, $sp, 8*35  // crmd    
    
    move    $a0, $sp
    csrrd   $t0, 0x1
    andi    $a1, $t0, 0x3   // if user or kernel
    bl      loongarch64_trap_handler

    // restore the registers.
    ld.d    $t1, $sp, 8*33  // era
    csrwr   $t1, 0x6
    ld.d    $t2, $sp, 8*32  // prmd
    csrwr   $t2, 0x1
    // Save kernel sp when exit kernel mode
    addi.d  $t1, $sp, {trapframe_size}
    csrwr   $t1, KSAVE_KSP 
    RESTORE_REGS
    // restore sp
    ld.d    $sp, $sp, 3*8
    ertn
```
首先保存用户态的寄存器，然后切换到内核栈，然后执行loongarch64_trap_handler：

```rust

fn loongarch64_trap_handler(tf: &mut TrapFrame, from_user: bool) {
    ... 
    match estat.cause() {
        Trap::Exception(Exception::Breakpoint) => handle_breakpoint(&mut tf.era),
        Trap::Exception(Exception::AddressNotAligned) => handle_unaligned(tf),
        Trap::Interrupt(_) => {
            let irq_num: usize = estat.is().trailing_zeros() as usize;
            crate::trap::handle_irq_extern(irq_num)
        }
        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::Syscall) => {
            enable_irqs();
            // jump to next instruction anyway
            tf.era += 4;
            let result = handle_syscall(
                tf.regs[11],
                [
                    tf.regs[4], tf.regs[5], tf.regs[6], tf.regs[7], tf.regs[8], tf.regs[9],
                ],
            );
            // cx is changed during sys_exec, so we have to call it again
            tf.regs[4] = result as usize;
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::FetchPageFault) => {
            let flags = MappingFlags::USER | MappingFlags::EXECUTE;
            handle_page_fault(addr.into(), flags, tf);
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::LoadPageFault) => {
            let flags = if from_user { MappingFlags::USER | MappingFlags::READ } else { MappingFlags::READ };
            handle_page_fault(addr.into(), flags, tf);
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::StorePageFault) => {
            let addr = tf.badv;
            let flags = MappingFlags::USER | MappingFlags::WRITE;
            handle_page_fault(addr.into(), flags, tf);
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::PageModifyFault) => {
            let addr = tf.badv;
            let flags = MappingFlags::USER | MappingFlags::WRITE | MappingFlags::DIRTY;
            handle_page_fault(addr.into(), flags, tf);
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::PagePrivilegeIllegal) => {
            let flags = MappingFlags::USER;
            handle_page_fault(addr.into(), flags, tf);
        }

        #[cfg(feature = "monolithic")]
        Trap::Exception(Exception::InstructionNotExist) => {

        }

        _ => {
            panic!(
                "Unhandled trap {:?} @ {:#x}:\n{:#x?}",
                estat.cause(),
                tf.era,
                tf
            );
        }
    }

    #[cfg(feature = "signal")]
    if from_user == true {
        handle_signal();
    }

    #[cfg(feature = "monolithic")]
    // 在保证将寄存器都存储好之后，再开启中断
    // 否则此时会因为写入csr寄存器过程中出现中断，导致出现异常
    disable_irqs();
}
```
然后执行相关的异常处理函数。

## Syscall执行流程

首先进入异常处理流程:
```rust
Trap::Exception(Exception::Syscall) => {
            enable_irqs();
            // jump to next instruction anyway
            tf.era += 4;
            let result = handle_syscall(
                tf.regs[11],
                [
                    tf.regs[4], tf.regs[5], tf.regs[6], tf.regs[7], tf.regs[8], tf.regs[9],
                ],
            );
            // cx is changed during sys_exec, so we have to call it again
            tf.regs[4] = result as usize;
        }

```
接着进入ulib/axstarry/syscall_entry/src/syscall.rs

```rust
fn handle_syscall(syscall_id: usize, args: [usize; 6]) -> isize {
        axprocess::time_stat_from_user_to_kernel();
        let ans = syscall(syscall_id, args);
        axprocess::time_stat_from_kernel_to_user();
        ans
}

pub fn syscall(syscall_id: usize, args: [usize; 6]) -> isize {
    #[cfg(feature = "futex")]
    syscall_task::check_dead_wait();
    let ans = loop {
        #[cfg(feature = "syscall_net")]
        {
            if let Ok(net_syscall_id) = syscall_net::NetSyscallId::try_from(syscall_id) {
                break syscall_net::net_syscall(net_syscall_id, args);
            }
        }

        #[cfg(feature = "syscall_mem")]
        {
            if let Ok(mem_syscall_id) = syscall_mem::MemSyscallId::try_from(syscall_id) {
                break syscall_mem::mem_syscall(mem_syscall_id, args);
            }
        }

        #[cfg(feature = "syscall_fs")]
        {
            if let Ok(fs_syscall_id) = syscall_fs::FsSyscallId::try_from(syscall_id) {
                break syscall_fs::fs_syscall(fs_syscall_id, args);
            }
        }

        {
            if let Ok(task_syscall_id) = syscall_task::TaskSyscallId::try_from(syscall_id) {
                break syscall_task::task_syscall(task_syscall_id, args);
            }
        }

        panic!("unknown syscall id: {}", syscall_id);
    };

    let ans = deal_result(ans);
    ans
}

```
此时，会根据传入的进程号，选择相应的处理流程：

- 1. syscall_net::net_syscall
- 2. syscall_mem::mem_syscall
- 3. syscall_fs::fs_syscall
- 4. syscall_task::task_syscall

例如执行clone syscall：

```rust
CLONE => syscall_clone(args[0], args[1], args[2], args[4], args[3]),

pub fn syscall_clone(
    flags: usize,
    user_stack: usize,
    ptid: usize,
    tls: usize,
    ctid: usize,
) -> SyscallResult {
    let clone_flags = CloneFlags::from_bits((flags & !0x3f) as u32).unwrap();

    let stack = if user_stack == 0 {
        None
    } else {
        Some(user_stack)
    };
    let curr_process = current_process();
    #[cfg(feature = "signal")]
    let sig_child = SignalNo::from(flags as usize & 0x3f) == SignalNo::SIGCHLD;

    info!("syscall_clone");

    if let Ok(new_task_id) = curr_process.clone_task(
        clone_flags,
        stack,
        ptid,
        tls,
        ctid,
        #[cfg(feature = "signal")]
        sig_child,
    ) {
        Ok(new_task_id as isize)
    } else {
        return Err(SyscallError::ENOMEM);
    }
}

```
执行完进程或者线程的clone之后，然后返回，将执行结果保存在中`TrapFrame` `tf.regs[4] = result as usize`。

此时系统调用完成，返回到trap_vector_base中：

```c
    // restore the registers.
    ld.d    $t1, $sp, 8*33  // era
    csrwr   $t1, 0x6
    ld.d    $t2, $sp, 8*32  // prmd
    csrwr   $t2, 0x1

    // Save kernel sp when exit kernel mode
    addi.d  $t1, $sp, {trapframe_size}
    csrwr   $t1, KSAVE_KSP 

    RESTORE_REGS

    // restore sp
    ld.d    $sp, $sp, 3*8
    ertn
```
将用户态的寄存器恢复后，保护执行环境栈sp，然后通过指令`ertn`返回到用户态继续执行。





