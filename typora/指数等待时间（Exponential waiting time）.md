# 指数等待时间（Exponential waiting time）

标签（空格分隔）： probability

---

1. 首先将连续的时间离散化
    a. 取一个最够小的片段 $\delta$, 使得在 $\delta$ 内最多只会发生一个事件。
    b. 那么 $T$ 可以被离散化为：
$$
T = 1\delta,2\delta,\cdots,n\delta 
$$
2. 令 $\delta$内事件发生的概率为 $p_\delta$.

3. 那么事件第一次发生等待时间$T$的概率为：
$$
P(T > n\delta) = (1-p_\delta)^n
$$
其中： $(1-p_\delta)^n$表示等了$n$个时间片段时间都没有发生, 如果我们知道 $t$ 时间内事件会发生 $c$ 次, 且有: $\delta = t /n;\ p_\delta = c/n$ 那么：
$$
P(T > n\delta) = (1-p_\delta)^n = (1+\frac{-c}{n})^n \\
\approx e^{-c};\ when\ \infty \gets n
$$



---
则有以下性质：

1. $E(T) = \delta/p_\delta$
证明：
  $$
    E(T) = \sum_{i=1}^{n}(1-p_\delta)^i\cdot\delta
  $$
令 $r = 1-p_\delta$, 则利用等比数列求和可以简化求和式子

2. 当 $0 \gets \delta$ 时，$E(T)$中 $\infty \gets n$, 那么 $\alpha \gets E(T)=\delta/p_\delta;$ 这里 $\alpha$ 为一个常数
$$
\delta/p_\delta = \frac{t/n}{c/n} = t/c;\ when\ \infty \gets n
$$
其中 $c$ 为 $t$ 时间内	发生的次数。

3. $n \approx t/\delta$
4. 当 $\delta$ 足够小时，有：
$$
P(T > t) \approx (1-p_\delta)^n = (1-\delta/\alpha)^{t/\delta}
$$
令 $1/k = \delta$, 所以$0\gets \delta$ 等价于 $\infty \gets k$ 
$$
(1-\delta/\alpha)^{t/\delta} = ((1+ \frac{-1/\alpha}{k})^{k})^t \\
= e^{-t/\alpha};  \infty \gets k
$$
把 $\alpha = \delta/p_\delta$ 替换一下：
$$
e^{-t/\alpha} = \frac{1}{e^{tp_\delta/\delta}} \\
= \frac{1}{e^{n\delta p_\delta/\delta}} \\
= \frac{1}{e^{n\cdot p_\delta}}
$$


因此，$P(T>t)$ 可以 解释为：首次时间发生等待时间的概率随着等待时间的增长而指数减小