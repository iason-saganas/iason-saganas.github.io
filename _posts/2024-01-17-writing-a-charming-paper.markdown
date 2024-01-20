---
layout: post-custom
title:  "Writing a CHARMing Paper"
date:   2024-01-17 10:10:58 +0100
categories: [the-process-of-writing-papers]
---

{% include katexLink.html%}
{% comment %}
    {% include table-of-contents-on-side-margin-for-post.html%} 
{% endcomment %}

In this document, I will outline important results and the process of writing the update paper to 

**[Porqueres et al. 2017: Cosmic expansion history from SNe Ia data via information field theory -- the charm code.](https://arxiv.org/abs/1608.04007){:target="_blank"}**

> CHARM: **C**osmic **H**istory **A**gnostic **R**econstruction **M**ethod. Inferring the past energy density
> evolution parameterized by redshift \\( \rho(z)\\) using Supernovae Ia data.

## 17.01: Starting Situation

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

## 18.01: Merging Datasets Together And New Data

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


## 19.01: Speeding Up The Code

As already mentioned, my code may take up to \\(15 \mathrm{hrs}\\) for one inference run (Pantheon+ data, 
\\(\approx 1700\\). datapoints). This is extremely problematic if the 
goals of the analysis is to find the best-fit parameters of the Correlated Field Model and the run of one set of 
parameters takes that long. 

I am running the code on my local CPU (1,4 GHz Quad-Core Intel Core i5 on MacOS).

The main culprit I believe is the diagonalization of the involved \\( 1700 \times 1700 \\) noise covariance matrix \\(N\\)
of the Pantheon+ catalogue. Per my current understanding, the matrix is not diagonalized once, but hundreds if not 
thousands of times in each update step of the Bayesian Inference. The matrix itself is also non-trivially changed 
per each update step, so it is not as if the same transformation is redundantly repeated. 

Of course, we could switch to running the code on a TPU, but it is important to re-check the code for any 
redundant numerical transformations for efficiency's sake.

In [this post]({% post_url 2024-01-19-optimize-kl-function %}) I explain my understanding of the 'optimize_kl()' function.