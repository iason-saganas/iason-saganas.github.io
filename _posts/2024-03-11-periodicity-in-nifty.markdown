---
layout: post-custom
title:  "The Nifty-Gritty: Reconstructing Non-Periodic Signals"
date:   2024-01-16 17:21:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}
 
This article collects the troubles I ran into in reconstructing a non-periodic signal in `NIFTy`. 
First and foremost, try not to reconstruct a non-periodic signal with a model that enforces periodic boundary conditions.
In my case (`CHARM`), the expansion of the universe is very different at different redshifts and the signal field values
increase monotonically with redshift - no periodicity. 

I started off using the correlated field model, in order to model uncertainties in the correlation structure.
The periodicity of the correlated field model can be broken using a zero padding operator. But still, the length of the
signal space affects the reconstruction thanks to the `fluctuations` parameter.
We opted to use a slightly different model to capture the monotonicity of the problem: A generative line
model, where the deviations about that line are modelled using a `cfm`.

> Side note: now,
the deviations have periodic boundary conditions, don't they!!! So now, we enforce that the reconstruction deviates from
the base line at high redshifts in the same way that it deviates at low redshifts. 
> Does it really make sense for the correlated field to model how one point in signal space deviates from the next 
> while adding the base line?
> So plot and analyze the difference between signal and the inferred alpha beta parameters (which shouldn't be harmonic?) 

## A non-periodic line as ground truth

Say you want to reconstruct the following ground truth: 

![Line model ground truth](../../../../media/nifty-posts/non-periodic-signal-inference/Line-Ground-Truth.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

From this set of data: 

![Line model data](../../../../media/nifty-posts/non-periodic-signal-inference/Line-Model-Data.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

The signal response is a simple line-of-sight integration: 

$$
R(s) = \int_0 ^x s(\tilde{x}) \hspace{1mm}\mathrm{d}\tilde{x}.
$$

First, we define the following functions in a file called `generative_line_model_helpers.py`: 

```python
import numpy as np
import nifty8 as ift
from nifty8.operators.operator import _OpChain
import pickle

__all__ = ['unidirectional_radial_los', 'build_response', 'attach_custom_field_method',
           'line_model', 'pickle_me_this', 'unpickle_me_this', 'kl_sampling_rate']


def unidirectional_radial_los(n_los: int) -> np.ndarray:
    """
    Generates an ordered array of lognormally distributed random numbers.
    :param n_los:   int,        Number of lines-of-sight to construct.
    :return: ends: np.array,    The synthetic lines-of-sight.
    """
    n_los = int(n_los)
    arr = np.random.lognormal(mean=0, sigma=.2, size=n_los)
    maximum = np.max(arr)
    ends = np.sort(arr / maximum)
    return ends


def build_response(num_of_dtps, los_ends, signal_domain):
    integrate = ift.LOSResponse(domain=signal_domain,
                                starts=[np.zeros(int(num_of_dtps))],
                                ends=[los_ends])
    return lambda s: integrate(s)


def line_model(signal_domain: ift.RGSpace, custom_alpha=None, custom_beta=None) 
               -> _OpChain:

    alpha = ift.NormalTransform(mean=0, sigma=1, key="slope")
    beta = ift.NormalTransform(mean=0, sigma=5, key="y-intercept")
    if custom_alpha is not None:
        if custom_beta is None:
            raise ValueError("Custom beta and must be specified.")
        alpha = ift.NormalTransform(mean=custom_alpha, sigma=.01, key="slope")
        beta = ift.NormalTransform(mean=custom_beta, sigma=.01, key="y-intercept")

    contraction = ift.ContractionOperator(domain=signal_domain, spaces=None)
    alpha = contraction.adjoint @ alpha
    beta = contraction.adjoint @ beta

    x_coord = ift.DiagonalOperator(diagonal=signal_domain.field())
    return x_coord @ alpha + beta


def attach_custom_field_method(space: ift.RGSpace):
    """
    Takes an instance of `RGSpace`, a regular cartesian grid, and attaches
    a new method called `field` which can then be called on the `RGSpace` instance.
    The `field` method returns a field of the values associated with the `distances`
    property of the raw, geometric object `RGSpace`.
    Example:

    Let the domain of a signal field be a regular, 1D grid space of length 100
    and consist of 5000 pixels. Then, each pixel has a width of 0.02.

    Calling this method will return a `NIFTy` field object whose values
    are [0.02, 0.04, 0.06, ..., 100]. This way, the distance values of the domain may
    be, e.g., added to the signal field, as needed in the signal
    response of `CHARM`.
    :param space: RGSpace,      The regular grid space to attach the 'field' method to.
    :return: space:             The regular grid with the method attached.
    """

    def field(rg_space):
        dimension = len(rg_space.shape)
        if dimension != 1:
            raise ValueError(f"Class method 'field' only works for 1D grid spaces, \n"
                             f"not for {dimension}D grid spaces.")

        values = []
        resolution = rg_space.shape[0]
        distances = rg_space.distances[0]
        for iindex in range(1, resolution + 1):
            values.append(iindex * distances)
        dom = ift.DomainTuple.make((rg_space,))
        return ift.Field(dom, val=np.array(values))

    space.field = lambda: field(space)
    return space


def pickle_me_this(filename: str, data_to_pickle: object):
    path = filename
    file = open(path, 'wb')
    pickle.dump(data_to_pickle, file)
    file.close()


def unpickle_me_this(filename):
    file = open(filename, 'rb')
    data = pickle.load(file)
    file.close()
    return data


def kl_sampling_rate(index):
    return 5 if index < 10 else 15

```

We then define the signal space as a regular grid space, the number of synthetic datapoints, a generative line model 
etc. inside of a file called `generative_line_model_setup.py`:

```python
import os.path
from generative_line_model_helpers import *
import nifty8 as ift
import numpy as np

config = {
    'Signal Field Resolution': 2048,
    'Number of synthetic datapoints': 580,
    'Length of signal space': 1,
    'Noise level': .01,
}

n_pix, n_dp, x_length, noise_level = [float(setting) for setting in config.values()]

x = ift.RGSpace(shape=n_pix, distances=x_length/n_pix)
x = attach_custom_field_method(x)  # Attach the `field` method

x_ground = ift.RGSpace(shape=2000, distances=x_length/n_pix)
x_ground = attach_custom_field_method(x_ground)

s_ground = line_model(signal_domain=x, custom_alpha=2, custom_beta=0)
s = line_model(signal_domain=x)  # The signal model guess

data_space = ift.UnstructuredDomain((n_dp,))

if os.path.isfile("los.pickle"):
    los = unpickle_me_this("los.pickle")
else:
    los = unidirectional_radial_los(n_dp)
    pickle_me_this("los.pickle", los)
R = build_response(num_of_dtps=n_dp, los_ends=los, signal_domain=x)
R_ground = build_response(num_of_dtps=n_dp, los_ends=los, signal_domain=x_ground)
N = ift.ScalingOperator(data_space, noise_level, np.float64)

# Iteration control for `MGVI` and linear parts of the inference
ic_sampling_lin = ift.AbsDeltaEnergyController(name="Precise linear sampling",
                                               deltaE=0.05,
                                               iteration_limit=100)

# Iteration control for `geoVI`
ic_sampling_nl = ift.AbsDeltaEnergyController(name="Coarser, nonlinear sampling",
                                              deltaE=0.5,
                                              iteration_limit=20,
                                              convergence_level=2)
# For the non-linear sampling part of geoVI, the iteration controller has to be
# "promoted" to a minimizer:
geoVI_sampling_minimizer = ift.NewtonCG(ic_sampling_nl)

# KL Minimizer control, the same energy criterion as the geoVI iteration control,
# but more iteration steps
ic_newton = ift.AbsDeltaEnergyController(name='Newton Descent Finder',
                                         deltaE=0.5,
                                         convergence_level=2,
                                         iteration_limit=35)
descent_finder = ift.NewtonCG(ic_newton)

```

Using a Gaussian Energy Likelihood, we can run the inference to get: 

![Line Model Reconstruction](../../../../media/nifty-posts/non-periodic-signal-inference/Line-Model-Reconstruction.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

which is in good accordance with the ground truth.
Notice that although the documentation of the `NIFTy8` `RGSpace` states that

> Topologically, a n-dimensional RGSpace is a n-Torus, i.e. it has periodic
boundary conditions.

no periodic boundary conditions are actually enforced on our signal. 

Furthermore, let the signal domain consist of `n_pix` many pixels and the domain length be `x_length=1.`.
The width of one regular grid space cell is defined via the `distances` parameter as 
\\(n_{\mathrm{pix}}/x_{\mathrm{length}}\\).

In numerical experiments we verified that the signal reconstruction does not change in the region of interest if:

- `distances` is kept the same, but `shape` is updated to twice, three or four times the old `n_pix` value 
- `n_pix` as well as `distances` according to the prescription above are updated to two, three or four times the old `n_pix`value

## Cosine signal model: \\(2\pi \\)-Periodicity 

Now, we switch to a cosine model, which is \\(2\pi \\) periodic and investigate effects of increasing signal domain 
resolution and length on the reconstruction.

```python
def cosine_model(signal_domain: ift.RGSpace, custom_alpha=None, custom_beta=None) 
                 -> _OpChain:

    alpha = ift.NormalTransform(mean=0, sigma=1, key="slope")
    beta = ift.NormalTransform(mean=0, sigma=5, key="y-intercept")
    if custom_alpha is not None:
        if custom_beta is None:
            raise ValueError("Custom beta must be specified.")
        alpha = ift.NormalTransform(mean=custom_alpha, sigma=.01, key="slope")
        beta = ift.NormalTransform(mean=custom_beta, sigma=.01, key="y-intercept")

    contraction = ift.ContractionOperator(domain=signal_domain, spaces=None)
    alpha = contraction.adjoint @ alpha
    beta = contraction.adjoint @ beta

    x_coord = ift.DiagonalOperator(diagonal=np.cos(signal_domain.field()))
    return x_coord @ alpha + beta
```

and in the setup file we only change

```python
s_ground = cosine_model(signal_domain=x, custom_alpha=-.5, custom_beta=0)
s = cosine_model(signal_domain=x)  # The signal model guess
```

![Cosine Model Data](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-data.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

![Cosine Model Ground Truth](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-ground-truth.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

And the reconstruction is: 

![Cosine Model Reconstruction](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-reconstruction.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

Because of the periodicity of the model, the data is not enough to properly constrain the reconstruction at high 
\\(x \\).  

Again, we verify that the reconstruction does not change if the signal domain length is increased or more pixel values 
are packed into the same domain length.

## Cosine signal model: Periodicity on the boundaries

This is a mock of the correlated field model, that is also periodic on the boundary conditions, i.e. the first and 
last pixel value of the `RGSpace`. 

Now, investigate the data, ground truth and reconstruction emerging from a cosine generative model with the
same signal response. For the sake of pedagogics, let this signal model have periodic boundary conditions, 
i.e. let \\(x_{\mathrm{length}}\\) be the length of the signal space, then multiply \\( x \\) in the definition of 
the model with \\(b=2\pi /x_{\mathrm{length}} \\) to ensure that it must return to its initial signal value 
at the end of the `RGSpace`.

```python 
def cosine_model_periodic_bc(signal_domain: ift.RGSpace, custom_alpha=None,
                             custom_beta=None, custom_period=None) -> _OpChain:
    alpha = ift.NormalTransform(mean=0, sigma=1, key="slope")
    beta = ift.NormalTransform(mean=0, sigma=5, key="y-intercept")
    if custom_alpha is not None:
        if custom_beta is None:
            raise ValueError("Custom beta must be specified.")
        alpha = ift.NormalTransform(mean=custom_alpha, sigma=.01, key="slope")
        beta = ift.NormalTransform(mean=custom_beta, sigma=.01, key="y-intercept")

    contraction = ift.ContractionOperator(domain=signal_domain, spaces=None)
    alpha = contraction.adjoint @ alpha
    beta = contraction.adjoint @ beta

    x_length = np.max(signal_domain.field().val)
    b = 2 * np.pi / x_length
    if custom_period is not None:
        b = 2 * np.pi / custom_period

    x_coord = ift.DiagonalOperator(diagonal=np.cos(b * signal_domain.field()))
    return x_coord @ alpha + beta
```

Let the signal ground truth itself have a custom period of \\(10\\). The ground truth and guessed signal model are:

```python
s_ground = cosine_model_periodic_bc(signal_domain=x_ground, custom_alpha=-.5, 
                                    custom_beta=0, custom_period=10)
# The ground truth model
s = cosine_model_periodic_bc(signal_domain=x)  # The signal model guess
```

The data are: 

![Cosine Model Reconstruction](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-model-periodic-bc-data.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

and the ground truth:

![Cosine Model Reconstruction](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-model-periodic-bc-ground-truth.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

Let's look at the reconstruction: 

![Cosine Model Reconstruction](../../../../media/nifty-posts/non-periodic-signal-inference/cosine-model-periodic-bc-reconstruction.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

Obviously, not good.
We see that the periodic boundary condition of our signal model hinders us at achieving a good reconstruction. 
How to solve this?  
Let's go with: 'You need to extend the signal space, make it longer, so `NIFTy` has the room to "breathe" and can 
reconstruct the signal field in the region of interest and at the same time fulfill the periodic boundary conditions 
of the signal in the rest of the domain'.

Let's double and triple the signal domain and look at the reconstructions. To be more specific, by extending we mean 
changing the signal domain such that the resolution of the regular grid space (`distances`) stay the same, but we 
just add more pixels at the end of the domain, i.e. `shape` will be `x_fac * n_pix` with some extension factor `x_fac`.

```python
x_fac = 2  # x_fac = 3
x = ift.RGSpace(shape=x_fac*n_pix, distances=x_length/n_pix)
x = attach_custom_field_method(x)  # Attach the `field` method
```

Expectation: Each reconstruction will look differently, because a different length of the signal domain fundamentally
changes the model! In our example: A "stretched" cosine is not the same model as a "squished" cosine. 

![Cosine Model Reconstructions Different Signal Domain Lengths](../../../../media/nifty-posts/non-periodic-signal-inference/wild-reconstruction.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

As expected. Obviously, our signal model, which has periodic boundary conditions, will squeeze and stretch arbitrarily as a 
reaction to the arbitrary extension of the signal domain length, in order to fit those boundary conditions.

Furthermore, potentially unphysical signal field values are now used to explain the data, the reconstructions 
cannot just agree up to the end of non-extended signal domain. 

Instead, 
what you need is a way to extend the signal domain that *really* differentiates between the region of interest and 
the "do whatever to fulfill your boundary conditions without affecting the reconstruction in the important part". 

This can be done via zero-padding.

## Zero-Padding the cosine signal model with periodic boundary conditions

The idea here is to extend the signal domain by applying an operator to the signal _field_ that fills the field with 
unnecessary zeroes. 
In case of convolutions used in a `Fast Fourier Transform`, this technique called "Zero Padding"
makes (or should at least make) the inference faster, if the domain is extended such that it contains 
\\(2^a, \hspace{2mm} a\in \mathbb{N}\\) values.  

A zero padder \\(\hat{X}\\) maps from \\(s(x)\\) to \\(s(x_{\mathrm{ext}})\\). The adjoint \\(\hat{X}^{\dagger}\\)
maps from \\(s(x_{\mathrm{ext}})\\) to \\( s(x) \\) by cutting away the pixels and associated field values at the end
of the domain, until the desired length of \\(x\\) is reached.


![Field zero padder illustration](../../../../media/nifty-posts/non-periodic-signal-inference/zero-padder-illustration.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

So now, in principle, the signal can be defined on an extended domain on which it may fulfill its periodic boundary 
condition, \\(s = s(x_{\mathrm{ext}})\\), but for the signal inference, we may only feed the part of the signal into the
likelihood that actually physically explains the data: \\(X^{\dagger}(s)\\).

Code wise we have: 

```python
x_vanilla = ift.RGSpace(shape=n_pix, distances=x_length/n_pix)
x_vanilla = attach_custom_field_method(x_vanilla)  
# Attach the `field` method

x_fac = 2
x_ext = ift.RGSpace(shape=x_fac*n_pix, distances=x_length/n_pix)
x_ext = attach_custom_field_method(x_ext)

X = ift.FieldZeroPadder(domain=x_vanilla, new_shape=(x_fac*n_pix, ))
s = cosine_model_periodic_bc(signal_domain=x_ext)  
# The signal model guess

# The response is: `R = build_response(num_of_dtps=n_dp, los_ends=los, 
# signal_domain=x_vanilla)`
# And the likelihood becomes: 
# `likelihood_energy = ift.GaussianEnergy(d, N.inverse) @ R(X.adjoint(s))`
```

![Field zero padder reconstruction cosine model](../../../../media/nifty-posts/non-periodic-signal-inference/Reconstruction-cosine-model-with-zero-padding.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

Against expectations, we still find quite a big deviation between the reconstructions extended by a factor of 3 and 
extended by a factors up to of 6.

> Note: The extension factor should be such that the signal field has the ability to decorrelate over the domain.
> For signal models with periodic boundary conditions and rotational or axial symmetry about the midway point/line,
> the signal domain should be extended by at least a factor of 2, whereas for, e.g., the correlated field model, 
> extension factors `1<x_fac<2` might work as well.


This could mean that the chosen model is especially bad behaved. 
It not only enforces periodic boundary conditions, but the signal field values in the domain of interest must be in 
a symmetric relation to those in the extended part of the domain.
Also, the monotonicity of the ground truth is only badly captured by one branch of the periodic cosine. 

Let's do one last example where we get rid of the monotonicity and build a correlated field model. 
The response, a line-of-sight integration, stays the same. 

## Zero-Padding a correlated field model

We construct the ground truth by drawing a random field from a correlated field model domain. 
As above, We pickle that object and reuse it in all inferences.  

We build the correlated field model: 

```python
def build_cf_model(signal_domain, custom_loglogavgslope=None, 
                   custom_fluctuations=None):
    standard_deviation = 1
    loglogslope = -5
    fluct = 1.5

    if custom_loglogavgslope is not None:
        if np.sign(custom_loglogavgslope) > 0:
            raise ValueError("Custom loglogavgslope "
            "needs to be negative")
        if custom_fluctuations is None:
            raise ValueError("Provide custom_fluctuations")
        standard_deviation = 1e-16
        loglogslope = custom_loglogavgslope
        fluct = custom_fluctuations
    args = {
        'offset_mean': 10,
        'offset_std': (5, 1e-16),
        'fluctuations': (fluct, standard_deviation),
        'loglogavgslope': (loglogslope, standard_deviation),
        'asperity': None,
        'flexibility': None,
    }
    return ift.SimpleCorrelatedField(target=signal_domain, **args)
```

and construct the ground truth model and signal model like this: 

```python
x_vanilla = ift.RGSpace(shape=n_pix, distances=x_length/n_pix)
x_vanilla = attach_custom_field_method(x_vanilla)  # Attach the `field` method

x_fac = 3
x_ext = ift.RGSpace(shape=x_fac*n_pix, distances=x_length/n_pix)
x_ext = attach_custom_field_method(x_ext)

X = ift.FieldZeroPadder(domain=x_vanilla, new_shape=(x_fac*n_pix, ))
s = build_cf_model(signal_domain=x_ext)  # The signal model guess

x_ground = ift.RGSpace(shape=2000, distances=x_length/2000)
x_ground = attach_custom_field_method(x_ground)

x_ground_ext = ift.RGSpace(shape=4000, distances=x_length/2000)
x_ground_ext = attach_custom_field_method(x_ground_ext)

X_ground = ift.FieldZeroPadder(domain=x_ground, new_shape=(4000, ))

s_ground = X_ground.adjoint @ build_cf_model(signal_domain=x_ground_ext, 
                                             custom_fluctuations=1,
                                             custom_loglogavgslope=-4)

```

The ground truth, 

![cf model ground truth](../../../../media/nifty-posts/non-periodic-signal-inference/cf-model-ground-truth.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

leads to this data:

![cf model data](../../../../media/nifty-posts/non-periodic-signal-inference/cf-model-data.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

and the reconstructions are:

![cf model data](../../../../media/nifty-posts/non-periodic-signal-inference/cf-model-reconstructions-500-datapoints.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

