---
layout: post-custom
title:  "IFT: How To Find The Instrument Response"
date:   2024-01-16 21:10:58 +0100
categories: [science-stuff-ift]
---
{% include katexLink.html%}

In this document, I will elaborate on how I view the instrument response equation, 

\\[
d = R(s) + n \tag{1}
\\]

and give an example of how [Natalia Porqueres et al.](https://arxiv.org/abs/1608.04007){:target="_blank"} found the Signal 
Response Operator \\(R(s)\\) 
for the problem of reconstructing the energy density of the universe parameterized by redshift.

In equation \\( (1) \\), The data is we measure and the noise encodes e.g. systematic uncertainties of the measurement which is carried out by some telescope, interferometer etc.
Our goal is to essentially invert this equation for a concrete instance of \\(R(s)\\) and find the most likely \\(s\\).

How can we find the Response Operator for a concrete problem? One way is the following. It is obvious that \\( R(s) \\) determines the
relationship between what we measured \\( (d) \\) and the quantity we want to infer \\( (s)\\).

Data has to be created and then measured, i.e. there has to be a physical process
that creates an observable something, e.g. a photon, and then an instrument that reacts to or processes that object. In a particularly simple case,
the photon is detected on the surface of a metal via the photoelectric effect and the current generated can be translated to its energy.

So, as I see it, the data \\( d \\) <span style="color:blue">(measured current)</span> is an instrument-specific representation of the "real" physical property
<span style="color:blue">(energy)</span> of an information-carrying object <span style="color:blue"> (photon) </span>. The signal
\\( s \\) is a representation of <span style="color:blue"> _our knowledge_ </span> of the physical process, that created e.g. the photon or left
an energetic imprint on it in the first place.
This is the object we want to know about, since, while we may have a clue about the physics encoded in the signal \\( s \\), we need to take data samples
and infer from them the specific realization of \\( s \\) in our universe.

Think: A universe can be static, expanding or collapsing (physical knowledge = GR). But which one is the case _in our_ universe?
Comparing different instruments might even lead to conflicting \\(s\\), in which
case either something with our instruments, our data processing and interpretation or even our understanding of the physics is wrong.

The entire process of the "creation" of data (the energetic imprint of a physical process of interest on an information-carrying-object e.g. a photon)
until its reception through an instrument and its interpretation by a human can look something like this: 

![Flowchart Diagram of the data creation till registration process](../../../../media/ift-posts/Detailed-Measurement-Process.png)


In IFT, this whole process, is reduced to: 

![Reduced Flowchart in IFT ](../../../../media/ift-posts/Reduced-Response-Operator-Flowchart.png)

Since the noise is just additive, it is the Response Operator that has to encode all of this behaviour from the 
creation to the registration of the data. Therefore, I conclude that these are valid descriptions of the concrete functionality of \\( R(s) \\): 

- \\( R(s) \\) describes how the physical process that \\(s\\) encapsulates (e.g. the energy density evolution encapsulates the expansion of the universe) leaves an energetic imprint on or creates an information-carrying-object (e.g. a photon). This is where we have to **inject knowledge about a physical theory**. For example: How does the process specifically look like that creates Cerenkov-Radiation photons? Or how does the expansion of the universe specifically affect the photon energy coming from far away stars?  


<div style="border-left: 5px solid black; padding-left: 25px; margin-top:25px; margin-left:25px">

<b>Note </b>: 
Here, we do <b>not</b> have to specify a physical model (e.g. the ΛCDM model to explain the expansion of the universe). We just have to assert the fact
that for example, to our best knowledge, light from far-away galaxies is much dimmer and the degree of how weaker the light is, is proportional to their distance.
This is very important since we do not have to assume a constraining, biased "high-level" physical model to infer a quantity we want to know something about - "low-level"
theoretical knowledge suffices. In this context, we may call a signal reconstruction that does not need to eliminate extra degrees of freedom by assuming a 
concrete physical model an <b>agnostic reconstruction</b>. 

</div>

<div style="border-left: 5px solid black; padding-left: 25px;margin-top:25px; margin-left:25px; margin-bottom:25px">

<b>Note </b>:
At the same time, the interest in the use of <b>statistical models</b> to describe physical processes is rising. For example: The Gaussian Process as a Prior
for Bayesian Inference. Keyword: <b>The Correlated Field Model</b>. 


</div>

([Reference](https://arxiv.org/abs/2004.01393){:target="_blank"} for the rising interest using Gaussian Processes to model physical processes.)

- \\(R(s)\\) assumes the existence of information-carrying-objects (photons, grav waves etc.) and their fundamental properties (energy of photon, column density of dust, frequency of grav waves) 
- \\( R(s)\\) encapsulates the instrumentation process (e.g.: Going to Fourier-Space in Radiotelescopy)
- \\( R(s) \\) _discretizes_ the signal

![Coherent gaseous structure as signal field, going from signal field to noiseless data.](../../../../media/ift-posts/From-field-to-noiseless-data.png)
![Coherent gaseous structure as signal field, going from noiseless data to real data.](../../../../media/ift-posts/From-noiseless-data-to-real-data.png)

As an example, I show the derivation of the Signal Response for the problem of reconstructing the total energy density \\( \rho(z) \\) parameterized by redshift.
This can be interpreted as reconstructing the cosmic expansion history since the expansion rate at any given time is determined by the existing 
energy density in the universe, according to General Relativity.

The data we can infer the expansion rate at any given time from tuples of \\( (\mu,z) \\) of Supernovae Ia.

- \\( \mu \\): Distance modulus: Difference between the apparent and absolute magnitude. Measure of the "luminosity distance" to an object. Light gets weaker and weaker with redshift, i.e. the further back in time we look.
- \\(z \\): Redshift

If we take the observed flux \\( F \\) of some astrophysical source, we can associate a distance to the effective sphere radius onto which
(from Earth's point of view) the source's power or **luminosity** \\( L \\) is projected on.

$$
F= \frac{L}{4\pi d_{\mathrm{L}}^2}    \tag{2}
$$

For a subclass of astrophysical sources, we can calculate \\( L \\) directly (e.g.: SNIa). We can compare the relative difference in brightness
of two objects by defining

$$
\begin{aligned}
m_1 - m_2 &= 2.5\mathrm{log}_{10}\bigg(\frac{F_1}{F_2} \bigg) \\
&= 2.5 \mathrm{log}_{10}\bigg( \frac{L_1 d_2^2}{L_2d_1^2} \bigg)
\end{aligned}
$$

If we specifically look at the one and the same object, \\( L_1 = L_2 \\) and compare an arbitrary distance \\( d=d_L \\) to the distance 
of exactly \\( 10 \mathrm{kpc}\\), the second apparent magnitude \\( m_2\\) becomes the object's absolute magnitude \\( M \\). 

$$
\mu := m-M = 5\mathrm{log}_{10}\bigg(\frac{d_{\mathrm{L}}}{10\mathrm{kpc}}\bigg) = 5 \mathrm{log}_{10}(d_{\mathrm{L}} \hspace{1mm} \mathrm{[kpc]} \hspace{1mm})-5    \tag{3}
$$

\\( \mu \\) is effectively the actually measured quantity. Now, we want to know how \\( \mu \\) varies with the redshift so we can track the 
weaking of the light at different periods in time of the universe. We can do this by injecting knowledge from GR: 

$$
\begin{aligned}
d_{\mathrm{L}} &= \frac{1}{a} d_{\mathrm{co}}    \tag{4} \\
&= \frac{1}{a} d_H \int_0^z \frac{1}{E(z)}\mathrm{d}z
\end{aligned}
$$

- \\(a \\): The scale factor
- \\(d_{\mathrm{co}} \\): The comoving distance
- \\(d_H \\): Hubble's Length, \\(:= c/H_0\\)
- \\(H_0 \\): The Hubble constant, the current rate of the expansion of the universe
- \\(E(z) \\): The dimensionless Hubble Parameter, measures the expansion rate at a given \\(z\\) in terms of the current expansion rate \\(:= H(z)/H_0\\)

We remember that the Friedmann Equation tells us 

$$
E(z) = \bigg( \frac{\rho(z)}{\rho_0} \bigg)^{1/2}   \tag{5}
$$

and insert equations \\( (4) \\) and \\( (5) \\) into \\(^{-1} (3) \\). We get: 

$$
\mu = 5 \mathrm{log}_{10}\bigg( \frac{1}{a} d_H \int_0^z \bigg( \frac{\rho(z)}{\rho_0} \bigg)^{-1/2} \mathrm{d}z  \bigg)-5 \tag{6}
$$

This already tells us how the data we measured, \\( \mu \\), is "created" from our quantity if interest, \\(\rho(z)\\). Now, we will take two
additional steps: 

- Define our signal field as \\(s(x) = \mathrm{log} (\frac{\rho(x)}{\rho_0} )  \\). Taking \\(\rho_0 \\) into account already here effectively centers the Gaussian Prior we will pick later
- Express everything in terms of a new coordinate, the redshift magnitude \\( x:= -\mathrm{log}(a) \\). This helps us in finding the covariance operator \\(S\\). Because of the scale factor - redshift relation \\(a = (1+z)^{-1}\\) this means that \\(x = \mathrm{log}(1+z)\\).

Parameterizing Equation \\( (6) \\) in terms of the signal yields our Response Operator. 

$$
    R(s) = 5 \mathrm{log}_{10} (e^x d_H \int_0^x e^{-\frac{1}{2}s(\tilde{x})+\tilde{x}} \mathrm{d}\tilde{x}) - 5\tag{7}
$$
