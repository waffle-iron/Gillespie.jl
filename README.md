[![Stories in Ready](https://badge.waffle.io/sdwfrost/Gillespie.jl.png?label=ready&title=Ready)](https://waffle.io/sdwfrost/Gillespie.jl)
# Gillespie

[![Build Status](https://travis-ci.org/sdwfrost/Gillespie.jl.svg?branch=master)](https://travis-ci.org/sdwfrost/Gillespie.jl)
[![Coverage Status](https://coveralls.io/repos/github/sdwfrost/Gillespie.jl/badge.svg?branch=master)](https://coveralls.io/github/sdwfrost/Gillespie.jl?branch=master)
[![](https://img.shields.io/badge/docs-stable-blue.svg)](https://USER_NAME.github.io/PACKAGE_NAME.jl/stable)
[![status](http://joss.theoj.org/papers/3cfdd80b93a9123b173e9617c1e6a238/status.svg)](http://joss.theoj.org/papers/3cfdd80b93a9123b173e9617c1e6a238)

## Statement of need

This is an implementation of [Gillespie's direct method](http://en.wikipedia.org/wiki/Gillespie_algorithm) for performing stochastic simulations, which are widely used in many fields, including systems biology and epidemiology. It borrows the basic interface (although none of the code) from the R library [`GillespieSSA`](http://www.jstatsoft.org/v25/i12/paper) by Mario Pineda-Krch, although `Gillespie.jl` only implements the standard exact method at present, whereas `GillespieSSA` also includes tau-leaping, *etc.*. It is intended to offer performance on par with hand-coded C code; please file an issue if you find an example that is significantly slower (2 to 5 times) than C.

## Installation

```Gillespie.jl``` can be installed from the Julia REPL using the following command.

```julia
Pkg.add("Gillespie")
```

## Example usage

An example of a [susceptible-infected-recovered (SIR) epidemiological model](https://en.wikipedia.org/wiki/Compartmental_models_in_epidemiology#The_SIR_model_without_vital_dynamics) is as follows.

```julia
using Gillespie
using Gadfly

function F(x,parms)
  (S,I,R) = x
  (beta,gamma) = parms
  infection = beta*S*I
  recovery = gamma*I
  [infection,recovery]
end

x0 = [999,1,0]
nu = [[-1 1 0];[0 -1 1]]
parms = [0.1/1000.0,0.01]
tf = 250.0
srand(1234)

result = ssa(x0,F,nu,parms,tf)

data = ssa_data(result)

p=plot(data,
  layer(x="time",y="x1",Geom.step,Theme(default_color=color("red"))),
  layer(x="time",y="x2",Geom.step,Theme(default_color=color("blue"))),
  layer(x="time",y="x3",Geom.step,Theme(default_color=color("green"))),
  Guide.xlabel("Time"),
  Guide.ylabel("Number"),
  Guide.manual_color_key("Population",
                            ["S", "I", "R"],
                            ["red", "blue", "green"]),
  Guide.title("SIR epidemiological model"))
```

![SIR](https://github.com/sdwfrost/Gillespie.jl/blob/master/sir.png)

Julia versions of the examples used in [`GillespieSSA`](http://www.jstatsoft.org/v25/i12/paper) are given in the [examples](https://github.com/sdwfrost/Gillespie.jl/blob/master/examples) directory.

Passing functions as arguments in Julia (currently) incurs a performance penalty. One can circumvent this by passing an immutable object, with ```call``` overloaded, as follows.

```julia
immutable G; end
call(::Type{G},x,parms) = F(x,parms)
```

An example of this approach is given [here](https://github.com/sdwfrost/Gillespie.jl/blob/master/examples/sir2.jl).

## Benchmarks

The speed of the SIR model in `Gillespie.jl` was compared to:
- A version using the R package `GillespieSSA`
- Handcoded versions of the SIR model in Julia, R, and Rcpp

Benchmarks were run on a Mac Pro (Late 2013), with 3 Ghz 8-core Intel Xeon E3, 64GB 1866 Mhz RAM, running OSX v 10.11.3 (El Capitan), using Julia v0.4.5 and R v.3.3. Jupyter notebooks for [Julia](https://gist.github.com/sdwfrost/8a0e926a5e16d7d104bd2bc1a5f9ed0b) and [R](https://gist.github.com/sdwfrost/afed3b881ef5742623b905a539197c7a) are available as gists.
