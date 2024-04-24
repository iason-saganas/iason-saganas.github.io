---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home-custom
title: About me 
---
<blockquote>Pursuing knowledge about the stars since 2006.</blockquote>

{% include katexLink.html%}
{% include home/plotly3D-welcome-plot.html%}

<p style="font-size: 12px; color:gray">
    <span style="text-decoration: underline">
        What's this plot?
    </span>
    This plot is an intermediate result of one my analyses about tidal dynamics in a hierarchical 4-body-system.
    That means a system consisting out of a sun, a planet and its moon, and another moon around that moon - which we
    call submoon. There is interesting gravitational and tidal dynamics happening in this system. The tides (second order
    gravitational moment more or less) act to remove
    the satellites, i.e. the moon and the submoon in this case.
    This plot demonstrates a transition region in the semi-major-axis phase space of this system; 
    The axes are proportional to the actual initial values of the semi-major-axis of planet, moon and submoon. 
    These values are then put into the differential equations describing the system, which are solved in python 
    as an initial value problem; if the integration runs successfully for 4.5 billion years, we call the inital value 
    point stable, give it a color and plot it into this diagram. In this case I plotted those initial values which lead 
    to a stability of around 75%-90% of 4.5 billion years.
</p>

<p style="font-size: 12px; color:gray">
    These points represents a transition region in the phase space; All points lying in front of the
    plotted surface i.e. towards you (if you haven't moved the plot yet) will lead to a stable submoon-system;
    All points lying behind this surface i.e. away from you lead to the submoon or moon escaping their hosts or 
    the bodies crashing into each other.
</p>

<p style="font-size: 12px; color:gray">
    Try moving around the plot and getting a feel for the shape of this transition surface.
</p>

Hello you! Thank you for visiting my landing page. On this website you will find 
blog posts, software, videos and any other media I find worthy of sharing that documents
my journey through physics academia. 

Since as long as I can remember I am fascinated by nature. I was officially hooked 
after having read the book "Das Große Antwortbuch der Astronomie" ("The Great Q&A on Astronomy")
which was in approximately 2006. 


![Das große Antwortbuch der Astronomie](../../../../media/general/Große-Antwortbuch.jpg){:style="display:block; margin-left:auto; margin-right:auto"}

My goal is to contribute to the scientific knowledge of humanity in the fields of astrophysics,
data-science and cosmology, which is an incredibly exciting adventure to embark on in 
this day and age of the 21st century.

I want to excite kids and young adults to be curious about nature and the process of science 
which we use to probe nature with. In the best case I can nudge them on 
a path to consider becoming students of the natural sciences, especially physics and mathematics. 

Furthermore, when I grow up, I want to educate the next generation of excellent physicists. 