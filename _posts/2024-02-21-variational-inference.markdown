---
layout: post-custom
title:  "IFT: Variational Inference"
date:   2024-02-21 11:06:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}
Information from [this paper ("Encoding prior knowledge in the structure of the likelihood").](https://arxiv.org/pdf/1812.04403.pdf){:target="_blank"}

Only for a small subset of problems, an analytical posterior solution can be found. 
In many cases, the evidence \\(\mathcal{P}(d)\\) is the source of the unsolvability 
of a problem.

In VI, the idea is to find a family of approximations \\(\tilde{\mathcal{P}}(s\mid \varphi)\\) 
that are dependent on some parameters \\(\varphi \\) and let that family converge to the 
optimal approximation via minimizing the KL-divergence w.r.t to the 
approximation parameters \\(\varphi \\).

$$
\tilde{\mathcal{P}}(s \mid \varphi )_{\text{optimal}} = 
\mathrm{optimize} \hspace{1mm}
\mathcal{D}_{\mathrm{KL}}\bigg( \tilde{\mathcal{P}}(s\mid \varphi) \mid \mid \mathcal{P}(s\mid d)\bigg)
$$

Say you have made some initial guess for the first approximating distribution.
To find the KL-divergence, the true posterior is needed. But when minimizing the KL-divergence,
constants (e.g., the problematic renormalization constant \\(\mathcal{P}(d)\\) ) are
irrelevant. So the _relevant_ parts of posterior are 

$$
\mathcal{H}(s\mid d) \hspace{1mm} \hat{=} \hspace{1mm} \mathcal{H}(d,s)=\mathcal{H}(d \mid s) + \mathcal{H}(s) \tag{2}
$$

and the KL-divergence is

$$
\begin{aligned}
\mathcal{D}_{\mathrm{KL}}\bigg( \tilde{\mathcal{P}}(s\mid \varphi) \mid \mid \mathcal{P}(s\mid d)\bigg) 
&= \int  Ds \hspace{1mm} \tilde{\mathcal{P}}(s\mid \varphi) \mathrm{log}\bigg(\frac{\mathcal{P}(s\mid d)}{\tilde{\mathcal{P}}(s\mid \varphi)}\bigg) \\
&= \int  Ds \hspace{1mm} \tilde{\mathcal{P}}(s\mid \varphi) \bigg( \mathrm{log}(\mathcal{P}(s\mid d))-\mathrm{log}(\tilde{\mathcal{P}}(s\mid \varphi ))\bigg) \\
&= \int  Ds \hspace{1mm} \tilde{\mathcal{P}}(s\mid \varphi) \bigg( -\mathcal{H}(s\mid d)+\tilde{\mathcal{H}}(s\mid \varphi )\bigg) \\
&= \langle \tilde{\mathcal{H}}(s\mid \varphi) \rangle - \langle \mathcal{H}(s \mid d)\rangle
\end{aligned}
$$

where I believe in the paper cited above, equation \\(7\\) a sign error was made. Moving on: The expectation is a linear
operator! I.e. 

$$
\begin{aligned}
\mathcal{D}_{\mathrm{KL}}\bigg( \tilde{\mathcal{P}}(s\mid \varphi) \mid \mid \mathcal{P}(s\mid d)\bigg)
&= \langle \tilde{\mathcal{H}}(s\mid \varphi) \rangle - \langle  \mathcal{H}(d \mid s) + \mathcal{H}(s) - \mathcal{H}(d) \rangle \\ 
&= \langle \tilde{\mathcal{H}}(s\mid \varphi) \rangle - \langle  \mathcal{H}(d \mid s) + \mathcal{H}(s) \rangle - \mathcal{H}(d) \\
&= \langle \tilde{\mathcal{H}}(s\mid \varphi) \rangle - \langle  \mathcal{H}(d,s) \rangle - \mathcal{H}(d)
\end{aligned}
$$

Finally, when optimizing the KL-Leiber, the term \\(H(d)\\) cancels: 

$$
\mathrm{optimize} \hspace{1mm} \mathcal{D}_{\mathrm{KL}}=\mathrm{optimize} \hspace{1mm} 
\bigg( \langle \tilde{\mathcal{H}}(s\mid \varphi) \rangle - \langle  \mathcal{H}(d,s) \rangle \bigg)
\color{red} - \cancel{\mathcal{H}(d)}
$$

The KL-divergence does not even have to be calculated explicitly, drawing samples from it is sufficient
for the optimization
([see here](https://projecteuclid.org/journals/bayesian-analysis/volume-8/issue-4/Fixed-Form-Variational-Posterior-Approximation-through-Stochastic-Linear-Regression/10.1214/13-BA858.full){:target="_blank"}).

To perform the minimization, a target functional needs to be optimized numerically. This might be a
non-convex problem, and we are not guaranteed to find a global minimum. For a Newton or quasi-Newton 
algorithm, the current position and derivatives of that position in the parameters space need to be
known. Therefore, the convergence properties of the minimization as well as the numerical conditioning number
rely on the underlying coordinate space.  

It will almost always be numerically more powerful to do the inference in a coordinate system, where 
the posterior given by Bayes-Theorem (minus the renormalization constant) and the approximating distribution
are e.g., Gaussians. 

To perform coordinate transforms, the differential volume change has to be taken into account in form
of the Jacobi determinant. 

**Coordinate systems matter for variational inference**.

[See here]({% post_url 2024-01-26-the-amplitude-transform %}){:target="_blank"}
for a post on how the conditioning number of inference algorithms is improved by "Standardizing" random variables.