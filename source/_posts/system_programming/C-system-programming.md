---
title: 『system programming-1』C system programming
date: 2024-03-27 22:39:38
tags: system programming
---

## C语言系统级编程

---

### 一、内存的基本概念

- 字节：内存的基本单位，由8个0/1比特位组成
- 机器位宽：表示内存中byte的编号，如32位机表示共可以**访问$2^{32}$个字节**（0 ~ $2^{32} - 1$）

---

### 二、非数组与数组变量的声明

- 内存的相关操作：分配内存、内存赋值、读取内存、释放内存

- 分配内存：C语言中主要有两种方式，即“变量声明”（无需手动free）与“使用malloc”（需要手动free）

- 变量要素：变量类型（约定了<u>内存类型与大小</u>） + 变量名称（标记<u>内存的别名</u>）

  - 非数组变量类型：包括int、char、结构体等

  - 数组变量类型：由多个**相同变量类型**（可以是非数组 or 数组类型）组成的<u>**一维**</u>构造类型
    <br>如int[2]表示由2个int类型构成的数组；double\[2\]\[3\]\[4\]表示由2个double\[3\]\[4\]类型构成的数组

    **注意**：可以使用 **sizeof** 关键字获取某个变量类型的占用内存大小

  - 指针变量类型：实质上是一种**非数组变量类型**，任意一种类型都有指向其的指针类型：

    1. 指向<u>非数组变量类型</u>变量的指针：Var_T $\rightarrow$ Var_T\*
    2. 指向<u>数组变量类型</u>的指针：Var_T\[10\] $\rightarrow$ Var_T(\*) \[10\]，注意加**括号**
       <br>**注意**：指针类型也是一种普通的变量类型，要将Var_T\* 看作一个**整体**PTYPE
        <br>如int\* \[2\]对应的指针变量为int\*(\*) \[2\]，其中int\* 可看作一个整体PINT
    3. 指向<u>指针变量类型</u>的指针：Var_T\* $\rightarrow$ Var_T\*\*，Var_T(\*) \[10\] $\rightarrow$ Var_T(\*\*) \[10\]

    **注意**：使用sizeof计算任意指针的大小都是4字节，这与机器位宽有关

---

### 三、**内存**的变量类型和表示值类型

- 值（Value）：包括值本身 + 表示值类型 \<V, V_T\>
  <br>**注意**：C语言中**任何表达式都是有值的**，如10 对应值\<10, int\>，(10 > 20) 对应值\<0, int\>
- 变量类型（Var_T）与表示值（V_T）类型：
  - 变量类型：包括**非数组**和**数组**变量类型，是系统分配内存的依据（物理视角）
  - 表示值类型：从系统**外部观察**内存得到的值的类型，**由变量类型Var_T决定**：
    1. Var_T为非数组类型：V_T = Var_T，V表示通过 Var_T **解释内存**比特串代表的值
    2. Var_T为数组类型：V_T表示<u>数组元素</u>对应的**指针变量类型**，V表示数组在内存中**首字节的编号**
       如变量类型为char \[2\]\[3\]\[4\] 对应的表示值类型为 char(\*)\[3\]\[4\]
- “内存六元组”模型：M = \{Address, Var\_T, Name, Size, Value, V\_T \}
  - Address：该段内存的首字节编号，由系统分配，一旦确定直至回收都无法更改
  - Var\_T：分配内存的**变量类型**，在<u>变量声明语句</u>中确定
  - Name：分配内存的**变量名称**，在<u>变量声明语句</u>中确定
  - Size：该段内存的**大小**（字节数量）
  - Value：该段内存的**表示值**，取值由<u>Var\_T</u>确定
    <br>**注意**：对于结构体或联合体变量，由于其内存块可能包含多个成员变量，故无法获取内存块的具体值
  - V\_T：分配内存的**表示值类型**，类型由<u>Var\_T</u>确定
- malloc( )：分配**指定大小**的连续内存，其变量类型与表示值类型均不确定

---

### 四、声明与赋值

- 变量声明：利用声明语句，在内存中某处（**Address**）分配**特定字节的Size**，并为其标记**Name**，设置**两个类型T**
  <br>**注意**：仅声明不赋值，内存中的表示值**Value**是随机的（undefined）

- 变量赋值：对一块内存进行赋值

  - 赋值表达式：包括左值（用于定位待赋值内存块）与右值（用于为内存块赋值）
    - 左值：通过**变量名称或\*运算符**（V_T为指针类型）定位特定的内存
    - 右值：一个**有效表达式**，可求出对应的值
  - 赋值流程：
    1. 根据左值定位待赋值内存
    2. 计算右值表达式的值\<V, V_T\>
    3. 检查**内存表示值类型**与**右值表示值类型**是否一致，若不一致则warning/error
       <br>如 `int a; a = "123";` 会报错
    4. 将右值V按V_T类型**转化为比特串**放入待赋值内存中

  **注意**：无法直接向数组类型变量赋值（**数组类型变量**的<u>表示值Value</u>总是等于其<u>首地址Address</u>，改不了）

---

### 五、观察内存的三种视角

- &：对于变量类型为Var\_T的变量a，表达式&a的返回值为\<Address, Var\_T\*\>；故有赋值语句`int* p = &a;`

- sizeof：对于内存大小为Size的变量a，表达式sizeof(a)的返回值为\<Size, size\_t\>；故有赋值语句`size_t n = sizeof(a);`
  <br>**注意**：size\_t是C语言中表示**内存大小**的类型

- 变量名：对于变量类型为Var\_T，表示值为V的变量a，表达式a的返回值为\<V, V\_T\>，即**直接取其表示值**；故有赋值语句`int b = a;`

  **注意**：与变量有关的三种表达式依次代表了**内存**地址、**内存**大小以及**内存**表示值

  <figure style="text-align:center;">
    <img src = "内存六元组.png" width=50% height=50%>
    <figcaption>内存六元组模型图示</figcaption>
  </figure>

---

### 六、左值与右值

- 表达式：一般由变量、常量、操作符构成
- 左值：能够定位内存的表达式
  <br>**注意**：等号左侧识别出的内存符号，一旦<u>与任何**操作符**结合</u>，就会变成对表达式的**取值操作**，即不再能够定位一块内存，**会报错**
- 右值：一个表达式的返回值\<V, V\_T\>

---

### 七、sizeof 操作符

- sizeof(Var\_T)：计算变量类型Vat\_T的空间大小
- sizeof(Name)：计算变量Name占用的空间大小
- sizeof(exp)：计算表达式exp**返回值的类型**的大小
  <br>**注意**：对于表达式exp的情形，并不需要先计算exp的返回值\<V, V\_T\>再求出sizeof(V\_T)
  <br>这是因为sizeof是**编译时关键字**，exp的返回值类型早在**编译时**（而非运行时）就已经确定了
- 有关字面量的问题：由于'a'、'b'等字符在C语言中被视为**Integer** Character Constant，故sizeof('a') = 4 != 1
  <br>**注意**：在C++中 sizeof('a') = 1，故需要在移植C/C++程序时考虑这个问题

---

### 八、 \* 操作符

- 定位内存的两种方式：变量名称 or \*操作符

- \*操作符：用于**定位**指针变量指向的**内存**，定位流程如下：设`int* p; *p = 10;`

  1. p不与&和sizeof结合，故定位p的内存并仅**取其值**（即某块内存的始址，表示值类型为指针）

  2. p与\*操作符结合，定位指针变量p指向的内存

  3. $$
     \text{对变量内存 (*p) 的操作}
     \begin{cases}
     \text{左值操作：对内存赋值，注意会检查类型是否匹配} \\
     \text{右值操作：包括取地址\&、求大小sizeof、或与其它操作符结合}
     \end{cases}
     $$

  **注意**：\*操作符与指针变量结合**获得该指针变量指向的类型**；若不与指针变量结合就会报错

---

### 九、指针加减法

- 对任意**表达式exp**，若exp的返回值类型是一个**有效的指针类型**，则有：
  $$
  *(\text{exp+n}) \Leftrightarrow \text{exp[n]}
  $$
  故任何一个返回值为<u>指针类型</u>的表达式，都蕴含着位于对应内存的**数组**（<u>大小未知</u>），其元素类型为指针变量类型指向的类型（Reference Type）

  <br>如 (p + 1)[2] $\Leftrightarrow$ \*(p+1**+2**) $\Leftrightarrow$ \*(p + 3) $\Leftrightarrow$ p\[3\]

  ---

- 指针加常数：设 p : \<Value, Value\_T \>，则 p+n : \<Value + n $\times$ sizeof(\*p), Value\_T \>

- 指针相减：设 p : \<Value1, int\*\>，q : \<Value2, int\*\>，则 p-q :  \< (Value1 - Value2) / sizeof(\*p),  **ptrdiff\_t** \>
  <br>**注意**：相减的指针类型必须**相同**；相减得到的表示值是**两个地址之间的变量个数**

---

### 十、数组的内存访问

- 设数组变量g为int g\[2\]\[3\]，则有：
  - &g：\<address of g,  int(\*)\[2\]\[3\]\>
  - sizeof(g)：\<size of g,  size\_t\>
  - g + 1：\<(**address of g**) + 1 $\times$ sizeof(\*g),  int(\*)\[3\]\>，注意V\_T为数组元素的**指针类型**保持不变
- 数组名：是一个**标识符**，一个**左值表达式**（可以定位一块内存）

---

### 十一、参数传递

- 实参与形参：先获取实参表达式exp的**返回值**，再用该返回值初始化函数**形参名对应的内存中**（Pass By Value）
  <br>**注意**：<u>实参表示值类型</u>与<u>形参类型</u>必须一致
- 非数组变量传参：将实参返回值写入形参的内存块中
- 数组变量传参：可以省略形参的第一维大小，即省略数组大小信息
  <br>**注意**：由于函数<u>形参</u>与<u>实参</u>的表示值类型都是**元素的指针变量类型**，故数组的大小信息可以被省略

---

### 十二、malloc

- 使用malloc分配堆内存的特点：

  1. 内存**没有名称Name**
  2. malloc返回值类型为void\*，指向分配内存的首字节，需要先强转类型
     如`Var_T* p = (Var_T*)malloc(sizeof(Var_T)*N);`
  3. malloc分配的内存空间未初始化
  4. 需要用 **free(p)** 释放内存，有多少次malloc就有多少次free（避免memory leak）

  **注意**：一般使用 malloc(sizeof(Var\_T) \* N) 分配内存空间，表示申请了一个 Var\_T\[N\] 类型的空间，其表示值类型为Var\_T\*

---

### 十三、typedef

- 为非数组类型提供别名：`typedef char INT1;`，即将char重定义为INT1类型
- 为数组类型提供别名：`typedef int AINT[2];`，即将int\[2\]重定义为AINT类型

---

### 十四、const限定符

- 限定<u>非数组</u>变量类型：`Var_T const a;`，Var\_T const是一个<u>**整体**</u>，表示Var\_T指向的内存是**只读的**
  <br>**注意**：由const修饰的变量类型，其表示值类型为对应的**非限定变量类型**；如 int\* const 的表示值类型为 int\*

- 限定<u>数组</u>变量类型：Var\_T const\[2\]\[3\]， 其对应的表示值类型为Var\_T const(\*)\[3\]，表示其元素类型是**只读的**
  <br>**注意**：此时应把 Var\_T const 看作一个<u>**整体**</u>

  ---

- Var\_T const对应的**指针**变量类型：Var\_T const\*，其表示值类型也是Var\_T const\*
  <br>**注意**：只有最后一个“干净”的const修饰只读变量，如：
  <br>由 Var\_T const\* 定义的变量是可以修改的，但指向的内存是**只读的**（其V\_T为Var\_T const\*，解引用后为Var\_T const）
  <br>由 Var\_T const\* const 定义的变量是不可被修改的（const修饰），其指向的内存也是**只读的**（其V\_T为Var\_T const\*，解引用后为Var\_T const）

- Var\_T const VS. const Var\_T：推荐使用前者，避免解读歧义；C语言中对const的位置无明确语义规范
  <br>**注意**：指针变量 const Var\_T\* 会存在解读歧义：不确定\*是与整个 const Var\_T 结合还是只跟 Var\_T 结合

---

### 十五、字符串

- 字符数组：变量类型为char\[N\]，表示值类型为char\*，与其它数组变量类型类似
- 字符串初始化：使用**双引号括起来**的字符串常量，其末尾包含一个隐藏的'\0'
  <br>**注意**：字符串"hello"被放置在**静态区**，其元素无法被修改，如`"hello"[0] = 'e'`会运行崩溃
- String Literal **VS.** String：前者字符串内部可以包含任何字符（包括'\0'），后者只是在最后有'\0'
  <br>**注意**：两者使用 sizeof 得到的大小是相同的，但使用 strlen 得到的长度是不同的
- 访问字符串：字符数组**本身**可以定位一块内存，如&"hello"，sizeof("hello")，"hello"[0]等

---

### 十六、左值与表达式

- 表达式：C语言中由一系列**操作符**和**操作对象**组成的序列

- 左值（**lvalue**）：一种可以<u>定位一个**对象**</u>的**表达式**
  <br>**注意**：C语言中Type分为Object Type和Function Type

  1. 变量标识符：int **a**
  2. 字符串："hello"
  3. 解引用 \*exp：\*(p + 1)
  4. 数组取值 exp1\[exp2\]：e\[1\]
     **注意**：E1\[E2\] $\Leftrightarrow$ \*(E1 + E2)，要求其中一个表达式返回**指针类型**、另一个E返回**整数类型**
  5. 复合字面量 (type-name){ initializer }：如(int){1}，(int\[3\]) {1, 2, 3}
  6. 成员变量引用：h.name，p->name

- 可被放在等号左边的左值（**modifiable lvalue**）：需同时满足以下条件：

  - 非数组变量类型：表示值总是等于内存首地址，无法被修改
  - 不是不完全类型：如extern int **a**\[\]，其中不完全类型是指除函数类型外**大小无法确定**的类型
  - 不能被 const 修饰；对于结构体类型变量，其各成员变量不能被 const 修饰

  **注意**：若对非modifiable lvalue赋值，则会报编译错误

- 右值（rvalue）：即表达式的值<V, V\_T>，表达式之间通过**表示值**计算返回值

---

### 十七、函数指针

- 函数类型（Function Type）：规定了**函数返回值类型**与**参数数量及类型**，如int(char, int)
  <br>**注意**：区别于对象类型，函数类型不是lvalue，故函数类型变量不能被赋值
- 函数类型对应的**指针类型**：如int(\*)(char, int)
- 函数类型六元组：仿照<u>对象类型六元组</u>提出
  1. Address：函数入口的地址
  2. Var\_T：函数类型，如int(int, int)
  3. Name：函数标识符
  4. Size ：N/A，函数类型无大小，不能与 sizeof 结合
  5. V：表示值与Address的值相同
  6. V\_T：表示值类型是**函数类型对应的指针类型**，如int(\*)(int, int)
- 函数调用：函数标识符 + 参数列表，其中函数标识符即为函数名称
  <br>**注意**：函数标识符func属于基础**表达式**，其返回值为\<函数**入口地址**,  函数对应的**指针类型**\>
- &func与\*pfunc：
  - &func：由&操作符的性质，&func返回**函数指针变量**，其表示值即为address
    由此可见<u>&func</u>与<u>func</u>表达式的返回值是完全相同的
  - \*pfunc：由解引用操作的性质，\*pfunc返回**函数类型变量**，其表示值类型为函数指针
    由此可见<u>\*pfunc</u>与<u>pfunc</u>表达式的返回值是完全相同的

---

### 十八、Compound Literal

- 复合字面量：**匿名**对象类型，其定义为 (type-name)\{ initializer-list \}

  - type-name：该对象的**变量类型**
  - initializer-list：对该对象进行**初始化**的列表

  **注意**：符合字面量即**匿名的变量**，除了不含Name外与非数组/数组变量类型类似

- 赋值问题：复合字面量本身是合法的**左值**，但其是否为 modifiable lvalue 取决于存储位置：

  - 位于函数内部：生命周期为 automatic storage duration，是 modifiable lvalue
  - 作为全局变量：生命周期为 static storage duration，不是 modifiable lvalue，<u>**只能初始化**</u>

  **注意**：复合字面量是合法的左值，可以<u>定位一块内存</u>，并能够与 &、sizeof 结合

- 空间分配规则：<u>同一作用域内</u>，各 Compound Literal 在内存中**仅保持一份**
  <br>**注意**：由上述特点可知，在循环中反复声明**同值**复合字面量，实际获得的是**同一内存块中的数据**

- 使用**同一个**复合字面量：使用对应的**指针类型存储**变量地址，通过指针解引用操作访问

---

### 十九、Type

- Object Type：定位一个**内存块**，可通过变量类型解析出一个值

  - Integer Related：主要包括以下类型：

    1. Signed Integer Type：标准有符号整型（signed）+ 扩展有符号整型
    2. Unsigned Integer Type：布尔型（\_Bool）+ 对应无符号整型（unsigned）+ 扩展无符号类型 

  - Floating Type：主要包括以下类型：

    1. Real Floating Type：标准浮点类型（float、double）+ 高精度浮点类型（\_Decimal32等）
    2. Complex Type：复数浮点类型（float\_complex、double\_complex）
       <br>**注意**：C语言中并未规定必须支持复数类型

  - Character Related：字符类型（char）+ 有符号字符类型（signed）+ 无符号字符类型（unsigned）
    <br>**注意**：对于平时使用的char，其是否为有符号类型取决于平台，一般只用于存储ASCII码（0~127）

  - Enumerated Type：定义一系列<u>文字表述的**整数**</u>

    ---

  - Basic Type ：char + Signed Integer Type + Unsigned Integer Type + Floating Type，均为**完全对象类型**

  - Integer Type ：char + Signed Integer Type + Unsigned Integer Type + Enumerated Type

  - Arithmetic Type ：Integer Type + Floating Type

  ---

- Derived Type：可由其它类型**递归**地构造出来，包括数组类型、结构体类型、**函数类型**、指针类型等

  - Array Type：给定任何一个Object Type，都可以其为元素类型构造对应的**一维数组**类型
    <br>**注意**：对于“高维数组”类型，均可通过“低维数组”类型递归地构造

  - Function Type：其定义包括返回值类型 + 参数数量及类型

  - Pointer Type：给定任何一个Type（Reference Type），都有对应的**指针类型**（Pointer Type）
    <br>**注意**：任何指针类型都是一个普通的Object Type

    ---

  - Scalar Type ：Arithmetic Type + Pointer Type（可进行标量加减操作）

  - Aggregate Type ：Array Type + Struct Type
    <br>**注意**：Union Type不是聚合类型

  - Incomplete Object Type：缺少确定object大小的信息，如数组缺少元素个数、void类型、包含不完全类型的结构体或联合体等

---

### 二十、赋值表达式

- 赋值操作符：包括 =、+=、>>= 等赋值操作符

- 赋值表达式：定义为 exp1 assignment\_operator exp2，其中exp1一定是modifiable lvalue

  - C：上面表达式不是lvalue，其返回值为exp1更新过的值
  - C++：上面表达式是lvalue，指向exp1定位的、已被更新的对象

- 后缀加减 or 前缀加减：exp++ or ++exp

  - exp++：<u>**返回值为exp的值**</u>，**副作用**是让exp对应对象的值+1；exp++不是lvalue
  - ++exp：等价于`exp += 1`，返回值是**exp更新后的值**，故其在C语言中是lvalue、而在C++中不是lvalue

  **注意**：为了维护程序的**可读性**，<u>GJB中规定禁止使用 ++、-- 操作符</u>

  ---

- Evaluation of Expression：主要包括两个步骤

  - Value Computation：计算表达式的返回值\<V, V\_T\>
  - Initiation of Side Effect：确定表达式的副作用，即执行状态的变化，如<u>改变内存值</u>
    <br>**注意**：副作用利用<u>等号右侧**表达式的返回值**</u>，改变等号左侧的**内存值**

  **注意**：a++ 和 ++a 这两个表达式都有各自的返回值，但什么时候真的+1（即副作用的出现顺序）则是**不确定**的

- Sequenced Before：描述两个表达式之间 evaluation 的先后顺序
  <br>exp1 Sequenced Before exp2 表示exp1的**求值与副作用**全部发生在exp2的**求值与副作用**之前

- Sequence Point ：exp1 Sequenced Point exp2 $\Leftrightarrow$ exp1 Sequenced Before exp2
  <br>常见的Sequence Point：<u>实参求值与函数调用</u>之间、<u>两个独立分隔的表达式</u>之间 等
  <br>**注意**：两个Sequence Point内部的表达式之间没有约定 evaluation 的先后顺序

- 有关自增/自减的undefined behavior：对于一个scalar object，如果满足其中一种条件：

  - 产生**两次副作用**，且两次副作用之间没有先后顺序的约定，如`int i = 1; i = ++i + 1;`
    <br>若 ++i 副作用先执行，则最后将 i 赋值为表达式 ++i + 1 的返回值3
    <br>若 i = ++i + 1 副作用先执行，则先将 i 赋值为 ++i + 1 的返回值3，最后 i 自增得到4
  - 产生的**副作用**与**对其取值**之间没有先后顺序的约定，如`int i = 1; a[i++] = i;`
    <br>若 i++ 的副作用发生在等号右侧对 i 求值之前，则 i 先自增为2，再将 **a\[1\] 赋值为2**
    <br>若 i++ 的副作用发生在等号右侧对 i 求值之后，则先将 **a\[1\] 赋值为1**，i 再自增为2
    <br>**注意**：表达式 i++ 的返回值始终为1，故均是对 a\[1\] 赋值

  则最终内存结果不确定，取决于编译器的优化方法等行为

---

### 二十一、volatile限定符

- 变量声明：`Var_T volatile N;`，其中变量N的表示值类型为Var\_T
  <br>volatile 限定符表示这块内存的值可能**以未知的方式变化**
- Volatile 在 MMIO 中的应用：
  - MMIO：内存和I/O共享同一个地址空间
  - 定义外设映射内存：`#define NUM (*(int volatile*)0x12340000)`
    <br>这样变量NUM的内存值可能被**硬件**而非程序改变，表现为NUM值不可控
- abstract machine 和 volatile ：“抽象机”是一种<u>不考虑优化</u>的模型；volatile 修饰的变量遵循“抽象机”模型
  <br>每次访问 volatile 修饰的变量都会严格从**内存**中获取值，而不是直接从**寄存器等缓存**中获取值
- volatile 的使用注意事项：由于每次都只从内存中读数据，而数据又可能被硬件改变，所以每次值都不确定
  - 编译器优化：如果从<u>寄存器等缓存</u>中读值，结果就是**正确**的；如果只从<u>内存</u>中读值，结果就是**不确定**的
  - volatile access：获取被 volatile 修饰的值的行为是**副作用**
- Var\_T volatile 对应的指针类型：变量类型 和 表示值类型 都是Var\_T volatile\*（Var\_T volatile是一个**整体**）
- Var\_T const volatile：从<u>程序代码角度</u>看是**只读**的，但从<u>硬件角度</u>看是会被**随机修改**的

---

### 二十二、Literal

- Literal的含义：仅从**字面义**去理解，不做任何演绎

- String Literal：按**字面意思**理解的字符串（不包含转义），其内存值无法修改（编译警告、运行出错）

- Compound Literal：按**字面意思**理解 (type-name)\{ initializer-list \} 这种组合形式，其内存值可以修改

  ---

- C语言常量（constant）：整数常量 + 浮点常量 + 枚举常量 + 字符常量
  <br>**注意**：常量不是左值，不能像 Literal 和 const修饰变量一样**定位一块内存**

- C++中的Literal：主要包括integer-literal、floating-literal、character-literal和string-literal

  - string-literal：在C++中，其返回值类型为**const** char\*；若尝试修改string-literal的值则会直接编译出错
    <br>**注意**：C语言中string-literal的返回值类型为char\*；**const** char\* 体现literal在C++中具有**常量**的含义
  - compound-literal ：C++中没有compound-literal，程序移植时要特别注意

  **注意**：C++中的literal，对应C中的constant

---

### 二十三、padding

- 字节大小：C语言中规定一个Byte含有 **CHAR\_BIT** 个二进制位，其中 **CHAR\_BIT** 规定要 $\ge$ 8 
  <br>**注意**：“1Byte = 8bits”是长期发展形成的工业标准，部分平台的字节大小大于8

- padding：**填充**类型中剩余的二进制位；设某类型的 size 为 n $\times$ CHAR\_BIT 个比特，即 n 个字节：

  - 无符号整数类型：包含 value bits 和 padding bits
    <br>设 value bits 共有N位（宽度为N），则该类型的表示值范围是 0 ~ $2^N - 1$

  - 有符号整数类型：包含 sign bit，value bits 和 padding bits
    <br>设 sign bit 和 value bits 共有N位（宽度为N），则该类型的表示值范围是 $-(2^{N-1})$ ~ $2^{N-1} - 1$

    **注意**：其中C语言规定 unsigned char 和 signed char 类型均不允许有 padding bits

 - 整型变量 int / unsigned int：规定宽度N必须$\ge$16，具体**宽度不确定**

 - intN\_t 和 uintN\_t：表示无 padding，且宽度**确定为N**的整型变量
   <br>**注意**：若某平台提供了无 padding，且宽度为32位的int类型，则应该 `typedef int int32_t;`

 - int\_leastN_t 和 uint\_leastN\_t：表示宽度**至少为N**的整型变量
   <br>**注意**：C语言规定编译器必须定义含 least 的整数类型；若编译器定义了intN\_t，则 int\_leastN\_t 和 intN\_t相同

 - int\_fastN\_t 和 uint\_fastN\_t：表示**宽度至少为N**，且处理速度最快的整型变量（最快最小）
   <br>**注意**：C语言规定编译器必须定义含 fast 的整数类型；fast不代表总是最快，而是代表**尽可能满足速度要求**

---

### 二十四、alignment

- 地址对齐：规定对象内存的**首地址**的限制；对齐值必须是2的n次方
  <br>**注意**：“对象对齐”是指 <u>对象首地址 % 对象的alignment == 0</u>，一般char的对齐数为1、int的对齐数为4

- \_Alignof ：对于类型为T的对象O，\_Alignof(T) 表示类型T的对齐数，\_Alignof(O) 表示对象O的对齐数
  <br>**注意**：对象O的对齐数默认为**其类型T的对齐数**，且可被修改

- \_Alignas：为类型为T的对象O设置**更大的对齐数**，声明句`_Alignas(N) T O;`表示将对象O的对齐数**调大到N**

  - 修改<u>非数组类型</u>的对齐数：\_Alignas(64) int a; $\Rightarrow$ \_Alignof(a) == 64；且int的**大小和类型对齐数均不变**
  - 修改<u>数组类型</u>的对齐数：数组类型的对齐数 == 其**元素类型**的对齐数（**递归定义**）
    \_Alignas(64) int a\[N\]; $\Rightarrow$ \_Alignof(a) == 64；且int\[N\]的**大小和类型对齐数均不变**

  **注意**：通过 \_Alignas 改变对象的对齐数，并不会改变该对象的size，也不会改变其类型的对齐数

- 结构体的对齐要求：对于类型T为为结构体类型的对象O，\_Alignof(T) == max(\_Alignof($\text{E}_i$)) 
  <br>其中$\text{E}_i$表示结构体中第 i 个**成员对象的对齐数**

  - 通过 \_Alignas($\text{E}_i$) 调整<u>结构体成员对象</u>的对齐数，会改变该结构体类型的对齐数
  - 通过 \_Alignas(stru) 调整<u>结构体对象</u>的对齐数，会使该对象的对齐数 > 结构体类型对齐数


---

### 二十五、结构体类型的size

- 计算结构体类型的大小：遍历所有成员对象，确定各成员对象的 offset 和 padding；对于每个成员对象：

  1. 根据<u>该**对象**的对齐数</u>，找到满足其**对齐要求**的**最小offset**
     该对象与上一个成员对象间的空白距离称为 **internal padding**
  2. 将该成员对象O填充在 offset 处，其占据了sizeof(O) 的内存空间

  填写了所有成员变量后，根据<u>该结构体**类型**的对齐数</u>，在末尾留出**最小trailing padding**

  $\Rightarrow$ sizeof(T) = $\sum$ sizeof($\text{O}_i$) + $\sum$ internal padding + trailing padding

  **注意**：由上述过程可知，结构体类型的size是其对齐数的**整数倍**

- \#pragma pack(n) ：调整结构体**内部**对象的对齐要求
  <br>对于结构体类型T，设其第 i 个成员对象为$\text{E}_i$，则 \_Alignof($\text{E}_i$) = min(\_Alignof($\text{E}_i$),  n)
  <br>**注意**：对于结构体类型T，\_Alignof(T) = max(\_Alignof($\text{E}_i$))，该规则保持不变，用于最后计算 trailing padding

- 申请**指定对齐要求**的空间：`void* aligned_alloc(size_t alignment, size_t size);`

- 指针类型强转的对齐问题：对指针表达式的类型强制转换后需要保持**地址对齐**
  <br>如 char\*  类型的内存对齐数为1，int\* 类型的内存对齐数是4，则`char* p; (int*)(p + 1);`会不对齐，属于ub

---

### 二十六、restrict限定符

- **指针**变量声明：`Var_T* restrict p;`
  <br>**注意**：restrict 只能修饰指针变量类型
- restrict 的使用：设 p 为 restrict 修饰的指针变量，则程序员需要保证<u>在指针 p 的生命周期内</u>，其指向的对象不会被其它指针**同时引用**
- restrict 与编译器优化：对于两个位于<u>同一作用域</u>的指针变量，若其指向了不同的内存块，编译器可以采用**载入立即数**等方式修改两个内存块数据，以达到**优化代码**的目的（减少访存次数）
