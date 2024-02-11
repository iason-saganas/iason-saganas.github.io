---
layout: post-custom
title:  "IFT: Introduction To The Correlated Field Model"
date:   2024-01-16 21:10:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}


We assume the knowledge in the blog post ["The Amplitude Transform and Standardized Coordinates"]({% post_url 2024-01-26-the-amplitude-transform %}){:target="_blank"}. 

First, we will describe the correlated field model (_CFM_) in a more general context
([arxiv.org/abs/2002.05218](https://arxiv.org/abs/2002.05218){:target="_blank"}),
and then in a more practical fashion
([arxiv.org/abs/2008.11435](https://arxiv.org/abs/2008.11435){:target="_blank"} and NIFTy code).

## Starting Point 

Since the transformation of a random variable into a different coordinate system does not change 
the contained information of the system, but variational inference algorithms like `MGVI` and `geoVI` are 
dependent on the choice of coordinate-system, it is fruitful to infer so-called **latent parameters** \\(\xi_s \\)
instead of \\(s \hookleftarrow \mathcal{P}(s,S)\\). 

That is because the latent parameters are standard-normally
distributed degrees of freedom (DOFs). Concretely, there is a transform \\(A\\) with

$$
S= AA^{\dagger}
$$

and 

$$
s = A\xi_s,
$$

such that 

$$
\xi_s \hookleftarrow \mathcal{G}(\xi_s ,\mathbf{1}).
$$

\\(A \\) is called the **amplitude operator**. We can apply this same standardization process to 
the power spectrum and infer it along the data just by updating the likelihood to contain the power
spectrum prior. 

In case of the problem in the blog post ["The Amplitude Transform and Standardized Coordinates"]({% post_url 2024-01-26-the-amplitude-transform %}){:target="_blank"}
the \\( A \\) transform for the power spectrum was found explicitly, because the correlation Kernel \\( T\\)
could be specified by the assumption of smoothness. But what if we don't even know whether the power spectrum is smooth?
Then, we need to build a model for \\( A \\) to regularize the problem.

These model-parameters, on which the latent parameters \\(\xi_s \\) depend, are called _hyperparameters_. 
This is the starting point of the paper in which the first description of the CFM is given. 

In each variation inference step, instances of the hyperparameters are created, which create instances of the 
latent parameters, which at the end of the inference are transformed back to signal samples.
Because of this chain of variable generations, this approach is also called a _generative model_ approach. 
It can be visualized by a _computational graph._ 

To quote from the paper: 

> We assume power spectra to be close to power laws with deviations modelled as an integrated Wiener Process on a double logarithmic scale

> The main idea is to model the correlations in space, time and frequency independently using the same underlying model
> and combine them via outer products. Doing this naively results in degenerate and highly un-intuitive model parameters.
> The model we introduce in the following avoids these issues, but [...] requires a certain complexity.

> Our next task is to develop a flexible model for this spectrum that expresses our uncertainty and is compatible
> with a wide range of possible systems. 

## Correlations in space, time and frequency 

Assume you have a strictly positive signal and you enforce positivity by introducing a to-be-inferred meta-variable \\(\tau \\) via 

$$
s=e^{\tau}
$$

$$
\tau \hookleftarrow \mathcal{G}(\tau, T).
$$

(not necessary for the CFM, but starting situation in the paper where it was first described!).

Again, instead of inferring \\( \tau \\) with some potentially complex correlation \\( T\\), we standardize to 
\\( \xi_s \hookleftarrow \mathcal{G}(\xi_s, \mathbf{1}) \\) via 

$$
\tau = A \xi_s, \hspace{5mm} AA^{\dagger}=T.
$$

Because of the relationship \\( AA^{\dagger} = T \\), the amplitude power spectrum and log signal power spectrum 
are related by: 

$$
p_A(k) = \sqrt{p_T(k)}.
$$

But what if we don't have enough information to specify \\( T \\) and therefore \\( A\\)? We build a model for \\( A \\)
directly. We do this by assuming a statistical homogeneity a priori and applying the Wiener-Khinchin-Theorem to get 

$$
A^{kk^{\prime}} = (2\pi)^d \delta(k-k^{\prime})p_A(k),
$$

so \\( A \\) is modeled as a diagonal operator in Fourier-Space. 
**IF** the power spectrum was of the form of something like 

$$
p_A(k) = \alpha k^{- \beta}, \tag{1}
$$

\\(\alpha, \beta > 0 \\), **then** the resulting power spectrum would look linear in a log-log space, i.e., be smooth! 
But here's the catch: We don't know whether \\( p_A(k) \\) is smooth. 

While \\( A \\) is modeled via a diagonal operator
in Fourier-Space, we don't assume a functional basis like in equation \\( (1) \\). 
Instead, potential "roughness" or "zig-zagness" is introduced by modeling the diagonal itself as part of a 
random process. 


### A Model For The Amplitude Power Spectrum

Since the amplitude power spectrum has to be strictly positive definite, let 

<div style="background-color: yellow ">
\[
p_A(k) \propto e^{\gamma (k)}.
\]
</div>

Goals: 

- Model \\( \gamma (k) \\)
- Find proportionality constant

The reason that here a proportionally is used instead of an equality, is that \\(\gamma(k)  \\) will be 
modeled as an _integrated Wiener-Process_ and the proportionality constant will implicitly fix its offset. 
So step by step: 

### What Is An Integrated Wiener Process And Why Use It Here?

To the latter question, "why use it here", my answer is: Why not? 
<span style="color:green"> But seriously, if you, the reader, has an idea as to why Philipp et al. opted to use this process,
shoot me a message! 
</span>

What I can say is that an IWP is a general continuous stochastic process. The defining equation that describes
the evolution of the random variable under consideration in terms of some parameter (we often like to think 
of as time but in this case is \\( l := \mathrm{log}(k) \\)) is

$$
\frac{\mathrm{d}^2\gamma (k)}{\mathrm{d}l^2} = \eta(l), \tag{2}
$$

where \\( \eta(l) \\) is Gaussian standard distributed (taken from [this Arras 2021 paper](https://arxiv.org/pdf/2008.11435.pdf){:target="_blank"}). So \\( \gamma (k )\\) is especially two time differentiable.

<div style="border-left: 5px solid indianred; padding-left: 25px; margin:25px 0 25px 25px">

<b>Warning </b>:
Because we set \( l = \mathrm{log}(k)\) all following considerations do not apply to \(k=0\), which 
has to be handled separately. 
Effectively, the zero-modes will be fixed by renormalization of the
amplitude operators.

</div>

<div style="border-left: 5px solid black; padding-left: 25px; margin:25px 0 25px 25px">

<b>Note </b>:
Why parameterize \( l = \mathrm{log}(k)\) in the first case? Sure, smooth functions look linear in a log-log
space, which is a nice property. <span style="color:green"> Do you know of any other reason for the 
\(\mathrm{log}(k)\) parameterization?</span> 

</div>

Equation \\( (2) \\) is basically saying: Let the variable of interest \\( \gamma(k) \\) be such that its
"acceleration" is given by a normal standard variable \\( \eta(l) \\). The _Wiener Process_ would be given by:

$$
\frac{\mathrm{d} \gamma}{\mathrm{d} l} = \eta (l),
$$

meaning the "velocity" is given by a random variable in each point "in time". 
The following interactive figure executes a script that sketches the evolution of the probability
of \\( \gamma(k) \\).
It starts at zero. Then for each point in time, it draws
from a standard normal distribution and simply solves the second order differential equation, 
which by then has become trivial since the right-hand side is no longer time dependent.

{% include ift/integrated-wiener-process-chart-Js.html %}



Another way of writing \\( (2) \\) is by splitting the second order differential equation into two first order
differential equations by substitution. 

$$
v = \frac{\mathrm{d}\gamma(k)}{\mathrm{d}l} \tag{3}
$$

$$
\frac{\mathrm{d}v}{\mathrm{d}l} = \eta(l) \tag{4}
$$

Integrating both of these equations on both sides with respect to \\( l \\) yields: 

$$
\int_{l_0}^l v(l^{\prime}) \hspace{1mm}\mathrm{d}l^{\prime} = \gamma (k) - \gamma(0)  \stackrel{!}{=} \gamma (k) \tag{5}
$$

$$
v(l^{\prime})-v(l_0) =\int_{l_0}^{l^{\prime}} \eta(l^{\prime \prime}) \hspace{1mm}\mathrm{d}l^{\prime \prime} \tag{6}
$$

And inserting equation \\( (6) \\) into equation \\( (5) \\) and renaming \\( v(l_0)=:m \\):

$$
\begin{aligned}
\gamma (l) &= \int_{l_0}^l \int_{l_0}^{l^{\prime}} \eta (l^{\prime \prime}) \hspace{1mm} \mathrm{d}l^{\prime \prime} +m \hspace{1mm} \mathrm{d}l^{\prime} \\

 &= \int_{l_0}^l \int_{l_0}^{l^{\prime}} \eta (l^{\prime \prime}) \hspace{1mm} \mathrm{d}l^{\prime \prime} \hspace{1mm} \mathrm{d}l^{\prime}+m(l-l_0) \\

&\stackrel{\color{red} l_0=0? \color{black}}{=}
\int_{l_0}^l \int_{l_0}^{l^{\prime}} \eta (l^{\prime \prime}) \hspace{1mm} \mathrm{d}l^{\prime \prime} \hspace{1mm} \mathrm{d}l^{\prime}+ml \tag{7}

 \end{aligned}
$$

Now, it is _my understanding_, that to arrive at equation \\( (20) \\) of the [Arras Nature paper](https://arxiv.org/abs/2002.05218){:target="_blank"}
you would assume that \\( \eta (l^{\prime \prime}) \\) is generated by a set of random excitations \\( \xi_W \\) and are of amplitude \\( \eta \\), i.e.

$$
\eta (l^{\prime \prime}) = \eta \xi_W(l^{\prime \prime}), \hspace{5mm} \xi_W \hookleftarrow \mathcal{G}(\xi_W, \mathbf{1}).
$$

In that case, inserting into equation \\( (7) \\) 

<div style="background-color: yellow ">
\[
\gamma (l) = \eta \int_{l_0}^l \int_{l_0}^{l^{\prime}} \xi_W (l^{\prime \prime}) \hspace{1mm} \mathrm{d}l^{\prime \prime} \hspace{1mm} \mathrm{d}l^{\prime}+ml \tag{8}
\]
</div>

gives us the integrated Wiener Process.

<div style="border-left: 5px solid black; padding-left: 25px; margin:25px 0 25px 25px">

<b>Note </b>:
Please note that I am uncertain about the accuracy of the following paragraphs. 
<span style="color:green"> Self-Note: Maybe Mrinal can help here?
</span>
</div>


Now, define a new operator \\( \tilde{U} \\), as the integral of the power spectrum \\( p_T \\) over all Fourier-Modes,
except the zero mode. We might call \\( \tilde{U} \\) the Total Variance Operator.

$$
\tilde{U} = \int_{k\neq 0} p_T(k) \hspace{1mm} \mathrm{d}k
$$

_In my understanding_, the significance of this operator is as follows: The smoother the power spectrum, the steeper
it drops in the log-log space, so the sooner it arrives at its last \\(k\\)-mode. Or, in other words, the smaller the 
area under the power spectrum. That means, a bigger \\( \tilde{U} \\), the area under the power spectrum curve, corresponds
to a "rougher" or less steep power spectrum, in each case more total variation around the signal mean is expected.

\\( \tilde{U} \\) is therefore a measure of total expected variation and an important quantity, and up to now fixed 
by the integrated Wiener Process. 
The IWP controls the slope of the power-law like behavior, as well as the strength 
of the integrated part of the process. 
The total variation \\( \tilde{U} \\) is 
given by the summed contributions of the IWP: 

$$
\tilde{U} = \int_{k \neq 0}e^{2\gamma(k)} \hspace{1mm} \mathrm{d}k
$$

Now, _as I see it_, the authors of the [2020 Arras Nature paper](https://arxiv.org/abs/2002.05218){:target="_blank"} separate the roles and functionalities
of the different hyperparameters as follows: 
The IWP takes on the responsibility of fixing:
- the power spectrum slope 
- the non-linear deviation from it, 
- **as well as** the total expected variation through its volume. 

Now, divide out the volume contribution of the IWP and introduce new parameters that act as **direct dials** to control
the total expected variation. Then, the **IWP only handles how the power spectrum changes from one point to the next**.

To do this, divide out the volume \\( \sqrt{\tilde{U}} \\) (which in the paper is explicitly called the Variance Amplitude
Operator). This suggests that 

$$
\sqrt{\tilde{U}}=\sqrt{\int_{k \neq 0}e^{2\gamma(k)} \hspace{1mm} \mathrm{d}k} = \int_{k \neq 0}e^{\gamma(k)} \hspace{1mm} \mathrm{d}k
$$

<span style="color:green">(1. right? 2. Why not write it a priori as int e^gamma, why take the square root of U?). </span>

Introducing a new hyperparameter \\(a \\) we can finally fix the model for the power spectrum of \\( A \\) as

<div style="background-color: yellow ">
\[
p_A(k)=a\tilde{U}^{-1/2}e^{\gamma}(k). \hspace{5mm} \forall k\neq 0 \tag{9}
\]
</div>



### Hyperparameterizing The IWP And The Power Spectrum 

<br/>
<br/>
![Sketch of Integrated Wiener Process](../../../../media/ift-posts/hyperparameterizing-iwp.png)
<br/>
<br/>

### Taking The Outer Product Of All Sub-Spaces: Fixing The Zero Modes

A final undertaking. Say you have a correlation in time, frequency and space, then it is natural that the final correlation
structure is given by 

$$
A = \alpha \bigotimes_{i\in \{x, t, \nu\}} \tilde{A}^{(i)},
$$

where \\( \bigotimes \\) denotes the _outer product_ (see [Wikipedia-Article](https://en.wikipedia.org/wiki/Outer_product){:target="_blank"}).
\\( \tilde{A}^{(i)} \\) are the normalized Amplitude Οperators with our model built-in in each sub-space. 
During the normalization process, the zero-mode of the power spectrum \\( p_A(0)\\) can be fixed as: 

$$
p_A(0)= \int_{\Omega^{(i)}}\bigg( F^{(i)} \bigg)^{-1}p_A^{(i)} \hspace{1mm} \mathrm{d}\Omega^{(i)},
$$

which is an integral over real-space \\( \Omega^{(i)} \in \{ \Omega^x, \Omega^{\nu}, \Omega^t \} \\) and \\( F \\) is 
the Fourier-Transform.

The hyperparameter \\( \alpha \\) arises from a degeneracy of taking the outer product and sets the scale globally in all
sub-domains. It is the last to-be-inferred parameter and follows statistics 

$$
\alpha = e^{\mu_{\alpha}+\sigma_{\alpha}\xi_{\alpha}}.
$$

## Identifying The Model Parameters With `NIFTy8` Keywords

<br/>
<br/>
![Identification of Integrated Wiener Process, Power Spectrum and Overall Correlation Structure Parameters With Nifty8 Keywords](../../../../media/ift-posts/identifying-with-nifty8-kw.png)
<br/>
<br/>

## What `NIFTy` Spits Out At The End Of An Inference

## Materials

- The theory behind the CFM: "Variable structures in M87* from space, time and frequency resolved interferometry", Arras et al., 2022, [arXive](https://arxiv.org/pdf/2002.05218.pdf){:target="_blank"}
- The more concrete numerical implementation which I cannot link to the theory yet: "Comparison of classical and Bayesian imaging in radio interferometry - Cygnus A with CLEAN and resolve", Arras et al., 2021, [arXive](https://arxiv.org/pdf/2008.11435.pdf){:target="_blank"}
- Showcase of the CFM parameters: [Link to NIFTy project page](https://ift.pages.mpcdf.de/nifty/user/getting_started_4_CorrelatedFields.html){:target="_blank"}
