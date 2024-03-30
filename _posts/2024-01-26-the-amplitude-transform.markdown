---
layout: post-custom
title:  "IFT: The Amplitude Transform and Standardized Coordinates"
date:   2024-01-26 11:06:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}


In IFT papers and the NIFTy project page, you often might see the equations 

$$
s= A\xi \hspace{10mm} S=AA^{\dagger} \tag{1}
$$

These are transformations that come in extremely handy when building multi-modal prior
distributions. A "Gaussian Mixture" of random variables come together to form a model
of the situation at hand, with the special property that their joint likelihoods are Gaussian.
The inference mechanism that uses such a prior implements a "Gaussian Process Regression" and
in NIFTy, the implementation of the Gaussian Process is called the "correlated field model." 

In this document, I will try to shed some light onto these transforms and onto variational
inference in general. Information from [this paper ("Encoding prior knowledge in the structure of the likelihood").](https://arxiv.org/pdf/1812.04403.pdf){:target="_blank"}

This post may be a good basis for understanding the
[main correlated field model paper](https://arxiv.org/abs/2002.05218){:target="_blank"}.

# Variational Inference

At this point, a brief digression into Variational Inference (VI) is useful. 
See [this post]({% post_url 2024-02-21-variational-inference %}){:target="_blank"}.

# Reparameterizing the Gaussian

Say you have a family of Gaussians (_Gaussian Process_) as Priors. They take the following form: 

$$
\mathcal{P}(s) = \mathcal{G}(s,S) = \frac{1}{\mid 2\pi S\mid^{-1/2}}e^{-\frac{1}{2}s^{\dagger}S^{-1}s}
\tag{3}
$$

Instead of inferring \\(s \hookleftarrow \mathcal{G}(s,S)\\) with some potentially complex correlation
structure \\(S \\), it'd be much nicer to infer \\(\xi \hookleftarrow \mathcal{G}(\xi, \mathbf{1} )\\).

This can be done as follows. First, do an eigen-decomposition of \\(S\\), denote the diagonal matrix with
\\(\hat{S}\\).

$$
S = F^{\dagger} \hat{S}F.
$$

Then, for the inverse correlation, the eigenvalues simply have to be inverted,

$$
S^{-1}= F^{\dagger}\hat{S}^{-1}F. 
$$

\\( \hat{S}^{-1} \\) is still diagonal. Input this into the exponent of equation \\( (3) \\): 

$$
\frac{1}{2} s^{\dagger}S^{-1}s = \frac{1}{2} s^{\dagger} \bigg(  F^{\dagger}\hat{S}^{-1}F \bigg) s
$$

The first step of the transform is to define \\( \tilde{s}=Fs \\). Since the transform is linear,
we do not have to track the change in differential volume since there is none. 

$$
\frac{1}{2} s^{\dagger} \bigg(  F^{\dagger}\hat{S}^{-1}F \bigg) s = 
\frac{1}{2} \tilde{s}^{\dagger}\hat{S}^{-1}\tilde{s}. \tag{4}
$$

Diagonal matrices have a square-root decomposition, i.e. 

$$
\hat{S}^{-1} = \bigg( \hat{S}^{-1/2} \bigg)^{\dagger} \bigg( \hat{S}^{-1/2} \bigg).
$$

Inserting into equation \\( (4) \\): 

$$
\frac{1}{2} \tilde{s}^{\dagger}\hat{S}^{-1}\tilde{s} = 
\frac{1}{2} \tilde{s}^{\dagger} \bigg( \hat{S}^{-1/2} \bigg)^{\dagger} \bigg( \hat{S}^{-1/2} \bigg) \tilde{s}
$$

and we perform our final transform \\(\xi = \hat{S}^{-1/2}\tilde{s} \\) to arrive at 

<div style="background-color: yellow ">
\[
\frac{1}{2}s^{\dagger}S^{-1}s = \frac{1}{2}\xi^{\dagger}\mathbf{1}\xi  \tag{5}
\]
</div>

and the full transform reads 

$$
\xi = \hat{S}^{-1/2}Fs \hspace{5mm} \text{or} \hspace{5mm} s= F^{-1}\hat{S}^{1/2}\xi
$$


where we define the Amplitude Operator \\(A \\) via

<div style="background-color: yellow ">
\[
s= F^{-1}\hat{S}^{1/2}\xi =: A\xi \tag{6}
\]
</div>

and it is obvious that the relationship between \\( A \\) and the original correlation is 


<div style="background-color: yellow ">
\[
S = AA^{\dagger} \tag{7}.
\]
</div>

With that, we can do the inference in the standardized \\( \xi \hookleftarrow \mathcal{G}(\xi, \mathbf{1})\\)
space and backtransform to \\( s \hookleftarrow \mathcal{G}(s,S) \\) when needed.

# Variationally Inferring Correlation Structure

Say you don't know the Correlation Structure \\( S \\) _exactly_. Can the data inform you about 
\\(S\\) as well as the signal? Yes! It very much can, this is a crucial use case of Gaussian
Processes in NIFTy. 

Say your signal has a stationary correlation, then the Wiener-Khinchin theorem can be applied that tells you 
that \\(S\\) has an Eigendecomposition in Fourier-Space according to 


$$
S = F^{\dagger} (\widehat{\mathbb{P}p}) F    \tag{8}
$$

The hat indicates diagonality. The functional eigenvalues are specified by the power spectrum \\(p \\) and the diagonal operator that
contains those eigenvalues is "built" using an "isotropy operator \\( \mathbb{P} \\), that distributes a one
dimensional power spectrum into the diagonal [...]".

<div style="border-left: 5px solid black; padding-left: 25px;margin-top:25px; margin-left:25px; margin-bottom:25px">

<b>Note </b>:
If there is just one eigenvalue \( \lambda \), you can construct the diagonal eigenvalue matrix simply by taking the unit matrix
and placing that eigenvalue everywhere along the diagonal: \( \mathbf{1} \lambda\). But in this case, we have infinitely many
eigenvalues, specified by the power spectrum, and each of those eigenvalues has its own unique place on an infinite diagonal.

</div>

The transformation operator \\( F \\) is the Fourier-Transform. Now, let \\(p \\) be a random variable.
Since we pretty much know only that the power spectrum should be strictly positive, the lognormal prior 
is a suitable choice:

$$
p \hookleftarrow \mathcal{LN}(p, T).
$$

The covariance structure \\( T \\) determines the smoothness of the power spectrum (variation of value from mean at one 
position and correlation between values at different position). In this context it is also called the _smoothing kernel_.

Positivity is a tough (numerical) constraint. Instead, we can reparameterize to a logarithmic power spectrum and say 
that the new random variable \\( \tau \\) is normal distributed:

$$
\tau := \mathrm{log}(p) \hspace{5mm} \tau \hookleftarrow \mathcal{G}(\tau, T).
$$

This way, we enforce positivity only indirectly. When the posterior mean of \\( \tau \\) is found the backtransform
ensures that \\( p = \mathrm{exp}(\tau) \\) is positive.

We can make \\( p \\) part of the inference process by not only inputting the joint likelihood

$$
\mathcal{H}(d ,s )
$$

into the to-be-minimized KL-divergence, but by inputting the likelihood 

$$
\mathcal{H}(d,s,\tau).
$$

Now, we can standardize \\( s \\) as discussed: 

$$
s = A\xi = F^{-1}\hat{S}^{1/2}\xi = F\hat{S}^{1/2}\xi = F (\widehat{\mathbb{P}p})^{1/2}\xi = 
F (\widehat{\mathbb{P}e^{\tau}})^{1/2}\xi = F \mathbb{P}e^{\frac{1}{2}\tau}\xi
$$

since the Fourier Transform is in general unitary.

We can apply this very same procedure to transform \\(\tau \\) into standardized coordinates, let's
say called \\( \zeta \\). For this, we need the eigendecomposition of the smoothness Kernel \\(T\\). 
Since we expect a lack of curvature in the logarithmic coordinate system (because exponential functions 
\\( p = k^{-\alpha}\\), \\(\alpha >0 \\) are linear there), we can show that the Kernel has to be given by 

$$
T^{-1} = \sigma^{-2} \Delta^{\dagger}\Delta,
$$

where the biharmonic operator \\(\nabla^4 \\) is a measure of total curvature over the whole domain of the operator.
See [this post ("IFT: How To Find The Signal Covariance")]({% post_url 2024-01-20-how-to-find-the-signal-covariance-operator %}){:target="_blank"}
for a derivation of the same concept in a different problem setting.
The expected deviation from a smooth power spectrum is given by \\(\sigma\\).

We reserve the character \\(k\\) for the harmonic coordinate of the signal space.
We denote the harmonic coordinate of the log power space as \\(l\\).
It is straightforward to show that the Laplacian is diagonal in its harmonic space as follows: 

$$
\Delta = V^{\dagger}l^2V
$$

and the standardized coordinates \\(\zeta \\) are given by: 

$$
\tau = B\zeta = V^{-1}\hat{T}^{1/2}\zeta = V \bigg(
\sigma^{-2}
\widehat{
\Delta^{\dagger}\Delta
}\bigg)^{-1/2}\zeta 
=
V\sigma \hat{\Delta}^{-1} \zeta = V \sigma V^{\dagger}l^{-2}V\zeta = \frac{\sigma}{l^2}V\zeta
$$

where again, \\(V\\) is a unitary transform, and as per my understanding the Fourier transform. Although I am 
not sure why in the paper I am describing here [("Encoding prior knowledge in the structure of the likelihood")](https://arxiv.org/pdf/1812.04403.pdf){:target="_blank"}
a new variable is assigned to it.

With this, the posterior that the approximating distributions need to convergence towards is
entirely dependent on random variables drawn from unit gaussians and the observed data. 
The complexity lies in the sequence of linear and non-linear transformations onto those variables 
and not in their correlation structures. The full information Hamiltonian is: 

$$
\begin{aligned}
\mathcal{H}(\xi, \tau \mid d) \hspace{1mm} \widehat{=} \hspace{1mm}  \mathcal{H}(\xi, \tau, d)
\hspace{1mm} &\widehat{=} \hspace{1mm}
\frac{1}{2}\bigg( d- RF(\widehat{Pe^{\frac{1}{2}V\frac{\sigma}{l^2}\zeta}})\xi \bigg)^{\dagger} N^{-1}
\bigg( d- RF(\widehat{Pe^{\frac{1}{2}V\frac{\sigma}{l^2}\zeta}})\xi \bigg) \\
& \hspace{1mm} + \frac{1}{2}\xi^{\dagger}\mathbf{1}\xi + \frac{1}{2}\zeta^{\dagger}\mathbf{1}\zeta \tag{9}
\end{aligned}
$$

In Gaussian Variational Inference, the approximating distribution is 

$$
\mathcal{P}(s,\tau \mid \text{ best parameters }) = \mathcal{P}(s,\tau \mid m, D, \tau^{\ast})
= \mathcal{G}(s-m,D)\delta (\tau - \tau^{\ast}) \tag{10}
$$

and our task is to find \\(m, D\\) and \\(\tau^{\ast} \\) such that the KL divergence between the 
approximant and the true posterior, equation \\( (9) \\), is minimal. 
But how can we minimize the KL divergence through stochastic sampling?

<div style="border-left: 5px solid black; padding-left: 25px;margin-top:25px; margin-left:25px; margin-bottom:25px">

<b>Note </b>:
Equation \( (10) \) is a good approximation in case the prior is Gaussian and the data only slightly updates the posterior.
In case the Gaussian prior was not a good guess, i.e. the data is highly informative leading to a complex 
prior, the algorithms of Metric Gaussian Variational Inference <b>MGVI</b> and Geometric Variational Inference <b>geoVI</b> provide 
better approximants.
</div>

To see how stochastic sampling is used to minimize the KL divergence, I refer to the post explaining the 
`kl_optimize()` function and more specifically the description of the `sampledKLEnergy()` therein:
[The Nifty-Gritty: The `optimize_kl()` function.]({% post_url 2024-01-19-optimize-kl-function%}){:target="_blank"}