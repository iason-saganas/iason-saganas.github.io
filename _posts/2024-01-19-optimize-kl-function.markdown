---
layout: post-custom
title:  "The Nifty-Gritty: The `optimize_kl()` function"
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
that allows only light from a finite amount of different points of the picture to fall into its eyes to protect them.

![Aliens eyes are protetcted by a screen](../../../../media/nifty-posts/alien-uses-eyes-protector.png)

<a href="https://www.flaticon.com/free-icons/alien" title="alien icons">Alien icon created by Freepik - Flaticon.</a>

Furthermore, after a selection of a photons from different spots hits its eye, the eye transforms the 
light signal to an electrical signal. Through a special dispersal mechanism, the photons are decomposed into 
corresponding RGB values and weighted with a specific factor.
That means that the eye converts the signal from a position \\( q\in \mathbb{R}^2\\) in the coordinate
grid like: 

$$
\begin{aligned}
d_q^{(r)} &= 0.9\cdot r + n^{(r)}  \\
d_q^{(g)} &=  0.5\cdot  g + n_q^{(g)} \\
d_q^{(b)} &= 0.2 \cdot b + n_q^{(b)} 
\end{aligned} \tag{1}
$$

![Data Tile](../../../../media/nifty-posts/data-tile.png)

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
0.9 & 0 & 0 \\
0 & 0.5 & 0 \\
0 & 0 & 0.2
\end{array} \right),
$$


where it assumes that the signal and noise are both drawn from Gaussians and are statistically independent of 
each other: 

$$
s \hookleftarrow \mathcal{G}(s,S)
$$

$$
n \hookleftarrow \mathcal{G}(n,N)
$$

Furthermore, the noise operator is diagonal and it estimates the covariance as: 

$$
S=\left( \begin{array}{ccc}
0.028 & 0.010 & -0.0037 \\
0.010 & 0.021 & -0.0010 \\
-0.0037 & -0.0010 & 0.0006
\end{array} \right)

\hspace{10mm} N_{ij} = (0.001)^2 \delta_{ij}.
$$

Over millions of years of evolution, the alien has learned to recognize a signal with a linear response, 
Gaussian and independent signal and noise statistics and implement the Wiener Filter to reconstruct the
most probable representation of reality. 
It knows that the most probable signal is 

$$
m = Dj,
$$

where 

$$
D = (R^{\dagger}N^{-1}R+S^{-1})^{-1} \hspace{10mm} j = R^{\dagger}N^{-1}d,
$$

It then easily works out the most probable picture that actually is before its eyes: 

![Biological Wiener Reconstruction By Alien](../../../../media/nifty-posts/alien-uses-biological-wiener-reconstruction.png)
![Multicolored tile ground truth](../../../../media/nifty-posts/ground-truth-tile-without-cmap-bar.png)

## Throwing The Whole Weight Of NIFTy8 And geoVI Onto The Problem

We found a dead alien in a corn field. 
Upon transferring it into a forensic clinic and dissecting its brain, we stimulate its eyes with a fuzzy image
and observe the current flux in its brain using an encephalogram to study the signal processing happening 
inside its brain. 

For this, we numerically simulate the process happening inside the alien's brain.

But what does the `optimize_kl()` function do? Well, let's look at its mandatory arguments and their documentation: 

### Input Parameters

``` python
def optimize_kl(
    likelihood_energy,
    
    ''' ~> Type:            Operator or Callable.
        ~> Documentation:   Likelihood energy shall be used for inference.
        ~> Note:            Function is accepted as an input that takes the 
                            global iteration variable and returns a specific 
                            value/instance of object that should be used in that
                            iteration.
    '''
    
    total_iterations,
    
    ''' ~> Type:            Int.
        ~> Documentation:   Number of resampling loops. The higher, the higher 
                            the computational expense, the more accurate the 
                            posterior approximation.
    '''
    
    n_samples,
    
    ''' ~> Type:            Int or Callable.
        ~> Documentation:   Number of samples used to sample Kullback-Leibler 
                            divergence. 0 corresponds to maximum-a-posteriori. 
                            In VI methods, the true posterior which can't be 
                            found analytically is approximated by minimizing 
                            the KL divergence between an approximant and the 
                            accessible part of the true posterior via Bayes 
                            Theorem (renormalization constants irrelevant in 
                            optimization). The KL divergence must not be 
                            calculated explicitly, the optimization can be 
                            done on a stochastic sample of the KLD. See 
                            Saliman et al. 2012 for further info 
                            (https://arxiv.org/abs/1206.6679).
    '''
    
    kl_minimizer,
    
    ''' ~> Type:            Minimizer or Callable.
        ~> Documentation:   Controls the minimizer for the KL optimization.
    '''
    
    sampling_iteration_controller,
    
    ''' ~> Type:            IterationController or None or Callable.
        ~> Documentation:   Controls the conjugate gradient for inverting the 
                            posterior metric.  If `None` only suited for 
                            maximum-a-posteriori solutions.
    '''
    
    nonlinear_sampling_minimizer, 
    
    ''' ~> Type:            Minimizer or None or Callable.
        ~> Documentation:   Controls the minimizer for the non-linear geometric 
                            sampling of the KL. If None, MGVI 
                            algorithm is used instead of geoVI.
    '''
)
```

### Output Directory and Sanity Checks

In the first few lines after the function definition, all arguments are made to callables via a `lambda` 
function, in case they aren't yet callables.

Two `if`-loops follow, that specify
1. the contents collected in the `output_directory` (intermediate results of the inference)
2. whether all domains of likelihoods and transitions fit together

``` python
if output_directory is not None:
    ...

if sanity_checks:
    ...
```

### Creating and Picking a 'Null-Field' as Initial Position

After, an initial position is picked: 

```python
# if MPI computing ressources are used, check that tasks are well-synchronized
check_MPI_synced_random_state(comm(initial_index)) 
if initial_position is None: 

    '''
    initial_position is an optional argument in the 'optimize_kl()' call. So 
    in general, this part of the if-loop will run
    '''
    
    from ..sugar import full, makeDomain
    mean = full(makeDomain({}), 0.) # the initial mean
    
    '''
    'full' is a convenvience function that takes a domain and creates from it 
    a Field that is filled everywhere with the supplied value. In this case, 
    an empty domain is filled with zeroes, creating a 'null' field.
    '''
    
else:
    mean = initial_position
```

### More Sanity Checks

```python
def trans(iglobal, prev_dom, next_dom):
    ...
    
    '''
    If 'likelihood_energy' is a callable consisting of different energy 
    operators, this provides information on those likelihoods. See for 
    example the next sanity check. 
    '''
    
if sanity_checks:
    ...
    
    '''
    For example: If 'likelihood_energy' is a callable, check that the 
    target of the current lh-energy is the domain of the next lh-energy.
    '''
    
if output_directory is not None:
    ...
    
    '''
    Document some more stuff.
    '''
```

### The Main For-Loop ~ 90 Lines of Code

```python
    for iglobal in range(initial_index, total_iterations):
    
        '''
        Creating the for loop over which the posterior is approximated 
        better and better. Starting at 'initial_index' (an optional 
        argument to the 'optimize_kl()', default value 0) and ending at 
        'total_iterations' (a mandatory argument, one of the factors 
        determining the accuracy of the posterior approximation).
        'initial_index' is an int, not to be confused with 'inital_position', 
        which is the "null-field" initialized as a starting position and 
        stored in the variable "mean"! So 'iglobal' is the iteration 
        variable of the "global" resampling loop.
        '''
        
        lh = likelihood_energy(iglobal) 
        
        '''
        Get the current likelihood energy. Will always be the same object,
        if the 'likelihood_energy' argument was not a-priori inputted 
        as a callable.
        ''' 
        
        mean = trans(iglobal, mean.domain, lh.domain)(mean)
        dom = mean.domain
        
        '''
        Remember, mean is the 'null-field' created as a starting position.
        If the argument 'likelihood_energy' is not a callable, 
        trans(...)(mean) is the identity. 'dom' is in general a multidomain.
        '''

        count = CountingOperator(lh.domain)  
        
        '''
        This is an operator, defined on the current likelihood domain,
        that has attributes called '_count_apply' and '_count_apply_lin' 
        that get incremented each time the 'count' instance is applied to 
        another operator (_count_apply_lin += 1 if operator applied on 
        is "linearized-like", otherwise _count_apply += 1).
        'count' returns the operator it was applied to itself.
        '''
        
        ham = StandardHamiltonian(lh @ count, sampling_iteration_controller(iglobal))
        
        '''
        Gaussian Energy Information Hamiltonian with unit covariance. 
        This is the approximating distribution in MGVI and geoVI.
        The first argument is just the likelihood energy 
        (application of '@ count' just increased the internal count attributes 
        of 'count'). The second argument is the currently in-use 
        'sampling_iteration_controller' (mandatory argument of the 'optimize_kl()' 
        function). The iteration controller says which convergence criterion 
        to use to draw Gaussian samples.  
        '''
        
        minimizer = kl_minimizer(iglobal)
        
        '''
        The currently in-use 'kl_minimizer' (mandatory argument to 
        the 'optimize_kl()' function).
        '''
        
        mean_iter = mean.extract(ham.domain)
        
        '''
        checks whether the domain of the multi-field instance 'mean' that 
        represents the initial position is equal to the domain of the energy 
        hamiltonian associated with the likelihood. If no error is thrown, 
        the 'mean' multi-field is returned again. So if this line runs correctly, 
        this essentially does 'mean_iter = mean'
        '''
        
        if n_samples(iglobal) == 0:
        
            '''
            'n_samples' is a mandatory argument to the 'optimize_kl()' function, 
            that determines how many samples to draw from the KL divergence to 
            estimate the approximation quality. If zero, imlement MAP solution
            ''' 
            
        else: 
        
            '''
            sample KL divergence for non-MAP solution using 
            'SampledKLEnergy()' function. 
            '''
            mean = something ... # The returned quantity!
            sl = something ... # The returned quantity!
            
        if output_directory is not None:
            ... # document even more
            
        del lh # delete the currently in-use likelihood instance to use the next one
        
    return (sl, mean) if return_final_position else sl
    # return statement of the 'optimize_kl()' function
        
```

At the heart of the `optimize_kl()` function lies the `if`-loop starting with `if n_samples(iglobal) == 0:`. 
Particularly interesting is the `else` statement that calls the `SampledKLEnergy()` function. 
Let us investigate this `if-else` statement further. 

### The function lying at the heart of `optimize_kl()`: `SampledKLEnergy()`

```python
if n_samples(iglobal) == 0:
            ...
            
            '''
            Ignored here, see paragraph above for a small description. 
            ''' 
else: 

    '''
    sample KL divergence for non-MAP solution using 
    'SampledKLEnergy()' function. 
    '''
    
    mean = something ... # The returned quantity!
    sl = something ... # The returned quantity!
```

In Gaussian Variational Bayes Methods, the true, not-accessible posterior is approximated by a Gaussian
through optimizing the KL divergence between the approximant Gaussian and the _accessible_ part of the true
posterior through Bayes Theorem. See the 'Variatonal Inference' section in this blog post: 
[IFT: The Amplitude Transform and Standardized Coordinates.]({% post_url 2024-01-26-the-amplitude-transform %}){:target="_blank"}

To optimize the KL divergence, one does not have to calculate it explicitly. 
Rather, the optimization can be done via a stochastic sample of the KL divergence.
This is what the `SampledKLEnergy()` function implements. 
See [Saliman et al. 2012](https://arxiv.org/abs/1206.6679){:target="_blank"}.

Let's dive into the `else` statement. 

```python
if ...:
            ...
else:

    '''
    this is not the documentation, this is the actual function call with 
    my notes written inside.
    '''
    
    e = SampledKLEnergy(
                mean_iter, 
                
                '''
                The initial field position. After this function is run, 
                the initial field is updated to 'mean', i.e. after the 
                first iteration, this contains the current signal field 
                estimation! Expansion point of the coordinate transformation.
                '''  
                
                ham,
                
                '''
                The approximating distribution. Gaussian for MGVI and geoVI.
                '''
                
                n_samples(iglobal),
                
                '''
                Number of samples used to stochastically estimate the KL. 
                Mandatory argument to the 'optimize_kl()' function.
                '''
                
                nonlinear_sampling_minimizer(iglobal),
                
                '''
                Minimizer used to perform the non-linear part of geoVI 
                sampling. If it is None, only the linear (MGVI) 
                approximation for sampling is used and no further 
                non-linear steps are performed.
                '''
                
                comm=comm(iglobal),
                constants=constants(iglobal),
                point_estimates=point_estimates(iglobal)
                
                '''
                Some other stuff, inferring constants and invariant 
                positions in the signal field.
                '''
                
                )
    
    '''
    'SampledKLEnergy' provides the sampled KL and 
    returns a class object called 'SampledKLEnergyClass'.
    'SampledKLEnergyClass' is a base class for Energies 
    representing a sampled Kullback-Leibler divergence for the 
    variational approximation of a distribution with another 
    distribution.
    
    The sampling happens via a call to the function 
    'draw_samples()'

    '''    
                
    e, _ = minimizer(e)
    
    '''
    I think: 'minimizer()' finds the descent direction using 
    a Newton-Conjugate Gradient Scheme, 'performs the step' 
    towards that direction and returns an object containing 
    information about the new environment. The information is 
    stored in the variable 'e', the second returned variable is 
    surpressed.
    '''
    
    mean = MultiField.union([mean, e.position]) if mf_dom else e.position
    
    '''
    Update the new mean to the new position in the found 
    descent direction 'e.position'! ('mf_dom' = "Multifield-Domain")
    '''
    
    sl = e.samples.at(mean)
    
    '''
    Create a sample list of the updated posterior mean position in the new 
    environment in parameter space, as encoded in the 'e' variable! 
    '''
    
    energy_history.append((iglobal, e.value))
    
    '''
    Documentation.
    '''
```

Now, it should be clear how through the iteration of this code, an estimation of the mean posterior field 
is found through recursive updating. 
The returned quantities are the sample list `sl` and if the argument `return_final_position` is set to `True`, 
the mean field in the last iteration of the `for-loop`. Then, the posterior samples can be analyzed e.g. like
this: 

```python
samples = ift.optimize_kl(**arguments)
print("Kullback Leibler Divergence has been minimized, Posterior Samples found.")

# ------- Grab Posterior Data ------- #

posterior_realizations, final_posterior_position = samples
mean, var = posterior_realizations.sample_stat(signal)
```

There is only one more step to go down into in the code hierarchy: Inside `SampledKLEnergy()`, the 
`draw_samples()` function hides.

### The function lying at the heart of `SampledKLEnergy()`: `draw_samples()`

```python
'''
This is more verbose pseude code!
'''

def sample_list = draw_samples(
        position,
        
        '''
        The current signal field estimation.
        '''
        
        approximating_hamiltonian, 
        
        '''
        The approximating distribution. Gaussian for MGVI and geoVI.
        '''
        
        minimizer_sampling, 
        
        '''
        If None, use MGVI else use geoVI.
        '''
        
        n_samples,
        mirror_samples, 
        napprox=napprox, 
        comm=comm
        
        '''
        Other things
        '''
        ):
        
        sampling_position = position.extract(approximating_hamiltonian.domain)
        
        '''
        Get the position to sample at. if the Multifield's domain 
        (position._domain) is equivalent to the approximating 
        Hamiltonian's domain, return the position itself again.
        Else, return the Multifield defined on a domain subset. 
        '''
        
        if use_geo_VI: 
            transformation = approximating_hamiltonian
                                .likelihood_energy
                                .get_transformation()
                                
            '''
            Returns a tuple: The used data type and an operator or 
            chain of operators representing the coordinate transformation 
            that maps into a coordinate system in which the metric of a 
            likelihood is the Euclidean metric.
            '''

```

A transformation example follows. 
The central transformation in geoVI represents a change in coordinate system,
such that the metric of the likelihood is mapped to Euclidean Metric.
Say the likelihood is Gaussian, i.e.

$$
\mathcal{H}(d\mid s) \hspace{1mm} \hat{=} \hspace{1mm} \frac{1}{2} \bigg( d- R(s) \bigg)^{\dagger} N^{-1} \bigg( d- R(s) \bigg).
$$

This is the case if the Noise \\(n \\) can be described by a signal-independent Gaussian:

$$
\mathcal{P}(n\mid s) = \mathcal{G}(n,N).
$$

Then, the mentioned transform is simply given by the operator square root of the inverse Covariance as seen from 
this code snippet: 

```python 
class GaussianEnergy(LikelihoodEnergyOperator):

    """
    
    Computes a negative-log Gaussian.
    Represents up to constants in d:

        E(f) = - \\log G(f-d, D) = 0.5 (f-d)^\\dagger D^{-1} (f-d),

    an information energy for a Gaussian distribution with data d and
    covariance D.
    
    """
    
    ...
    def get_transformation(self):
        return self._icov.sampling_dtype, self._icov.get_sqrt()
```

The math markup in the docstring translates to: 

$$
E(f) = \frac{1}{2}(f-d)^{\dagger}D^{-1}(f-d).
$$

Going back to the `draw_samples()` snippet: 

```python
    if use_geo_VI: 
                transformation = approximating_hamiltonian
                                    .likelihood_energy
                                    .get_transformation()
                                    
                '''
                Returns a tuple: The used data type and an operator or 
                chain of operators representing the coordinate transformation 
                that maps into a coordinate system in which the metric of a 
                likelihood is the Euclidean metric.
                '''
                
                data_type, likelihood_transform = transformation
                
                linearization_transform = likelihood_transform(
                                                Linearization.make_var(sampling_position)
                                                )
                
                '''
                1.) Create a Linearization Object from the current signal 
                    field estimate 'sampling_position' by calling the 
                    '.make_var' method and passing the signal field 
                    estimate as a value. A Linearization Object stores
                    information on the field values and the Jacobian
                    of a field and how that changes once operators are 
                    applied to it. In this case, since no operation has 
                    yet been applied, a unit Jacobian is used via a 
                    Scaling Operator.
                2.) Apply the coordinate transform onto that 
                    Linearization.
                '''
                
                coordinate_system_transform_dom = linearization_transform.domain
                operator1 = ScalingOperator(coordinate_system_transform_dom, 1.)
                
                adjoint_jac = linearization_transform.jac.adjoint
                operator2 = adjoint_jac @ likelihood_transform
                
                transformation = operator1 + operator2
                transformation_mean = sampling_position + 
                                      adjoint_jac(linearization_transform.val)
                metric = SamplingEnabler(...)
                
                '''
                I believe this formulas correspond exactly to equations from the 
                geoVI paper. To Do. Once we have the transformation,
                transformation_mean and metric, samples from the KL can 
                be finally drawn for its optimization! 
                '''
                
```

It is worthy to note that by using the python `inspect` module,
we can see that  the `apply` method of the signal response is activated on three different occasions
inside the `draw_samples` function call.

The original lines that activate the apply method of the signal response are: 

```python
- fl = f_lh(Linearization.make_var(sam_position))
- transformation_mean = sam_position + fl.jac.adjoint(fl.val)
- y, yi = met.special_draw_sample(True) # most of all
```

or a bit more verbose: 

```python
- linearization_transform = likelihood_transform(
                                Linearization.make_var(sampling_position)
                                )
                                
- transformation_mean = sampling_position +
                        adjoint_jac(linearization_transform.val)
                        
- b, energy.position = metric.special_draw_sample(True) # most of all
```