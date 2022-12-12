---
title: "Physically Based Renderer"
excerpt: "Rendering techniques study and implementation"
header:
  image: /assets/images/renderer/water_caustic.png
  teaser: assets/images/renderer/water_caustic.png
sidebar:
  - title: "Role"
    #image: http://placehold.it/350x250
    #image_alt: "logo"
    text: "Developer"
  #- title: "Responsibilities"
    #text: "Reuters try PR stupid commenters should isn't a business model"
gallery:
  - url: /assets/images/renderer/water_caustic.png
    image_path: assets/images/renderer/water_caustic.png
    alt: "placeholder image 1"
  - url: /assets/images/renderer/volume_caustic.png
    image_path: assets/images/renderer/volume_caustic.png
    alt: "placeholder image 2"
  - url: /assets/images/renderer/sssdragon.png
    image_path: assets/images/renderer/sssdragon.png
    alt: "placeholder image 3"
  - url: /assets/images/renderer/dragon.png
    image_path: assets/images/renderer/dragon.png
    alt: "placeholder image 4"
gallery2:
  - url: /assets/images/renderer/lte-orb-single.png
    image_path: assets/images/renderer/lte-orb-single.png
    alt: "placeholder image 1"
  - url: /assets/images/renderer/lte-orb-multi.png
    image_path: assets/images/renderer/lte-orb-multi.png
    alt: "placeholder image 2"
---

[This](https://github.com/wenjianzhou2931/Renderer) physically based renderer was developed when I was reading [Physically Based Rendering: From Theory To Implementation](https://pbr-book.org/) for self-study purpose. It doesn't have any user interface yet, all the scenes must be set using code. I'll add user interface in the future.

Features:
* Integrators: path tracing and volumetric path tracing.
* Materials: matte, glass, metal, plastic, microfacet model.
* Implemented [Multiple-Scattering Microfacet BSDFs with the Smith Model](https://eheitzresearch.wordpress.com/240-2/), below is the comparison with classic microfacet model.

{% include gallery id="gallery2" layout="half" caption="Left: microfacet model, Right: Heitz's multiple scattering model" %}

Below are some nice render results:

{% include gallery id="gallery" layout=" " caption="These are some render results from my renderer." %}