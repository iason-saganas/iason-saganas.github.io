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

See [the DES paper](https://arxiv.org/pdf/2401.02929.pdf){:target="_blank"} 

and [the Union3 paper](https://arxiv.org/pdf/2401.02929.pdf){:target="_blank"}.

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

In [this post]({% post_url 2024-01-19-optimize-kl-function %}){:target="_blank"} I explain my understanding of the 'optimize_kl()' function.

- Note 26.01: I think setting a GP might lead to several possibly unnecessary standardization transforms => Maybe try setting the power spectrum MORE explicitly than just loglogavgslope = (-4, something very small). ,
- Time what time is needed for the code to run before the `optimize_kl()` call, what time during it, what time during the `sampledKL_energy()` call and what time is spent diagonalizing matrices
- To run geoVi, you need to get the coordinate transform of the parameter space in which the likelihood distribution metric is the Euclidean one. In case of Gaussian Likelihoods, like we have here, the transform is the square root of the inverse Noise Covariance Matrix. I went about this problem by doing an eigendecomposition and returning the eigendecomposition with a squared eigenvalues diagonal matrix. There may be a more efficient process to do this. See [this stackoverflow thread ("Find the square root of a matrix")](https://math.stackexchange.com/questions/59384/find-the-square-root-of-a-matrix){:target="_blank"} for inspiration. 

#### Note 28.01 

We can visualize which part of our code takes up how much time and benchmark that against the demos NIFTy comes shipped,
which we assume to be efficient. 
We compare against 'getting_started_3.py', since a 'LOSResponse' Operator is employed.
In the CHARM2.0 code, our response operator is an operator chain at whose heart also lies the 'LOSResponse' Operator.

We use the radial line of sights mode and infer a signal field of \\(90 \times 90\\) pixels, 8100 pixels in total,
from a synthetic dataset of 580 points. 
We compare the computation time to that of Charm2.0 in case the Union2.1 dataset is used.
There, a signal field of resolution \\(8000 \times 1\\) is inferred, also from 580 datapoints.

Red \\( + \\) Yellow Bar \\( \approx \\) Blue Bar, 

Green \\( + \\) Tourquoise Bar \\( \approx \\) Red Bar.

{% include Computational-Benchmark-Absolute-ChartJS.html %}

It will probably be more informative to check the relative time taken:

{% include Computational-Benchmark-Relative-ChartJS.html %}

INFO: Time it took for diagonalizing some manipulated noise uncertainty matrix object: 
about 10 minutes, so 30% of time (Union2.1 data).

Code for finding the descent direction: 

```python
>>> optimize_kl()

e, _ = minimizer(e)
mean = MultiField.union([mean, e.position]) if mf_dom else e.position
sl = e.samples.at(mean)
energy_history.append((iglobal, e.value))
```

Code for transforms and metric that calls the apply method of the signal response: 

```python
>>> SampledKLEnergy() > draw_samples()


# Construct transformation
geometric = minimizer is not None
if geometric:
    tr = H.likelihood_energy.get_transformation()
    if tr is None:
        raise ValueError("Geometric sampling only works for likelihoods")
    dtype, f_lh = tr
    if isinstance(dtype, dict):
        myassert(all([dtype[k] is not None for k in dtype.keys()]))
    else:
        myassert(dtype is not None)
    scale = ScalingOperator(f_lh.target, 1., dtype)



    fl = f_lh(Linearization.make_var(sam_position))

    transformation = ScalingOperator(f_lh.domain, 1.) + fl.jac.adjoint @ f_lh

    transformation_mean = sam_position + fl.jac.adjoint(fl.val)

    # Note: This metric is equivalent to H.metric, except for the case of a
    # `VariableCovarianceGaussianEnergy` with `use_full_fisher = True`.
    met = SamplingEnabler(SandwichOperator.make(fl.jac, scale),
                              ScalingOperator(fl.domain, 1., float),
                              H.iteration_controller)


else:
    met = H(Linearization.make_var(sam_position, want_metric=True)).metric
    if napprox >= 1:
    met._approximation = makeOp(approximation2endo(met, napprox))
    # /Construct transformation
```

Code for actually sampling KL: 

```python
>>> SampledKLEnergy() > draw_samples()


# Draw samples
sseq = random.spawn_sseq(n_samples)
if mirror_samples:
    sseq = reduce(lambda a, b: a+b, [[ss]*2 for ss in sseq])
local_samples = []
local_neg = []
utilities.check_MPI_synced_random_state(comm)
utilities.check_MPI_equality(sseq, comm)
y = None

ntask, rank, _ = get_MPI_params_from_comm(comm)
for i in range(*shareRange(len(sseq), ntask, rank)):
    with random.Context(sseq[i]):
        neg = mirror_samples and (i % 2 != 0)
        if not neg or y is None:  # we really need to draw a sample
            y, yi = met.special_draw_sample(True)

        if geometric:
            m = transformation_mean - y if neg else transformation_mean + y
            pos = sam_position - yi if neg else sam_position + yi
            en = GaussianEnergy(m) @ transformation
            en = EnergyAdapter(pos, en, nanisinf=True, want_metric=True)
            en, _ = minimizer(en)
            local_samples.append(en.position - sam_position)
            local_neg.append(False)
        else:
            local_samples.append(yi)
            local_neg.append(neg)
```