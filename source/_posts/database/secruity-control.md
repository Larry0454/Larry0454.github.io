---
title: 『database-3』secruity control
date: 2024-03-27 21:16:12
tags: database
---

## 关系数据库标准语言 SQL

---

### 一、有关SQL

- SQL的特点：

  - 综合统一：结合了 DDL、DML、DCL 三种关系数据库语言的功能
  - 高度**非过程化**、易学易用
  - 面向集合的操纵方式

  <figure style="text-align:center">
      <img src="SQL功能.png">
      <figurecap>9个动词实现SQL核心功能</figurecap>
  </figure>
  
 - 基本表与导出表：

   - 基本表：实际存在的表，真正存储在物理文件中
   - 导出表：从**基本表导出**的表，是一个虚表，其真实数据不存在数据库中，只是在<u>数据字典</u>中存储其**定义**
     <br>**注意**：视图（View）经过定义（Create View）就可以像基本表那样操作
   
 - 关系数据库的三级模式结构：SQL的关系数据库同样支持三级模式结构

   <figure style="text-align:center">
       <img src="三级模式.png">
       <figurecap>三级模式结构</figurecap>
   </figure>

   **注意**：存储文件的**逻辑结构**组成了内模式；存储文件的物理存储结构对用户隐藏

---

### 二、SQL数据查询功能

- SQL查询语句块：SELECT-FROM-WHERE，其中：
  - SELECT：选取**目标列**，表示目标检索项
  - FROM：指明**基本表或视图**，可以由<u>一个或多个表</u>组成
  - WHERE：选取符合检索条件的**元组**
  
- SQL投影查询：**SELECT** A1, ..., An FROM S ...，表示从关系S中抽取A1列、A2列
  <br>SELECT **DISTINCT** A FROM S ... 表示从关系S中抽取A列后，**去除重复行**
  
- SQL选取检索：通过**WHERE**子句引出查询条件；检索条件由<u>属性名、常量、比较与布尔运算符</u>（AND等）组成

- SQL排序检索：在<u>查询语句块**最后**</u>接 **ORDER BY** A1, ..., An (ASC or DESC)，将各行**按指定列排序**
  <br>**注意**：若不指定升序或降序，默认按升序排列；可以按照单列或多列（多关键字）排序
  
- SQL连接操作：查询涉及两个以上表的信息，将多个由**连接属性**相关联的表按照一定条件连接起来
  1. SELECT：指出最终选取的所有**列名**（可来自多个表，必要时使用关系名修饰）
  2. FROM：指出需要进行连接的**各关系名称**
  3. WHERE：指出**元组连接谓词**（表1.属性A $\theta$ 表2.属性B） + 连接后的**选取条件**
  
- 自连接操作：给一张表<u>复制为两个非同名表</u>，对其进行连接操作

- 外连接操作：在连接谓词的某一端加一个**星号\***，表明为所在端的表增加一**空行**
  <br>该空行可以与另一表中<u>不满足连接条件</u>（即被舍弃）的元组连接，从而**可将悬浮元组输出**
  
  ---
  
- SQL子查询嵌套检索：查询间可以嵌套（**结构化查询**的体现）
  - 子查询的分类：普通子查询 + 相关子查询
    - 普通自查询：子查询的查询条件**不依赖于父查询**
    - 相关子查询：子查询的查询条件**依赖于父查询**
    
    **注意**：不相关子查询可以先直接求出子查询的返回值，再求解父查询；相关子查询中父子要交替进行
    
  - 查询语句块：在父查询的**WHERE后**用括号包裹**新的S-F-W查询块**
  
  - 子查询的返回值：可以返回一个单值、也可以返回一组值
  
    - 返回单值：可在 WHERE 中直接使用比较运算符
    - 返回集合：需使用 IN、ANY 等关键字
  
  - 嵌套 IN 谓词：满足筛选条件的属性值存在于**子查询的返回值集合**中，如 IN (child)
  
  - 嵌套比较运算符：若能够确认子查询**仅返回单值**，则父查询的 WHERE 中可以使用比较运算符
  
    <figure style="text-align:center">
        <img src="嵌套查询.png" height=75% width=75%>
    </figure>
  
  - 嵌套 ANY（ALL）的子查询：子查询<u>返回多值</u>时使用 **比较运算符 + ANY（ALL）**，如：
    <br>“> ANY(child)” 表示大于子查询child 返回的**某个值**
    <br>“$\le$ ALL(child)” 表示小于等于子查询child 返回的**全部值**
    <br>“ = ANY(child)” 表示等于子查询child 返回的**某个值**
    **注意**：=ANY 等价于 IN 谓词；!= ALL 等价于 NOT IN 谓词
  
  - 嵌套 EXISTS 的子查询：**EXISTS(child) = true** $\Leftrightarrow$ child 查询结果**非空**；EXISTS 不返回查询数据
  
    - EXISTS表示**全称**：p总是满足 $\Rightarrow$ **不存在**p不满足的情况
  
      <figure style="text-align:center">
          <img src="存在表全称.png">
          <figurecap>全称常用于表示“全选”</figurecap>
      </figure>
  
    - EXISTS表示**蕴含**：若p满足则q满足 $\Rightarrow$ 不存在p满足但q不满足的情况
  
      <figure style="text-align:center">
          <img src="存在表蕴含.png">
          <figurecap>蕴含常用于表示“条件全选”</figurecap>
      </figure>
  
  ---
  
- 并、差、交检索

  - 并集：SELECT ... **UNION** SELECT ...
  - 交集：SELECT ... **INTERSECT** SELECT ...
  - 差集：SELECT ... **MINUS** SELECT ...

- 库函数检索：聚集函数

  - COUNT(A)：求出<u>属性值为A</u>的行数；COUNT(\*)：求出<u>总</u>行数
  - SUM(A)：对数值列A求总和
  - AVG(A)：对数值列A求平均值
  - MAX(A)：在列A中找出最大值；MIN(A)：在列A中找出最小值

  **注意**：库函数检索只能在 SELECT 或 HAVING 子句中出现

- 分组检索：**GROUP BY** 若干列名 \[**HAVING** 条件表达式\]

  - GROUP BY 若干列名：表示**分组依据**；同组元组聚在一起，同一组内的列值不可完全相同
  - HAVING 条件：将不满足条件的组筛除（可选项）

  **注意**：分组后SELECT中的**聚集函数**会<u>为每个组都计算一个结果</u>，不分组则直接按整列计算
  <br>注意**编写顺序**：WHERE（先选出符合条件的行）$\rightarrow$ GROUP BY（确定分组列）$\rightarrow$ HAVING（最后筛选组）

- 算术表达式的检索：SELECT子句中可包含由<u>属性列 + 常量 + 库函数 + 算术运算符</u>构成的**算术表达式**

- 部分匹配查询：列名  **LIKE** / **NOT LIKE** 字符串常量（模糊查询）

  - 列名：必须为字符串型
  - 字符串常量：可包含 %（匹配任意**多个**字符）和 _ （匹配任意**单个**字符）

- 基于派生表的查询：SELECT ... FROM **(child) AS <u>child\_table</u>(A1, ..., An)**，即需要给派生表指定一个**别名**
  <br>**注意**：若子查询中<u>未使用聚集函数</u>，就不必为派生表指定列名，其列名默认为子查询SELECT的列名

---

### 三、SQL数据定义功能

- 定义基本表：其基本格式为如下

  <figure style="text-align:center">
      <img src="定义表.png">
      <figurecap>定义表的格式</figurecap>
  </figure>

  **注意**：新建立的表是空表，表的定义及有关约束存放在数据字典中

  - SQL92的数据类型：

    <figure style="text-align:center">
        <img src="数据类型.png">
        <figurecap>部分基本数据类型</figurecap>
    </figure>

  - 完整性约束：

    <figure style="text-align:center">
        <img src="完整性约束.png">
    </figure>

- 删除基本表：**DROP TABLE** <表名> \[RESTRICT | CASCADE\]

  - RESTRICT：表示删除表**有限制条件**，默认RESTRICT有效
  - CASCADE：表示删除表**没有限制条件**

- 修改基本表：其基本格式如下

  <figure style="text-align:center">
      <img src="修改表.png">
      <figurecap>修改表的格式</figurecap>
  </figure>

  ---

- 定义索引：索引可以建立在**一列**或**多列**上，其基本格式如下

  <figure style="text-align:center">
      <img src="建立索引.png">
  </figure>

  - UNIQUE：表明该索引的<u>每个索引值</u>仅对应<u>唯一的数据记录</u>
  - CLUSTER：表明要建立的索引是**聚簇索引**
  - 次序：可以选择为该列上的索引值设置**排列次序**，默认为ASC

  **注意**：索引是<u>加快查询速度</u>的有效手段，属于内模式的范畴，无需用户干预

- 修改索引**名**：**ALTER INDEX** <旧索引名> RENAME TO <新索引名>

- 删除索引：**DROP INDEX** <索引名>
  <br>**注意**：数据的增删改会导致<u>维护索引的开销</u>，可以考虑删除不必要的索引
  
  ---
  
- 定义视图：视图是一个虚表，其数据存于原先的基本表中，<u>随基本表改变而变化</u>
  
  <figure style="text-align:center">
      <img src="建立视图.png">
  </figure>
  
  - 子查询：任意的 SELECT 语句
  - WITH CHECK OPTION：子查询中的谓词条件表达式
  
  **注意**：视图的各列名可以省略（或全部指定），默认为 SELECT 子句的各属性字段
  <br>不能省略的情况：目标列使用了**聚集函数**；多表连接中将**各表同名列**作为目标列；需要换列名称
  
- 删除视图：**DROP VIEW** <视图名> \[CASCADE\]
  
- 查询视图：使用<u>与基本表查询相同</u>的方式（SELECT-FROM-WHERE）对视图进行查询
  
  - 视图消解：从数据字典中取出**视图定义**，将<u>视图子查询</u>和<u>用户查询</u>结合，转化为**等价的对基本表的查询**
  
    <figure style="text-align:center">
        <img src="视图查询.png">
    </figure>
  
  - 视图的作用：视图是数据库的“窗户”，可以观察用户感兴趣的数据及变化
  
    1. 为同一数据赋予**多种观察角度**
    2. 提供一定的**逻辑独立性**
    3. 对数据提供**安全保护**
  
---

### 四、SQL数据更新功能

- 插入数据：可分为插入元组或插入子查询结果

  - 插入元组：**INSERT INTO** \<表名\> (若干属性列) VALUES (若干元组)
  - 插入子查询结果：**INSERT INTO** \<表名\> （若干属性列）\<SELECT 子查询\>
  
- 修改数据：**UPDATE** \<表名\> **SET** (若干 \<列名\> **=** \<表达式>)  \[WHERE \<条件\>\]，表示修改符合条件的**行的属性值**
  <br>**注意**：修改元组使用UPDATE，而修改整个表的特性使用ALTER
  
- 删除数据：**DELETE FROM** \<表名\> \[WHERE \<条件\>\]，即**删除符合条件的行**
  
---

### 五、SQL数据控制功能

- 授权与回收权限：安全控制
  - 授权：**GRANT** \<权限> \[ON <对象类型> \<对象名>\] **TO** \<用户\>
  - 回收权限：**REVOKE** \<权限> \[ON \<对象类型\> \<对象名\>\] **FROM** \<用户>

---

### 六、空值运算

- 空值（NULL）的含义：不知道（如漏填）、数据缺失（如缺席）、不便填写（如隐私信息）

- 判断空值：WHERE 中使用条件语句 IS NULL 或 IS NOT NULL

- 空值的计算：空值 ArithmaticOp 任意值 = 空值、空值 CompareOp 任意值 = Unknown

- 空值的逻辑运算：T AND U = U、F OR U = U、NOT U = U

  **注意**：含 NOT NULL、UNIQUE **约束** 或 **主码属性**都不能取 NULL

---

### 七、嵌入式SQL

- SQL的实际应用：内嵌在高级语言（如C语言）中，将<u>SQL的特性</u>与<u>程序设计语言的特性</u>相结合

- 内嵌SQL：将SQL**预编译**为高级语言源码，再按诸语言方式进行编译

  - 区分SQL与宿主语言：在SQL语句前**增加前缀EXEC SQL**
  - SQL与宿主语言通信：**SQL通信区**负责向宿主语言传递执行状态；宿主语言设置**主变量**与SQL交换数据
  - SQL与宿主语言的协调：通过**游标**缓存SQL的执行结果

- ODBC与JDBC：基于X/Open和ISO/IEC的SQL调用级接口规范

  - 功能：建立与数据库的连接；发送SQL语句；处理结果

    **注意**：JDBC在设计思想上沿袭了ODBC

  <figure style="text-align:center">
      <img src="JDBC.png">
      <figurecap>JDBC四大组件</figurecap>
  </figure>
