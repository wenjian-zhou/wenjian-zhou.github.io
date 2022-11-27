---
layout: posts
title: Conclusion of 'Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs' by Eric Heitz
mathjax: true
---

# Derivation of the Masking Function

In this section, the author showes how the *projected area* of the microsurface can be used to introduce the constraint on physically based masking functions. 

## Measuring Radiance on a Surface

The integral form of the outgoing radiance $$L(\omega_o, \mathcal{M})$$ of a given surface is:

$$
L(\omega_o, \mathcal{M}) = \frac{\int_{\mathcal{M}}\mathrm{projected \  area}(p_m)L(\omega_o, p_m)d p_m}{\int_{\mathcal{M}}\mathrm{projected \ area}(p_m)d p_m} \tag{1}
$$

>  the denominator is a normalization coefficient, and it gives the expression radiance units, or it'll be something like intensity.

## Microfacet Statistics

The author here relates integrals over the microsurface to integrals over the sphere:

*The Distribution of Normals*, $$D(\omega) = \int_{\mathcal{M}}\delta_{\omega}(\omega_m(p_m))d p_m$$, the unit is $$(m^2 / sr)$$. It tells us how much area we can see from direction $$\omega$$.

So integrating $$D$$ by the upper-hemispherical space gives the area of the microsurface:


$$
\mathrm{microsurface \ area} = \int_{\mathcal{M}}dp_m = \int_{\Omega}D(\omega_m)d\omega_m \tag{2}
$$

## Microfacet Projections

The projected area of the geometric surface onto the outgoing direction is the projected area of the *visible* microsurface.

At first the author uses a *spatial masking function* $$G_1$$ to define the projected area of the microsurface:

$$
\mathrm{projected \ area} = \int_{\Omega}G_1(\omega_o, p_m)\langle \omega_o, \omega_m(p_m) \rangle d p_m \tag{3}
$$
After transforming the *spatial masking function* to *statistical masking function*:

$$
\mathrm{projected \ area} = \int_{\Omega}G_1(\omega_o, \omega_m)\langle \omega_o, \omega_m \rangle D(\omega_m) d\omega_m \tag{4}
$$

>   $$D(\omega_m)$$ returns the differential area oriented to direction $$\omega_m$$, the clamped dot product ($$\langle \omega_o, \omega_m \rangle$$) projects the differential area to direciton $$\omega_o$$, and then times $$G_1$$ to get the fraction of visible area.

## Constraint on the Masking Function

By convention the geometric area is 1 $$m^2$$, so equation (3) can be written as

$$
\cos{\theta_o} * 1 m^2 = \int_{\Omega}G_1(\omega_o, \omega_m)\langle \omega_o, \omega_m \rangle D(\omega_m) d\omega_m \tag{5}
$$

An intuitive way to think about $$D$$ is that it's like a histogram, describing the *proportion* of each normal on the microsurface. But it doesn't provide information on how the normals are organized, so we need a *microsurface profile*, which describes the shape of the microsurface, to determine the exact $$G_1$$.

# Microfacet-Based BRDFs

In this section, the author defines the distribution of visible normals and shows how microfacet models are constructed. Also the author shows that the masking function is the normalization coefficient of the distribution of visible normals, and discusses the link with energy conservation for BRDFs.

## Distribution of Visible Normals

Distribution of visible normals is the distribution of normals weighted by the projected area of each normal, and by the masking function:

$$
D_{\omega_o}(\omega_m) = \frac{G_1(\omega_o, \omega_m)\langle \omega_o, \omega_m \rangle D(\omega_m)}{\cos{\theta_o}} \tag{6}
$$

>  $$\cos{\theta_o}$$ is the normalization factor

We will use it as a weighting function to average radiances:

$$
L(\omega_o, \mathcal{M}) = \int_{\Omega} L(\omega_o, \omega_m) D_{\omega_o}(\omega_m) d \omega_m
$$

## Construction of the BRDF

The radiance $$L(\omega_o, \omega_m)$$ of each microfacet can be expressed by the micro-BRDF $$\rho_{\mathcal{M}}(\omega_o, \omega_i, \omega_m)$$:


$$
L(\omega_o, \omega_m) = \int_{\Omega_{i}}\rho_{\mathcal{M}}(\omega_o, \omega_i, \omega_m) \langle \omega_i, \omega_m \rangle L(\omega_i) d\omega_i \tag{6}
$$


Next, we differentiate the radiance on a microsurface:


$$
dL(\omega_o, \mathcal{M}) = \int_{\Omega}dL(\omega_o, \omega_m)D_{\omega_o}(\omega_m)d\omega_m \tag{7}
$$


substitute with equation (6):


$$
dL(\omega_o, \mathcal{M}) = L(\omega_i)d\omega_i \int_{\Omega}\rho_{\mathcal M}(\omega_o, \omega_i, \omega_m)\langle \omega_i, \omega_m \rangle D_{\omega_o}(\omega_m)d\omega_m \tag{8}
$$


Since the macro-BRDF is defined by the equation:


$$
dL(\omega_o, \mathcal M) = \rho(\omega_o, \omega_i) \cos{\theta_i}L(\omega_i)d\omega_i \tag{9}
$$


we can arrive at the following


$$
\rho(\omega_o, \omega_i) = \frac{1}{\cos{\theta_o}\cos{\theta_i}}\int_{\Omega}\rho_{\mathcal M}(\omega_o, \omega_i, \omega_m)\langle \omega_o, \omega_m \rangle \langle \omega_i, \omega_m \rangle G_1(\omega_o, \omega_m)D(\omega_m)d\omega_m \tag{10}
$$


But this equation only describes single scattering in microsurface, multiple bounces have to be removed, it is achieved by introducing a *masking-shadowing function*  $$G_2$$


$$
\rho(\omega_o, \omega_i) = \frac{1}{|\omega_g \cdot \omega_o||\omega_g \cdot \omega_i|}\int_{\Omega}\rho_{\mathcal M}(\omega_o, \omega_i, \omega_m)\langle \omega_o, \omega_m \rangle \langle \omega_i, \omega_m \rangle G_2(\omega_o, \omega_i, \omega_m)D(\omega_m)d\omega_m \tag{11}
$$

## Construction of the BRDF with specular microfacets

because the micro-BRDF for mirror-like microfacets is 


$$
\begin{aligned}
\rho_{\mathcal M}(\omega_o, \omega_i, \omega_m) &= \| \frac{\partial \omega_h}{\partial \omega_i} \| \frac{F(\omega_o, \omega_h) \delta_{\omega_h}(\omega_m)}{|\omega_i \cdot \omega_h |}  \\
&= \frac{F(\omega_o, \omega_h)\delta_{\omega_h}(\omega_m)}{4 |\omega_i \cdot \omega_h |^{2}}
\end{aligned} \tag{12}
$$


Substitude into equation (11), we arrive at


$$
\rho(\omega_o, \omega_i) = \frac{1}{|\omega_g \cdot \omega_o||\omega_g \cdot \omega_i|}\int_{\Omega}\frac{F(\omega_o, \omega_h)\delta_{\omega_h}(\omega_m)}{4 |\omega_i \cdot \omega_h |^{2}}\langle \omega_o, \omega_m \rangle \langle \omega_i, \omega_m \rangle G_2(\omega_o, \omega_i, \omega_m)D(\omega_m)d\omega_m \tag{13}
$$


the delta function $$\delta_{\omega_h}(\omega_m)$$ allows a replacement at $$\omega_m = \omega_h$$, and after doing some cancellation:


$$
\rho(\omega_o, \omega_i) = \frac{F(\omega_o, \omega_h)G_2(\omega_o, \omega_i, \omega_h)D(\omega_h)}{4|\omega_g \cdot \omega_o||\omega_g \cdot \omega_i|} \tag{14}
$$


This is the equation for specular microfacet-based BRDFs

## Construction of the BRDF with diffuse microfacets

Substitute $$\rho_{\mathcal M} = \frac{1}{\pi}$$ into equation(11) we will get the result. 

## The BRDF Normalization Test

If the Fresnel term is 1, then rays are never transmitted, so the scattering is entirely defined by the BRDF. In this case, the rays are reflected without energy loss. This is modeled by the *White Furnace Test*:


$$
\int_{\Omega_i}\rho(\omega_o, \omega_i)| \omega_g, \omega_i | d\omega_i = 1 \tag{15}
$$


However, the *White Furnace Test* cannot be used to validate common BRDF models, which incorporate only the first scattering event. So we can design a less restrictive test. This test verifies that the distribution of rays reflected just after the first bounce and before leaving the surface is normalized. This is achieved by replacing masking-shadowing by masking alone, without Fresnel and shadowing, the BRDF becomes:


$$
\rho(\omega_o, \omega_i) = \frac{G_1(\omega_o, \omega_h)D(\omega_h)}{4|\omega_g \cdot \omega_o||\omega_g \cdot \omega_i|} \tag{16}
$$


and substitude it into equation (15), the *Weak White Furnace Test* equation is:


$$
\int_{\Omega_i}\frac{G_1(\omega_o, \omega_h)D(\omega_h)}{4|\omega_g \cdot \omega_o|}d\omega_i = 1
$$

# Common Physically Based and Non-Physically Based Masking Functions

In this section, the author reviews the Smith and V-cavity microsurface profiles, derives the masking functions, and discusses their properties. He also reviews some non-physically based masking functions.

## The Smith Microsurface Profile

The Smith microsurface profile assumes that the microsurface is not autocorrelated. The figure is like a random set of microfacets instead of a continuous surface. The height (or the normal) at one point has no correlation with any neighboring point.

The consequence is that the orientation of the microfacet is independent of masking that will potentially occur at this point, because the microsurface is not continuous.

So the masking function can be separated into two parts: the local masking function and a distant masking function. The local masking function determines whether the microfacet is visible, the distant masking function determines the probability of occlusion by a distant point of the microsurface.

The local masking function is:

$$
G^{\mathrm{local}}_{1}(\omega_o, \omega_m) = \chi^{+}(\omega_o \cdot \omega_m)
$$

It's a binary discard of backfacing microfacets.

The distant masking function is:

$$
G^{\mathrm{dist}}_{1}(\omega_o)
$$

It's not related to the normals of microfacets.

The masking function is expressed in the separable form:

$$
G_{1}(\omega_o, \omega_m) = G^{\mathrm{local}}_{1}(\omega_o, \omega_m)G^{\mathrm{dist}}_{1}(\omega_o)
$$

After substitution into equation(4), we have:

$$
G_1(\omega_o, \omega_m) = \chi^{+}(\omega_o \cdot \omega_m)\frac{\cos{\theta_o}}{\int_{\Omega} \langle \omega_o, \omega_m \rangle D(\omega_m) d\omega_m}
$$

This integral form was proposed by Ashikhmin et al. They precomputed the masking function and store in a table for evaluation at runtime.

### The Smith Masking Function

By changing the integration domain from normal to slope space, the masking function becomes:

$$
G_1(\omega_o, \omega_m) = \frac{\chi^{+}(\omega_o \cdot \omega_m)}{1 + \Lambda(\omega_o)}
$$

## The V-Cavity Microsurface Profile

Nope. Because it's not very physical.

## Non-Physically Based Masking Functions

Nope.

# Stretch Invariance of the Masking Function

In this section, the author investigates the invariance property of the masking function and of the distribution of slopes.

## Masking Probability Invariance

The masking probability is invariant to configuration stretching when all of the slopes involved in the configuration are scaled at the same time. When stretching a 1D configuration by a factor of 2, the slopes of the configuration are scaled by a factor of 1/2.

## The Distribution of Slopes

If the microsurface is a heightfield, the distribution of heights is often denoted $$P^{1}(h)$$. The slopes of the microsurface are the gradients of the heights: $$(x_{\tilde{m}}, y_{\tilde{m}}) = \nabla h$$, the distribution of the slopes is denoted $$P^{22}(x_{\tilde{m}}, y_{\tilde{m}})$$.

The distribution of normals is constructed from it:

$$
D(\omega_m) = \frac{P^{22}(x_{\tilde{m}}, y_{\tilde{m}})}{\cos^4{\theta_m}}
$$

When roughness must be explicit, we use $$P^{22}(x_{\tilde{m}}, y_{\tilde{m}}, \alpha)$$.

## Isotropic Shape-Invariant Distributions of Slopes

Several isotropic parametric distributions of slopes depend on a roughness parameter $$\alpha$$. In this case, the distribution of slopes depends on the ratio $$\frac{\tan{\theta_m}}{\alpha}$$.

$$
P^{22}(x_{\tilde{m}}, y_{\tilde{m}}, \alpha) = \frac{1}{\alpha^2}f(\frac{\sqrt{x^2_{\tilde{m}} + y^2_{\tilde{m}}}}{\alpha}) = \frac{1}{\alpha^2}f(\frac{\tan{\theta_m}}{\alpha})
$$

where $$f$$ is a 1D function that defines the shape of the distribution.

## Anisotropic Shape-Invariant Distributions of Slopes

In this section, the author extends the isotropic version to anisotropic version with azimuth-dependent factors.

$$
P^{22}(x_{\tilde{m}}, y_{\tilde{m}}, \alpha_x, \alpha_y) = \frac{1}{\alpha_x \alpha_y}f(\sqrt{\frac{x^2_{\tilde{m}}}{\alpha_x^2} + \frac{y^2_{\tilde{m}}}{\alpha_y^2}}) = \frac{1}{\alpha_x \alpha_y}f(\tan{\theta_m}\sqrt{\frac{\cos^2{\phi_m}}{\alpha_x^2} + \frac{\sin^2{\phi_m}}{\alpha_y^2}})
$$

## More Generalization

In this section, the author introduces another 2 extended forms of distributions: *Arbitrary Shape-Invariant Distributions* and *Vertical Shearing and Non-Centered Distributions*.

# The Smith Joint Masking-Shadowing Function

In this section, the author reviews the use of Smith masking function with the light direction and its joint form with the masking function: the masking-shadowing function.

## Separable Masking and Shadowing

It's the most widely used variant of the masking-shadowing function proposed by Walter et al.[2007].

$$
\begin{aligned}
G_2(\omega_o, \omega_i, \omega_m) &= G_1(\omega_o, \omega_m)G_1(\omega_i, \omega_m) \\
&= \frac{\chi^{+}(\omega_o \cdot \omega_m)}{1 + \Lambda(\omega_o)}\frac{\chi^{+}(\omega_i \cdot \omega_m)}{1 + \Lambda(\omega_i)}
\end{aligned} \tag{98}
$$

This form does not model correlations between masking and shadowing, and therefore always overestimates shadowing since some correlation exists.

## Height-Correlated Masking and Shadowing

It's a more accurate form of the masking-shadowing function that models the correlation between masking and shadowing due to the height of the microsurface. It's proposed by Ross et al.[2005].

$$
G_2(\omega_o, \omega_i, \omega_m) = \frac{\chi^{+}(\omega_o \cdot \omega_m)\chi^{+}(\omega_i \cdot \omega_m)}{1 + \Lambda(\omega_o) + \Lambda(\omega_i)}
$$

The author suggests us to use this form in practice, it's more accurate than the separable form. But this form is accurate when the outgoing and incident directions are far away from each other, but overestimates shadowing when the directions are close.

## Direction-Correlated Masking and Shadowing

Masking and shadowing are strongly correlated when the outgoing and incident directions are close. When $$\omega_o = \omega_i$$, microfacets are visible from both direction $$\omega_o$$ and $$\omega_i$$. In this case, the shadowing should be removed from the BRDF. This is known as the "hotspot effect": when the view and light directions are parallel, shadows disappear.

Ashikhmin et al. [2000] account for directional correlation by blending the separable form of Equation(98) with the case where both directions are fully correlated:

$$
G_2(\omega_o, \omega_i, \omega_m) = \lambda(\phi)G_1(\omega_o, \omega_m)G_1(\omega_i, \omega_m) + (1 - \lambda(\phi))\min{(G_1(\omega_o, \omega_m),G_1(\omega_i, \omega_m))}
$$

where $$\lambda(\phi)$$ is an empirical factor similar to Ginneken et al.'s.

## Height-Direction-Correlated Masking and Shadowing

The directional correlation between masking and shadowing can be modeled by incorporating a directional correlation factor $$\lambda$$ into the height-correlated form:

$$
G_2(\omega_o, \omega_i, \omega_m) = \frac{\chi^{+}(\omega_o \cdot \omega_m)\chi^{+}(\omega_i \cdot \omega_m)}{1 + \max (\Lambda(\omega_o), \Lambda(\omega_i)) + \lambda(\omega_o, \omega_i) \min (\Lambda(\omega_o), \Lambda(\omega_i))}
$$

# Discussion and Future Work

In this section, the author discusses some ideas possible for future work:

1. Deriving the Smith Masking Function for Other Commonly Used Models
2. Correlation of Masking-Shadowing
3. Multiple Scattering

# Reference

* Heitz, Eric. "Understanding the masking-shadowing function in microfacet-based BRDFs." *Journal of Computer Graphics Techniques* 3.2 (2014): 32-91.