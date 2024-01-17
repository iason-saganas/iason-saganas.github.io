---
layout: post
title:  "Writing a CHARMing Paper"
date:   2024-01-17 10:10:58 +0100
categories: [charming-paper]
---

{% include katexLink.html%}

In this document, I will outline important results and the process of writing the update paper to 

**[Porqueres et al. 2017: Cosmic expansion history from SNe Ia data via information field theory -- the charm code.](https://arxiv.org/abs/1608.04007){:target="_blank"}**

> CHARM: **C**osmic **H**istory **A**gnostic **R**econstruction **M**ethod. Inferring the past energy density
> evolution parameterized by redshift \\( \rho(z)\\) using Supernovae Ia data.

## Starting Situation

In my Bachelor's Thesis, we extended Natalia's analysis to the recent [Pantheon+ Supernova Compilation](https://pantheonplussh0es.github.io){:target="_blank"} 
We also looked at the [Union2.1 dataset](https://supernova.lbl.gov/Union/){:target="_blank"} which was the dataset originally used in Natalia's paper. (Both of those catalogues stitch many different SN samples together).

Instead of an iterative MAP approach, the updated **CHARM2.0** code uses [geometrical variational inference](https://arxiv.org/pdf/2105.10470.pdf){:target="_blank"} 
and a correlated field model (**"CMF"**, NIFTy implementation of a Gaussian Process, [first described in this paper](https://arxiv.org/pdf/2002.05218.pdf)) as 
prior probability distribution.

In my Bachelor's thesis we laid the code groundwork:

- Implementation of the [Response Operator]({% post_url 2024-01-16-instrument-response %}) in NIFTy8
- Modelling the signal via a CFM 
- Defining necessary energy objects (likelihood)
- Setting up geoVI
- Synthetic data catalogue
- Rudimentary analysis of posterior samples 

### Goals that warrant further analysis 

- Speed up the current version of the code. Time it takes for the inference: \\(40-60\mathrm{min}\\) (Union2.1 data) and \\(10-15\mathrm{hrs}\\) (Pantheon+ data)
- Based on more efficient inference: Find the optimal parameters of the underlying Gaussian Process that describe the dataset best and interpret them physically (See variety of inference outcomes from Bachelor Thesis!)
- Benchmark new algorithm against original maximum-a-posteriori + perturbation approach.  Pro's and con's of each algorithm?
- Merge datasets (see section down below)
- Find or constrain fundamental cosmological parameters  (Hubble constant \\(H_0\\)) 

## Merging Datasets Together And New Data

So long the Union2.1 and Pantheon+ compilations are independent, it could be possible to merge those datasets
into a super-catalogue and construct the corresponding uncertainty matrix \\(\mathbf{\mathcal{N}}\\) on our own.

Data distribution: 

{% include Supernovae-Data-Comparison-ChartJS.html%}

Now, after 8 years since the initial announcement in 2016, the Union3 SN compilation came out last November (of 2023) containing 2000 
Supernovae samples instead of 500. Furthermore, this January (2024), after five years of dedicated search of Supernovae, 
the Dark Energy Survey paper came out, releasing ~1600 datapoints that quintuple the amount of high-redshift supernovae (\\(z>0.5\\)) we had priorly.


Further datasets that could constrain cosmological parameters are: 

- CMB Radiation and Polarization data
- BAO data
- Weak lensing measurements of galaxy clusters

It might be the case that for each of these categories the corresponding Signal Response has to be constructed and
the signal inferred. 

# Cosmographic And Other Approaches To Data

- fCDM cosmography: _Benetti_
- Analytic, auto-differentiable ΛCDM cosmography: _Karchev_
- Principal Component Analysis: _Sharma_
- Neural Network Reconstruction: _Mukherjee_