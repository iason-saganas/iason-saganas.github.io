---
layout: post
title:  "Tides Short Story"
date:   2024-02-20 11:47:58 +0100
categories: [science-stuff-planetary-blogs]
---
{% include katexLink.html%}

<div style="
    border-bottom: 2px solid #0a0082;
    margin: 20px 0 20px 0;
">
<b style="color: #0a0082;">Description</b> <br>
This article is a short description of what tides are and how they work, in the style of the short articles at for example
    <a 
        href="https://outreach.spp1992-exoplanetdiversity.de/neptune-sized-exoplanets/" 
        target="_blank" 
        style="color: #0a0082; font-weight:bold;">
            https://outreach.spp1992-exoplanetdiversity.de/neptune-sized-exoplanets/.
    </a>
</div>

# Tides

In Newtonian Mechanics, the moon, planets and stars are modelled as **point-particles**: Bodies whose volume is close 
to zero but that carry the whole lunar, planetary or stellar mass. The particles then interact via Newton's Law of 
Universal Gravitation: 

![Point particles interact via newton's law](../../../../media/planetary-blogs-posts/point-particle-interaction-via-newtons-law.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 40vw"}

where \\( m_1\\) and \\(m_2 \\) are the masses of the two bodies attracting each other, \\(r \\) is their separation and
\\( G \\) is the gravitational constant.

The point-particle model is only a **first order approximation**. In reality, celestial bodies have volume.

Consider the system of a moon accompanying its host planet. A natural consequence due to the bodies' spatial extent,
is that the planet "feels" gravity more strongly on one side.  

![Gravitational gradient building up accros surface](../../../../media/planetary-blogs-posts/gravitational-gradient-across body.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 40vw"}

As a result, a gravitational gradient builds up across the planet's surface. This difference in gravitational interaction
between the less and more attracted side, forces the particles that can be thought of to make up the planet, to rearrange.

More specifically, the planet is "squished" toward the direction of the moon. Because of a symmetry in the gravitational 
potential, the same "squish" needs to happen in the opposite direction; Otherwise the planet would disintegrate. 

So-called "radial tidal bulges" have formed.

![Tidal Bulges have formed](../../../../media/planetary-blogs-posts/tidal-bulges-have-formed.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 60vw"}

In physics, no process is perfect; Energy is always lost due to friction, and tidal interactions are no exception. 
A model called the "constant geometric lag model" explains how the tidal response of the planet, that is the formation 
of the tidal bulge, lags a bit behind the tidal perturbation by the moon, _spatially_.

In other words, the moon-facing tidal bulge does not form directly under the tide-raising satellite, but is shifted by
a small amount clockwise or counter-clockwise relative to it (depending on initial conditions). 

How big this shift is, is encoded in the so-called **Quality Factor** \\( Q \\) of the planet.
Because there is a higher concentration of mass around the tidal bulge, and the tidal bulge is now shifted away 
from the moon's position in the sky, the gravitational equilibrium is disrupted; 

![Tides lead to a cosmic wrench](../../../../media/planetary-blogs-posts/cosmic-wrench.png){:style="display:block; margin-left:auto; margin-right:auto; padding-top: 20px; width: 40vw"}

The moon and the tidal bulge attract each other. In the example shown in this picture, the tidal bulge is carried 
ahead of the moon. Due to the gravitational attraction, the satellite forces the spinning planet to slow down, so as
to catch up with the tidal bulge.

Effectively, a cosmic wrench, the size of a planet, is implemented by this interplay.

Due to the law of the conservation of angular momentum, if the planet _slows down_, that loss in angular momentum has 
to be _gained somewhere else_; In fact, the moon itself gains this angular momentum by sapping it out and absorbing it 
into its orbital motion. As a result, by Kepler's Third Law, the moon widens its orbit around the planet, increasing 
their mutual separation. 

This is the underlying mechanism of a process called **tidal migration**.
Tidal migration is one of the limiting factors on the lifetime of moons (or even moon's moons, called _submoons_).
This happens right now, for our earth-moon system. Due to tidal migration, the moon pushes itself away from us at a rate 
of about \\(3.8\mathrm{cm}\\) per year (this can be measured with lasers). That rate is not enough for the moon to
escape earth's gravitational influence before both getting obliterated by the sun's red giant phase in about \\(5 \mathrm{bn}\\)
years.

Depending on the initial conditions of the system, the moon could come crashing down, being disintegrated at 
the so-called "Roche-Limit" and forming a "lunar-ring" around the planet, instead of escaping it.

Mathematically, the tidal potential is the second term in the taylor expansion of the gravitational potential and 
therefore captures the gravitational quadrupole moment.





















































