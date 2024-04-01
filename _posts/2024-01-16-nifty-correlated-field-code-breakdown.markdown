---
layout: post-custom
title:  "The Nifty-Gritty: Breakdown Of The Correlated Field Code"
date:   2024-03-31 15:21:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}

What actually happens when you do: 

```python
# Arguments of the correlated field model
args_cfm = {
    'offset_mean': 0,
    'offset_std': None,
    'fluctuations': (1.8, 1.8),
    'loglogavgslope': (-2, 1e-16),
    'asperity': None,
    'flexibility': None,
}

s = ift.SimpleCorrelatedField(target=signal_domain, **args)
```
?
Let's break down the code step by step-ish. You probably should have read the [theory on the correlated field model]({% post_url 2024-02-08-ift-the-correlated-field-model %}){:target="_blank"}
first.

We will omit the `if-loops` corresponding to `offset_mean`, `offset_std`, `asperity` and `flexibility`, as well as 
stuff that has to do with domain-target sanity checks.

We will denote `fluctuations`by \\( a \\) and `loglogavgslope` by \\(m\\). 
We will refer to these symbols as `<a>` and `<m>` in the comments of the code we will document.

```python
def SimpleCorrelatedField(
    target,  # The signal domain that field realizations should be 
    # realized for
    offset_mean,
    offset_std,
    fluctuations,
    flexibility,
    asperity,
    loglogavgslope,
    prefix="",
    harmonic_partner=None,
):

    avgsl = NormalTransform(*loglogavgslope, prefix + 'loglogavgslope', 0)
    # Draw the average slope <m> from a normal distribution
    fluct = LognormalTransform(*fluctuations, prefix + 'fluctuations', 0)
    # Draw the fluctuations parameter <a> from a Lognormal; Details: 
    # `LognormalTransform` draws necessary lognormal moments in the background:
    # `LognormalTransform` returns the exponential of a normal distribution 
    # around `logmean`, where `logmean` is `np.log(mean)` 
    # (if standard deviation is 0). I.e., the returned values are ensured to
    # be positive and in the magnitude given by the mean of e.g. `fluctuations`.
    
    pspace = PowerSpace(harmonic_partner)  # Power space defined on the harmonic
    # domain
    twolog = _TwoLogIntegrations(pspace)  # you don't need this if flexibility 
    # is `None`. In the next step, `twolog.domain` is just `pspace`
    expander = ContractionOperator(twolog.domain, 0).adjoint # Same
    ps_expander = ContractionOperator(pspace, 0).adjoint # Define an 
    # expander in the power space, i.e. take a number drawn from a distribution
    # and distribute it over the whole space if applied on that number
    vslope = makeOp(makeField(pspace, _relative_log_k_lengths(pspace)))
    # This constructs a diagonal operator whose entries will be pixel-wise
    # multiplied with the field its applied on. Its defined on the power 
    # space and its values are the logarithmic fourier modes!
    # I.e. <l:=log(k)>.
    slope = vslope @ ps_expander @ avgsl 
    # Here, two things happen: `avgsl` (<m>) is distributed over the whole
    # power space (I.e. in each harmonic bin there is the same value <m>) 
    # and multiplied with `vslope` which are our <l>'s.
    # So this does: <m*l> 
    a = slope  # Now we start to construct the power spectrum of the 
    # AMPLITUDE operator, <p_A>, here denoted by `a`, not to be 
    # confused with out notation for the fluctuations parameter <a>.
```

So, up until know, the code did: \\(p_A(k) = m\cdot l=m\cdot \mathrm{log}(k)\\). Let's move on: 

```python
    a = _Normalization(pspace, 0) @ a  # `_Normalization` constructs 
    # the variance operator, multiplies it on the exponentiated `a` 
    # operator and takes the square root! So this line does
    # <p_A = (U^-1 * e^(m*l))^(1/2) = U^(-1/2) * e^(1/2 * m * l)> Important!!! 
```

Now, we have \\(p_A = \tilde{U}^{-1/2}e^{\frac{1}{2}ml}\\)! An extra factor of \\(1/2 \\) when compared to equation 
\\( (9) \\) of the blog post on the [theory of the correlated field model]({% post_url 2024-02-08-ift-the-correlated-field-model %}){:target="_blank"}.

Here you see the `apply` method of `_Normalization` that is used on \\(p_A=ml\\): 

```python
    def apply(self, x):
        self._check_input(x)
        spec = x.ptw("exp")
        # NOTE, see the note in the doc-string on why this is not a proper
        # normalization!
        # NOTE, this "normalizes" also the zero-mode which is supposed to be
        # left untouched by this operator. Since the multiplicity of the
        # zero-mode is set to 0, the norm does not contain traces of it.
        # However, it wrongly sets the zeroth entry of the result. Luckily,
        # in subsequent calls, the zeroth entry is not used in the CF model.
        return (self._specsum(spec).reciprocal()*spec).sqrt()
```

So, the **effective** average slope of the amplitude power spectrum is \\(\frac{1}{2}m\\)??? 

```python
    maskzm = np.ones(pspace.shape) # The zero mode needs to be handled 
    # seperately => mask it. Create first an array with ones
    maskzm[0] = 0
    # Set the value of the new array the position of the zeroth k-mode
    # to zero
    maskzm = makeOp(makeField(pspace, maskzm))
    # make a new diagonal operator from this new array
    a = (maskzm @ ((ps_expander @ fluct)*a))
    # 1. set first entry of the amplitude power spectrum to zero by 
    # applying the `maskzm` operator. If `offset_std` is not None,
    # a new zero mode will be inserted instead of `0`.
    # 2. Multiply the fluctuations parameter `fluct` (or <a>) onto the
    # amplitude power spectrum after setting the value of 
    # `fluct` to be the same in each position of the Fourier Space
    # (`ps_expander @ fluct`). So this line does:
    # <p_A = a * U^(-1/2) * e^(1/2 * m * l)>, k != 0.
    a = a.scale(target.total_volume)
    # Multiply the power spectrum by the total volume of the 
    # signal domain. I think because Volume in Fourier Space should be 
    # = to volume in real space? 
```

So after this step, we have

$$
p_A (k) = a \cdot V_{\Omega_x} \cdot \tilde{U}^{-1/2}e^{\frac{1}{2}m\cdot l}, \hspace{5mm} k\neq 0.
$$

Finally we get signal realizations through: 

```python
    ht = HarmonicTransformOperator(harmonic_partner, target) 
    # Fourier transform 
    pd = PowerDistributor(harmonic_partner, pspace)
    # This takes a power spectrum and returns an operator in the 
    # harmonic domain, so when applied on `a`, it essentially does:
    # pd(a) ~ < (2π)^u δ(k-k') p_A (k) =: A>
    xi = ducktape(harmonic_partner, None, prefix + 'xi')
    # This `xi` thing I don't really get
    op = ht(pd(a)*xi)
    # This finally creates signal realizations by doing s = F[Aξ_s], where 
    # F[.] denotes the Fourier transform, which I presume needs to
    # be applied because the ξ_s are harmonic?
    op.amplitude = a
    op.power_spectrum = a**2
    # Attribues can be attached to any function in python
    # via dot notation. `op.amplitude = a` sets the amplitude power 
    # spectrum of the returned operator to the constructed `a`,
    # `op.power_spectrum = a**2` means the signal power spectrum.

```

Boom! Signal realizations

$$
\begin{aligned}
s &= \mathcal{F} (\color{blue}A \color{black}\xi_{s,\text{harmonic}} ) \\
&= \mathcal{F} (\color{blue}2\pi \delta(k-k^{\prime}) p_A(k) \color{black}\xi_{s,\text{harmonic}} )
\end{aligned}
$$

where \\( p_A(k) \\) is 

$$
p_A(k) = a \cdot V_{\Omega_x} \cdot \tilde{U}^{-1/2}e^{\frac{1}{2}m\cdot l}, \hspace{5mm} k\neq 0.
$$

This is probably the reason why reconstructions depend on the length of the signal domain, the effective 
fluctuations parameter is \\(a\cdot V_{\Omega_x} \\) where \\( V_{\Omega_x} \\) is the volume of the signal 
domain.

The signal power spectrum constructed is then

$$
\begin{aligned}
p_T(k)=p_A(k)^2 &= a^2 V_{\Omega_x}^2 \cdot \tilde{U}^{-1}(e^{\frac{1}{2}m\cdot l})^2, \hspace{5mm} k\neq 0 \\
&= a^2 V_{\Omega_x}^2 \cdot \tilde{U}^{-1}e^{ml} \\
&= a^2 V_{\Omega_x}^2 \cdot \tilde{U}^{-1}k^{m}
\end{aligned}
$$

From which I conclude that the value for \\( m \\) given by `loglogavgslope` is actually the expected signal 
power spectrum slope in log-log space!

I still assume that \\(\tilde{U} \\) is such, that the expected signal variance 

$$
\langle \sigma^2_s\rangle = \int_{k\neq 0}p_T(k) \hspace{1mm} \mathrm{d}k
$$

still is soley given by \\( a^2V_{\Omega_x}^2 \\). This can only be the case if 

$$
\tilde{U} = \int_{k\neq 0} e^{\gamma (k)} \hspace{1mm} \mathrm{d}k,
$$
 

<span style="color:red"> not </span>

$$
\tilde{U} = \int_{k\neq 0} e^{2\gamma (k)} \hspace{1mm} \mathrm{d}k,
$$

as stated in the Nature m87 paper.
