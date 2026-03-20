---
layout: post
title: Derive RIS from scratch
date: 2026-03-20 17:00:00
description: 
tags: rendering math
categories: rendering
---

This post starts from the simplest possible integral and works toward the core idea behind RIS. The goal is to show, in a rough but direct way, how you might derive the RIS used in ReSTIR from scratch. By the time we reach the final form, the familiar RIS sampling procedure also falls out naturally.

This is mainly for readers who want to understand why RIS converges, not just memorize the usual sequence of steps.

If you want to justify RIS, the first step is to justify IS, or Importance Sampling. After that, you still need to show that resampling from those samples is also valid.

In rendering, what we are fundamentally doing is solving the rendering equation. To keep the discussion easy to read, we reduce the rendering equation to the most basic integral:

$$F = \int f(x)\,dx$$

(If we want to emphasize the integration domain, we could write $F = \int_\Omega f(x)\,dx$, but we will omit $\Omega$ below.)

Let us begin with importance sampling. A small but useful trick in the derivation is to multiply the integrand by 1. That "1" can be written using any probability density you like, as long as the integration domain matches. Let that density be $g(x)$. If we apply importance sampling and treat $\frac{f(x)}{g(x)}$ as a single quantity, with $g(x)$ being the density we introduced, we get:

$$F = \int f(x) \frac{g(x)}{g(x)} = \int \frac{f(x)}{g(x)}\,g(x)\,dx = \mathbb{E}_{X\sim g}\left[\frac{f(X)}{g(X)}\right]$$

This formula tells us that no matter which probability density $g(x)$ we sample from, the estimator still converges to the correct result. In that sense, the statement is almost trivial: it even includes the case where we do nothing especially clever and simply choose a density such as a uniform one. If we want to discuss which density is actually better, then the right quantity to study is variance rather than expectation, but that is outside the scope of this post.

Once we have this trick in hand, we can push it a little further. Again we multiply by 1, but this time both the numerator and denominator are replaced by the full integral itself:

$$ F = \frac{\int f(x)\,dx}{\int f(x)\,dx} \int f(x)\,dx $$

Next, multiply each integral inside by 1 once more. This time, the "1" is exactly the one we used above for importance sampling:

$$ F =  \frac{\int \frac{f(x)}{g(x)}\,g(x)\,dx}{\int \frac{f(x)}{g(x)}\,g(x)\,dx} \int \frac{f(x)}{g(x)}\,g(x)\,dx $$

Now rewrite the expression in expectation form:

$$ F = \frac{\mathbb{E}_g[\frac{f(x)}{g(x)}]}{\mathbb{E}_g[\frac{f(x)}{g(x)}]} \int \frac{f(x)}{g(x)}\,g(x)\,dx $$

Now let:

$$w_i = \frac{f(x_i)}{g(x_i)}$$

If we use $N$ samples $x_1, \dots, x_N$ to approximate the expectation, we get:

$$F = \frac{\frac{1}{N}\sum^{N}_{i = 1} w_i}{\frac{1}{N}\sum^{N}_{j = 1} w_j}\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

Now pull each term $w_i$ out of the numerator separately and drop the common factor $\frac{1}{N}$:

$$F = \sum^{N}_{i = 1} \frac{w_i}{\sum^{N}_{j = 1}w_j}\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

At this point, $\frac{w_i}{\sum^{N}_{j = 1}w_j}$ can be interpreted as a probability $q_i$:

$$F = \sum^{N}_{i = 1} q_i\int \frac{f(x)}{g(x)}\,g(x)\,dx$$

Then rewrite the integral on the right-hand side as an expectation:

$$F = \sum^{N}_{i = 1} q_i \cdot \mathbb{E}_g[\frac{f(x)}{g(x)}]$$

Now comes the key observation. If we treat that expectation as a whole, then multiply it by another probability and sum over it, we get yet another expectation, this time taken with respect to $q$:

$$F = \mathbb{E}_q[\mathbb{E}_g[\frac{f(x)}{g(x)}]]$$

This is the end of the derivation. What we have shown is that if we first perform importance sampling, and then resample from the candidates drawn from $g$ using a resampling distribution $q$, the final expectation still evaluates to the solution of the rendering equation.

The notation used in the SIGGRAPH 2023 course notes for *A Gentle Introduction to ReSTIR* looks slightly different, so it is useful to map the symbols one by one.

In the PDF, $\omega_i = \frac{\hat{p}(X_i)}{p(X_i)}$, which corresponds to $\frac{f(x)}{g(x)}$ in this article. The Unbiased Contribution Weight is defined as $W_X = \frac{1}{\hat{p}(X)}(\frac{1}{M}\sum^{M}_{i=1}\omega_i)$. Since this is the reciprocal of the probability density we want to sample from, we can flip it over. If we also treat $\frac{1}{M}$ as a probability density $g$, then in the notation of this article it becomes $\frac{1}{W_X} = \frac{\hat{p}(X)}{g(X)}(\frac{1}{\sum^{M}_{i=1}\omega_i})$, which is exactly the form $\frac{\omega_j}{\sum^{M}_{i=1}\omega_i}$.

At that point, we have derived the RIS used in ReSTIR by hand.