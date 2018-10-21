[![Build Status](https://travis-ci.org/matthieugomez/EconPDEs.jl.svg?branch=master)](https://travis-ci.org/matthieugomez/EconPDEs.jl)


This package includes the function `pdesolve` that allows to solve (system of) ODEs/PDEs that arise in economic models:
- ODEs/PDEs corresponding to HJB equations (i.e. differential equations for value function in term of state variables)
- ODEs/PDEs corresponding to market pricing equations (i.e. differential equations for price- dividend ratio in term of state variables)

This package proposes a new, fast, and robust algorithm to solve these ODEs / PDEs. I discuss in details this algorithm [here](https://github.com/matthieugomez/EconPDEs.jl/blob/master/src/details.pdf). It is based on finite difference schemes, upwinding, and non linear time stepping.

To install the package
```julia
using Pkg
Pkg.add("EconPDEs")
```
# Solving  PDEs
The function `pdesolve` takes three arguments: (i) a function encoding the ode / pde (ii) a state grid corresponding to a discretized version of the state space (iii) an initial guess for the array(s) to solve for. 

For instance, to solve the PDE giving the price-dividend ratio in the Campbell Cochrane model:
<img src="img/by.png">

```julia
using EconPDEs
# define state grid
state = OrderedDict(:μ => range(-0.05, stop = 0.1, length = 1000))

# define initial guess
y0 = OrderedDict(:V => ones(1000))

# define pde function that specifies PDE to solve. The function takes two arguments:
# 1. state variable `state`, a named tuple. 
# The state can be accessed with `state.x` where `x` denotes the name of the state variable.
# 2. current solution `sol`, a named tuple. 
# The current solution at the current state can be accessed with `sol.y` where `y` denotes the name of initial guess. 
# Its derivative can be accessed with `sol.yx` where `x` denotes the name of state variable.
# Its second derivative can be accessed with `sol.yxx`,
#
# It returns two outputs
# 1. a tuple with the value of PDE at current solution and current state 
# 2. a tuple with drift of state variable, used for upwinding 
function f(state, sol)
	μbar = 0.018 ; ϑ = 0.00073 ; θμ = 0.252 ; νμ = 0.528 ; ρ = 0.025 ; ψ = 1.5 ; γ = 7.5
	Vt = 1 / sol.V - ρ + (1 - 1 / ψ) * (state.μ - 0.5 * γ * ϑ) + θμ * (μbar - state.μ) * sol.Vμ / sol.V + 0.5 * νμ^2 * ϑ * sol.Vμμ / sol.V + 0.5 * (1 / ψ - γ) / (1- 1 / ψ) * νμ^2 *  ϑ * sol.Vμ^2/sol.V^2
	(Vt,), (θμ * (μbar - state.μ),)
end

# solve PDE
pdesolve(f, state, y0)
```

More complicated ODEs / PDES (including PDE with two state variables or systems of multiple PDEs) can be found in the `examples` folder. 

The `examples` folder contains code to solve
- Asset Pricing Models
	- Campbell Cochrane (1999) and Wachter (2005) Habit Model
	- Bansal Yaron (2004) Long Run Risk Model
	- Garleanu Panageas (2015) Heterogeneous Agent Models
	- Di Tella (2017) Model of Balance Sheet Recessions
- Consumption Saving Models
    - Wang Wang Yang (2016) Optimal consumption and savings with stochastic income and recursive utility
    - Achdou Han Lasry Lions Moll (2018) Consumption Saving Problem with Stochastic Labor Income
- Investment Model
	- Bolton Chen Wang (2009) A Unified Theory of Tobin's q, Corporate Investment, Financing, and Risk Management

# Boundary Conditions
In case the volatility of the state variable is zero at the boundaries of the state space, there is no need for supplementary boundary conditions.

In all other cases, I assume that the derivative of the value function is zero at the boundaries. This is the right boundary condition if boundaries are reflecting. This is typically the case for models in which the state space is theorically unbounded.

In some rare cases, however, one may want to solve a PDE with different boundary conditions.  For an example specifying different boundary conditions, see the `BoltonChenWang` model. in the `examples` folder.
