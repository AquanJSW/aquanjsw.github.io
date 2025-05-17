+++
title = '平方根倒数速算法'
date = 2025-05-17T00:36:08+08:00
draft = false
+++

{{< katex >}}

背景可见[wiki](https://en.wikipedia.org/wiki/Fast_inverse_square_root)

我们所关注的问题是，所谓的魔术数字是如何得来的。

McEniry论文[^McEniry]的DERIVATION部分给出了

```c
i = 0x5f3759df - (i >> 1);
```

这一形式的数学解释：

$$
I_y = R - \frac{1}{2}I_x
$$

此时，问题转换为\\(R\\)是如何得来的

由于

$$
R = \frac{3}{2}(B-\sigma)L
$$

这相当于问\\(\sigma\\)是如何得来的

\\(\sigma\\)是一个常数，源于对函数

$$
f(x)=log_2(1+x)
$$

的逼近

$$
g(x)=x+\sigma
$$

一种直观的想法是求一个\\(f(x)\\)的最佳逼近\\(g(x)\\)，这样就能得到\\(\sigma\\)

如果采用切比雪夫最佳逼近（可见数值分析[^chebyshev]的“6.9 最佳逼近：切比雪夫理论”的例1），相当于求解：

$$
\begin{equation}
\left\{
	\begin{array}{l}
	    g(0)-f(0)=\mu \\
		g(1)-f(1)=\mu \\
		f(\xi)-g(\xi)=\mu \\
		f'(\xi)-g'(\xi)=0
    \end{array}
\right.
\end{equation}
$$

> 配合McEniry论文[^McEniry]CALCULATIONS部分的第一个图更容易理解

解得

$$
\sigma=\frac{1}{2}(1-\frac{1}{ln2}-log_2(ln2))\approx0.043
$$

显然，这与根据魔术数字反推的\\(\sigma=0.045\\)仍有差距

我们不知道真正的作者是如何得到这个数字的

但一种合理的猜想是

上述展示函数逼近仅仅考虑了\\(\sigma\\)对\\(log_2(1+x)\\)的影响

如果要考虑整体的影响，则需要分析\\(\sigma\\)对\\(\frac{1}{\sqrt{x}}\\)的影响，这在McEniry论文[^McEniry]中有长篇讨论

[^McEniry]: C. McEniry, 《THE MATHEMATICS BEHIND THE FAST INVERSE SQUARE ROOT FUNCTION CODE》.
[^chebyshev]: 金凯德和切尼, 数值分析.
