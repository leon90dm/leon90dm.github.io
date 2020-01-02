# 

1. 给定时间序列： $(y_1, y_2, y_3, \dots y_n)$

2. 给定 $l$  个change points， 并且对应时间序列的下标表示为： $\tau_1 ,\tau_2, \tau_3, \dots \tau_l$

3. 追加 $\tau_0=0, \tau_{l+1} = n$

4. 由于 $l$ 个change points，将序列划分为 $l+1$ 个segments。给定一个segment $k$，绑定一个模型 $M_k$ 和一个向量 $\theta_k$, 其中每个segment表示为：
   $$
   \mathbf{y}_{\tau_{k}+1: \tau_{k+1}}, \text { for } k=0, \ldots, l
   $$

5. ?] 当 $k \ge 1$ 时, 令 $\theta_k$ 的分布依赖于 $k-1$的 $\tau_{k-1},\tau_k,\theta_{k-1}$ 等参数：
   $$
   \operatorname{Pr}\left(M_{k}=m\right) p_{m}\left(\theta_{k} | \theta_{k-1} \tau_{k}, \tau_{k-1}\right)
   $$
   

6. 对于第一个segment $k=0$, 我们<u>指定一个先验的 $\theta_0$</u> 

7. 给定一个segment，它的开始和结束的change point分别为 $s, t$ ，则likelihood model为：
   $$
   p_{m}\left(\mathbf{y}_{s+1: t} | \theta\right)
   $$
   

8. 



