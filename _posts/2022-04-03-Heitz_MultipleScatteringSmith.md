---
layout: posts
title: Multiple-Scattering Microfacet BSDFs with the Smith Model
mathjax: true
---

{% include figure image_path="/assets/images/multiplescatteringSmith/difference.png" alt="figure 1" caption="The difference between single scattering and single + multiple scattering." %}

# Problem

Compensate the missing energy due to multiple scattering on microfacet model. Because when roughness becomes higher, the traditional microfacet model becomes darker.

# Previous Work Limitations

* Not physically-based multiple scattering method, mostly approximate compensation.
* Not simulating the multiple scattering process.
* Some works in physics focus on the albedo with consideration of multiple scattering, but no full BSDF derived.

# Insight

The microfacet model for surfaces with the Smith model can be derived as a special case of the *microflake model* for volumes, with additional constraints to transform the volume into a surface. So we can compute using Monte Carlo estimator.

# Method

* Intersection with the microsurface by tracking the heights of the multiple intersections.
* Phase functions which are similar to generic microfacet BSDFs.
* With intersections and phase functions known, random walks on the microsurface can be deduced, which is similar to path tracing.
* The multiple-scattering BSDF is the expectation of the random walk.

{% include figure image_path="/assets/images/multiplescatteringSmith/randomwalk.png" alt="figure 2" caption="The random walk process." %}

# Limitations

* Stochastic evaluation introduces additional bias.
* Rendering cost increase as the method is generally doing path tracing.
* Not suited for real-time rendering.