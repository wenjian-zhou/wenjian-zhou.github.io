***

本文将从一个最简单的积分开始推导，大概说明如果你想从零开始推导ReSTIR中的RIS的话应该如何推，并且从最后的公式形式上给出如何采样，也就是一般大家了解的RIS的几个步骤。对想知道RIS为什么能收敛，不想局限于了解RIS的步骤的人有些帮助。

如果要做RIS的证明，那首先就要做IS的证明（重要性采样，Importance Sampling），之后再证明你从里面Resample（重采样）的时候也是正确的。

做渲染的时候本质是解渲染方程，为了省去阅读烦恼，我们把渲染方程极端简化为一个最基础的积分。

$$F = \int f(x)\,dx$$

（若需强调积分域，可写成 $F = \int_\Omega f(x)\,dx$，下文略去 $\Omega$。）

首先简单推导一下重要性采样，重要性采样推导中的一个小技巧就是在被积函数旁边乘个1，这个1用你想用的任何概率都可以替代，只要保证积分域是匹配的就可以。将这个概率密度计为 $g(x)$ ，做重要性采样，再将 $\frac{f(x)}{g(x)}$ 看作一个整体，$g(x)$ 为乘上去的概率，则：

$$F = \int f(x) \frac{g(x)}{g(x)} = \int \frac{f(x)}{g(x)}\,g(x)\,dx = \mathbb{E}_{X\sim g}\left[\frac{f(X)}{g(X)}\right]$$

这个公式证明了你用任意概率密度 $g(x)$ 去采样的话，最终都能收敛到正确的结果，不过这个证明看起来有点在讲废话，因为其实也可以代表你不做重要性采样，但最终选什么概率密度随你（比如均匀的概率密度），最后你想比较哪个概率密度更重要的话就不是算期望了，要算方差，但这个就不是本文的内容了。

知道了证明importance sampling的这个技巧之后，我们把它变换一下，同样也是乘上一个1，但这次我们乘上去的1的分子和分母都替换成整个被积函数：

$$ F = \frac{\int f(x)\,dx}{\int f(x)\,dx} \int f(x)\,dx $$

再对里面的每个积分都再乘1，这时候乘上的就是上面做importance sampling的那个1：

$$ F =  \frac{\int \frac{f(x)}{g(x)}\,g(x)\,dx}{\int \frac{f(x)}{g(x)}\,g(x)\,dx} \int \frac{f(x)}{g(x)}\,g(x)\,dx $$

再写成期望的形式：

$$ F = \frac{\mathbb{E}_g[\frac{f(x)}{g(x)}]}{\mathbb{E}_g[\frac{f(x)}{g(x)}]} \int \frac{f(x)}{g(x)}\,g(x)\,dx $$

这时候我们令：

$$w_i = \frac{f(x_i)}{g(x_i)}$$

并用N个样本 $\omega_1, \dots, \omega_N$ 来近似期望的话，可以得到：

$$F = \frac{\frac{1}{N}\sum^{N}_{i = 1}\omega_i}{\frac{1}{N}\sum^{N}_{j = 1}\omega_j}\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

将分子的每个 $\omega_i$ 单独提出来，省略$\frac{1}{N}$：

$$F = \sum^{N}_{i = 1} \frac{\omega_i}{\sum^{N}_{j = 1}\omega_j}\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

这时可以看出，可以把 $\frac{\omega_i}{\sum^{N}_{j = 1}\omega_j}$ 看作一个概率 $q_i$：

$$F = \sum^{N}_{i = 1} q_i\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

再将公式右边的积分写成期望形式：

$$F = \sum^{N}_{i = 1} q_i \cdot \mathbb{E}_g[\frac{f(x)}{g(x)}]$$

这时候我们发现，将期望看作一个整体，再这个整体上又乘上概率求和，那就形成了又一个以 $q$ 作为概率密度函数采样的期望：

$$F = \mathbb{E}_q[\mathbb{E}_g[\frac{f(x)}{g(x)}]]$$

那么所有的推导也就在这里结束了，最后我们证明了，你先通过importance sampling，**再通过对从概率密度** $g$ 里采样出来的样本进行重新采样，重新采样的概率密度为 $q$ 的话，你最后的期望依然是渲染方程的解。

在A Gentle Introduction to ReSTIR的SIGGRAPH 2023 Course Note PDF中，里面的形式稍微有些不同，为了方便也做一下一一对应。

PDF中的 $\omega_i = \frac{\hat{p}(X_i)}{p(X_i)}$ ，对应本文中的 $\frac{f(x)}{g(x)}$。而Unbiased Contribution Weight， $W_X = \frac{1}{\hat{p}(X)}(\frac{1}{M}\sum^{M}_{i=1}\omega_i)$ ，是需要采样的概率密度的倒数，所以把它翻转过来，并将 $\frac{1}{M}$ 看作是一个概率密度 $g$ 的话，对应在本文中就是 $\frac{1}{W_X} = \frac{\hat{p}(X)}{g(X)}(\frac{1}{\sum^{M}_{i=1}\omega_i})$，也就是 $\frac{\omega_j}{\sum^{M}_{i=1}\omega_i}$ 的形式了。

到这为止我们就通过自己手推的方式推导出了ReSTIR中的RIS。