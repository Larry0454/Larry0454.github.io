---
title: 『machine learning-6』SVM
date: 2024-03-28 08:30:05
tags: machine learning
---

## 支持向量机

---

### 一、支持向量

- 超平面：设训练样本集$D = \{(x_1, y_1), ..., (x_m, y_m\}$，其中$y_i \in \{-1, +1\}$ 

  超平面可将二分类样本集D划分为两部分，实现**正确分类**
  
- 支持向量与二分类：设超平面为$\omega^T x + b = 0$，其中$\omega = (\omega_1; \omega_2; ...; \omega_d)$ 为超平面**法向量**

  设对于训练样本$(x_i, y_i) \in D$，通过以下方式实现分类：
  $$
  \begin{cases}
  \omega^T x_i + b \ge +1 & y_i = +1 \\
  \omega^T x_i + b \le -1 & y_i = -1
  \end{cases}
  $$
  则称<u>距离超平面**最近**</u>的，<u>满足上式约束</u>的训练样本为**支持向量**

  两异类支持向量<u>到超平面的距离之和$\gamma$</u>称为**间隔**：
  $$
  \gamma = \frac{2}{|{|{\omega}||}}
  $$

- 支持向量机的**基本型**：求解两类样本“**正中间**”的划分超平面，即求解**间隔最大**的超平面
  $$
  \begin{align}
  & \underset{\omega, b}{\min} \quad \frac{1}{2} |{|{\omega}||}^2 \\ \\
  & \text{s.t.} \quad y_i(\omega^T x_i + b) \ge 1, \quad i=1,2,..,m
  \end{align}
  $$

---

### 二、对偶问题

- 拉格朗日函数：设超平面 $f(x) = \omega^T x + b$，SVM基本型的拉格朗日函数可以写成：
  $$
  L(\omega, b,  \alpha) = \frac{1}{2} |{|{\omega}||}^2 + \sum_{i=1}^m \alpha_i(1 - y_i(\omega^Tx_i + b))
  $$
  其中每个约束的拉格朗日乘子$\alpha_i \ge 0$，<u>令L对$\omega$和b的偏导为0</u>：
  $$
  \begin{align}
  \omega &= \sum_{i=1}^m \alpha_i y_i x_i \\
  0 &= \sum_{i=1}^m \alpha_i y_i
  \end{align}
  $$
  将上式代入拉格朗日函数得到**对偶函数**：
  $$
	\underset{\omega, b}{inf} L(\omega, b, \alpha) = \sum_{i=1}^m \alpha_i - \frac{1}{2} \sum_{i=1}^m \sum_{j=1}^m \alpha_i \alpha_j y_i y_j x_i^T x_j \\ \\
	$$
	
	SVM基本型的**对偶问题**（即求解SVM基本型的**最优下界**）：
	$$
	\begin{align}
	& \underset{\alpha}{max} \quad \underset{\omega, b}{\text{inf}} \quad L(\omega, b, \alpha) \\
	& s.t. \quad \sum_{i=1}^m \alpha_i y_i = 0 \\
	& \quad \quad \quad \alpha_i \ge 0
	\end{align}
	$$
	**注意**：由约束条件，总有$inf L(\omega, b, \alpha) \le L(\omega, b, \alpha) \le  \frac{1}{2} \abs{\abs{\omega}}^2 $
	
	求解对偶问题得到$\alpha$，再根据$\alpha$解出$\omega$与b，最终获得划分超平面 $f(x) = \sum_{i=1}^m \alpha_i y_i x_i^Tx + b$

---

### 三、核函数

- 核函数的用途：将样本空间**映射到高维**，以解决原始数据**线性不可分**的问题

- 核函数定义：$k(x_i, x_j) = <\phi(x_i), \phi(x_j)> = \phi(x_i)^T \phi(x_j)$

  其中$\phi(x)$表示将向量$x$映射到更高的特征空间，在<u>特征空间</u>中划分超平面的形式变成$f(x) = \omega^T \phi(x) + b$

  最终求解可得到 $f(x) = \sum_{i=1}^m \alpha_i y_i k(x_i, x_j) + b$，其中$\alpha_i \ge 0$ 是SVM基本型对应的拉格朗日乘子

- 核函数定理：设$X$为输入空间，$k(\cdot,\cdot)$是$X \times X$上的**对称函数**

  则$k$是核函数 $\leftrightarrow$ 对任意输入数据$D = \{x_1, ..., x_m\}$，核矩阵$(k(x_i, x_j))_{m \times m}$是**半正定**的

- 常用核函数：直接计算高维向量的内积较为复杂，故可使用核函数**直接应用于原始样本**

  1. 线性核：$k(x_i, x_j) = x_i^Tx_j$
  2. 多项式核：$k(x_i, x_j) = (x_i^Tx_j)^d$，其中d为多项式<u>次数</u>
  3. 高斯核：$k(x_i, x_j) = e^{-\frac{|{|{x_i - x_j}||}^2}{2\sigma^2}}$，其中$\sigma \gt 0$为高斯核<u>带宽</u>
  4. 拉普拉斯核：$k(x_i, x_j) = e^{-|{|{{x_i - x_j}}||^2}{\sigma}}$，其中$\sigma \gt 0$
  5. Sigmoid核：$k(x_i, x_j) = tanh(\beta x_i^Tx_j + \theta)$，其中$\beta > 0$，$\theta \lt 0$

	**注意**：核函数隐式定义了特征空间，应**选取合适的核函数**将样本空间映射到合适的特征空间
