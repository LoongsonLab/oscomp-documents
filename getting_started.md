
# 如何运行
这部分介绍如何现在获得源码，编译，指定运行参数等。生成IMG镜像，可以在QEMU模拟器上运行。


### 编译工具

#### 1. 获得 Rust 相关工具

``` shell 
# 获得rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 获得rust所支持的所有架构相关
rustc --print target-list

# 增加LoongArch工具
rustup target add loongarch64-unknown-none

# 增加 cargo-binutils （objcopy, objdump）
cargo install cargo-binutils
rustup component add llvm-tools
#


```

#### 2. 获取GNU GCC相关工具

```shell 
git clone https://github.com/LoongsonLab/oscomp-toolchains-for-oskernel.git

tar zxf gcc-8.3.0-loongarch64-linux-gnu-rc1.1.novec.tgz

#增加环境变量
export PATH=${PATH}:/your-gcc-dir/bin

#测试工具
loongarch64-linux-gnu-gcc -v

```

#### 3. 安装QEMU和binutils-gdb

```shell
# 在下载的oscomp-toolchains-for-oskernel中
tar zxf loongarch64-linux-gnu-gdb.tgz
tar zxf qemu.tgz

# 安装qemu所需要的依赖
sudo apt install libfdt-dev libcapstone-dev libspice-server-dev libsdl2-dev libusbredirparser1 libiscsi7 libaio1

# 增加环境变量
export PATH=${PATH}:/your-dir/bin

# 测试是否安装正确
loongarch64-linux-gnu-gdb -v
qemu-system-loongarch64 -M ?

```

### 代码获取
直接从LoongLAB的github上获取源码。

```shell
git clone https://github.com/LoongsonLab/StarryOS-LoongArch.git
```

获取的源码切换到main分支

### 制作镜像
```shell
cd StarryOS-LoongArch

./build_img.sh sdcard
#会在当前目录下生成 disk.img 文件

#可以查看文件类型
file disk.img

```
`./build_img.sh sdcard`会将testcases/sdcard的内容打包进disk.img

### 运行StarryOS

```shell
# 当前目录为StarryOS-LoongArch
# 编译内核
make ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp

# 运行内核
make run ARCH=loongarch64 TARGET=loongarch64-qemu-virt A=apps/oscomp
```

其中 ARCH指定架构类型。target指定平台，目前有两个平台支持，loongarch64-qemu-virt为Qemu模拟器平台，而loongarch64-2k1000是为2k1000开发板的配置。
A指定的是运行的哪一个应用，在目录apps/下有诸多目录，其中oscomp为比赛使用的应用，是入口main函数。

运行情况如下：

```shell
qemu-system-loongarch64 -m 16G -smp 1 -machine virt -kernel apps/oscomp/oscomp_loongarch64-qemu-virt.elf -nographic -vga none

       d8888                            .d88888b.   .d8888b.
      d88888                           d88P" "Y88b d88P  Y88b
     d88P888                           888     888 Y88b.
    d88P 888 888d888  .d8888b  .d88b.  888     888  "Y888b.
   d88P  888 888P"   d88P"    d8P  Y8b 888     888     "Y88b.
  d88P   888 888     888      88888888 888     888       "888
 d8888888888 888     Y88b.    Y8b.     Y88b. .d88P Y88b  d88P
d88P     888 888      "Y8888P  "Y8888   "Y88888P"   "Y8888P"

arch = loongarch64
platform = loongarch64-qemu-virt
target = loongarch64-unknown-none
smp = 1
build_mode = debug
log_level = warn

[  0.460358 0 fatfs::dir:140] Is a directory
[  0.474467 0 fatfs::dir:140] Is a directory
[  0.485735 0 fatfs::dir:140] Is a directory
[  2.482038 0:3 syscall_entry::test:428]  --------------- load testcase: ["busybox", "sh"] --------------- 
/ # ls
[ 15.300087 0:8 fatfs::dir:140] Is a directory
[ 15.307231 0:8 fatfs::dir:140] Is a directory
[ 15.311956 0:8 fatfs::dir:140] Is a directory
busybox  dev      lat_sig  libc.so  main.S   sys      tmp
c.S      hello    ld.so    main     proc     thread   var
/ # 
```
