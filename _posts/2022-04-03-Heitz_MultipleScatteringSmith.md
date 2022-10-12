---
layout: posts
title: Conclusion of 'Multiple-Scattering Microfacet BSDFs with the Smith Model' by Heitz et al. SIGGRAPH 2016
mathjax: true
---

# Problem

Modeling multiple scattering in microfacet theory is considered an important open problem. Because a non-negligible portion of the energy leaving rough surfaces is due to paths that bounce multiple times.

# Insight

The author's insight is that the microfacet theory for surfaces with the Smith model can be derived as a special case of the *microflake theory* for volumes, with additional constraints to transform the volume into a surface.

# Method

## Intersection with the Microsurface

If we want to simulate the full process of light bouncing multiple times inside the microsurface, we need to track the heights on the microsurface. So the author modeled the Smith microsurface with the equation of volumetric scattering, by defining the **extinction coefficient $$\sigma_{t}$$** and a **free-path distribution**.

After defining it, we can sample the heights and simulate the process of light bouncing between microfacets. Also, the height sampling match properties expected from a heightfield.

## Phase Function of the Microsurface

