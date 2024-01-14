---
layout: post
title:  "What are Gaussian Processes?"
date:   2024-01-14 15:39:58 +0100
categories: [science-stuff]
---
{% include katexLink.html%}


\\[
\mathcal{P} ( s \mid d ) = \frac{\mathcal{P}(s,d)}{\mathcal{P}(d)}=\frac{\mathcal{P}(d\mid s)\mathcal{P}(s)}{\mathcal{P}(d)} \tag{1}
\\]



> A Gaussian Process is to statistical processes what the normal distribution function is to probability distributions.

# Central Characteristic Of The Gaussian Process
Let  \\( x_1, ..., x_i \\)  be random variables, that follow some respective probability density functions ("PDF") 
\\( x_i \hookleftarrow \mathcal{P}_i\\). A <span style='color:blue;'>stochastic process</span> is a 
<span style='color:blue;'>Gaussian Process</span> if the joint probability distribution of the tracked random 
variables at a given index, is a multivariate Gaussian. A stochastic process in turn, in a nutshell, refers to the final and intermediate outcomes of a collection
of random variables that dynamically change. The dynamic evolution typically occurs in time. For example:


<div style="width:100%;display: flex; margin-bottom: 25px; color:#86ad6f; font-weight: bold">
    <div style="border-top: 3px solid #86ad6f; width: 20px; border-left: 3px solid #86ad6f;">
    </div>
    <div style="display: inline; padding-left: 15px; border-top: 3px solid #86ad6f; "> 
        Example:
    </div> 
    <div id="exampleDescription" style="display: inline; border-top: 3px solid #FFFFFF; padding-left: 10px" > 
        Tracking the evolution of two complementary variables. 
    </div>
</div>

```python
x_1 = [.1 , .3, .7, .9] 
x_2 = [.9 , .7, .3, .1] 
t = [1, 2, 3 ,4]

process = [(x1,x2,t) for x1,x2,t in zip(x_1, x_2, t)]
>>> [(0.1, 0.9, 1), (0.3, 0.7, 2), (0.7, 0.3, 3), (0.9, 0.1, 4)]

 ```


<div style="width:100%;display: flex; margin-bottom: 25px; color:#86ad6f; font-weight: bold">
    <div style="border-bottom: 3px solid #86ad6f; width: 20px; border-left: 3px solid #86ad6f;">
    </div>
    <div style="display: inline; padding-left: 15px; border-bottom: 3px solid #86ad6f; "> 
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </div> 
</div>

So the above process is a Gaussian Process if for each index \\( t\\),

$$
\mathcal{P}(x_1,...,x_i)(t) = \mathcal{P}(x_1,x_2)(t) = \mathcal{G}(y, Y)(t) = \frac{1}{\mid 2\pi Y \mid} \mathrm{exp}
\big(-\frac{1}{2}y^{\dagger}Yy \big)(t),
$$

where we collected all random variables in a vector \\(y= (x_1,...x_i)^{\mathrm{T}}\\) and \\( Y\\) refers to their 
covariance operator.

Assume it is valuable, even a valid description of our knowledge, to model some process of nature, such that the evolution 
of the joint probability density of observables of interest is a multivariate Gaussian that dynamically evolves in time.

pretty pretty pictures

How would we build such a process in the context of Information Field Theory, where observables in a more concrete sense 
refer to Blabla Data and stuff => Gaussian Priors => Basics => Product Rule for stochastically independent prob. density functions
=> Draw every random variable from a Gaussian thingy 

Check out my other blog post on <u>the markov process as another example of a stochastic process.</u>

### Joint Probabilities Or How To Calculate Logical Products 

As a reminder, the fundamental way to compute how likely it seems to be that random variables \\( a,b \\) are true at the same time
(conjunction) given some background Information \\( I \\), given that they follow some probability distributions \\( a\hookleftarrow P_a\\), 
\\( b\hookleftarrow P_b \\), is by using the product rule:  

$$ \omega (a,b \mid I) =: \omega(ab \mid I) = \omega(a \mid bI)\cdot \omega(b \mid I) = \omega(b \mid aI) \cdot \omega(a \mid I) $$

where \\( \omega \\) is the probability measure, a function that maps to the range \\( \[0,1\] \\). Notice the symmetry in how we can choose whether
to do the calculation with the "prior" of \\(a\\) or \\(b\\), \\( \omega (a\mid I) \\) or \\( \omega (b\mid I) \\). 

Notice especially, that if \\( a\\) and \\( b\\) are stochastically independent, that is to say that the probability measure of one variable 
does not conditionally dependent on the other, the product rule reduces to 

\\[
\omega(a,b\mid I) = \omega(a\mid bI) \cdot \omega(b\mid I) \stackrel{!}{=}  \omega(a\mid I) \cdot \omega(b\mid I)=: \omega(a)\cdot \omega(b)  \tag{2}
\\]

### Modeling Nature. The Correlated Field Model.

Therefore, it is obvious that a way to construct such a model is to sample each random variable from a univariate
which enforces normality in the multivariate case. Per definition, not every Gaussian Process has to have normally
distributed random variables, but normally distributed random variables inevitable lead to a Gaussian Process.

In Information Field Theory (to be more precise, its numerical implementation, NIFTY), the implementation of the Gaussian
Process is called the Correlated Field Model. 