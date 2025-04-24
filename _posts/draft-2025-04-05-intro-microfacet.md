---
layout: post
title: Deriving the D and G term in microfacet theory under the physical assumption
date: 2025-04-05 16:40:16
description: a deeper understanding of microfacet theory
tags: rendering microfacet math
categories: rendering
---

In this article, I'm going to talk about the physical assumption in microfacet theory and how we can derive the distribution of normals(D) and masking function(G) under this assumption, based on the paper: ["Understanding the masking-shadowing function in microfacet-based BRDFs."](https://jcgt.org/published/0003/02/03/). Also, I will include some explanations to help you understand the equations and (potentially) have a deeper understanding of microfacet theory. 

Since this article is about a "deeper" understanding, you should be familiar with microfacet BRDFs before you read this article.

# Radiance on a surface as radiance on a point

As it is known, the microfacet BRDF has the form:

$$
\frac{D \cdot F \cdot G}{4(\omega_i \cdot n)(\omega_o \cdot n)}
$$

Where $$D$$ and $$G$$ are the distribution of normals and the masking function. $$F$$ is fresnel term. $$\omega_i$$ and $$\omega_o$$ are incident direction and outgoing direction respectively, $$n$$ is the surface normal.

From its assumption, it's defined on a rough surface like this:

IMAGE!!!

And the microfacet theory tells us, let's treat this rough surface as a point! Which means we scale the rough surface to be infinitely small, as small as a shading point, then the whole area looks like a flat surface. Let's call the rough surface as "microsurface" and the flat surface as "macrosurface". We can observe the macrosurface but can't observe the microsurface. And the microsurface consists of many small microfacets.

IMAGE!!!!

But in reality, light interacts with the microsurface, the microsurface receives light, and then reflects the light to other directions. And for the shading point, we are interested in the outgoing radiance $$L_o$$ for the macrosurface, in order to get $$L_o$$, we need to accumulate all the $$L'_o$$ on the microsurface for that given direction $$\omega_o$$. Below is a visualization of it:

IMAGE!!!

In this image, $$M$$ represents the microsurface(rough surface), the percentage represents how much radiance do the microfacets with different normals take in the projected outgoing direction. You can also consider it as a regular surface, and we are just computing the total radiance for a given outgoing direction $$\omega_o$$ from this surface.

The equation to compute the outgoing radiance from this surface is:
$$
L(\omega_o, M) = \frac{\int_M \text{projected area}(p_m)L(\omega_o, p_m)dp_m}{\int_M \text{projected area}(p_m)dp_m}
$$
How to understand it? Think of it as we times the radiance $$L$$ with the projected area for a given point $$p_m$$, then we do an integration based on the $$p_m$$. We also divide it by the integral of the projected area, it's called normalization. We do an integration with the area times radiance, so we need to divide it by the total area to get the original unit of radiance.

TO BE CONTINUED...

# Reference

* Heitz, Eric. ["Understanding the masking-shadowing function in microfacet-based BRDFs."](https://jcgt.org/published/0003/02/03/) Journal of Computer Graphics Techniques 3.2 (2014): 32-91.