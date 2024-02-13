---
layout: post-custom
title:  "The Expectation Value Of The Variance Of A Signal Field"
date:   2024-02-12 17:47:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}

<div style="border-left: 5px solid black; padding-left: 25px; margin:25px 0 25px 25px">
<b>Note </b>:
Thanks @Mrinal Jetti for helping with the calculations.
</div>

This document connects back to the post ["An Introduction To The Correlated Field Model"]({% post_url 2024-02-08-ift-the-correlated-field-model %}){:target="_blank"}.
Here we summarize calculations that give some insight into the definition of the Signal Variance Operator 

$$
\tilde{U}=\int_{k \neq 0} p_T(k)\hspace{1mm}\mathrm{d}k,
$$

where \\(p_T \\) is the signal power spectrum, as defined by [Arras et al. in Nature](https://arxiv.org/abs/2002.05218){:target="_blank"}.

## The Signal Variance

The signal variance is given by 

$$
\mathrm{Var}(s) := \sigma_s^2:= \langle (s-\bar{s})^2\rangle,
$$

where we define 

$$
\bar{s} = \langle s \rangle .
$$

Now, assume <span style="color:red">(I am not sure how the volume comes into the play and where the PDF is that has to be integrated over 
in the definition of the variance) </span>:

$$
\sigma_s^2 = \frac{1}{V} \int (s-\bar{s})^2 \hspace{1mm} \mathrm{d}x = \frac{1}{V}(s-\bar{s})^{\dagger}(s-\bar{s})
$$

with the real volume \\( V=\int_{\Omega_x}\hspace{1mm}\mathrm{d}x \\). Then by changing notation to contra- and covariant
fields: 

$$
\begin{aligned}
V\sigma_s^2 &= (s-\bar{s})_x(s-\bar{s})^x \\
&= s_x s^x-s_x\bar{s}^x-\bar{s}_xs^x+\bar{s}_x\bar{s}^x

\end{aligned}
$$

## The Expectation Value 

Considering that \\(\bar{s}\\) is a scalar:

$$
\begin{aligned}
\langle V\sigma_s^2 \rangle 
&=  \langle  s_x s^x-s_x\bar{s}^x-\bar{s}_xs^x+\bar{s}_x\bar{s}^x \rangle \\
&= \langle s_x s^x \rangle - \langle s_x\bar{s}^x \rangle - \langle \bar{s}_xs^x \rangle + \langle \bar{s}_x\bar{s}^x \rangle \\
&= \langle \left| s^x \right|^2 \rangle - \bar{s}_x\bar{s}^x -  \bar{s}_x \bar{s}^x  +   \left|  \bar{s}^x \right| ^2 \\
&= \langle \left| s^x \right|^2 \rangle - \left|  \bar{s}^x \right| ^2 \tag{1}

\end{aligned}
$$

Now, we go into Fourier-Space. Let \\( F_x^k=e^{ikx} \\) be the Fourier-Transform operator. Then, 

$$
(F^{-1})_k^x = e^{-ikx}
$$

and integrals over the \\(k\\)-modes need to consider the volume \\( (2\pi)^u \\). Note that 

$$
s_x  = (s^x)^{\dagger} = ((F^{-1})_k^x s^k)^{\dagger} = (s^k)^{\dagger}((F^{-1})_k^x)^{\dagger}=s_kF_x^k,
$$

meaning that

$$
\begin{aligned}
\left| s^x \right|^2  &= s_x s^x =s_k F_x^k (F^{-1})_{k^{\prime}}^x s^{k^{\prime}}
\end{aligned}
$$

Transforming into Fourier-Space
gives us an integral over \\(\ k \\), \\( k^{\prime }\\) and \\( x \\): 



$$
\begin{aligned}
 \left| s^x \right|^2  
&= \int \mathrm{d}x \int \frac{\mathrm{d}k}{(2\pi)^u} \int \frac{\mathrm{d}k^{\prime}}{(2\pi)^u} e^{ikx}e^{-ik^{\prime}x}s_ks^{k^{\prime}} \\
&=\color{blue}\int \mathrm{d}x \color{black}\int \frac{\mathrm{d}k}{(2\pi)^u} \int \frac{\mathrm{d}k^{\prime}}{(2\pi)^u} \color{blue} e^{ix(k-k^{\prime})} \color{black} s_ks^{k^{\prime}} \\
&= \int \frac{\mathrm{d}k}{(2\pi)^u} \int \frac{\mathrm{d}k^{\prime}}{(2\pi)^u} \color{blue} (2\pi)^u \delta(k-k^{\prime}) \color{black} s_ks^{k^{\prime}} \\
&= \color{green} \int \mathrm{d}k \color{black} \int \frac{\mathrm{d}k^{\prime}}{(2\pi)^u} \color{green}  \delta(k-k^{\prime}) \color{black} s_ks^{k^{\prime}} \\
&= \int \frac{\mathrm{d}k^{\prime}}{(2\pi)^u} s_k^{\prime} s^{k^{\prime}} \\
&\stackrel{k^{\prime}\rightarrow k}{=} \int \frac{\mathrm{d}k}{(2\pi)^u} \left| s^k \right|^2 \\
&\stackrel{\color{red} (2\pi)^u???}{=}\int \mathrm{d}k \left| s^k \right|^2 \tag{2}


\end{aligned} 
$$

Now, remember the definition of the power spectrum: 

$$
P_s = \frac{\langle \left| s^k \right|^2 \rangle}{V} \tag{3}
$$

Combining equation \\( (1), (2) \\) and comparing to equation \\( (3) \\) gives: 

$$
\begin{aligned}
\langle \sigma_s^2 \rangle &= \frac{1}{V}\langle \int \mathrm{d}k \left| s^k \right|^2 \rangle - \langle \frac{1}{V}\left| \bar{s}^x \right|^2 \rangle \\
&=  \int \mathrm{d}k  \frac{1}{V} \langle\left| s^k \right|^2 \rangle - \frac{1}{V} \langle \left| \bar{s}^x \right|^2 \rangle \\
&= \int  P_s(k) \mathrm{d}k -\frac{1}{V} \langle \left| \bar{s}^x \right|^2 \rangle
\end{aligned} \tag{4}
$$

Furthermore, notice that 

$$
\begin{aligned}
P_s(0) &= \frac{1}{V}\langle \left| s^{k=0} \right|^2 \rangle \\
&= \frac{1}{V}\langle \left| \int \mathrm{d}x e^{i\cdot 0 x }s^x \right|^2 \rangle \\
&= \frac{1}{V}\langle \left| \int \mathrm{d}x s^x \right|^2 \rangle \\
&= \frac{1}{V}\langle \left| \bar{s}^x \right|^2 \rangle \tag{5}
\end{aligned}
$$

<span style="color:red">(Again, where is the PDF?)</span>
Which gives us our final equation when inputting into \\( (4) \\):

$$
\langle \sigma_s^2 \rangle = \int P_s(k) \hspace{1mm} \mathrm{d}k - P_s(0)
\tag{6}
$$

And now, if the model is parameterized via \\(l:=\mathrm{log}(k) \\), the zero-mode \\( k=0 \\) is excluded a priori,
meaning the expected signal variance is exactly equal to the operator \\( \tilde{U} \\): 

<div style="background-color: yellow ">
\[
\tilde{U}=\int_{k\neq 0} \mathrm{d}k P_s(k) = \langle \sigma_s^2(k\neq 0) \rangle
\]
</div>

Notice that if you go back to the post ["An Introduction To The Correlated Field Model"]({% post_url 2024-02-08-ift-the-correlated-field-model %}){:target="_blank"},
the signal power spectrum is indexed by \\(T\\), not \\(s\\).









