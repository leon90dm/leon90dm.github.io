# MCMC

<img src="https://tva1.sinaimg.cn/large/006tNbRwly1ga7x0twq7mj30is0egq44.jpg" alt="img" style="zoom: 33%;" />

通过蒙特卡洛 近似求 <u>**阴影部分**</u> 面积：

1. 随机生成一个点进行近似，如下图。这种近似非常的粗略。

   <img src="https://tva1.sinaimg.cn/large/006tNbRwly1ga7x21y35xj30hi0ga0up.jpg" alt="img" style="zoom: 33%;" />

2. 在 $[a,b]$ 之间均匀的生成 $n$ 个随机点：$x_1, x_2, \dots x_n$，如下图：

   ![img](https://tva1.sinaimg.cn/large/006tNbRwly1ga7x3itnh0j30im05wwf7.jpg)

3. 整体阴影面积近似为： $\frac{1}{n}\sum_i^nf(x_i)\cdot(b-a)$

   > 以上有一个前提： <u>$[a,b]$ 之间的密度是相等的</u> ，如何理解密度相等？
   >
   > 密度不等的意思是： 在 $[a,b]$ 之间随机生成的 $x_i$ 并不是均匀的铺满 $[a,b]$, 而是服从某种概率分布，例如正态分布，随机 $n$ 个点，在 $[-4,-2]$ 和 $[-2,0]$ 的点数量并不相等：
   >
   > <img src="https://tva1.sinaimg.cn/large/006tNbRwly1ga7u0a92tkj30wq0kgdgy.jpg" alt="image-20191224151304681" style="zoom: 25%;" />

对于，密度不等的情况（$x_i$ 取到 $[a,b]$ 之间的值的机会是不等的。）

