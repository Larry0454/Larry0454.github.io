---
title: 『machine learning-1』overview
date: 2024-03-28 08:18:27
tags: machine learning
---

## 概述

---

### 一、与机器学习相关的学科

- 人工智能
- 智能信息处理
- 模式识别
- 数据挖掘
- 计算机视觉
- 自然语言处理

---

### 二、机器学习算法

- 机器学习的主要问题：

  <figure style="text-align:center;">
    <img src = "典型算法.png" width=75% height=75%>
    <figcaption>机器学习典型算法</figcaption>
  </figure>

- 监督学习：利用**已标记**的数据训练模型；$\{\text{x}_i \in \mathbb{R}^d, y_i \in \mathbb{R} \}_{i=1}^N$

  <figure style="text-align:center;">
    <img src = "监督学习.png" width=75% height=75%>
    <figcaption>监督学习</figcaption>
  </figure>

- 非监督学习：利用**未标记**的数据训练模型，需要模型**自行发掘**数据中的特征；$\{\text{x}_i \in \mathbb{R}^d\}_{i=1}^N$

  <figure style="text-align:center;">
    <img src = "非监督学习.png" width=75% height=75%>
    <figcaption>非监督学习</figcaption>
  </figure>

- 半监督学习：训练数据集中同时混有**已标记和未标记**的数据

  **注意**：“监督学习”与“非监督学习”的差异在于训练集中是否含有**目标值**

---

### 三、训练与测试

- No Free Lunch ：没有一种**单一**的机器学习算法能够在**所有**问题上表现得最好

- 影响机器学习结果的因素：

  1. 训练的类型
  2. **初始背景知识**的形式与范围
  3. 训练**反馈**的类型
  4. 机器学习**算法**

- 机器学习的目标：

  - 监督学习：尽可能缩小**测试集**上的预测误差，其中测试集误差 $\text{Error}_{out} = \dfrac{1}{N} \sum_{i=1}^N (y_i \ne g(x_i))$
  - 非监督学习：缺少预先定义的**标准答案**；一般结合实际领域与需求对测试结果进行解释与判断

- “偏差”与“方差”：

  - 偏差：用所有可能的训练集训练出的**所有模型的输出<u>平均值</u>**与真实模型输出值之间的差异
  - 方差：用**不同训练集**训练出的模型输出值之间的差异

- “过拟合”与“欠拟合”：

  - 过拟合：模型过于**复杂**，学习了训练集中大量无关特征（噪声）
    <br>特点：低偏差 + 高方差（抗噪性弱）、低训练集误差 + 高测试集误差

  - 欠拟合：模型过于简单，无法充分学习数据中相关特征
    <br>特点：高偏差（算法拟合能力差） + 低方差、高训练集误差 + 高测试集误差

    **注意**：“偏差”与“方差”是矛盾的

  <figure style="text-align:center;">
    <img src = "模型复杂度.png" width=75% height=75%>
    <figcaption>训练集误差（Ein）、测试集误差（Eout）与模型复杂度（dvc）之间的关系</figcaption>
  </figure>

- 泛化误差：$\text{GE} = \text{noise}^2 + \text{bias}^2 + \text{variance}$
  <br>其中 noise 是**训练集的噪声**，不可避免；bias 由**模型错误的假设**造成；variance 由各**训练集间**的偏差造成
