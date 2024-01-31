---
layout: post-custom
title:  "The Nifty-Gritty: Minimizers And Iteration Controllers"
date:   2024-01-26 11:06:58 +0100
categories: [science-stuff-nifty]
---
{% include katexLink.html%}

The `optimize_kl()` function needs, amongst others, these input parameters:

```python
def optimize_kl()
    ...
    kl_minimizer, 
    '''
    Finds the descent direction
    '''
    
    sampling_iteration_controller, 
    
    '''
    Controlls the sampling of the approximating Gaussian Hamiltonian
    '''
    
    nonlinear_sampling_minimizer 
    
    '''
    Minimizer used to perform the non-linear part of geoVI sampling. If
    it is None, only the linear (MGVI) approximation for sampling is
    used and no further non-linear steps are performed. Inside 
    'SampledKLEnergy()'.
    '''
```

### What are energy controllers?

Energy controllers are subclasses of the abstract base class `IterationController`. 
Iteration controllers check the progress of a minimization process. 
They do this by checking some energy condition between the energies of the current and 
the last step for all iterations. If the condition is `True` they return `CONVERGE`, increasing
the convergence counter by one. If the convergence counter reaches the `int` specified by 
`convergence_level`, the minimization iteration terminates, `else` it continues.

Possible energy conditions to check for are: 

- \\(L^2\\) gradient energy: check if \\( \sqrt{\int (\nabla E_{i-1})^{\ast} \nabla E_i \mathrm{d}x } < \text{ some value}\\)
- \\(L^{\infty}\\) gradient energy: check if \\( \mathrm{inf} \lbrace C \geq 0 : \hspace{2mm} \mid \nabla E_i(x) \mid \hspace{2mm} \leq C \text{  for almost every x }\rbrace  < \text{ some value}  \\)
- Relative energy: check if \\( E_i - E_{i-1}  < \text{ some value}\\)
- Absolut energy: check if    \\( \mid E_i - E_{i-1} \mid  < \text{ some value}\\)
- Standard deviation energy: check if \\( \sigma (E_{i-1}) < \text{ some value} \\)

I assume because `geoVI` is more computationally expensive, any iteration control needed for `geoVI` needs
something like this as input:  

```python
ic_sampling_nl = ift.AbsDeltaEnergyController(name="Sampling (nonlinear)", 
                           deltaE=0.5, iteration_limit=20, convergence_level=2)
```
Whereas I think that in `MGVI` and linear parts of the inference, a much more precise iteration control is acceptable, 
maybe even necessary:

```python
ic_sampling_lin = ift.AbsDeltaEnergyController(name="Sampling (linear)", 
                           deltaE=0.05, iteration_limit=100)
```


### What are minimizers?

Minimizers find e.g. the energy descent direction (towards the most probable value) by implementing some algorithm.
Often, a Newton-Conjugate-Gradient scheme is used.
Minimizers need energy iteration control objects to properly work. For example: 

```python
ic_newton = ift.AbsDeltaEnergyController(name='Newton', 
                            deltaE=0.5,convergence_level=2, iteration_limit=35)
# the same energy criterion as the geoVI iteration control, but more iteration steps
descent_finder = ift.NewtonCG(ic_newton)
```
This object (`descent_finder`) is used on the `SampledKLEnergyClass` object returned by the `sampledKLEnergy` function 
and returns an energy object representing the most probable field realization in that iteration:

```python
# e is the by the 'SampledKLEnergy' function returned class object.
# this energy object has information on value, gradient and metric at a
# specific position in parameter space. The newton minimizer acts upon it.
e, _ = minimizer(e)
```

For the non-linear sampling part of `geoVI`, the iteration controller has to be 'promoted' to a minimizer:

```python
geoVI_sampling_minimizer = ift.NewtonCG(ic_sampling_nl)
```

### Calling optimize_kl()

```python
optimize_kl(
    kl_minimizer = descent_finder, 
    sampling_iteration_controller = ic_sampling_lin,
    nonlinear_sampling_minimizer = geoVI_sampling_minimizer
)

```