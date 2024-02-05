---
layout: post
title:  "Inspecting Stellar Evolution By Observing Mass Accretion"
date:   2023-11-08 00:47:58 +0100
categories: [science-stuff-planetary-blogs]
---
{% include katexLink.html%}

<div style="
    border-bottom: 2px solid #0a0082;
    margin: 20px 0 20px 0;
">
<b style="color: #0a0082;">TLDR!</b> <br>
Data suggests Brown Dwarfs evolve faster than sun-like stars. 
Exoplanet researchers create an extensive data table hinting at different accretion rate mechanisms
between different stellar populations.
    <a 
        href="https://arxiv.org/pdf/2310.00072.pdf#:~:text=Here%2C%20we%20present%20the%20Comprehensive,bound%20plan%2D%20etary%20mass%20companions." 
        target="_blank" 
        style="color: #0a0082; font-weight:bold;">
            https://arxiv.org/pdf/2310.00072.pdf
    </a>
</div>

# How To Inspect The Universe

Astronomers come in many different forms and colors. 
Some may have a knack for setting up sensitive experimental devices, 
while others scratch their head trying to connect the elegant equations 
of a mathematical model with the pen and paper.

In the 21st century, new state-of-the-art instruments and a plethora 
of sky surveys lead to ever growing datasets that demand more and more 
computing time and power to process.

Often, tools and models from various fields of physics have to be combined to properly 
interpret these datasets and only then can we see patterns emerging from large data.
From these patterns one can infer some truth about how a specific process and 
therefore also how the universe works.

# Accretion Disks

Researchers from Germany, Switzerland, the US and China have been working on expanding 
our knowledge in the field of dynamical stellar evolution. In various scenarios of 
stellar and planetary evolution, so-called accretion disks form.

In the case of a solar-mass star, a giant gas cloud fragments and collapses into itself. 
The centre of mass of the gas cloud will accrete matter as the cloud collapses, 
leading to a run-away process, in which a hot, young protostar is formed.

Due to the conservation of angular momentum particles do not fall directly onto the hot, 
dense protostar, but the spiral into it. Since many particles spiral into the same object 
from every direction, collisions between particles are unavoidable. Some particles get 
knocked into an orbit, where the have a statistical slightly smaller chance to collide 
with other particles. Over time, the particles accumulate into a single orbit, that 
represents the average orbital inclination and original velocity of the particles; 
An accretion disk is formed.

![Accretion Disk Forming Around Protostar.](../../../../media/planetary-blogs-posts/accretion.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px"}
<div id="Figure-1-Caption" style="
    margin: 0 auto; 
    width: 600px;
    font-size:12px;
    color: gray;
    padding-bottom:20px;
">
An accretion disk is an infalling flux of matter. The matter may fall for example
onto a young, hot protostar through the gravitational interaction between
the star and the gas cloud from which it initially formed. 
In this case, the accretion disk has a special name: 
The <b>circumstellar  or protoplanetary disk</b>. In the inner 
and outer regions of protoplanetary disks, accretion of gas and dust particles
leads to the formation of planets.
</div>

# Correlations

Gravity as we know it, only acts between objects that have mass (-and energy but let's put that aside).
Naturally, the mass \\(M\\) of an astronomical object is in many cases a central property of interest. 

Furthermore, the _accretion rate_, denoted by \\(\dot{M}\\) describes how much mass per unit 
time is gained through a process of accretion. Therefore it is also an important characteristic of the evolution
of a young stellar system. 

It turns out, that both of these quantities are _correlated_, in math terms: 

$$
\dot{M} \propto M.
$$

That means, that once scientists have a good estimate of \\(M\\), they also have a good estimate on 
\\( \dot{M} \\) and vice versa. 

This is an imprint of the deep physical connection between the two quantities. As a matter of fact, the 
correlation between the central mass and how much matter it accretes has an approximately parabolic trend: 

![Parabolic Correlation MP4](../../../../media/planetary-blogs-posts/accretion-rate-correlation.mp4){:style="display:block; margin-left:auto; margin-right:auto; padding: 20px 0 20px 0"}

In the research by [Betti et al.](https://arxiv.org/pdf/2310.00072.pdf){:target="_blank"} they find that the slope \\( a\\)
of the correlation in a "log-log-space" (a space where curved lines look linear), is 

$$
a = 2.02 \pm 0.06,
$$

confirming previous measurements.
This \\(M-\dot{M}\\) correlation is won when crunching the numbers together for accretion processes around
stars, brown dwarfs and planets. The authors argue the importance of describing these objects separately:

{% include planetary-blogs/ChartJS-accretion-mass-mass-correlation.html%}


We see differenecs in the mass accretion when splitting up in categories. Now, it seems that Brown Dwarfs
evolve faster than regular stars! 

Not only does the accretion mechanism depend on the mass of the central object, but age seems to play a role 
as well, since potentially different evolutionary mechanisms kick in at different times: 

{% include planetary-blogs/ChartJS-accretion-mass-change-age-correlation.html%}

The research shows the importance of data interpretation and questioning that interpretation in knowledge of 
the data uncertainty. Although for some data, the uncertainties (which were not presented here!) of the estimates were 
relatively high, the differences in correlation structure might potentially be due to different accretion mechanisms.  