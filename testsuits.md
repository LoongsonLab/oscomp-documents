
# 比赛使用的测试集

此部分主要演示比赛的测试集的编译与简单的运行。

### 获取测试集

```shell 
git clone https://github.com/LoongsonLab/oscomp-testsuits-loongarch.git

cd oscomp-testsuits-loongarch

git submodule update --init --recursive

```

### 目录结构
目录如下所示：
```
├── busybox
├── config                  （一些配置文件，用于busybox）
├── copy-file-range-test
├── interrupts-test
├── iperf
├── libc-bench
├── libc-test
├── linux-test-project      （ltp的测试子目录，用于兼容linux测试）
├── lmbench
├── lua
├── Makefile
├── netperf
├── prebuilt                 (交叉编译工具链)
├── README.md
├── rt-tests
├── runtime
├── scripts
├── time-test
├── true
└── UnixBench
```

### 如何编译执行

#### 1. 获取并且安装交叉编译器

```shell
cd /prebuilt
tar zxf gcc-8.3.0-loongarch64-linux-gnu-rc1.1.novec.tgz
```

#### 2. 编译测试集
```shell
# 安装好测试集后，进入oscomp-testsuits-loongarch
make all 
```

所有的基本相关的测试（除ltp）可执行文件会拷贝进sdcard目录。

然后将sdcard拷贝到Starry-LoongArch/testcases/下：

```shell 
cp -r oscomp-testsuits-loongarch/ramdisk Starry-LoongArch/testcases/
```

需要说明的是，我们在Makefile中，我们定义了`GNU_CROSS_COMPILER_GCC`和`MUSL_CROSS_COMPILER_GCC`变量，
指的是，在编译时选用glibc相关环境还是musl libc相关环境。


#### 3. 运行测试集
```shell
cd Starry-LoongArch

./build_img.sh sdcard # 将生成的测试集打包进disk.img

# 运行内核
make run ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp
```


### 需要修改kernel源码的部分

上述步骤会把所有的测试集打包，也可以单独的编译，具体可参看[`Makefile`](https://github.com/LoongsonLab/oscomp-testsuits-loongarch/blob/main/Makefile)

在执行apps/oscomp的main的时候，会调用run_testcases函数，如下：

``` rust
pub fn run_testcases(case: &'static str) {
    fs_init(case);
    let (mut test_iter, case_len) = match case {
        "junior" => (Box::new(JUNIOR_TESTCASES.iter()), JUNIOR_TESTCASES.len()),
        "libc-static" => (
            Box::new(LIBC_STATIC_TESTCASES.iter()),
            LIBC_STATIC_TESTCASES.len(),
        ),
        "libc-dynamic" => (
            Box::new(LIBC_DYNAMIC_TESTCASES.iter()),
            LIBC_DYNAMIC_TESTCASES.len(),
        ),
        "lua" => (Box::new(LUA_TESTCASES.iter()), LUA_TESTCASES.len()),
        "netperf" => (Box::new(NETPERF_TESTCASES.iter()), NETPERF_TESTCASES.len()),

        "ipref" => (Box::new(IPERF_TESTCASES.iter()), IPERF_TESTCASES.len()),

        "sdcard" => (Box::new(SDCARD_TESTCASES.iter()), SDCARD_TESTCASES.len()),

        "ostrain" => (Box::new(OSTRAIN_TESTCASES.iter()), OSTRAIN_TESTCASES.len()),
        _ => {
            panic!("unknown test case: {}", case);
        }
    };
    ... 
}
```

传入不同的参数，代表运行不同的测试样例。在函数中比较灵活的设置。

比如，如果传入的参数为"sdcard", 则会执行`SDCARD_TESTCASES`相关的测试用例：

```rust
pub const SDCARD_TESTCASES: &[&str] = &[
    "./hello",
    "./main",
    "busybox echo hello",
    // "busybox sh test_hello.sh",
    // "busybox sh busybox_testcode.sh",
    "busybox ls",
    "busybox sh",
    // "./lmbench_all",
    // "./lmbench_all lat_syscall -P 1 null",
    // "./libc-bench",
    // "sh",
    // "busybox sh lua_testcode.sh",
    // "./riscv64-linux-musl-native/bin/riscv64-linux-musl-gcc ./hello.c -static",
    // "./a.out",
    // "./time-test",
    // "./interrupts-test-1",
    // "./interrupts-test-2",
    // "./copy-file-range-test-1",
    // "./copy-file-range-test-2",
    // "./copy-file-range-test-3",
    // "./copy-file-range-test-4",
    // "busybox echo hello",
    // "busybox sh ./unixbench_testcode.sh",
    // "busybox echo hello",
    // "busybox sh ./iperf_testcode.sh",
    // "busybox echo hello",
    // "busybox sh busybox_testcode.sh",
    // "busybox echo hello",
    // "busybox sh ./iozone_testcode.sh",
    // "busybox echo latency measurements",
    // "lmbench_all lat_syscall -P 1 null",
    // "lmbench_all lat_syscall -P 1 read",
    // "lmbench_all lat_syscall -P 1 write",
    // "busybox mkdir -p /var/tmp",
    // "busybox touch /var/tmp/lmbench",
    // "lmbench_all lat_syscall -P 1 stat /var/tmp/lmbench",
    // "lmbench_all lat_syscall -P 1 fstat /var/tmp/lmbench",
    // "lmbench_all lat_syscall -P 1 open /var/tmp/lmbench",
    // "lmbench_all lat_select -n 100 -P 1 file",
    // "lmbench_all lat_sig -P 1 install",
    // "lmbench_all lat_sig -P 1 catch",
    // "lmbench_all lat_sig -P 1 prot lat_sig",
    // "lmbench_all lat_pipe -P 1",
    // "lmbench_all lat_proc -P 1 fork",
    // "lmbench_all lat_proc -P 1 exec",
    // "busybox cp hello /tmp",
    // "lmbench_all lat_proc -P 1 shell",
    // "lmbench_all lmdd label=\"File /var/tmp/XXX write bandwidth:\" of=/var/tmp/XXX move=1m fsync=1 print=3",
    // "lmbench_all lat_pagefault -P 1 /var/tmp/XXX",
    // "lmbench_all lat_mmap -P 1 512k /var/tmp/XXX",
    // "busybox echo file system latency",
    // "lmbench_all lat_fs /var/tmp",
    // "busybox echo Bandwidth measurements",
    // "lmbench_all bw_pipe -P 1",
    // "lmbench_all bw_file_rd -P 1 512k io_only /var/tmp/XXX",
    // "lmbench_all bw_file_rd -P 1 512k open2close /var/tmp/XXX",
    // "lmbench_all bw_mmap_rd -P 1 512k mmap_only /var/tmp/XXX",
    // "lmbench_all bw_mmap_rd -P 1 512k open2close /var/tmp/XXX",
    // "busybox echo context switch overhead",
    // "lmbench_all lat_ctx -P 1 -s 32 2 4 8 16 24 32 64 96",
    // "busybox sh libctest_testcode.sh",
    // "busybox sh lua_testcode.sh",
    // "libc-bench",
    // "busybox sh ./netperf_testcode.sh",
    // "busybox sh ./cyclictest_testcode.sh",
];
```

需要注意的是，我们在初始化环境的时候，会设置一些变量，如下所示：

```rust

pub fn fs_init(_case: &'static str) {
    // 需要对libc-dynamic进行特殊处理，因为它需要先加载libc.so
    // 建立一个硬链接

    // let libc_so  = &"ld-musl-loongarch64-sf.so.1";
    // let libc_so2 = &"ld-musl-loongarch64.so.1"; // 另一种名字的 libc.so，非 libc-test 测例库用
    let libc_so = &"ld-linux-loongarch-lp64d.so.1";
    let libc_so1 = &"ld.so.1";

    create_link(
        &(FilePath::new(("/lib64/".to_string() + libc_so).as_str()).unwrap()),
        &(FilePath::new("ld.so").unwrap()),
    );

    create_link(
        &(FilePath::new(("/lib64/".to_string() + libc_so1).as_str()).unwrap()),
        &(FilePath::new("ld.so").unwrap()),
    );
    create_link(
        &(FilePath::new("/lib/libc.so.6").unwrap()),
        &(FilePath::new("libc.so").unwrap()),
    );
    create_link(
        &(FilePath::new("/sbin/busybox").unwrap()),
        &(FilePath::new("busybox").unwrap()),
    );
    create_link(
        &(FilePath::new("/usr/sbin/busybox").unwrap()),
        &(FilePath::new("busybox").unwrap()),
    );
    create_link(
        &(FilePath::new("/usr/sbin/ls").unwrap()),
        &(FilePath::new("busybox").unwrap()),
    );
    create_link(
        &(FilePath::new("/usr/sbin/main").unwrap()),
        &(FilePath::new("main").unwrap()),
    );
    create_link(
        &(FilePath::new("/usr/sbin/hello").unwrap()),
        &(FilePath::new("hello").unwrap()),
    );
    create_link(
        &(FilePath::new("/ls").unwrap()),
        &(FilePath::new("/busybox").unwrap()),
    );
    create_link(
        &(FilePath::new("/sh").unwrap()),
        &(FilePath::new("/busybox").unwrap()),
    );
    create_link(
        &(FilePath::new("/bin/lmbench_all").unwrap()),
        &(FilePath::new("/lmbench_all").unwrap()),
    );
    create_link(
        &(FilePath::new("/bin/iozone").unwrap()),
        &(FilePath::new("/iozone").unwrap()),
    );
    let _ = new_file("/lat_sig", &(FileFlags::CREATE | FileFlags::RDWR));
}
```

由于我们使用的是fatfs，它不支持软链接，因此在执行之前，需要在代码中手动的创建链接。

如果在执行过程中，产生了文件不存在，或者"not found"的错误，则可以考虑此处是否
增加了相应的链接。
