---
title: 『machine learning-3』Bayes
date: 2024-03-28 08:22:30
tags: machine learning
---

## 贝叶斯决策

---

### 一、贝叶斯公式

- 相关术语：

  - 样本：$x \in \text{R}^d$，表示共有d个属性的样本
  - 类别（状态）：$\omega_i$，表示第 i 个标签
  - 先验概率：$P(\omega_i)$，表示类别为 $\omega_i$ 的样本的分布
  - 类条件概率：$P(x \text{ | } \omega_i)$，表示**类别 $\omega_i$**中样本$x$的比例
  - 后验概率：$P(\omega_i \text{ | }x)$，表示给定**任意样本**$x$，判断其属于类别 $\omega_i$ 的概率

- 贝叶斯公式：已知<u>先验概率</u> $P(\omega_i)$ 和<u>条件概率</u>$P(x \text{ | }\omega_i)$，计算**后验概率** $P(\omega_i \text{ | }x)$
  $$
  \begin{align}
  P(\omega_i \text{ | }x) &= \frac{P(x \text{, }\omega_i)}{P(x)} \\
  &= \frac{P(x \text{ | }\omega_i)P(\omega_i)}{\sum_{i=1}^c P(x \text{ | }\omega_i)P(\omega_i)}
  \end{align}
  $$
  **注意**：贝叶斯公式中，分子使用**乘法公式**，分母使用**全概率公式**

---

### 二、贝叶斯决策论

- 条件风险：设共有N种类别 $C = \{\omega_1, \dots \omega_N\}$，将真实类别为 $\omega_j$ 的样本$x$<u>误分类</u>为 $\omega_i$ 的**损失**为$\lambda_{ij}$
  则进行 $\omega_i$ 分类的条件风险 $R(\omega_i \text{ | }x)$ 为：
  $$
  R(\omega_i \text{ | }x) = \sum_{j=1}^N \lambda_{ij}P(\omega_j \text{ | }x)
  $$
  **注意**：$\lambda_{ii} = 0$，即**没有误分类**的损失为 0；否则为对应的误分类损失值
  
- **最小风险**贝叶斯决策：已知先验概率$P(\omega_i)$，类条件概率$P(x \text{ | }\omega_i)$（i = 1, 2, ..., N），待分类样本为 $x$

  1. 后验概率：根据<u>**贝叶斯公式**</u>计算各后验概率 $P(\omega_i \text{ | }x)$（i = 1, 2, ..., N）
  2. 考虑风险：根据所求<u>后验概率</u>和<u>损失函数</u>$\lambda$，计算各条件风险 $R(\omega_i \text{ | }x)$（i = 1, 2, ..., N）
  3. 做出决策：$\omega = \underset{i}{\text{argmin } }R(\omega_i \text{ | }x)$，即选出使**决策风险最小**的类别

- **最小错误率**贝叶斯决策：已知先验概率$P(\omega_i)$，类条件概率$P(x \text{ | }\omega_i)$（i = 1, 2, ..., N），待分类样本为 $x$

  1. 后验概率：根据<u>**贝叶斯公式**</u>计算各后验概率 $P(\omega_i\text{ | }x)$（i = 1, 2, ..., N）
  2. 做出决策：$\omega = \underset{i}{\text{argmax }}{P(\omega_i \text{ | }x)}$，即选出使**错误率最小**的类别

  **注意**：最小错误率贝叶斯决策即为<u>损失函数$\lambda$为 0 - **1** 条件下</u>的最小风险贝叶斯决策

- **朴素**贝叶斯决策：

  - 贝叶斯决策的问题：类条件概率$P(x \text{ | }\omega_i)$ 是样本$x$在**所有属性上的联合概率**，难以从<u>有限样本</u>中获取

  - 属性条件<u>独立性</u>假设：假设样本$x$的各个属性$x_i$之间**相互独立**，则类条件概率可做**拆分**：
    $$
    \begin{align}
    P(x \text{ | }\omega_i) &= P(x_1x_2\dots x_d \text{ | }\omega) \\
    &= \Pi_{i=1}^d P(x_i \text{ | }\omega)
    \end{align}
    $$
    **注意**：独立性假设有助于**降低对样本集的大小需求**，从而降低复杂度

  - 贝叶斯公式 + 属性独立性条件：朴素贝叶斯公式可以写为：
    $$
    \begin{align}
    P(\omega \text{ | }x) &= \frac{P(\omega)P(x\text{ | }\omega)}{P(x)} \\
    &= \frac{P(\omega)}{P(x)} \Pi_{i=1}^d P(x_i \text{ | }\omega)
    \end{align}
    $$
    相应的决策转变为：$\omega_k = \underset{j}{\text{argmin }}{P(\omega_j)\Pi_{i=1}^dP(x_i \text{ | }\omega_j)}$（假设对各类别 $P(x)$ 均相等）
  - 基于训练样本的估计：
  
    - 先验概率估计：$P(\omega_i) = \dfrac{|D_{\omega_j}| + 1}{|D| + N}$
      <br>其中 $|D_{\omega_j}|$表示样本集D中类别为$\omega_j$的样本数量，N表示样本集D中可能的类别数
    - 类条件概率估计：$P(x_i \text{ | }\omega_j) = \dfrac{|D_{\omega_j, x_i}| + 1}{|D_{\omega_j}| + N_i}$
      <br>其中 $|D_{\omega_j, x_i}|$表示样本$D_{\omega_j}$中样本第i个属性取值为$x_i$的数量，$N_i$表示第i个属性可能的取值个数
