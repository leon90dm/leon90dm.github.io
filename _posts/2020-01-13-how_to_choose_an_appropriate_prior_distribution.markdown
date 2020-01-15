---
layout:     post
title:      "如何选择合适的先验分布"
subtitle:   ""
date:       2020-01-13
author:     "BilboDai"
catalog: true
tags:
    - probability
---

### 两类 priors

1. 主观priors :  先验时多加入参与者的观点
2. 客观priors：让数据更多参与影响后验分布



### 有用的先验分布

1. Gamma分布

   
   $$
   f(x | \alpha, \beta)=\frac{\beta^{\alpha} x^{\alpha-1} e^{-\beta x}}{\Gamma(\alpha)}
   $$
   

2. The Wishart distribution

   把随机变量从单个值拓展到多维

   ![wischard](https://tva1.sinaimg.cn/large/006tNbRwly1gawfvzjo4fj30ks098749.jpg)

3. Beta 分布

   
   $$
   f_{X}(x | \alpha, \beta)=\frac{x^{(\alpha-1)}(1-x)^{(\beta-1)}}{B(\alpha, \beta)}
   $$
   

   如果我们从先验分布 $$Beta(1,1)$$ 实际是一个均匀分布开始，当观察到 $$X \sim Binomial(N,p)$$ 样本后的后验分布为：$$Beta(1+X, 1+N-X)$$ 

   这里， $$Beta$$ 也被称作 $$Binomial$$ 的共轭先验，形似满足如下等式的 $$p_{\beta}$$：

   
   $$
   \underbrace{p_{\beta}}_{\text { prior }} \cdot \overbrace{f_{\alpha}(X)}^{\text {data }}=\overbrace{p_{\beta^{\prime}}}^{\text {posterior }}
   $$
   

   换 $$p_{\beta} = Beta,\ f_{\alpha} = Binomial$$ 得：

    

$$
\underbrace{\text { Beta }}_{\text { prior }} \cdot \overbrace{\text { Binomial }}^{\text {data }}=\overbrace{\text { Beta }}^{\text {posterior }}
$$

​	

​		👉 共轭先验的好处是：**可以避免使用MCMC进行近似推断，而直接进入后验。**

### Bayesian Multi-Armed Bandits

multi-armed bandits 的目标就是要找出那个能产生最大收益概率的 arm。算法流程如下：

1. Sample a random variable $$X_b$$ from the prior of bandit $$b$$, for all $$b$$.
2. Select the bandit with largest sample, i.e. select $$B = argmax\ X_b$$.
3. Observe the result of pulling bandit $$B$$ , and update your prior on bandit $$B$$.
4. return to step 1.

👉 拓展

1） 在更新prior的时候，可加上一个 `rate` 

```python
self.wins[choice] = rate*self.wins[choice] + result
self.trials[choice] = rate*self.trials[choice] + 1
```

### The Resulting ranking algorithm

The Resulting Ranking algorithm is quite straightforward,  each new time the comments page is loaded, the score for each comment is sampled from a Beta(1+U,1+D), comments are then ranked by this score in descending order.

> This randomization has a unique benefit in that even untouched comments (U=1,D=0) have some chance of being seen even in threads with 5000+ comments (something that is not happening now), but, at the same time, the user is not likely to be inundated with rating these new comments.

### Stock Returns

假设 $$S_t$$ 是已知股票在第 $$t$$ 天的价格，那么第 $$t$$ 天的收益率为：


$$
r_t = \frac{S_t - S_{t-1}}{S_{t-1}}
$$


那么这只股票的天收益率期望为： $$\mu = E[r_t]$$.















