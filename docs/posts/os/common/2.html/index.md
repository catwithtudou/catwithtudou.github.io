# os中的内核


> 以下主要来自 LMOS 大佬的操作系统课程中记录的笔记

## 计算机中的资源管理——内核

计算机中资源主要分为：

- 硬件资源（如总线、CPU、内存、硬盘、网卡、显卡、I/O设备等）
- 软件资源（如各种文件、软件程序等）

![](https://img.zhengyua.cn/img/202203071032147.png)

内核作为硬件资源和软件资源的管理者，其内部组成在逻辑上大致如下：

> 内核除了下面这些必要组件，根据功能不同还有安全组件等，且为了**管理和控制各种硬件编写的代码，即称为“驱动程序”**。

1. **管理CPU**

由于CPU是执行程序的，而内核把运行时的程序抽象成进程，所以又称为进程管理。

2. **管理内存**

由于程序和数据都要占用内存，内核要非常小心地分配、释放内存。

1. **管理硬盘**

硬盘主要存放用户数据，而内核把用户数据抽象成文件，即管理文件，文件需要合理地组织，方便用户查找和读写，所以形成了文件系统。

4. **管理显卡**

负责显示信息，而现在操作系统都是支持 GUI（图形用户接口）的，管理显卡自然而然地就成了内核中的图形系统。

5. **管理网卡**

网卡主要完成网络通信，网络通信需要各种通信协议，最后在内核中就形成了网络协议栈，又称网络组件。

6. **管理各种 I/O 设备**

键盘、鼠标、打印机、显示器等统称为 I/O（输入输出）设备，在内核中抽象成 I/O 管理器。

## 经典内核结构

如何组织内核的组件，让系统稳定和高效运行，这就是内核结构需要考虑的事情。现有的经典内核结构有以下：

- 宏内核结构
- 微内核结构
- 混合内核结构

### 宏内核结构

宏内核就是把以上诸如管理进程的代码、管理内存的代码、管理各种 I/O 设备的代码、文件系统的代码、图形系统代码以及其它功能模块的代码，**把这些所有的代码经过编译，最后链接在一起，形成一个大的可执行程序**：

- 大程序里有实现支持这些功能的所有代码，向用户应用软件提供一些接口（系统 API 函数）
- 大程序会在处理器的特权模式下运行（宏内核模式）

宏内核主要的缺点：

- **没有模块化，扩展性，移动性**，开发新的能需要重新编译、链接、安装内核
- **高度耦合**，造成“牵一发而动全身”

而主要的有点就在于：

- **性能很好**（组件之间互相直接调用，没有过多消耗）

![](https://img.zhengyua.cn/img/202203071043129.png)

比如宏内核提供内存分配功能的服务过程，具体如下：

1. **调用API**：应用程序首先调用内存分配的 API（应用程序接口）函数；
2. **内核运行**：处理器切换到特权模式，开始运行内核代码；
3. **内核分配内存**：内核里的内存管理代码按照特定的算法，分配一块内存；
4. **API返回**：把分配的内存块的首地址，返回给内存分配的 API 函数；
5. **获取内存地址**：内存分配的 API 函数返回，处理器开始运行用户模式下的应用程序，应用程序就得到了一块内存的首地址，并且可以使用这块内存了。

### 微内核结构

微内核架构正好与宏内核架构相反，**它提倡内核功能尽可能少**：仅仅只有进程调度、处理中断、内存空间映射、进程间通信等功能。而实际完成功能（如进程管理、内存管理、设备管理、文件管理等服务功能）是**由服务进程（特殊的“应用程序”）专门负责**。

微内核定义了一种良好的进程间通信的机制——消息，**服务进程的编程模型就是循环处理来自其它进程的消息，完成相关的服务功能**：

- 应用程序请求服务，则向微内核发送消息，微内核将消息转发给服务进程，然后服务进程完成服务后返回消息

微内核结构主要的缺点在于：

- **系统性能较低**，转发消息和服务进程切换开销较大

而主要的优点在于：

- **系统结构清晰**，利于协作开发
- **良好的移植性，且代码量较少**
- **良好的伸缩性和扩展性**，服务进程成本较低

> 微内核的代表作有 MACH、MINIX、L4 系统，这些系统都是微内核，但是它们不是商业级的系统，商业级的系统不采用微内核主要还是因为性能差。

![](https://img.zhengyua.cn/img/202203071055747.png)

而比如微内核提供内存分配功能的服务过程，具体如下：

1. **调用API发送消息**：应用程序发送内存分配的消息，这个发送消息的函数是微内核提供的，相当于系统 API，微内核的 API（应用程序接口）相当少，极端情况下仅需要两个，一个接收消息的 API 和一个发送消息的 API；
2. **内核运行**：处理器切换到特权模式，开始运行内核代码；
3. **内核转发消息给服务进程**：微内核代码让当前进程停止运行，并根据消息包中的数据，确定消息发送给谁，分配内存的消息当然是发送给内存管理服务进程；
4. **服务进程分配内存**：内存管理服务进程收到消息，分配一块内存；
5. **服务进程返回消息给内核**：内存管理服务进程，也会通过消息的形式返回分配内存块的地址给内核，然后继续等待下一条消息；
6. **内核返回消息给应用程序**：微内核把包含内存块地址的消息返回给发送内存分配消息的应用程序；
7. **应用程序得到内存**：处理器开始运行用户模式下的应用程序，应用程序就得到了一块内存的首地址，并且可以使用这块内存了。


## 内核设计

### 分离硬件的相关性

分层的主要目的和好处在于**屏蔽底层细节，使上层开发更加简单**。

> 计算机领域的一个基本方法是增加一个抽象层，从而使得抽象层的上下两层独立地发展。

**分离硬件的相关性**，就是要把操作硬件和处理硬件功能差异的代码抽离出来，**形成一个独立的软件抽象层，对外提供相应的接口，方便上层开发**。

> 比如进程调度模块，主要有两个机制：进程调度和进程切换。
> 不管是在 ARM 还是 X86 硬件平台，选择进程调度的代码不容易发生改变，而需要改变的代码是进程切换的相关代码，因为不同的硬件平台上下文是不同的。
> 所以这里最好是将进程切换分离到独立的层实现，当操作系统要运行在不同的硬件平台上时只需要修改该层的相关代码，提高其移植性。

### 自主设计的选择

将内核分为三个大层分别是：

- **内核接口层**：定义了一系列接口
- **内核功能层**：主要完成各种实际功能，这些功能按照其类别分成各种模块
- **内核硬件层**：主要包括一个具体硬件平台相关的代码

这么设计的考虑：

- 吸取微内核优势（内核小维护成本小，且扩展性强），内核实现的功能很少；
- 吸收宏内核优势（高耦合代来的性能提升），把文件系统、网络组件、其它功能组件作为虚拟设备交由设备管理；

![](https://img.zhengyua.cn/img/202203071122381.png)

## 业界成熟内核架构

### Linux 内核

- 基本思想：**一切都是文件**
- 支持**类 UNIX、POSIX 标准接口**，也支持**多用户、多进程、多线程、多核运行**
- **宏内核结构**

![](https://img.zhengyua.cn/img/202203071143971.png)

> [原图地址](https://makelinux.github.io/kernel/map/)

![](https://img.zhengyua.cn/img/202203071144376.png)


### Darwin-XNU 内核

- 主要特点就是**两个内核层即 Mach 层与 BSD 层**。
- **微内核结构**

> 在调用 Darwin 系统 API 时，会传入一个 API 号码，用这个号码去索引 Mach 陷入中断服务表中的函数。此时，API 号码如果小于 0，则表明请求的是 Mach 内核的服务，API 号码如果大于 0，则表明请求的是 BSD 内核的服务，它提供一整套标准的 POSIX 接口。

![](https://img.zhengyua.cn/img/202203071131577.png)

### Windows NT 内核

- **高内聚，低耦合**
- **硬件抽象层 HAL** ，并在之上定义了**小内核**
- **混合内核结构**

> Windows NT从设计架构上看属于微内核结构，HAL层之上的内核层属于微内核的核心，之上的执行体属于内核级别的应用层。Windows NT从权限的角度看属于宏内核，内核模式之下功能完备，并不是微内核那样单薄。这种设计兼顾了结构清晰和性能良好两个优点。

![](https://img.zhengyua.cn/img/202203071135866.png)

## 参考

https://time.geekbang.org/column/article/372609
