---
title: 『link and load-3』runtime library
date: 2024-03-27 20:53:59
tags: link and load
---

## 库与运行库

---

### 一、程序的内存布局

- 应用程序的内存空间：从上往下依次是：

  - 栈：维护**函数**调用的上下文，从高地址向下扩展
  - 动态链接库映射区：装载依赖的**动态链接库**
  - 堆：容纳应用程序**动态分配**的内存区域（malloc or new），从低地址向上扩展
  - 可执行文件映像：**可执行文件**的可读可写区、只读区等段
  - 保留区：被**禁止访问**的低地址区域

  **注意**：“segment fault” 常发生在**非法指针解引用**，如<u>试图写0地址，随机地址</u>等

  ---

- 栈：遵循**先入先出**（FIFO）的**动态**内存区域

  - 栈帧：保存函数调用所需要维护的信息，又称活动记录，主要包括以下内容：

    - 函数**接收的参数与返回地址**
    - 临时变量：函数内的**非静态局部变量**，或编译器自动生成的**其它临时变量**
    - 保存上下文：函数调用前后需要**保持不变的寄存器**

  - “烫”与“屯”：Debug模式下，分配的栈空间（未初始化的局部变量）被初始化为0xCC或0xCD

  - 与栈相关的两个寄存器：esp和ebp
  
    - esp：始终指向栈顶，其位置随函数执行而不断变化
    - ebp：又称“帧指针”，其位置固定不变，指向栈中<u>调用函数前的ebp旧值</u>

  - i386标准函数调用流程：2、3两步由call指令一起执行

    1. 把所有或部分**参数压栈**，或使用特定寄存器传参

    2. 将当前调用指令的**下一条指令的地址**（即返回地址）压入栈中

    3. 保存好<u>参数</u>和<u>返回地址</u>后，跳转到函数体：
  
     ```c
     /* 将ebp寄存器的 旧值 保存在栈上 */
     push ebp
     /* 将ebp固定在当前栈顶位置 */
     mov ebp, esp
     /* 开辟x字节的栈空间 */
     sub esp, x
     /* 保存n个寄存器（可选） */
     push reg_1
     ...
     push reg_n
     /* ... FUNCTION BODY ... */
     /* 恢复n个寄存器（可选） */
     pop reg_n
     ...
     pop reg_1
     /* 回收开辟的栈空间 */
     mov esp, ebp
     /* 恢复ebp寄存器的旧值 */
     pop ebp
     /* 调用ret指令返回 */
     ret
     ```
    
       **注意**：对于声明为static（或仅在本编译单元被调用）的函数，可能不会按照上面的标准流程执行
    
    ---
    
  - 调用惯例：函数的<u>调用方</u>和<u>被调用方</u>之间的约定
  
    - 函数参数的传递顺序和方式：规定参数的**压栈顺序**，**传参寄存器**等
    - 栈的维护方式：约定**负责弹栈的一方**，保持栈在函数调用前后保持一致
    - 名字修饰策略：通过**对函数本身的名字进行修饰**，以区分不同的调用惯例
  
  - 常见的调用惯例：
  
    - cdecl：默认调用惯例
      - 压栈顺序：**从右向左**将参数列表压栈
      - 负责出栈方：函数**调用方**
      - 名字修饰：下划线 + 函数名，如_foo
    - stdcall：
      - 压栈顺序：**从右向左**将参数列表压栈
      - 负责出栈方：函数**本身**
      - 名字修饰：下划线 + 函数名 + @ + 参数列表的总字节数，如_foo@12
    - fastcall：通过寄存器传参优化性能
      - 压栈顺序：头两个4字节（或更少）的参数通过寄存器传参，其余参数**从右向左**压栈
      - 负责出栈方：函数**本身**
      - @ + 函数名 + @ + 参数列表的总字节数，如@foo@12
  
  - 函数返回值的传递：
  
    - 返回值 $\le$ 4字节：使用eax寄存器传值；4字节 $\lt$ 返回值 $\le$ 8字节：eax存低四字节、edx存高位
    - 返回值 $\gt$ 8字节：在**栈上**开辟空间保存返回值对象，并将**返回值的地址**通过eax寄存器传出
  
  ---
  
- 堆：占据绝大部分的虚拟空间

  - Linux 进程堆管理：两个**系统调用**，分别是brk( )和mmap( )

    - brk: 设置进程数据段的**结束地址**，其C函数原型为 int brk(void\* end_data_segment);
    - mmap: C函数原型为 void\* mmap(void\* start, size_t length, int prot, int flags, int fd, ...);
      - start与length：指定需要申请的**空间始址**和**长度**
      - prot：指定申请空间的**权限**（可读 or 可写 or 可执行）
      - flags：指定申请空间的**映射类型**（文件映射 or **匿名空间**）
      - fd：若为文件映射，则指定**被映射文件的描述符**

  - Windows 进程堆管理：

    - Windows 进程地址空间：

    <figure style="text-align:center;">
      <img src = "Windows进程地址空间.png" width=35% height=35%>
      <figcaption>Windows进程地址空间</figcaption>
    </figure>

    - EXE一般位于地址 0x00400000；运行库DLL一般位于地址 0x10000000

    - VirtualAlloc: Windows提供的API，用于向系统申请虚拟空间

    - 堆管理器：Windows中提供与堆相关的一套API

      - HeapCreate：创建一个堆（通过VirtualAlloc实现）
      - HeapAlloc：在一个堆里分配一块较小的内存
      - HeapFree：释放已经分配的内存
      - HeapDestroy：销毁一个堆

      **注意**：Windows中，当一个堆空间不够时，会创建更多的堆，故进程中可能存在多个堆

    ---

  - 堆分配算法：用于管理堆空间，做到按需分配与回收释放

    - 空闲链表：将堆中各空闲的块按链表串起来
      - header：每个空闲块的开头有一个头结构，记录上一个和下一个空闲块的地址
      - prev与next：分别指向上一个空闲块地址和下一个空闲块地址
      - 分配方案：在空闲链表中查找足够大的空闲块，并将其中一部分分配出去

    - 位图：将堆均分为大量的块，并使用数组管理块
      - 分配方案：每次分配整数个块，第一个块为Head，其余块为Body
      - 块状态：Head or Body or Free 


---

### 二、运行库

- 入口函数（Entry Point）：一个程序<u>初始化</u>或<u>结束</u>的部分

  - GLIBC入口函数：

    - 调用_start函数前：装载器依次将**环境变量**和**用户参数**压入栈中
  
    - 调用_start函数：\_start入口由ld链接器的链接脚本决定
  
    ```c
    libc\sysdeps\i386\elf\Start.S
    _start:
    	/* 将ebp寄存器置0 */
    	xorl %ebp, %ebp
        /* esi寄存器指向argc */
        popl %esi
        /* ecx寄存器指向argv */
        movl %esp, %ecx
        
        /* 下面向 __libc_start_main 函数传参 */
        /* 将 栈顶地址 压栈 */
        pushl %esp
        /* 将 rtld_fini函数地址 压栈 */
        pushl %edx
        /* 将 __libc_csu_fini函数地址 压栈 */
        pushl $__libc_csu_fini
        /* 将 __libc_csu_init函数地址 压栈 */
        pushl $__libc_csu_init
        /* 将 用户命令 压栈 */
        pushl %esi
        /* 将 main函数地址 压栈 */
        pushl main
        /* 跳转到 __libc_start_main 函数*/
        call __libc_start_main
        /* 若函数调用失败，hlt会强行把程序停下来 */
        hlt
    ```
    
    - __libc_start_main：包括全局对象的构造与析构、main函数的调用与退出
    
    ```c
    int __libc_start_main(
        int (*main) (int, char**, char**),
        int argc,
        /* 用户命令，即argv */
        char** ubp_av,
        /* init 负责 main 调用前的初始化工作 */
        __typeof (main) init,
        /* fini 负责 main 结束后的收尾工作 */
        void (*fini) (void),
        /* rtld_fini 负责和动态加载相关的收尾工作 */
        void (*rtld_fini) (void),
        void* stack_end
    ) {
        /* 取出环境变量 */
        char** ubp_ev = &ubp_av[argc + 1];
        __environ = ubp_ev, __libc_stact_end = stack_end;
        ...
        /* 若干初始化操作 */
        /* ⬇ 通过 __cxa_atexit 函数注册退出函数 ⬇ */
        /* __cxa_atexit(rtld_fini, NULL. NULL); */
        /* __cxa_atexit(fini, NULL, NULL); */
        ...
        int result = main(argc, argv, __environ);
        exit(result);
    }
    ```
    
    - exit与_exit：
    
    ```c
    void exit(int status) {
        /* 遍历 __exit_funcs函数链表，依次调用由 __cxa_atexit 注册的函数 */
        while (__exit_funcs != NULL) {
            ...
            __exit_funcs = __exit_funcs -> next;
        }
        ...
        _exit(status);
    }
    
    _exit:
    	movl 4(%esp), %ebx
        movl $__NR_exit, %eax
        /* 执行exit系统调用，结束程序 */
        int $0x80
        /* 若程序终止失败，hlt会强行把程序停下来 */
        hlt
    ```
    
    **注意**：程序可以在main函数返回后调用exit退出，也可以直接调用exit退出
    
    ---
    
  - MSVC CRT 入口函数：
  
    - mainCRTStartup：
  
    ```c
    int mainCRTStartup(void) {
        ...
        /* 在堆初始化之前只能使用 _alloca 分配空间 */
        posvi = (OSVERSIONINFO *)_alloca(sizeof(OSVERSIONINFO));
        posvi -> dwOSVersionInfoSize = sizeof(OSVERSIONINFOA);
        /* 获取当前操作系统的版本信息，初始化系统全局变量 */
        GetVersionExA(posvi);
        /* 填写平台信息 */
        _osplatform = posvi -> dwPlatformId;
        /* 填写主版本号 */
        _winmajor = posvi -> dwMajorVersion;
        _winminor = posvi -> dwMinorVersion;
        /* 填写操作系统版本 */
        _osver = (posvi -> dwBuildNumber) & 0x07fff;
        ...
        /* 先通过_heap_init初始化堆 */
        if (!_heap_init(0)) {
            fast_error_exit(_RT_HEAPINIT);
        }
        
        __try {
            /* 通过_ioinit初始化IO */
            if (_ioinit() < 0) {
                _amsg_exit(_RT_LOWIOINIT);
            }
            /* 获取命令行与环境变量 */
            _acmdln = (char*)GetCommandLineA();
            _aenvptr = (char*)__crtGetEnvironmentStringA();
            if (_setargv() < 0) {
                _amsg_exit(_RT_SPACEARG);
            }
            if (_setenvp() < 0) {
                _amsg_exit(_RT_SPACEENV);
            }
            /* 设置其它C语言库 */
            initret = _cinit(TRUE);
            if (initret != 0) {
                _amsg_exit(initret);
            }
            _initenv = _environ;
            /* 调用 main 并退出 */
            mainert = main(__argc, __argv, _environ);
            _cexit();
        }
        __except {
            ...
            /* 异常处理 */
        }
    }
    ```
    
      **注意**：try-except块是Windows结构化**异常处理机制**SEH的一部分
    
    ---
    
  - 运行库（CRT）与I/O：
  
    - I/O：操作系统中代指**程序与外界的交互**，如文件、管道、网络，命令行等
    
    - 文件：操作系统中具有**输入输出**概念的设备实体
      - 文件操作：借助FILE结构体指针实现
      - 文件管理：在Linux中使用**文件描述符**（fd）；在Windows中使用**句柄**
      
    - 打开文件表：<u>指针数组</u>，其中每个元素指向**内核的打开文件对象**
    
      <figure style="text-align:center;">
        <img src = "FILE.png">
        <figcaption>操作系统中的文件</figcaption>
      </figure>
    
      **注意**：FILE结构与fd是**一一对应**的
    
  - MSVC CRT 的入口函数初始化：
  
    - 堆初始化：_heap_init( )通过调用**HeapCreate**创建系统堆
    - I/O初始化：_ioinit( )
      1. 建立**打开文件表**
      2. 从父进程获取继承的**文件句柄**
      3. 初始化**标准输入输出**
  
  
  ---
  
- C/C++ 运行库：

  - C语言运行库：主要包括启动与退出函数、标准库函数、I/O功能函数等**运行时依赖**

    **注意**：世界上第一个C语言标准由ANSI于1989年制定（即C89）

  - C语言标准库：主要包括文件操作、字符串处理、数学函数等C头文件，是**运行库的主要部分**

    - 变长参数：C语言中特殊参数形式，通过省略号追加<u>任意数量、任意类型</u>的参数

      1. 头文件：stdarg.h
      2. 实现方式：cdecl调用惯例，即**从右向左压栈**，且调用方负责弹栈
      3. va_list：指向各不定参数的**指针**类型，一般为 char* 类型
      4. va_start：将va_list定义的指针指向**第一个不定参数**
      5. va_arg：获取当前不定参数的值，并将va_list**移向下一个参数**
      6. va_end：将va_list指针**清0**

      **注意**：GCC和MSVC均支持<u>定义宏</u>时使用变长参数

    - 非局部跳转：

      - 头文件：setjmp.h
      - 实现方式：调用 longjmp( ) 跳转至 **setjmp( ) 函数返回的时刻**，并重新指定其返回值
  
  ---

  - glibc与MSVC CRT ：C语言程序与操作系统平台之间的**抽象层**

    - glibc ：GNU C Library，即Linux平台下的C标准库

      - 主要组成：标准头文件 + 库的二进制文件部分

      - glibc启动文件：包括crt1.o、crti.o、crtn.o等
        
        - crt1.o：包含入口函数_start
        - crti.o：包含\_init( )和 \_fini( )函数的**开头代码**
        - crtn.o：包含\_init( )和\_fini( )函数的**结尾代码**
        
        最终输出文件的“.init”段只包含函数_init( )，“.fini”段只包含函数 _fini( )
        
        **注意**：链接时实际顺序为 <u>crti.o -> crtbegin.o -> someobjs.o -> crtend.o -> crtn.o</u>
        
      - GCC平台相关目标文件：包括crtbegin.o、crtend.o等
      
        - crtbegin.o：实现C++的全局对象构造
        - crtend.o：实现C++的全局对象析构
        - libgcc.a：执行不同硬件平台下的数学运算函数
        - libgcc_eh.a：支持C++异常处理的相关函数
      
        **注意**：.init和.fini仅在main( )执行**前后**运行，实际上与全局对象的构造析构无关
      
      ---
      
    - MSVC CRT ：Microsoft Visual C++ C Runtime，即Windows平台下的C标准库
    
      - 主要组成：不同分类指标下的多种**子版本**，如静态/动态链接版、单线程/多线程版等
        					 									 				   不同属性之间可以互相组合
                               编译器cl根据传入参数选择对应的CRT，如选项 /MDd 对应选择msvcrtd.lib
    
      - 静态运行库的命名规则：libc \[p] \[mt] \[d] .lib，其中：
    
        - p表示CPlusplus，即**C++标准库**
        - mt表示Multi-Thread，即表示**支持多线程**
        - d表示Debug，即表示**调试版本**
    
      - 动态运行库：含有用于链接的.lib文件、以及运行时使用的.dll动态链接库
    
        动态链接库的命名规则中增加了**版本号**，如多线程 + **动态链接**的 msvcr**90**.dll
    
      **注意**：若DLL分别使用<u>不同版本的CRT</u>，则各DLL间难以传递共享资源（如堆空间、文件等）
    
    ---
    
  - 运行库与多线程

    - 线程的访问权限：线程可以访问以下数据
    
      - **进程**内存中的所有**公共数据**，如全局变量、堆、静态变量
      - 线程的私有数据，如栈、寄存器、线程局部存储（TLS）
    
    - 多线程运行库：提供<u>多线程操作</u>的接口，并支持<u>多线程环境</u>下的正确运行
    
      - 多线程操作接口：用于线程的**创建与退出**
      - 多线程环境运行：确保使用运行库接口时的**线程安全**
    
    - 多线程安全问题的解决措施：
    
      - 使用TLS：将变量存储在各线程的私有环境中
      - 加锁：在线程不安全的函数中加锁（如malloc、printf 等）
      - 改进函数调用：如 strtok( ) $\rightarrow$ strtok_s( )（C11引入）
        **注意**：strtok( )通过在函数内部设置**静态指针**跟踪字符串地址，多线程调用会有冲突
        strtok\_s( )通过保存<u>上次调用时</u>**静态指针的地址**，保证了线程安全
    
      ---
    
    - 线程局部存储（TLS）
    
      - TLS的隐式定义：Linux使用关键字\_thread、Windows使用\_declspec(thread)
    
        **注意**：被定义为TLS的全局变量在每个线程中保存有**独立的副本**
    
      - Windows中TLS的实现：
    
        - .tls段：专门负责存放被声明为TLS的全局变量
    
        - TLS表：保存所有TLS变量的构造函数和析构函数的**地址**
    
          **注意**：TLS表的信息存于PE数据目录中的IMAGE_DIRECT_ENTRY_TLS项中
    
        - 线程环境块（TEB）：保存线程ID、堆栈地址、TLS数组等信息
    
          **注意**：TLS数组在TEB中的偏移是0x20，可以通过该偏移量找到TLS数组
    
        - TLS数组：用于查找TLS变量在线程中的地址，其首元素存放.tls段的地址
    
      - TLS的显式定义：需借助库函数API<u>手动申请与释放</u>，不便使用（不推荐）
    
        - 实现方式：借助**TLS数组**保存TLS数据
        - 二级TLS数组：需额外申请，用于存放更多的TLS数据
    
    ---
    
  - C++全局构造与析构
  
    - glibc全局构造与析构
  
      - GLOBAL\_\_I\_Hw ：负责<u>本编译单元</u>所有的全局与静态对象的**构造与析构**
  
      - .ctor段：每个目标文件含有一个.ctor段，存放指向本目标文件**全局构造函数的指针**
  
        **注意**：各目标文件的.ctor段合并成最终输出文件的.ctor段，形成<u>构造函数指针数组</u>
  
        - .crtbegin.o：其.ctor段定义符号\_\_CTOR\_LIST\_\_，指向**合并后.ctor段的始址**
        - 目标文件：其.ctor段中存放一个全局构造函数的**指针**
        - crtend.o：其.ctor段定义符号\_\_CTOR\_END\_\_，指向**合并后.ctor段的末尾**
  
      - \_\_do\_global\_ctors\_aux：由GCC提供（不属于glibc），在**_\_libc\_csu\_init**中被调用
  
        - \_\_CTOR_LIST\_\_：全局构造函数的**指针数组**，由 .crtbegin.o 定义
        - 初始化过程：依次执行\_\_CTOR\_LIST\_\_列表中记录的各全局构造函数
  
        <figure style="text-align:center;">
          <img src = ".ctor段.png">
          <figcaption>.ctor段的形成</figcaption>
        </figure>
  
        ---
  
      - \_\_tcf\_1：由GLOBAL\_\_I\_Hw中的**\_atexit**注册，在最后的exit( )中负责**全局对象的析构**
  
        **注意**：全局对象的构造与析构的<u>流程是类似的</u>，<u>顺序是相反的</u>
  
      ---
  
    - MSVC CRT的全局构造与析构
  
      - \_initterm：由mainCRTStartup函数调用
        借助全局指针\_\_xc\_a和\_\_xc\_c，依次遍历执行所有的全局构造函数
      
        - .\_\_xc\_a：被分配在段.CRT\$XCA中，指向首个构造函数
          （类似\_\_CTOR\_LIST\_\_）
        - .\_\_xc\_c：被分配在段.CRT\$XCZ中，指向构造函数的结束地址
          （类似\_\_CTOR\_END\_\_）
      
        **注意**：各.CRT\$XC\*段合并后被分配在.rdata段（只读），且按照<u>字母\*的字典序</u>链接
      
        ---
      
      - dynamic atexit destructor \*：由**atexit( )**注册，退出时负责<u>全局对象\*的析构</u>
  
  
  ---
  
  ### 三、系统调用与API
  
  - 系统调用：由**操作系统提供**的，用于访问**临界资源**的一套接口
    					 如创建（退出）进程、访问系统资源（如文件、网络等）
    
  - Linux系统调用：由0x80中断完成，由各通用寄存器<u>传递系统调用参数</u>
  
  - 运行库与系统调用：运行库作为“中间层”，解决了系统调用**跨平台**不兼容的问题，**简化**了系统调用
  
       **注意**：运行库只能为各平台**功能的交集**提供中间层，无法完全维持各平台间的兼容性
  
  - 特权级与中断：
  
       - CPU的两种特权级：“用户模式”与“内核模式”，限制代码操作的权力
  
       - 中断：由<u>用户态</u>切换到<u>内核态</u>的桥梁
  
            - 终端类型：硬件中断（如外设）与软件中断（如执行异常）
  
            - 中断号：不同的中断拥有不同的中断号，用于查找对应的**中断处理程序**
  
            - 中断处理程序：由中断号索引，程序指针存于**中断向量表**中
            
            **注意**：Linux下借助中断号0x80触发**所有<u>系统调用</u>**，并通过**寄存器eax**获取对应系统调用号
       
       ---
       
  - Linux下基于int指令的系统调用：
  
       1. 触发中断：执行内嵌汇编代码`int $0x80`保存现场，并切换到内核态，查找0x80号中断
  
       2. 切换堆栈：调用**0x80**中断，从用户栈切换至内核栈，再将esp等用户态的寄存器压入内核栈
  
       3. 执行中断处理程序：根据**eax寄存器**的值，在sys_call_table中查找对应的内核函数sys_\*
  
          <figure style="text-align:center;">
            <img src = "Linux系统调用流程.png">
            <figcaption>Linux系统调用流程图解</figcaption>
          </figure>
  
          **注意**：内核函数sys_\*从**内核栈**上获取参数，其中各参数寄存器由**宏SAVE_ALL**压入内核栈中
  
       ---
  
  - Linux的新型系统调用
  
       - 系统调用的改进：
         - 改进前：基于int指令的系统调用在Pentium IV上表现不佳
         - 改进后：Pentium II开始使用专门针对系统调用的指令**sysenter**和**sysexit**
       - 虚拟动态共享库（VDSO）：用于**支持新型系统调用**，总是被加载到 0xffffe000 上
         - \_\_kernel\_vsyscall：位于VDSO中，负责新型系统调用，其地址为 0xffffe400
         - sysenter：新型系统调用指令，位于函数 \_\_kernel\_vsyscall 中
           1. 可直接跳转到<u>寄存器指定的函数</u>并执行
           2. 可自动实现<u>特权级切换</u>与<u>堆栈切换</u>等功能
       
       ---
       
  - Windows API ：Windows系统提供的**应用程序编程接口**
  
       - 系统服务与API ：程序员无法像使用Linux那样直接使用系统调用，只能使用上层API
            **注意**：系统调用的一般流程为 <u>Application -> CRT -> (API) -> Kernel</u>
  
       - SDK ：Windows API DLL导出函数声明的头文件、导出库、相关文件与工具的集合
            头文件windows.h中包含了Windows API中的核心部分
  
       - 使用API的优势：
  
            - 屏蔽了不同平台下硬件结构的差异，确保了**系统调用的正常执行**
            - 为Windows各版本使用互不相同的内核提供了可能，极大程度实现了**向后兼容**
  
            ---
  
       - Windows API的版本演进：Win16 -> **Win32** -> Win64，其中数字表示位数
            Win32是应用最广泛最成熟的版本
  
       - Win32的主要功能类别：Windows API是以**DLL导出函数**的形式暴露给开发者的
  
            1. 基本服务（kernel32.dll）：包括文件系统、进程与内存管理等Windows的**基本功能**
            2. 图形设备接口（gdi32.dll）：包括绘图、打印等**与图形设备相关**的功能
            3. 用户接口（user32.dll）：包括滚动条、按钮等**与Windows窗口交互**的操作
            4. 高级服务（advapi32.dll）：包括注册表、系统关闭重启、账号管理等额外功能
            5. 通用对话框（comdlg32.dll）：包括打印窗口、选择字体与颜色等功能
            6. 网络服务（ws2\_32.dll）：包括Winsock、NetDDE等**网络相关服务**
                 **...**
  
            **注意**：在Windows NT系列平台上，上述DLL还会依赖更底层的NTDLL.DLL
  
       - “银弹”：通过在软件体系结构中增加中间层以解决**兼容性问题**，如Windows API
  
       ---
  
  - 子系统：Windows NT提供了<u>其它操作系统</u>的执行环境，以**兼容它们的应用程序**，如Win32子系统
       <br>**注意**：API是架设在<u>应用层和内核</u>之间的中间层；子系统是架设在<u>应用层和API</u>之间的另一个中间层
