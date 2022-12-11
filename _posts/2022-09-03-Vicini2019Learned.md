---
layout: posts
title: A Learned Shape-Adaptive Subsurface Scattering Model
mathjax: true
---

# Problem

Subsurface scattering is crucial for visual realism. BSSRDF models are efficient but have some assumptions that are often violated, and explicit simulation is too costly.

# Previous Work Limitations

* Improved sampling strategy but still costly.
* Diffusion models have plane-parallel assumption that will introduce bias when rendering non-planar objects.
* Neural network methods run training concurrently with rendering.

# Insight

Subsurface scattering can be represent by a set of latent variables, which can characterize the stochastic volume rendering process.

# Method

* A combination of three neural networks that together constitute a model for BSSRDF sampling.
* 1) A conditional variational autoencoder (VAE) that can sample from a distribution produced by a volumetric path tracer.
* 2) A MLP that regresses the distribution's scale factor to allow for absorbing materials.
* 3) A feature network.
* Approximate the local geometry using a low-order polynomial.

{% include figure image_path="/assets/images/learnedBSSRDF/network.png" alt="figure 1" caption="The network overview." %}

{% include figure image_path="/assets/images/learnedBSSRDF/polynomial.png" alt="figure 2" caption="The polynomial that approximates local geometry, because outgoing positions are often placed around the incident positions." %}

# Limitations