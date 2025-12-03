***

# From Integrals to Pixels: Understanding Resampled Importance Sampling

In computer graphics, we often bridge the gap between abstract probability theory and the concrete task of lighting a pixel. While the general theory of Resampled Importance Sampling (RIS) is well-documented, intuition significantly improves when we frame these formulas explicitly within the context of rendering.

Here, we derive the mechanics of RIS by treating paths as being conditioned on a specific pixel.

### 1. The Rendering Context: From General Integrals to Pixel Radiance

In rendering, our primary goal is to compute the radiance, $L(p)$, for a specific pixel $p$ on the image plane. This radiance is the solution to the **Rendering Equation**. For practical Monte Carlo rendering, we solve its path integral formulation.

The radiance for a pixel $p$ is the integral of contributions from all possible light transport paths, $x$, that interact with that pixel. The set of all such paths forms the **path space**, denoted as $\Omega_p$.

Our target integral becomes the radiance $L(p)$:

$$L(p) = \int_{\Omega_p} f(x|p) \,d\mu(x)$$

Where:
*   $p$ is the pixel being rendered.
*   $x$ represents a complete light transport path (a sequence of vertices connecting lights to the camera via pixel $p$).
*   $f(x|p)$ is the **measurement contribution function**. It evaluates the throughput (light and color) transported along path $x$ to pixel $p$.
*   $\Omega_p$ is the high-dimensional manifold of all paths contributing to pixel $p$.
*   $d\mu(x)$ is the geometric measure over the path space.

### 2. Importance Sampling in Path Space

To solve this integral numerically, we require a strategy to sample paths. We introduce a source probability density function (PDF), $g(x|p)$, which might represent a standard BSDF or light-sampling strategy.

We rewrite the pixel radiance integral as an expectation:

$$L(p) = \int_{\Omega_p} \frac{f(x|p)}{g(x|p)} g(x|p) \,d\mu(x) = \mathbb{E}_{X \sim g(\cdot|p)}\left[\frac{f(X|p)}{g(X|p)}\right]$$

This is the classic Monte Carlo formulation: we average the contribution of paths, each weighted by the ratio $w(x) = f(x)/g(x)$.

### 3. The Resampling Step

In Resampled Importance Sampling (RIS), we do not immediately use these samples to estimate the image. Instead, we generate a pool of "candidate" paths and choose the best among them.

Assume we generate a set of $N$ candidate paths, $\{x_1, x_2, \ldots, x_N\}$, drawn from $g(x|p)$. For each path, we calculate its **unnormalized importance weight**:

$$w_i = \frac{f(x_i|p)}{g(x_i|p)}$$

We then construct a discrete probability mass function (PMF), $q(\cdot|p)$, to select one index $j$ from this pool. The probability of picking the $j$-th path is proportional to its weight:

$$q_j \equiv q(j|p) = \frac{w_j}{\sum_{i=1}^{N} w_i}$$

This step effectively biases our selection toward paths that carry more light (high $f$) relative to how often we find them (low $g$).

### 4. Deriving the Nested Expectation

To understand why this works mathematically, we can derive the relationship between the standard estimator and the resampling interactions.

We start with the definition of our pixel radiance $L(p)$. Because the sum of our resampling probabilities $\sum q_j = 1$, we can introduce this sum into the equation without changing the value:

$$L(p) = \left(\sum_{j=1}^{N} q_j\right) \cdot \mathbb{E}_{X \sim g(\cdot|p)}\left[\frac{f(X|p)}{g(X|p)}\right]$$

However, to see the algorithmic implementation on the right-hand side, we must look at how the *estimator* behaves. If we approximate the expectation using our $N$ samples (the Monte Carlo approximation), we can see the structure of the RIS estimator emerge.

Below is the derivation showing how the standard Monte Carlo estimator transforms into the RIS selection probability:

$$
\begin{aligned}
L(p) &= \frac{\int_{\Omega_p} f(x|p) \,d\mu(x)}{\int_{\Omega_p} f(x|p) \,d\mu(x)} \cdot \int_{\Omega_p} f(x|p) \,d\mu(x) \\
\\
&= \frac{\int_{\Omega_p} \frac{f(x|p)}{g(x|p)} g(x|p) \,d\mu(x)}{\int_{\Omega_p} \frac{f(x|p)}{g(x|p)} g(x|p) \,d\mu(x)} \cdot L(p) \\
\\
&= \frac{\mathbb{E}_{X \sim g}\left[w(X)\right]}{\mathbb{E}_{X \sim g}\left[w(X)\right]} \cdot L(p)
\end{aligned}
$$

*Approximating the expectations with sums over $N$ samples:*

$$
\begin{aligned}
L(p) &\approx \frac{\frac{1}{N}\sum_{j=1}^N w_j}{\frac{1}{N}\sum_{i=1}^N w_i} \cdot L(p) \\
\\
&= \left(\sum_{j=1}^N \frac{w_j}{\sum_{i=1}^N w_i}\right) \cdot \mathbb{E}_{X \sim g}\left[\frac{f(X)}{g(X)}\right] \\
\\
&= \sum_{j=1}^N q_j \cdot \mathbb{E}_{X \sim g}\left[\frac{f(X)}{g(X)}\right]
\end{aligned}
$$

This final form reveals the nested nature of RIS. Let $Y$ be the random variable representing the index chosen according to $q$. We can express the final radiance calculation as an expectation over the resampling choice, which is itself fed by an expectation over the candidate generation:

$$L(p) = \mathbb{E}_{Y \sim q(\cdot|p)}\left[\mathbb{E}_{X \sim g(\cdot|p)}\left[\frac{f(X|p)}{g(X|p)}\right]\right]$$

### Conclusion

This identity is the theoretical core of algorithms like ReSTIR. It demonstrates that the radiance of a pixel can be solved by first casting a wide net (sampling from