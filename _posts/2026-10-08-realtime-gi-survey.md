---
title: A Survey of Real-Time GI Techniques with Game Case Studies
date: 2026-10-08 12:00:00
description: an overview of real-time global illumination techniques and the games that ship them
tags: [rendering, GI]
categories: [rendering]
math: true
---

## Introduction

I have recently been reading a lot of GI-related material and papers, and it became apparent that the landscape of GI techniques is rather fragmented. New "hybrid solutions" continue to emerge, making the field difficult to follow. So, with the aim of presenting a complex topic as clearly as possible, this post summarizes the GI techniques commonly used in real-time rendering today, together with examples of games that use them.

Unlike typical introductory articles on GI, this post, in the section on dynamic spatial lighting cache GI, treats the **cache structure**, the **cached radiometric quantity**, and the **Final Gather method** as three parallel axes and combine them. Hybrid solutions such as DDGI, Surfel GI and Lumen can all be derived from these combinations.

For the papers behind specific GI techniques, or the GDC / SIGGRAPH Advances talks from game studios, readers may use AI-assisted tools to locate and review them. This post will not go into the details of any individual technique.

Note that this post only covers techniques for **diffuse** GI. I plan to write a separate post on GI light-leaking prevention in the future.

## Precomputed / Baked GI

The essence of precomputed (baked) GI is to solve or approximate the rendering equation offline and cache the result. Methods used for this include Monte Carlo, Photon Mapping, Radiosity, and so on.

Based on what is cached, these approaches fall into two categories: **the result of light transport, and the process of light transport**.

Each category in turn has two storage options: **baking onto surfaces, and baking into points placed in world space**.

Representative methods of the first category are the Lightmap (baked on surfaces) and the Volumetric Lightmap (baked into points placed in world space). The points-in-space approach usually stores an SH vector at each point and interpolates samples at runtime, which is equivalent to storing an expression of the lighting at a given point in space. This pattern of storing sample points in space is sometimes called an Irradiance Volume, or Probes; naming conventions vary, but the underlying idea is the same: sample points distributed in space. How exactly those points are placed (manually by artists / automatically / with an octree) is an implementation choice.

![Unreal Lightmass](/assets/img/realtime-gi-survey/1.jpg)
_Unreal Lightmass. Source: [Epic Games documentation](https://dev.epicgames.com/documentation/unreal-engine/lightmass-basics-in-unreal-engine)_

The representative method of the second category is PRT (Precomputed Radiance Transfer). Depending on how data is stored, PRT can likewise be split into baking on geometric surfaces and baking into points placed in space. Surface PRT is stored as an SH transfer vector (or, depending on whether only diffuse or additionally glossy reflection is stored, as SH scaling coefficients / an SH transfer matrix). Spatial PRT is stored as an SH matrix: once the incident lighting has been converted into an SH vector, the goal is to compute a new incident lighting expression that accounts for occlusion by the various geometry in the scene. That result is still an SH vector, so an SH matrix is needed to perform the transformation in between.

Because the first category bakes the complete result of light transport, it needs multiple sets of baked data to handle time of day (TOD), dynamic weather, and so on. The second category only bakes the light transport process, so it can support a variety of lighting conditions with a single set of baked data. However, for interiors, where direct sunlight and skylight contribute relatively little and local lights such as point and spot lights dominate, it may be necessary to fall back to the first category and bake a separate Lightmap or Volumetric Lightmap. For example, in Ghost of Yōtei, the open world uses a PRT-like approach, while artists can define bounding volumes around interiors and other important areas; these regions are subdivided with an octree, probes are generated, and then baked.

Game examples: Ghost of Tsushima (outdoors, PRT-like), The Division (PRT), Delta Force (baked probes + voxel interpolation).

![Ghost of Tsushima (outdoors)](/assets/img/realtime-gi-survey/2.jpg)
_Ghost of Tsushima (outdoors). Source: [SIGGRAPH 2021 Advances in Real-Time Rendering](https://advances.realtimerendering.com/s2021/jpatry_advances2021/index.html#/18)_

## Screen-Space GI

The main idea of screen-space GI is to perform a linear trace or a Hi-Z/HZB trace against the depth buffer in screen space. Its advantage is low cost, since it requires no additional world-space scene representation. Its drawback is that it cannot see geometry that is off-screen, occluded, or otherwise absent from the current GBuffer.

Typical methods include SSGI (Screen-Space Global Illumination) and the Screen Trace in Lumen.

![SSGI in UE, Quality=4](/assets/img/realtime-gi-survey/3.jpg)
_SSGI in UE, Quality=4. Source: [Epic Games documentation](https://dev.epicgames.com/documentation/unreal-engine/screen-space-global-illumination?application_version=4.27)_

## Dynamic Spatial Lighting Cache GI

The goal of dynamic spatial lighting cache GI is to update lighting in real time while the game runs, store that lighting in some kind of spatial cache, and then reuse the cache for GI shading.

How the cache is laid out in space, which radiometric quantity it stores, and how the Final Gather is done yield **three largely independent classification axes**:

- **How the cache is laid out in space**: World-Space Probes, Screen-Space Probes, Surfels, Surface Atlas, Voxels, etc.
- **Which radiometric quantity is stored**: Radiance, Irradiance, etc.
- **The Final Gather method**:
  - World-Space Probe Gather: reconstruct the world-space position of each pixel and interpolate from the surrounding probes.
  - Surfel Neighborhood Gather: reconstruct the world-space position of each pixel and interpolate from the surrounding surfels.
  - Screen-Probe Gather: place screen probes on the screen; each probe shoots rays in multiple directions, and the sampled results are interpolated back to pixels.
  - Per-Pixel Raytrace Gather (full / half / quarter res): shoot rays per pixel, hit the surface cache, and read the lighting stored there.
  - And hybrid Final Gather schemes.

**Final Gather can be understood as a mapping from the spatial cache to pixels. If the current frame's spatial cache is viewed as a set, then the act of shading a pixel can be abstracted as accumulating elements of that set onto the pixel with different weights (i.e. different Final Gather methods).**

Combining these three axes **yields a wide range of GI algorithms**. This is the origin of the so-called "hybrid solutions", which are essentially combinations along these three axes. The first two axes combine to give various caching schemes, while the third axis serves as an independent scheme for the final per-pixel shading.

Below are some concrete examples.

**World-Space Probe + Irradiance + World-Space Probe Gather (complete solution):**

The representative method is DDGI. Its pipeline can be roughly described as:

1. Generate rays in multiple directions from each world-space probe.
2. Shade the hits/misses to obtain incident radiance.
3. Optionally query the previous frame's DDGI to recursively approximate diffuse multiple bounces.
4. Filter the radiance and cosine-convolve it into octahedral irradiance.
5. Update the distance moments, used to prevent light leaking.
6. At shading time, interpolate the irradiance of the surrounding probes, weighted by position, normal, visibility, etc.

The ray-hit data in DDGI is sometimes referred to as "surfels", but those are not the same representation as the surfels discussed later in this post.

Original DDGI paper: [https://jcgt.org/published/0008/02/01/](https://jcgt.org/published/0008/02/01/)

![DDGI](/assets/img/realtime-gi-survey/4.jpg)
_DDGI. Source: [Morgan McGuire's blog](https://morgan3d.github.io/articles/2019-04-01-ddgi/)_

**World-Space Probe + Irradiance + hybrid Per-Pixel Raytrace Gather and World-Space Probe Gather (complete solution):**

The main difference from the previous scheme is the hybrid Final Gather. The game example here is Assassin's Creed Shadows.

The pipeline is:

1. Reconstruct world-space positions from GBuffer pixels (the first hit) and shoot rays.
2. If a ray hits an object (the secondary hit), compute direct lighting at that point.
3. Also query and blend the 8 surrounding world-space irradiance probes.
4. Return the result and accumulate it as SH at the first hit, then upsample it to obtain the final shading value.

![Assassin's Creed Shadows](/assets/img/realtime-gi-survey/5.jpg)
_Assassin's Creed Shadows. Source: [SIGGRAPH 2025 Advances in Real-Time Rendering](https://advances.realtimerendering.com/s2025/content/Advances%202025%20-%20Raytracing%20the%20world%20of%20Assassin%27s%20Creed%20Shadows.pdf)_

**World-Space Probe + Radiance (caching scheme):**

The representative method is the world-space probes in Lumen. In Lumen, world-space probes are distributed in clipmaps over world space and store directional radiance $$L_i(x, \omega)$$.

They have low spatial density but high directional density, which allows them to be re-integrated against the BRDF, e.g. for metallic reflections within a certain roughness range. Lumen also uses them to supply distant lighting.

**Screen-Space Probe + Radiance + Screen Probe Gather (complete solution):**

The representative method is Lumen's Screen Probe Gather. The main steps are:

1. Adaptively place probes on screen tiles based on depth, normals, and other information.
2. Shoot rays in multiple directions from each probe.
3. Query radiance via Screen Trace, SDF, or hardware RT.
4. When a ray misses, look up distant radiance from the world-space radiance cache.
5. Apply filtering and temporal reprojection.
6. Interpolate probe radiance to pixels and perform the integration.

**Surface Atlas + Irradiance (caching scheme):**

The representative method is Lumen's Surface Cache, although the Surface Cache stores more than just irradiance. Lumen calls the way it updates indirect lighting in the Surface Cache "Surface Cache Radiosity". However, the algorithm involves neither the form-factor matrix for light transport between surface patches nor the hemicube from traditional radiosity, and it should therefore not be confused with classic radiosity.

The main steps are:

1. Sparsely place probes on Surface Cache texels; each probe shoots rays over the hemisphere.
2. Use Monte Carlo sampling with hardware ray tracing or SDF tracing; on a hit, read the Surface Cache's final lighting.
3. Write the ray results into a radiance atlas and filter them.
4. Project the radiance onto SH to complete the integral of the rendering equation, yielding irradiance, which is written into the indirect lighting atlas.
5. After accumulation, regenerate the final lighting atlas.

Combining the three schemes above yields an approximate outline of the Lumen pipeline:

1. Surface Atlas + Irradiance produces the final lighting atlas, which serves as a cache.
2. World-Space Probe + Radiance gathers lighting from the Surface Atlas, acting as a sparse cache and as the fallback for screen probes.
3. Screen-Space Probe + Radiance produces the lighting needed for final pixel shading, using Screen Trace, SDF, or hardware RT, or falling back to world-space probes.

**UE 5.8 adds two new Final Gather schemes, which essentially amount to adding a World-Space Probe Gather and a Per-Pixel Raytrace Gather, where the Per-Pixel Raytrace Gather uses ReSTIR.**

In effect, Lumen is a multi-level cache hierarchy: the Surface Atlas is one level, the world-space probes are another, and only at the screen-space probe level is the data used for shading. The remaining task is to handle the conversion between cache levels.

![Lumen](/assets/img/realtime-gi-survey/6.jpg)
_Lumen. Source: [Epic Games documentation](https://dev.epicgames.com/documentation/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine)_

**Surfel + Irradiance + Surfel Neighborhood Gather (complete solution):**

The representative method is GIBS (Surfel GI). Its pipeline can be roughly described as:

1. Update and recycle existing surfels.
2. Check surfaces from the GBuffer; where there are gaps in surfel coverage, spawn new surfels.
3. Build the surfel acceleration structure.
4. Shoot a limited number of rays from each surfel.
5. Evaluate radiance for hits/misses.
6. Filter the radiance and cosine-integrate it into surfel irradiance.
7. Update radial depth moments, variance, and ray-guiding data.
8. At shading time, gather irradiance from nearby surfels.
9. Use world-space probes as a fallback.

Game examples: EA Sports College Football 25, Love and Deepspace (a surfel GI, but not EA's GIBS).

![GIBS](/assets/img/realtime-gi-survey/7.jpg)
_GIBS. Source: [SIGGRAPH 2024 Advances in Real-Time Rendering](https://advances.realtimerendering.com/s2024/content/EA-GIBS2/Apers_Advances-s2024_Shipping-Dynamic-GI.pdf)_

**As these examples show, every so-called GI technique, whether a single scheme or a hybrid, is essentially a combination along the three axes above. Consequently, any GI technique, regardless of its apparent complexity, can be systematically decomposed along these three axes.**

## Voxel / SDF / Sparse Voxel GI

These methods are beyond the scope of this post and are only briefly outlined here.

Voxel-based GI discretizes the scene's geometry, materials and other information into a 3D voxel grid or a sparse voxel structure, then injects lighting as radiance and performs voxel ray tracing or cone tracing at runtime.

SDFs mainly provide visibility and hit queries and do not necessarily store lighting. When used effectively, they allow empty space to be skipped efficiently.

## Hardware Ray Tracing and Sample-Reuse GI

Hardware ray tracing is primarily a backend for ray intersection, e.g. DXR, or Vulkan 1.3 + `VK_KHR_acceleration_structure`. A variety of GI algorithms can be built on top of it: RTGI, DDGI probe updates, radiance caches, path tracing, ReSTIR, and so on.

Among these, ReSTIR and RTGI are the most common today. ReSTIR provides a better PDF while also accumulating a larger effective sample count, but it has some issues, such as color noise and disocclusion (some recent papers attempt to address these). ReSTIR also does not integrate well with DLSS-RR (Ray Reconstruction): DLSS-RR expects i.i.d. samples as input, but the samples ReSTIR produces are temporally correlated, which leads to "boiling" artifacts. In practice, alternative denoisers such as NRD can be used instead.

Game examples: Cyberpunk 2077 (ReSTIR), Indiana Jones and the Great Circle (RTGI).

![Cyberpunk 2077](/assets/img/realtime-gi-survey/8.jpg)
_Cyberpunk 2077. Source: [SIGGRAPH 2023 ReSTIR Course](https://intro-to-restir.cwyman.org/presentations/2023ReSTIR_Course_Cyberpunk_2077_Integration.pdf)_

![Indiana Jones and the Great Circle](/assets/img/realtime-gi-survey/9.jpg)
_Indiana Jones and the Great Circle. Source: [NVIDIA](https://www.nvidia.com/en-sg/geforce/news/dlss-full-ray-tracing-indiana-jones-and-the-great-circle/)_

---

Original post (in Chinese): [实时渲染中的GI方案整理以及游戏案例](https://zhuanlan.zhihu.com/p/2068657071654974329)
