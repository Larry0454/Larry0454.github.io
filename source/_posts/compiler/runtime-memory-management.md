---
title: 『compiler-5』runtime memory management
date: 2024-03-27 21:42:09
tags: compiler
---

## 运行时的存储组织及管理

---

### 一、静态存储分配

- 静态分配概念：在<u>编译阶段</u>由**编译程序**实现对存储空间的管理，为源程序中的变量分配存储
  <br>**注意**：静态分配要求能够确定变量在运行时的数据空间大小，且运行时不变

- 分配策略：无可变长的串or数组，且不允许递归调用

  1. 开辟**数据区**，其首地址在加载时确定
  2. 按编译顺序给每个**模块**分配存储空间
  3. 在模块内部按顺序给模块的变量分配存储（使用**相对地址**）
  4. 目标地址填入**变量的符号表**中

- FORTRAN 子程序的数据区：一个子模块主要包括<u>隐式参数区</u>、<u>形式参数区</u>、<u>局部变量和临时参数区</u>

  - 隐式参数区：存放调用**返回地址** or 不便从寄存器返回的**函数返回值**
  - 形式参数区：存放**实参**的地址或值
  - 局部变量和临时变量区：记录**临时变量**的存储空间

  **注意**：该情况下程序运行栈是由各子模块组成的，运行栈是自顶向下生长的

---

### 二、动态存储分配

- 动态分配概念：在<u>目标程序运行阶段</u>由**目标程序**实现对存储空间的管理，为源程序中的变量分配存储
  <br>**注意**：动态存储分配中，要求编译程序能够生成**有关存储分配**的目标代码

- 分配策略：数据区为一个**栈**

  1. 进入一个过程：在栈顶为其**开辟**一个数据区
  2. 退出一个过程：**撤销**栈顶的过程数据区

- 活动记录：创建于**运行栈**上的专有数据区，包括 <u>局部数据区</u>、<u>参数区</u>、<u>display</u> 区

  - 局部数据区：存放本模块定义的**各局部变量**

  - 参数区：存放隐式参数和显式参数

    - 显式参数区：存放模块调用的各个实参值 or 实参地址
    - 隐式参数区：
      - prevabp：指向**caller**模块的**基地址**
      - ret addr(n)：本模块**返回到模块n**
      - ret value：函数的**返回值**

  - display区：存放本模块的**各外层模块**AR的**基地址abp(1 ~ i-1)**

    - 变量的二维编址：(BL, ON)，其中 BL 指**嵌套深度**、ON 指变量**在本层的变量顺序号**（$\ge 0$）

      <figure style="text-align:center">
          <img src="变量的二维编址.png">
          <figurecap>内层模块可以引用外层模块的变量</figurecap>
      </figure>

    - 构造AR的display区：假设从某第 i 层模块进入某第 j 层模块，若要创建第 j 层的display区：

      - 若 j = i + 1（即 i call j 或 i begin-j-end），则有：

        <figure style="text-align:center">
            <img src="规则一.png">
            <figurecap>j比i恰深一层</figurecap>
        </figure>

      - 若 j $\le$ i，即从 i 跳到某个外层or同层的模块 j，则有：

        <figure style="text-align:Center">
            <img src="规则二.png">
            <figurecap>j外层的所有AR基址（1 ~ j-1）赋给i的display区</figurecap>
        </figure>

  - 运行时的变量地址计算：设要在 LEV 层访问的变量为 (BL, ON)，且当前模块基址为 abp

    - 若 BL = LEV ：ADDR = abp + (BL - 1) + nip + ON
    
      <figure style="text-align:center">
          <img src="同层地址计算.png">
          <figurecap>同层地址计算</figurecap>
      </figure>
    
    - 若 BL < LEV ：ADDR = **display\[BL\]** + (BL - 1) + nip + ON
    
      <figure>
          <img src="外层地址计算.png">
      </figure>
    
    - 若 BL $\ge$ LEV ：说明要访问内层模块的变量，不合法
    
      **注意**：(BL - 1)+nip 表示要<u>向上跨过</u> display区和隐式参数区，才能到达**显式参数区**和局部数据
      <br>一个AR的结构如图所示：多个AR组成了运行栈（栈的方向在图中是**向上增长**的）
    
      <figure style="text-align:center">
          <img src="活动记录.png">
      </figure>
