---
layout: article
title: Conclusion of 'A Comprehensive Framework for Rendering Layered Materials' by Jakob et al. SIGGRAPH 2019
mathjax: true
---

BSDFs of layered materials

Ingredients：

* transport-theoretical models of scattering layers
* smooth/rough boundaries of conductors and dielectrics

expansion:

* directional basis that supports arbitrary composition

layers:

* scattering and/or absorbing media, separated by interfaces that might be smooth or rough

contributions:

* numerical methods for projecting a range of transport-theoretic reflectance models into a basis that supports arbitrary composition
* inherently sparse nature of the problem
* a multiple scattering term that avoids energy loss

# Background

introduce the adding equations, which are used to compute the scattering matrix of a composite layer ---> describe the directional basis that underlies these matrix, and how to use it to discretize the radiative transfer equation and boundary conditions ---> how to solve the radiative transfer equation

## Adding equations

consider a glass plate, the total reflectance and transmittance can be calculated in a explicit form:
$$
\tilde{R} = R + TRT + TRRRT + TRRRRT + \dots = R + \frac{RT^2(1-R^{2n})}{1-R^2} \\
\tilde{T} = TT + TRRT + TRRRRT + \dots = \frac{T^2(1 - R^{2n})}{1 - R^2}
$$
After so many bounces, $$R^{2n}$$ is close to 0, so it can be removed.

A more general case:
$$
\Phi^{\uparrow}(t) = R^t \Phi^{\downarrow}(t) + T^{bt}\Phi^{\uparrow}(b) \\
\Phi^{\downarrow}(b) = R^b \Phi^{\uparrow}(b) + T^{tb}\Phi^{\downarrow}(t)
$$
$$\Phi^{\uparrow}(t)$$ represents all the vectors at height t pointing upward.

With the derivation of [Grant and Hunt 1969], it can be expanded to:
$$
\tilde{R}^{t} = R^{t}_{1} + T^{bt}_{1}(I - R^{t}_{2}T^{b}_{1})^{-1}R^{t}_{2}T^{tb}_{1} \\
\tilde{R}^{b} = R^{b}_{2} + T^{tb}_{2}(I - R^{b}_{1}T^{t}_{2})^{-1}R^{b}_{1}T^{bt}_{2} \\
\tilde{T}^{tb} = T^{tb}_{2}(I - R^{b}_{1}R^{t}_{2})^{-1}T^{tb}_{1} \\
\tilde{T}^{bt} = T^{bt}_{1}(I - R^{t}_{2}R^{b}_{1})^{-1}T^{bt}_{2}
$$
This is the adding equation. We can compute the scattering matrix of a layered material by repeatedly using this equation to combine layers and boundaries based on a structural description of the material. But it requires us knowing the matrices of the interior of layers, which will be discussed in next sections.

## Problem statement
