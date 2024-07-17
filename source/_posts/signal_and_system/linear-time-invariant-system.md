---
title: 『signal and system-2』LTI
date: 2024-06-29 22:03:46
tags: signal and system
---

## 线性时不变系统（LTI）

### 一、线性时不变系统的定义

- 一个系统同时是<u>**线性**</u>且<u>**时不变**</u>的，全称是 Linear Time-Invariant system
- 研究 LTI 的意义：LTI 足够简单，如果已知一个 $x(t) \rightarrow y(t)$，那么可以知道其它所有 $x(t) \rightarrow y(t)$
  - 研究线性系统的意义：现实世界中没有可以**无限制放大**输出信号倍数，**无限制叠加**输出信号的系统
  - 研究时不变系统的意义：现实世界中没有可以**永远不随时间改变输出**的系统

---

### 二、离散 LTI 卷积

- 单位脉冲响应：记作 $h[n]$，定义为 $\delta[n] \xrightarrow{LTI} h[n]$，即<u>**单位脉冲序列**</u>经过 LTI 系统输出得到的信号

  > **注意**：单位脉冲响应 $h[n]$ **唯一标识**了一个 LTI；已知 $h[n]$，能够求出任意信号 x 输入该 LTI 输出的信号

- 列表法：计算时间复杂度为 $O(N^2)$

  - 已知 $h[n]$ 和输入 $x[n]$，求解卷积结果 $x[n] * h[n]$：设 $h[n]$ 中 n 的取值范围是 -2 ~ 0，$x[n]$ 中 n 的取值范围是 -1 ~ 2

    1. 确定 $x[n] * h[n]$ 的左右取值边界：

       - 卷积左边界 = $x[n]$ 的左边界 + $h[n]$ 的左边界 = -2 + -1 = -3
       
        - 卷积右边界 = $x[n]$ 的右边界 + $h[n]$ 的右边界 = 0 + 2 = 2
       
     > **注意**：$x[n]$ 取值长度为 $N_1$，$h[n]$ 取值长度为 $N_2$，则卷积结果长度为 $N_1 + N_2 - 1$
    
    2. 列表法求解卷积结果：将输入信号 $x[n]$ 的各取值依次与每一个 $h[n]$ 的取值**相乘**，并依次<u>**右移一个单位**</u>
    $$
       \begin{vmatrix}
     h[n] & h[-2] & h[-1] & h[0]  \\
       x[-1] & x[-1] * h[-2] & x[-1] * h[-1] & x[-1] * h[0]  \\
       x[0] & & x[0] * h[-2] & x[0] * h[-1] & x[0] * h[0]  \\
       x[1] & & & x[1] * h[-2] & x[1] * h[-1] & x[1] * h[0]  \\
       x[2] & & & & x[2] * h[-2] & x[2] * h[-1] & x[2] * h[0]  \\
       \text{列求和} & o(-3) & o(-2) & o(-1) & o(0) & o(1) & o(2) \\
       \end{vmatrix}
    $$
    
    3. 表格中<u>各列求和</u>即是输出信号中<u>从左到右</u>的取值，上表中各列的求和即为 -3 ~ 2 的值
    
    > **注意**：对于**无穷范围**的离散信号，同样可以使用列表法（无限延伸的表格），**找规律**求解

- 离散 LTI 卷积公式​⭐​：$x[n] * h[n] = \sum_{k = -\infty}^{+\infty} x[k]h[n-k]$，以下是证明过程​
    $$
    \begin{align}
    &\text{由单位脉冲响应的定义可知 } \delta[n] \xrightarrow{LTI} h[n] \\
    &\text{由LTI的时不变性可知 } \delta[n-k] \xrightarrow{LTI} h[n-k] \\
    &\text{由LTI的齐次性可知 }x[k]\delta[n-k] \xrightarrow{LTI} x[k]h[n-k] \\
    &\text{由LTI的叠加性可知 }\sum_{k = -\infty}^{+\infty} x[k]\delta[n-k] \xrightarrow{LTI} \sum_{k = -\infty}^{+\infty} x[k]h[n-k] \\
    &\text{由已知推论 }x[n] = \sum_{k = -\infty}^{+\infty} x[k] \delta[n-k]\text{ ，可知 } x[n] \xrightarrow{LTI} \sum_{k=-\infty}^{+\infty} x[k]h[n-k] \\
    &\text{由卷积运算的定义，}x[n]*h[n] = \sum_{k=-\infty}^{ +\infty} x[k]h[n-k]\text{ ，证毕。}
    \end{align}
    $$

    1. 确定 $x[n] * h[n]$ 的左右取值边界，该步骤与列表法完全一样

    2. <u>**翻转**</u> $h[n]$，再依次<u>**平移 $h[-n]$**</u> 与 $x[n]$ 逐元<u>**相乘**</u>再<u>**求和**</u>（$h[n]$ 翻转前是 $h[-2] \, h[-1] \, h[0]$）

       1. 将 $x[-1]*h[-2]$ 填入卷积结果的<u>第一个位置</u>：
          $$
          \begin{align}
          &x[-1] \,\,\,x[0] \,\,\, x[1] \,\,\, x[2] \\
          h[0] \,\,\, h[-1] \,\,\, &h[-2] \\
          \end{align}
          $$
          
       2. 将 $h[-n]$ 右移一个单位，并将 $x[-1] * h[-1] + x[0] * h[-2]$ 填入卷积结果的<u>第二个位置</u>：
          $$
          \begin{align}
          &x[-1] \,\,\,x[0] \,\,\, x[1] \,\,\, x[2] \\
          h[0] \,\,\, &h[-1] \,\,\, h[-2] \\
          \end{align}
          $$
       
       3. 依此类推，直至将 $h[n]$ 右移至以下情形，并将 $x[2] * h[0]$ 填入卷积结果的<u>最后一个位置</u>：
          $$
          \begin{align}
          x[-1] \,\,\,x[0] \,\,\, x[1] \,\,\, &x[2] \\
          &h[0] \,\,\, h[-1] \,\,\, h[-2] \\
          \end{align}
          $$
    
    > **注意**：离散 LTI 卷积公式要求"翻转谁"、就"平移谁"，如上例中翻转"翻转 $h[n]$"、就"平移 $h[-n]$"

---

### 三、连续 LTI 卷积

- 几个重要函数：

  - 无限窄方波函数：记作 $\delta_{\Delta}(t)$，表示时间宽度为 0 ~ $\Delta$，高度为 $\dfrac{1}{\Delta}$ 的一小段方波（面积 = 1）
  - 冲激函数：$\delta(t) = \lim_{\Delta \rightarrow 0} \delta_{\Delta}(t)$，即 无限窄方波函数的**极限**
  - 冲激响应：$x(t) \xrightarrow{LTI} h(t)$，其中 $h(t) = \lim_{\Delta \rightarrow 0} h_{\Delta}(t)$

- 连续 LTI 卷积公式⭐：$x(t) * h(t) = \int_{-\infty}^{+\infty}x(\tau)h(t-\tau)d\tau$，证明过程如下：
  $$
  \begin{align}
  &\text{由冲激响应定义可知 } \delta_{\Delta}(t) \xrightarrow{LTI} h_{\Delta}(t) \\
  &\text{由LTI的时不变性可知 } \delta_{\Delta}(t-k\Delta) \xrightarrow{LTI} h_{\Delta}(t - k\Delta) \\
  &\text{由LTI的齐次性可知 } x(k\Delta) \delta_{\Delta}(t - k\Delta) \Delta \xrightarrow{LTI} x(k\Delta)h_{\Delta}(t - k\Delta)\Delta \\
  &\text{由LTI的叠加性可知 } \sum_{k = -\infty}^{+\infty} x(k\Delta)\delta_{\Delta}(t - k\Delta)\Delta \xrightarrow{LTI} \sum_{k = -\infty}^{+\infty}x(k\Delta)h_{\Delta}(t - k\Delta)\Delta \\
  &\text{由已知推论 } x(t) = \lim_{\Delta \rightarrow 0} x_{\Delta}(t) = \lim_{\Delta \rightarrow 0} \sum_{k = -\infty}^{+\infty}x(k\Delta)\delta_{\Delta}(t - k\Delta)\Delta \text{ 可知，}x(t) \xrightarrow{LTI} \lim_{\Delta \rightarrow 0} \sum_{k = -\infty}^{+\infty}x(k\Delta)h_{\Delta}(t - k\Delta)\Delta \\
  &\text{由冲激响应的极限定义可知 } x(t) \xrightarrow{LTI} \lim_{\Delta \rightarrow 0} \sum_{k = -\infty}^{+\infty}x(k\Delta)h(t - k\Delta)\Delta = \int_{-\infty}^{+\infty} x(\tau)h(t-\tau)d\tau \text{ ，证毕。}
  \end{align}
  $$

- $\delta(t)$ 函数的 5 个性质：

  - $\int_{-\infty}^{+\infty} \delta(t) dt = 1$ = ($\int_{-\infty}^{+\infty} [\lim_{\Delta \rightarrow 0} \delta_{\Delta}(t)]dt = \lim_{\Delta \rightarrow 0} \int_{-\infty}^{+\infty} \delta_{\Delta}(t) dt = \lim_{\Delta \rightarrow 0} 1$)，即 $\delta(t)$ 在 $\mathbb{R}$ 上的积分为 1

  - $\int_{-\infty}^{+\infty} x(t) \delta(t) dt = x(0)$，证明如下：
    $$
    \begin{align}
    & \int_{-\infty}^{+\infty} x(t) \delta(t) dt \\
    =& \int_{-\infty}^{+\infty} x(t) (\lim_{\Delta \rightarrow 0} \delta_{\Delta}(t)) dt \\
    =& \lim_{\Delta \rightarrow 0} \int_{0}^{\Delta} x(t) \dfrac{1}{\Delta} dt = \lim_{\Delta \rightarrow 0} \dfrac{1}{\Delta} \int_{0}^{\Delta}x(t)dt \text{ （该积分仅在0到}\Delta\text{上有取值）} \\
    & \text{由积分中值定理，} \lim_{\Delta \rightarrow 0} \dfrac{1}{\Delta} \int_{0}^{\Delta}x(t)dt = \lim_{\Delta \rightarrow 0} \dfrac{1}{\Delta} x(\xi) \Delta = \lim_{\Delta \rightarrow 0} x(\xi) = x(0) \text{ ，证毕。}
    \end{align}
    $$

    > **注意**：性质2是性质1的特例，令 性质2 中的 x(t)=1 即可得到性质1

  - $x(t) \delta(t) = x(0) \delta(t)$，证明如下：
    $$
    \begin{align}
    &\text{Lebesgue 对函数 }f_1(x) \text{和} f_2(x) \text{ 相等的定义：对}\textbf{任意函数 }y(t)\text{ 有}\int_{-\infty}^{+\infty}y(t)f_1(t) dt = \int_{-\infty}^{+\infty} y(t)f_2(t) dt \\
    &\text{上式左侧}\Rightarrow \int_{-\infty}^{+\infty} y(t) [x(t)\delta(t)]dt = \int_{-\infty}^{+\infty}[y(t)x(t)]\delta(t)dt \text{ ，由性质2可知 } \int_{-\infty}^{+\infty} [y(t)x(t)]\delta(t) dt = y(0)x(0) \\
    &\text{上式右侧}\Rightarrow \int_{-\infty}^{+\infty} y(t)[x(0)\delta(t) ] dt = \int_{-\infty}^{+\infty} [y(t)x(0)]\delta(t) dt \text{ ，由性质2可知 } \int_{-\infty}^{+\infty} [y(t)x(0)]\delta(t) dt = x(0)y(0) \\
    &\text{由 }\int_{-\infty}^{+\infty} y(t) [x(t)\delta(t)]dt = \int_{-\infty}^{+\infty} y(t)[x(0)\delta(t) ] dt \text{ ，} x(t)\delta(t) = x(0)\delta(t) \text{ ，证毕。}
    \end{align}
    $$

    > **注意**：该性质的变式为 $x(t) \delta(t - t_0) = x(t_0) \delta(t-t_0)$

  - $\delta(at) = \dfrac{1}{|a|}\delta(t)$（$a \ne 0$），证明如下：
    $$
    \begin{align}
    &\text{上式右侧} \Rightarrow \int_{-\infty}^{+\infty} \dfrac{1}{|a|} y(t) \delta(t) dt \text{ ，由性质3可知 } \int_{-\infty}^{+\infty} \dfrac{1}{|a|} y(t) \delta(t) dt = \dfrac{1}{|a|}y(0) \\
    &\text{右侧证明类似，注意讨论a的正负} \\
    &\text{由 } \int_{-\infty}^{+\infty} y(t) \delta(at)dt = \int_{-\infty}^{+\infty} \dfrac{1}{|a|} y(t) \delta(t) dt \text{ ，}\delta(at) = \dfrac{1}{|a|}\delta(t) \text{ ，证毕。}
    \end{align}
    $$

    > **注意**：该性质的变式为 $\delta(at + b) = \dfrac{1}{|a|}\delta(t + \dfrac{b}{a})$

  - $\delta(f(t)) = \sum_{\forall{t_0}\text{, }f(t_0)=0} \dfrac{1}{|f'(t_0)|}\delta(t-t_0)$，使用 Lebesgue 函数相等定义证明。

    - 分解 $\delta(\cos(t))$（例题）：令 $\cos(t_0) = 0$，解得 $t_0 = k\pi + \dfrac{\pi}{2}$  $\Rightarrow$ $|\cos'(t_0)| = |-\sin(t_0)| = 1$ 
      <br>故 $\delta(\cos(t)) = \sum_{k = -\infty}^{k=+\infty} 1 * \delta(t-k\pi-\dfrac{\pi}{2})$（$k \in \mathbb{Z}$）

    > **注意**：该性质是性质 4 的推广，取 性质 5 中的 $f(t) = at$ 即可

---

### 四、卷积的性质

- 什么是"卷积"（简单理解）："卷"指对响应函数的翻转操作，"积"指 $x$ 和 $h$ 的乘积操作
  1. **翻转**：对响应函数的 对称翻转 操作
  2. 平移：对翻转后的函数进行 平移 操作
  3. **乘积**：将平移后的函数和响应函数进行乘积
  4. 求和：对乘积式进行求和（或积分）
- 交换律：连续型有 $x(t) * h(t) = h(t) * x(t)$；离散型有 $x[n] * h[n] = h[n] * x[n]$
- 结合律：连续型有 $[x(t) * h_1(t)] * h_2[t] = [x(t) * h_2(t)] * h_1(t)$；离散型同理
- 分配律：连续型有 $x(t) * [h_1(t) + h_2(t)] = x(t) * h_1(t) + x(t) * h_2(t)$；离散型同理

---

### 五、LTI 系统的性质

- LTI 系统**稳定** $\Leftrightarrow$ $\int_{-\infty}^{+\infty} |h(t)| dt < +\infty$，对于离散型有 $\sum_{k = -\infty}^{+\infty} |h[n]| < +\infty$，连续型的证明如下：
  $$
  \begin{align}
  &\text{先证明必要性：设 }x(t) \text{ 是有界的，则有 }|x(t)| \le M \\
  &\text{则输出 }y(t) \text{ 也是有界的，即 } |y(t)| = |x(t) * h(t)| = |\int_{-\infty}^{+\infty}h(\tau)x(t-\tau)d\tau| \le \int_{-\infty}^{+\infty}|x(t-\tau)||h(\tau)| d\tau \le M \int_{-\infty}^{+\infty} |h(\tau)| d\tau \\
  &\text{由 }\int_{-\infty}^{+\infty} |h(t)| dt \text{ 有界可知，} |y(t)| \text{ 有界，即LTI是稳定的。} \\ \\
  &\text{再证明充分性：设LTI稳定，则对于有界的一个信号 }x(t) \text{，有 }|y(t)| = |x(t) * h(t)| \le +\infty \\
  &\text{不妨设当 }h(-t) \ge 0 \text{ 时 }x(t) = +1 \text{，当 }h(-t) < 0 \text{ 时 }x(t) = -1。\\
  &\text{则 }y(0) = \int_{-\infty}^{+\infty} x(\tau)h(0-\tau)d\tau = \int_{-\infty}^{+\infty}|h(-\tau)|d\tau < +\infty \text{ ，证毕。}
  \end{align}
  $$

- LTI 系统**因果** $\Leftrightarrow$ $h(t) = 0$（$t < 0$），对于离散型有 $h[n] = 0$（$n < 0$），连续型的证明如下：
  $$
  \begin{align}
  &t_0\text{ 时刻输出信号 }y(t_0) = x(t_0) * h(t) = \int_{-\infty}^{+\infty}x(\tau)h(t_0 - \tau)d\tau \text{ ，由因果性 }y(t_0)\text{ 仅与 }\tau < t_0 \text{ 的 }x(\tau) \text{ 相关。}  \\
  &y(t_0)\text{ 仅与 }t_0 \text{ 之前的 }x\text{ 的取值有关 } \Leftrightarrow h(t_0 - \tau) \text{ 在 }t_0 < \tau \text{ 时（未来时）取 }0 \Leftrightarrow t < 0 \text{ 时 } h(t) = 0 \text{ ，证毕。}
  \end{align}
  $$

> **注意**：以上两个性质仅对 **LTI** 系统成立
