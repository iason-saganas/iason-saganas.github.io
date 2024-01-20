---
layout: post-custom
title:  "IFT: How To Find The Signal Covariance"
date:   2024-01-16 21:10:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}

In this document, I will show how [Natalia Porqueres et al.](https://arxiv.org/abs/1608.04007){:target="_blank"} found the Signal Covariance
Operator for the problem of reconstructing the cosmic energy density \\(\rho(z)\\) in terms of the redshift
from Supernovae Type Ia data.

<div style="border-left: 5px solid blue; padding-left: 15px; margin: 15px 0 15px 0">

<span style="color:blue"> <b> Definition 1 </b> (The Cosmological Principle ) </span>
<br>
<span></span>
The hypothesis that the universe, on large scales, is spatially homogeneous and isotropic.
Forces should act equally on large scales throughout the universe and should therefore no observable
inequalities in larges scale structures.
</div>

This is the central assumption of the Friedmann-Lemaître-Robertson-Walker (FLRW) metric. 
The FLRW metric is one of the simplest, analytical solutions to Einstein's Field Equations. 

In CHARM, 

> CHARM: Cosmic History Agnostic Reconstruction Method

we assume the FLRW metric when building the [Response Operator]({% post_url 2024-01-16-instrument-response %}){:target="_blank"} 
Concretely: the relationship between the 
luminosity distance \\(d_{\mathrm{L}}\\) and the dimensionless Hubble constant \\( E(z)\\): 

$$
d_{\mathrm{L}} = \frac{1}{a} d_H \int_0^z \frac{1}{E(z)}\mathrm{d}z
$$

This is the base gravitational model in `CHARM`. 

For a Bayesianist, hearing that a signal has stationary correlation is comparable to hearing that a variable follows
Gaussian Statistics. 

To be more precise, by 'stationary correlation' I mean that the autocorrelation of the signal field, as encoded in the 
signal covariance \\( S\\), is only a function of the relative distance \\(x-y\\) of \\(s^x\\) and \\(s^x\\) in the 
signal space (statistical homogeneity).
(By autocorrelation I mean the correlation of variables \\( s^x, s^y\\) elements of the same signal field \\(s\\)).

If the correlation statistics of a signal field are homogeneous, the Wiener-Khinchin-Theorem can be applied, which says
that the full covariance can be parameterized solely by one function, the **Power Spectrum**.