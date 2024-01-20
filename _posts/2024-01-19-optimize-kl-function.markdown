---
layout: post-custom
title:  "The Nifty-Gritty: The `optimize_kl() function"
date:   2024-01-16 17:21:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}

In this post I will try to explain (mostly to myself) the Bayesian Inference process happening inside the 
central 'optimize_kl()' function in NIFTy.

To do this, let's look at an analytically solvable example, visualize its analytical solution and then set up 
a NIFTy program that uses geoVI to numerically solve the problem. 

## An Analytical Problem

Assume an alien is looking at a fuzzy picture like this: 

![Multicolored tile ground truth](../../../../media/nifty-posts/ground-truth-tile.png)

The alien doesn't immediately receive the picture like this. The alien's eyes are covered by a grid shape
that allows only light from 100 different points of the picture to fall into its eyes to protect them.

![Aliens eyes are protetcted by a screen](../../../../media/nifty-posts/alien-uses-eyes-protector.png)

<a href="https://www.flaticon.com/free-icons/alien" title="alien icons">Alien icon created by Freepik - Flaticon.</a>

Furthermore, after a selection of a photons from 100 different spots hits its eye, the eye transforms the 
light signal to an electrical signal. Through a special dispersal mechanism, the photons are decomposed into 
corresponding RGB values and the alien is oversensitive to green values and undersensitive to red values.
That means that the eye converts the signal from a position \\( q\in \mathbb{R}^2\\) in the coordinate
grid like: 

$$
\begin{aligned}
d_q^{(r)} &= 0.5\cdot s_q^{(r)} + n_q^{(r)} \\
d_q^{(g)} &= 2 \cdot s_q^{(g)} + n_q^{(g)} \\
d_q^{(b)} &= s_q^{(b)} + n_q^{(b)}
\end{aligned} \tag{1}
$$

We can collect the random variables \\(s_q^{(rgb)}\\) in a random vector which stands for the to be inferred color
at location \\(q\\) in the coordinate system. The to be inferred random field is then simply the mapping 

$$
\begin{aligned}
s: \mathbb{R}^2 &\longrightarrow \mathbb{R}^3 \\
q &\mapsto (s_q^{r},s_q^{g},s_q^{b})^{\mathrm{T}} 
\end{aligned}
$$

In order to process environmental signals correctly, the alien's brain has involved to implement architecture 
for Bayesian Inference. Internally it rewrites equations \\( (1) \\) to 

$$
\begin{aligned}
d_q &= R s + n 
\end{aligned}
$$

$$
R = \left( \begin{array}{ccc}
0.5 & 0 & 0 \\
0 & 2 & 0 \\
0 & 0 & 1
\end{array} \right)
$$


where it assumes that the signal and noise are both drawn from Gaussians and are statistally independent of 
each other.

## Throwing The Whole Weight of NIFTy8 and geoVI Onto The problem