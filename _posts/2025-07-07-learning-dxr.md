---
layout: post
title: Learning DXR
date: 2025-07-07 16:30:00
description: My studying notes of DXR.
tags: graphics DX12
categories: StudyNotes
---

## Initialting raytracing

1. Set pipline state containing raytracing shaders using `SetPipelineState()`.
2. Invoke raytracing by `DispatchRays()` just like rasterization and compute are invoked by `Draw()` and `Dispatch()`.
3. Tier 1.1: 
   * adds support for GPU initiated `DispatchRays()` via `ExecuteIndirect()`.
   * adds support for a variant of raytracing that can be invoked from any shader stage(<span style="color:red">*What???*</span>).

## Ray generation shaders

`DispatchRays()` invokes threads of ray generation shaders, and each of them knows it's location in the "grid", and can generate rays via `TraceRay()`.

## Rays

Which includes: origin, direction, and parametric interval `(TMin, TMax)`, positions along the ray is `origin + T * direction`.

A ray is visible to the *caller* of `TraceRay()`, it comes with a user defined payload that is modifiable as the ray interacts with geometry in a scene(<span style="color:red">*What???*</span>)

`TMin` never changes over the lifetime of a ray, the system reduces `TMax` to reflect the closest intersection.

## Raytracing output

Shaders output results such as color samples for an image, manually through UAVs.

## Ray-geometry interaction diagram

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/rayGeometryInteraction.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Geometry and acceleration structures

Geometry in a scene is described as a system using two levels of acceleration structures: **Bottom-level acceleration structures(B-LAS)** consist of a set of geometries that are building blocks for a scene, while **Top-level acceleration structures(T-LAS)** represents a set of instances of B-LAS structures.

Within a given BLAS there can be:

1. Triangle meshes
2. Procedural primitives initially described by an AABB

A BLAS can only contain a single geometry type。

Given a set of BLAS, the application defines a set of instances which points to a BLAS structure and includes information for specializing the instance, like matrix transform and a user defined InstanceID. Then this set of instances are used to generate TLAS into GPU memory.

## Acceleration structure updates

BLAS only allows vertex positions to be updated. TLAS is more flexible.

## Built-in ray-triangle intersection - triangle mesh geometry

Uses built-in intersection for triangles.

## Intersection shaders - procedural primitive geometry

An alternative representation for geometry in a BLAS which uses procedural primitives.

## Any hit shaders

Can be defined to run whenever a ray intersects a geometry instance.

Any hit shaders can:

1. read intersection attributes
2. modify ray payload
3. indicate a hit should be ignored
4. accept the hit and continue
5. accept the hit and tell system to stop

When accepts a hit, the `T` value becomes the new `TMax`. It can't trace new ray.

## Closest hit shaders

Can:

1. read intersection attributes
2. modify ray payload
3. generate additional rays

A typical use would be to evaluate the color of a surface and either contribute to the ray payload or store data to memory via UAV.

If both any-hit and closest-hit shader types are defined for the geometry instance at the closest hit's `T` value, the any-hit shader will always run before the closest hit shader.

## Miss shaders

For rays that do not intersect any geometry. Can:

1. modify ray payload
2. generate additional rays

## Hit groups

A hit group is one or more shaders consisting of:

* 0 or 1 intersection shader
* 0 or 1 any hit shader
* 0 or 1 closest hit shader

Individual geometries in a given instance each refer to a hit group.

## TraceRay control flow

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/traceRayControlFlow.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Instance masking

<span style="color:red">*will finish this part when I understand it*</span>

## Callable shaders

Defined through *shader table*, implementations are expected to schedule callable shaders for execution separately from the calling shader.

## Resouce binding

### Local root signatures vs global root signatures

* *local root signature*, enables each shader to have unique arguments.
* *global root signature*, whose arguments are shared across all raytracing shaders and compute PSOs on CommandLists, set via `SetComputeRootSignature()`

If any local root signature makes a sampler definition such as `s0`, all local root signatures that define `s0` must have the same definition.

## Shader identifier

A shader identifier identified one of the raytracing shaders: ray gen, hit group, miss, callable. It can be thought of as *a pointer to a shader*.

For separate raytracing pipelines or collections of code, the same shader may not return the same identifier depending on the implementation.

## Shader record

```
shader record = {shader identifier, local root arguments for the shader}
```

A shader record refers to a region of memory owned by the application in the above layout.

## Shader tables

```
shader table = {shader record A}, {shader record B}, ...
```

A shader table is a set of shader records.


---

## Example in Microsoft DirectX Raytracing Sample

Before Rendering:

1. OnInit
   * Create device resources
   * Scene initialization: setup primitives, materials, cameras, lights...
   * Create resources that depend on the device
     * Any auxilary device resources
     * Create raytracing interfaces: device, commandlist
     * Create root signatures for shaders: global root signature, local root signature (<span style="color:orange">how to design local root signature? make separate LHS or a shared LHS? Looks like it can be used with "bindless"</span>)
     * Create raytracing PSO: DXIL library, hit group types, shader config, local/global root signature, pipeline config
     * Create descriptor heap
     * Build geometry
     * Build TLAS and BLAS
     * Create constant buffer
     * Create AABB primitive attribute buffers(<span style="color:orange">not neccessary</span>)
     * Create shader table(ray gen, miss, hit group...)
     * Create output resources(for storing raytracing result)
2. When rendering
   * Do any update you want: animate camera, animate geometries...
   * Do raytracing(dispatch)
3. After rendering
   * Clean up neccessary things
