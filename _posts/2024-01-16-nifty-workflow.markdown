---
layout: post-custom
title:  "The Nifty-Gritty: A CHARMing Project Example"
date:   2024-01-16 17:21:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}

> A play on words and a play on acronyms. This has to be good.  

This document describes the workflow of the code I've written in my Bachelor's thesis, and it may help 
as a template and an example for beginners as to how the NIFTy workflow works (next to the other excellent
examples in the [NIFTy Project Page](https://ift.pages.mpcdf.de/nifty/user/getting_started_0.html){:target="_blank"} ).

I will also slightly outline the theory specific to my Bachelor's Thesis which is 
then actually implemented in NIFTy. 

Steps:
 
1. **Determine what your data is.** (Let's say a \\(n\\)-dim array of tuples \\( (x_i,y_i) \\) ). 
2. **The Information Field Theory part** (In no specific order):
   - Define your to-be-inferred signal and pick a prior, e.g. a Gaussian \\(\mathcal{P}(s)=\mathcal{G}(s,S)\\).
   - Find the Signal Response Operator \\(R(s)\\) for your problem. The Response Operator determines the relationship between the signal and the data see [here]({% post_url 2024-01-16-instrument-response %}){:target="_blank"}  for a post about my personal view of the Instrument Response Equation and an example of how [Natalia Porqueres et al. 2017](https://arxiv.org/abs/1608.04007){:target="_blank"} found the Signal Response for the problem of determining the evolution of the cosmic energy density.
   - Find the Covariance Operator \\(S\\) for your problem. 
   - Find the likelihood for \\( \mathcal{P}(d\mid s) \\) your problem.
3. **The Numerical Information Field Theory part**:
   1. Read your data
   2. Define the signal space, i.e. the space that the signal 'lives in'
   3. Define the data space, i.e. the space the data 'lives in'. Define the data field by filling the data space with the read data values
   4. Define the to be inferred signal field as a simple correlated field model based on the signal space. If you are able to use the Wiener-Khinchin-Theorem to parameterize the Signal Covariance operator \\(S \\) in terms of the power spectrum, this is where that information is injected into.
   5. Implement the Response Operator \\(R(s) \\) and the Noise Covariance Operator \\(N \\)
   6. Define necessary minimization and sampling controllers
   7. Implement the likelihood energy 
   8. Run the geoVI algorithm by calling 'optimize_kl'
   9. Analyze the posterior samples

In the following, I will showcase an exemplary workflow of step 3. The contents are the code I've written for my
Bachelor's thesis, which can be found [on github](https://github.com/iason-saganas/CHARM2.0_final).

