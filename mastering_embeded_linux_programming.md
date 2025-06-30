# embedded-linux-slides
这个培训教材还是挺好的。

## 目录，
第一部分，嵌入式linux的组成
第一章：开始
第二章：了解工具链
第三章：关于Bootloader
第四章：配置与构建内核
第五章：构建根文件系统
第六章：构建系统选择
第七章：使用Yocto开发
第八章：Yocto 深入探讨

第二部分，系统架构与设计决策
第三部分，编写嵌入式应用程序
第四部分，调试与性能优化

### 第一章
嵌入式linux的四个组成元素，都从获取定制和部署这四部分组成。---工具链，bootloader，内核，根文件系统。


#### 为嵌入式linux选硬件， 
   根据架构来看，    我目前有2个板子，树莓派pico，这个跑zephyr，另外一个是飞凌的，跑linux，还有一个qemu作为补充

- 先选择架构，有arm，risc-v，
- ram，最小是16M。        i.mx6DualLite，是1GHZ的cortex-A9，属于ARMv7架构 ，但是720m的ram？
- 非易失性存储，通常是闪存，8M以上。（而其实非易失性存储包括闪存，硬盘驱动器HDD，ROM）Linux 广泛支持闪存设备，包括原始NOR和NAND闪存芯片，以及SD卡、eMMC芯片、USB闪存等形式的托管闪存。
- 串口，最好是基于uart的
- 加载软件的方法。jtag（联合测试行动组）。还有些启动方式是sd卡和串口方式加载引导代码。

我选了2块，一块是买的，飞凌的板子，OKMX6DL-C的ram技术规格
  i.mx6DualLite  1Ghz的2核cortedx-a9（ARMv7）的64位soc
1Gb的DDR3 RAM                 ---很奇怪为什么大小free是600多m
8G的板载闪存emmc
支持wifi，蓝牙，不支持ble
    5路UART（包含一个调试串口）
    3路I2C
    1路SPI
有usb
网络：1路10/100/1000M自适应以太网接口     
3个SD/MMC接口
显示：

    1路RGB
    2路单8位LVDS
    1路HDMI
    1路MIPI

音频：

    1路Phone
    1路Mic
    2路Speaker

还有qemu的使用，这玩意也可以加强进行使用。居然还可以配网络。

### 第二章节，工具链

#### 工具链
为我的处理器使用最佳的指令集来利用我的硬件，支持所需要的语言，具有系统接口实现。
工具连，可以下载和安装，也可以从源代码构建。   使用crosstool-NG，来进行对工具链更清楚的理解。
这玩意，就是将源码编译到目标设备上运行可执行文件的工具，包括编译器，链接器和运行时库。
目前GUN是最成熟的工具链，Clang和llvm正在加大应用。
GUN包括：Binutils、GNU Compiler Collection、c库三部分。

在编译c库的时候，还需要linux内核头文件的副本。

工具链需要根据目标的cpu能力进行构建，包括，cpu架构，大小端，浮点数支持，应用程序二进制接口ABI。
ABI也分好些个标准，OABI，EABI，EABIHF（最新的）。这个工具链是有相应的命名的，CPU-供应商-内核-操作系统

就是我之前使用的哪些工具链，比如arm-none-eabi

c库也包括很多，常见的是glibc
工具链获取，1是现成的，2是嵌入式系统生成的，3是自己创建的。

crosstool-NG，一套开发交叉工具链的工具，也可以用buildroot和yocto进行构建。























工程服务
引导程序/固件开发
Linux内核移植和驱动程序开发
Linux BSP的开发、维护和升级
嵌入式Linux构建系统
嵌入式Linux集成
开源的上游


课程：
嵌入式linux系统开发; linux内核驱动开发；yocto 项目系统开发；buildroot系统开发；linux的图形栈，gui啥的吧；
嵌入式linux启动时间优化；实时linux补丁什么的；linux的调试追踪性能分析。


什么是嵌入式linux？
嵌入式系统中使用linux内核和各种开源组件。

系统建立在自己的基础上，内核、编译器、c库、基本的程序

嵌入式linux的硬件：
大多数架构都支持，x86，arm，risc-v
mmu and no-mmu 都支持，后者会有点限制

![](vx_images/592487753068248.png)

嵌入式linux 的一个架构。这也把linux 的内核给看出来长啥样子。

bsp：包含引导加载器和内核，为目标硬件提供合适的驱动。

## 交叉编译器，是什么？怎么获取
binutils 、内核头、c/cpp库与编译器、gdb

包含 binutils（二进制工具），给定的CPU架构生成和操作二进制文件（通常使用ELF格式）•as，汇编器，从汇编器源代码生成二进制代码•ld，链接器•ar， ranlib，生成.a archive（静态库）•objdump, readelf, size, nm, strings，

c库和编译后的程序也需要与内核交互，因此需要包含相应的头文件。

内核更新具有向下兼容。

c库：glibc, uClibc, musl, klibc, newlib

Linux工具链
包括一个Linux就绪的C标准库，它使用Linux系统调用来实现系统服务•可用于构建Linux用户空间应用程序，但也裸机代码（固件，引导加载程序，Linux内核）•由工具链元组中的Linux操作系统标识符标识：
arm-linux， arm-none-linux-gnueabihf
▶裸机工具链•是一个不包括C标准库的工具链，或者一个非常小的不绑定到特定操作系统的工具链
可用于构建裸机代码（固件，引导加载程序，Linux内核）
由工具链元组中的无操作系统标识符标识：arm-none-eabi， arm-none-none-eabi（供应商为无，操作系统为无）



大多数嵌入式Linux项目使用基于GNU项目的工具链：GCC编译器，binutils， 


cpu优化参数：
GNU工具（gcc, binutils）一次只能编译一个特定的目标架构（ARM, x86, RISC-V…）▶gcc提供了更多的选项：•-march允许选择一个特定的目标指令集•-mtune允许为特定的CPU优化代码•例如：
-march=armv7 -mtune=cortex-a8•-mcpu=cortex-a8可以用来允许gcc推断目标指令集（-march=armv7）和cpu优化（-mtune=cortex-a8）•https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html▶在GNU工具链编译时，可以选择值。
它们的用途：•作为交叉编译工具的默认值，当没有传递其他-march， -mtune， -mcpu选项时•编译C库▶注意：LLVM （Clang, LLD…）实用程序一次支持多个目标体系结构



手工构建一个交叉编译工具链是一个相当困难的过程▶
典型的过程是:
•构建依赖binutils / gcc (GMP, MPFR ISL等)
•构建binutils•
建立baremetal,第一阶段gcc•
从Linux源代码中提取内核头文件•
构建C库使用gcc•
第一阶段构建第二阶段和最终gcc支持Linux操作系统和C库

已经编译好的工具链：
•优点：它是最简单和最方便的解决方案•缺点：你不能微调工具链以满足你的需求
▶确保你找到的工具链满足你的要求：CPU，端序，C库，组件版本，内核头文件的版本，ABI，软浮动或硬浮动等
▶一些可能性：•由你的发行版打包的工具链，例如Ubuntu包gcc-arm-linux-gnueabihf或Fedora gcc-arm-linux-gnu。
通常仅限于使用glibc的ARM/ARM64。
•Bootlin的GNU工具链，大多数CPU架构，与glibc/uClibc/musl， https://toolchains.bootlin.com•ARM和ARM64工具链由ARM发布


此外还有很多构建 的项目可以寻找，比如yocto、openEmbedded



## bootloaders and firmware

引导加载程序是一段代码，负责：
•基本硬件初始化•
从闪存、网络或其他类型的非易失性存储加载应用程序二进制文件，通常是操作系统内核。
•可能解压缩应用程序二进制文件•
执行应用程序
▶除了这些基本功能外，大多数引导加载程序提供shell或菜单
•菜单选择要加载的操作系统
•shell带有命令从存储或网络加载数据，检查内存，执行硬件测试/诊断
▶由处理器运行的第一块代码可以由我们的开发人员修改。


引导程序例子： legacy BIOS；UEFI booting
x86上的，legacy BIOS，
UEFI ，x86上的一个新的

在嵌入式平台启动：ROM code
大多数嵌入式处理器包含一个ROM代码实现引导过程的初始步骤
▶ROM代码写的处理器供应商和直接内置处理器•
•不能更改或更新处理器数据表中描述其行为是▶负责寻找合适的引导装载程序,加载并运行它
•从NAND flash,也不是从USB,从SD卡,从eMMC等等。
•定义良好的位置/格式▶通常与外部RAM运行没有初始化,
•引导加载程序的大小有限，由于SRAM的大小
•强制引导过程分为两个步骤：第一阶段引导加载程序（小，从SRAM运行，初始化外部DRAM），第二阶段引导加载程序（较大，从外部DRAM运行）


bootloaders

U-boot，嵌入式体系结构中事实上的标准和最广泛使用的引导加载程序：ARM、ARM64、RISC-V、PowerPC、MIPS等。
▶也支持x86与UEFI固件。

类似的有 Barebox


▶Linux类RISC-V处理器有几个特权级别•m模式：机器模式•s模式：Linux内核运行的级别•u模式：
一些特定的硬件资源在s模式下无法访问▶一个更特权的固件在m模式下运行▶RISC-V定义了SBI， Supervisor Binary Interface•操作系统和固件之间的标准化接口•https://github.com/riscv-non-isa/riscv-sbi-doc▶OpenSBI是SBI的一个参考，开源实现https://github.com/riscv-software-src/opensbi

Uboot中的设备树

▶设备树是一种描述硬件拓扑的数据结构▶允许软件知道哪些硬件外设可用以及它们如何连接到系统▶最初主要用于Linux，现在也用于U-Boot， Barebox， TF-A等▶U-Boot在大多数平台上使用。
▶设备树文件位于arch/ arch/ dts为每个板一个.dts：需要创建一个，如果你建立一个自定义板▶U-Boot默认配置通常指定一个默认的设备树，但它可以使用DEVICE_TREE变量被覆盖▶在本课程后面的设备树的更多详细信息。


TF-A（Trusted Firmware-A）是一个开源的软件框架，用于实现安全、可信的启动和运行环境，特别是在ARM架构的系统中

## linux内核

概念掠过。


Linux通过伪文件系统（有时也称为虚拟文件系统）在用户空间中提供系统和内核信息。伪文件系统允许应用程序看到不存在于任何真实存储上的目录和文件：它们是由内核动态创建和更新的。两个最重要的伪文件系统是：
proc，通常挂载在/proc上；操作系统相关信息（进程，内存管理参数…）
sysfs,挂载在/sys；
将系统表示为通过总线连接的设备的树状结构。
管理这些设备的内核框架收集的信息。


linux的版本控制和发展

说白了得LTS

Linux v5.18源代码：接近80k文件，35M行，1.3GiB▶但一个压缩的Linux内核大小只有几兆字节。
▶那么，为什么这些资源如此之大？
因为它们包括许多设备驱动程序、网络协议、体系结构、文件系统……
内核非常小！
▶内核版本v5.18（占总行数的百分比）：

因此在构建内核的时候，可以有选择性地进行构建

### 内核构建和编译

1、确定目标架构

2、选择编译器

传参
3、初始化

内核映像是一个单一的文件，由所有与配置中启用的功能相对应的目标文件的链接产生。•这是由引导加载程序加载到内存中的文件。•因此，一旦内核启动，在没有文件系统存在的时候，所有内置功能都可用。▶一些功能(设备驱动程序，文件系统，
这些插件可以动态加载/卸载以添加/删除内核特性。每个模块作为文件系统中的单独文件存储，因此使用模块访问文件系统是强制性的。在内核的早期引导过程中，这是不可能的，因为没有可用的文件系统

因此内核配置的参数文件，kconfig。
有不同类型的选项,Kconfig文件中定义:
▶bool选项,他们要么•真实(包括内核中的功能)或•假(排除功能从内核)
▶三态选项,他们要么•真实(包括内核映像的特征)或•模块(包括功能作为一个内核模块)或•假(排除功能)
▶int选项,指定整数▶十六进制值选项,指定十六进制值的例子:
CONFIG_PAGE_OFFSET=0xC0000000
▶string选项，用于指定字符串值
示例：CONFIG_LOCALVERSION=-no-network，用于区分由不同选项构建的两个内核

启用网络驱动程序需要启用网络堆栈，因此配置符号有两种方式来表达依赖关系：
▶依赖于依赖：配置B依赖于a•B是不可见的，直到a被启用
•适用于依赖链▶选择依赖：配置A选择B•当A被启用时，B也被启用（并且不能手动禁用）

•最好不要选择依赖依赖的符号•用于声明硬件特性或选择库

这块还得实践才知道是什么回事。



### 编译和安装内核


本机示例
嵌入式系统示例


### 启动内核

内核硬件描述，嵌入式系统的是 设备树、

自定义板子的设备树

内核开发人员编写设备树源（DTS），一旦编译就变成设备树blob （DTB）。
▶内核支持的每个板/平台都有一个不同的设备树，可在arch//boot/dts//中找到。Linux 6.5之前的ARM 32)。
▶作为董事会用户，您可能有合法的需求来定制您的董事会设备树：•描述连接到不可发现的总线的外部设备并配置它们。
•配置引脚复用：选择主板外部连接器上可用的SoC信号。
请参阅http://linux.tanzilli.com/了解以交互方式执行此操作的web服务。
•配置一些系统参数：flash分区，内核命令行（其他方式存在）


使用uboot启动
除了编译时配置，内核行为可以调整不重新编译使用内核命令行
console=ttyS0 root=/dev/mmcblk0p2 rootwait最重要的参数记录在内核文档中的admin-guide/kernel-parameters中。


还有内核log

配置Linux内核并为嵌入式硬件平台进行交叉编译。
▶通过U-boot的TFTP客户端在主板上下载内核。
▶启动内核。
▶自动化内核启动过程与U-Boot脚本。

## linux文件系统
linux的文件系统是有多个的，比如df -h



文件系统挂载在目录树，使用mount

内核初始状态的时候，怎么安装？
以下多种方法
一个特定的文件系统被挂载在层次结构的根目录，由/▶标识，这个文件系统被称为根文件系统▶因为挂载和卸载是程序，它们是文件系统中的文件。
在挂载至少一个文件系统之前，它们是不可访问的。
▶由于根文件系统是第一个挂载的文件系统，它不能用正常的挂载命令挂载▶它是由内核直接挂载的，根据root= kernel选项▶当没有根文件系统可用时，内核恐慌：请添加正确的“root=”引导选项内核恐慌-不同步：VFS：无法在未知块（0,0）上挂载根fs

它从不同的位置可以安装•从硬盘的分区•分区的分区的一个USB密钥•SD卡•分区的NAND闪存芯片或相似类型的存储设备从网络,使用NFS协议从内存,使用预装文件系统(由引导装载程序)•等等。▶由系统设计师选择的配置系统,并配置内核行为与根=

例子略

还有虚拟文件系统，proc 。在linux启动的时候就存在了

cat /proc/3840/cmdline•包含进程打开的文件、CPU和内存使用情况等详细信息
/proc/interrupts、/proc/iomem、/proc/cpuinfo包含与设备相关的一般信息。▶/proc/cmdline包含内核命令行。▶/proc/sys包含许多可以写入以调整内核参数的文件。它们称为sysctl


sysfs，基于内存的虚拟文件系统

各种用户空间应用程序需要列表和查询可用的硬件,例如udev或mdev(见后)使用sysfs▶所有应用程序期望它被安装在/ sys目录▶命令来挂载/ sys:
 mount - t sysfs nodev / sys
 ▶$ ls / sys /块总线类开发设备固件fs内核模块的力量


### 最小文件系统

对于一个linux系统，最少需要什么？
init、shell

为了工作，Linux系统至少需要几个应用程序
▶一个init应用程序，这是内核在挂载根文件系统后启动的第一个用户空间应用程序（参见https://en.wikipedia.org/wiki/Init）：•内核尝试运行由init=命令行参数指定的命令，如果可用。否则，会尝试运行/sbin/init、/etc/init、/bin/init和/bin/sh。在initramfs的情况下，它只会查找/init。另一个路径可以通过rdinit= kernel参数提供。
•如果这些都不工作，内核恐慌和引导过程停止。
•init应用程序负责启动所有其他用户空间应用程序和服务，并作为父进程终止的通用父进程。
▶一个shell，用于实现脚本，自动执行任务，并允许用户与系统交互▶基本的UNIX可执行文件，用于系统脚本或交互式shell: mv， cp, mkdir, cat, modprobe, mount， ip等。▶这些基本组件必须集成到根文件系统中才能使用





### 启动总结
1、bootloader：  加载DTB和内核到RAM（内存），启动内核
2、内核：初始化硬件设备和内核子系统；挂载根文件系统，通过root=  ；  启动init应用程序，默认为/sbin/init
3、 启动其余用户设施和应用程序 ： 包括shell和其余应用


使用initramfs  ，内存方式启动的文件系统？
1、bootloader： 将initramfs存档加载到ram（如果单独）；将DTB +内核加载到RAM，启动内核

2、内核：初始化硬件设备和内核子系统；将initramfs存档提取到文件缓存中；如果找到，启动/init可执行文件；（否则回退到挂载由root=指定的设备）

3、init:  启动早期用户空间命令（显示到屏，启动时间关键应用…）；加载访问最终根文件系统所需的驱动程序；挂载根文件系统并切换到它（switch_root）

/sbin/init，正常启动系统


## busybox

百科这玩意是许多工具的集合，命令行工具等等。


文件操作和系统配置的各种基本实用程序，这个也是linux的一个潜在的基本要求。正常可能是多个组件，而busybox就是一个这样的合集。嵌入式linux的瑞士军刀，哈

busybox make menuconfig，这个是可以自定义选择功能编译进busybox，通过这个menuconfig

所以编镜像的时候，加一个这玩意就很舒服。我电脑也安装了这个

可以使用busybox 建立根文件系统

## 访问硬件设备

### 内核驱动

用于硬件访问的典型软件堆栈。

![](vx_images/285203539333485.png)
从底到上：

内核中的总线控制器驱动程序 I2C， SPI， USB， PCI控制器
总线子系统为驱动程序提供了一个API访问特定类型的总线：I2C， SPI， PCI，USB等。
内核中的设备驱动程序驱动特定的连接到给定总线的设备
▶驱动子系统暴露了某些类的设备，通过一个标准内核和用户空间接口
▶应用程序可以通过这个标准的内核/用户空间接口直接或通过

 Networking stack for Ethernet, WiFi,CAN, 802.15.4, etc.
▶ GPIO
▶ Video4Linux for camera, video encoders/decoders
▶ DRM for display controllers, GPU
▶ ALSA for audio
▶ IIO for ADC, DAC, gyroscopes, sensors, and more
▶ MTD for flash memory
▶ PWM
▶ Input for keyboard, mouse, touchscreen, joystick
▶ Watchdog
▶ RTC for real-time clocks
▶ remoteproc for auxilliary processors
▶ crypto for cryptographic accelerators
▶ hwmon for hardware monitoring sensors
▶ block layer for block storage

/dev这个目录向用户提供设备访问的接口





















