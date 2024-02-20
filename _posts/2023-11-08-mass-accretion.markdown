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
<b style="color: #0a0082;">Summary</b> <br>
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

Astronomers come in many different shapes and sizes. 
Some may have a knack for setting up sensitive experimental devices, 
while others scratch their head trying to connect the elegant equations 
of a mathematical model with the pen and paper.

In the 21st century, new state-of-the-art instruments and a plethora 
of sky surveys lead to ever growing datasets that demand more and more 
computing time and power to process.

Often, tools and models from various fields of physics have to be combined to properly 
interpret these datasets. 
This interplay allows us to detect the patterns emerging from large data.
From these patterns, one can infer some truth about how a specific process, and 
therefore also how the universe works.

# Accretion Disks

Researchers from Germany, Switzerland, the US and China have been working on expanding 
our knowledge in the field of dynamical stellar evolution. In various scenarios of 
stellar and planetary evolution, so-called accretion disks form.

In the case of a to-be solar-mass star, a giant gas cloud has to first fragment and collapse into itself. 
The gas cloud's center-of-mass will accrete matter as the cloud collapses, 
leading to a run-away-process, in which a hot, young star is formed.

Due to the conservation of angular momentum, particles do not fall directly onto the hot, 
dense star, but rather they spiral into it. 
Since many particles spiral into the same object 
from every direction, collisions between particles are unavoidable. 
Some particles get knocked into an orbit, where they have a statistically slightly smaller chance to collide 
with other particles. 
Over time, the particles accumulate into a single orbit, that 
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
An accretion disk is an infalling flux of matter. The matter may fall, e.g,
onto a young, hot star; The infalling matter originates from the gas cloud the star initially formed from. 
In this case, the accretion disk has a special name: 
The <b>circumstellar  or protoplanetary disk</b>. In the inner 
and outer regions of protoplanetary disks, accretion of gas and dust particles
leads to the formation of planets.
</div>

# Correlations

Gravity, as we know it, only acts between objects that have mass (and energy according to Einstein's Theory of General
Relativity).

Naturally, in many cases, the mass of an astronomical object is a central property of interest. 

Furthermore, the _accretion rate_ is a quantity that describes how much mass is gained through a process of accretion. 
Together, the mass and accretion rate play a crucial role in the evolution of a young stellar system. 

It turns out that, because of their intricate connection, these two quantities are _correlated_.
That means: If scientists observe and calculate the properties of a stellar system and have a good estimate on the mass of the central star,
they _also_ have a good estimate on the accretion rate of the central star (and vice versa).

Two for one!

The _correlation function_ characterizes the strength of the connection between two quantities. 
For example, in case 
of the mass and accretion rate, the correlation function follows approximately a _parabolic_ trend:

![Parabolic Correlation MP4](../../../../media/planetary-blogs-posts/accretion-rate-correlation.mp4){:style="display:block; margin-left:auto; margin-right:auto; padding: 20px 0 20px 0"}

Consider, e.g., a fictitious system in which a young, accreting star has a mass of \\(0.4\\) times that of our sun. 
The correlation function would then tell us, that the accretion rate must be proportional to \\(0.4^2=0.16\\) \\(\times\\)
times the mass of our sun per year. 
In fact, accounting for a constant of proportionality and correct units, the accretion rate in the case 
discussed above would equate to a gain of approximately \\(0.00005\\) sun's masses per year! 
Or, in other words, the young star would swallow the equivalent of the sun's mass in only about \\(20.000 \\) years. 

This is a tiny number, compared to the usual astronomical timescales. 
Our sun's age for example, is over \\(4,6 \mathrm{bn} \\) years old.
"Birthing" a star, depending on the definition, may take up hundreds of thousands 
to millions of years.

In the research by [Betti et al.](https://arxiv.org/pdf/2310.00072.pdf){:target="_blank"} they find that the slope parameter of the correlation,
denoted by \\( a\\), is 

$$
a = 2.02 \pm 0.06,
$$

confirming previous measurements.
This figure is won when calculating the statistics of accretion processes around
stars, brown dwarfs and planets—alltogether! 
The authors argue the importance of describing these objects separately:

{% include planetary-blogs/ChartJS-accretion-mass-mass-correlation.html%}


Now, differences in the correlation function are visible when splitting up celestial bodies into categories!
It seems that Brown Dwarfs evolve faster than regular stars. 

Not only does the accretion mechanism depend on the mass of the central object, but age seems to play a role 
as well, since potentially different evolutionary mechanisms kick in at different times: 

{% include planetary-blogs/ChartJS-accretion-mass-change-age-correlation.html%}

The research shows that important information can be won by looking at data through different lenses
(e.g., categorization of the correlation function according to different types of stars).
It is always important to consider data uncertainty when interpreting or drawing conclusions from data. 
The mean of the correlation function curve for planets (blue-dashed line in the first figure), represents reality 
less likely than the other two curves. 

Although for some data the uncertainties of the estimates were 
relatively high, the differences in correlation structure might actually be caused by different accretion mechanisms. 
This hints at the diversity of stars and the many factors at play relevant for their creation.