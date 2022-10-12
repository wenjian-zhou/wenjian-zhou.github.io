---
layout: article
title: Conclusion of 'A Learned Shape-Adaptive Subsurface Scattering Model' by Vicini et al. SIGGRAPH 2019
mathjax: true
---

Still reading...

# Abstract

focus on subsurface scattering problem

explicit simulation of the internal scattering is too costly

BSSRDFs are efficient but impose unreal assumptions:

* planar geometry, isotropy, low absorption, spatio-directional separability.

This paper introduces a new **shape-adaptive** BSSRDF that retains the **efficiency** while improving accuracy

## Approach

based on a conditional variational autoencoder, which learns to sample from a reference distribution produced by a brute-force volumetric path tracer. The autoencoder directly samples outgoing locations on the object surface, bypassing a potentially lengthy internal scattering process

# Intro

Materials include subsurface scattering often use 2 ways to render. One is Monte Carlo methods, which simulates the long sequences of scattering interactions in a medium, but it's very costly for many real-world materials due to the high albedo. The other one is BSSRDF, it models surface-to-surface transport and are typically based on analytic solutions of simplified light transport problems.

BSSRDF are efficient but it imposes severe assumptions that are almost always violated. 

* postulate half-space or slab geometry and low anisotropy/absorption.
* separability, i.e. that the BSSRDF decomposes into a product of functions with spatial or directional dependence.

So the discrepancies between model and usage lead to objectionable errors in rendered images, especially near geometric features that violate planarity.

# Method

combination of three neural networks that together constitute a probabilistic generative model for BSSRDF sampling on curved surfaces:

1. a conditional variational autoencoder, which learns to **sample from a reference distribution** produced by a volumetric path tracer.
2. multiplayer perceptron that **regresses the distribution's scale factor** to allow for absorbing materials
3. The two neural networks are **conditioned on a number of parameters that are pre-processed by a third feature network**. Include **material properties** (albedo, extinction, anisotropy, and index of refraction), the **incident direction**, and a **descriptor characterizing geometric variation in a neighborhood of the incident location**.

## VAE

The VAE consists of 2 parts: 

* a neural network **g** in the role of a high-capacity function approximator maps a latent vector to a surface position, this is referred to as *decoding* **z**.
* a separate *encoder* network **q** takes a surface position **x** as input and returns randomly sampled latent variables corresponding to light transport that produces **x** with high probability
