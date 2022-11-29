---
layout: posts
title: Multiple-Scattering Microfacet BSDFs with the Smith Model
mathjax: true
---

# Problem

Modeling multiple scattering in microfacet theory is considered an important open problem. Because a non-negligible portion of the energy leaving rough surfaces is due to paths that bounce multiple times. But no physically based method was derived before.

# Insight

The insight is that the microfacet theory for surfaces with the Smith model can be derived as a special case of the *microflake theory* for volumes, with additional constraints to transform the volume into a surface.

# Method

The whole process of multiple scattering in this paper is like path tracing. First, use the microflake theory to derive a 3d probability space of microfacets, then track the heights of the microfacets, like we track specific points on geometries after ray intersects them. Then we sample the outgoing direction using surface-like phase functions. Track next height...until we have a sample that the ray leaves the microsurface.