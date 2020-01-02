# Yate's Correction

## 是什么？

- Yate's Correction 是用来矫正卡方检验。

- Yate's Correcton 适用在有cell的值小于5的情况
- Yate's Correction 会过度修正，增大<u>**pvalue**</u>，增大犯II型错误的概率

## 如何矫正？

​	正常卡方的计算公式为：
$$
\chi^{2}=\sum_{i=1}^{N} \frac{\left(\left|O_{i}-E_{i}\right|\right)^{2}}{E_{i}}
$$
​	其中 $$E_i$$ 为 $$i$$ 格子的期望值， $$O_i$$ 为对应的观测值。Yate’s correction 增加了一项 $$0.5$$ 的修正：
$$
\chi_{\mathrm{Yates}}^{2}=\sum_{i=1}^{N} \frac{\left(\left|O_{i}-E_{i}\right|-0.5\right)^{2}}{E_{i}}
$$




