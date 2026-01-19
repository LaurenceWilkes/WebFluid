# WebFluid
This is a 2D semi-Lagrangian fluid simulation based on the document [Real-Time Fluid Dynamics for Games](https://graphics.cs.cmu.edu/nsp/course/15-464/Fall09/papers/StamFluidforGames.pdf) by Jos Stam.

## Overview
The simulation works on a grid of pixels, each representing a $`1/N \times 1/N`$ square in the square domain.
Each pixel stores a scalar density, representing the concentration of a dye in that square, and a velocity vector, which represents the movement of the fluid.

At each step of the simulation, the source is added (both to the dye matrix and to the velocity matrix), the velocity _diffuses_, _projects_, _advects_, and then _projects_ again, and the density _diffuses_ and then _advects_.

The simulation works as a fractional step method where Navier&ndash;Stokes is split with a first-order splitting; the local error is $`O(\Delta t^2)`$ per step.

### Diffusion
The diffusion step is a finite difference version of solving the equation 
```math
    \frac{\partial \boldsymbol{x}}{\partial t} = \nu \nabla^2 \boldsymbol{x}
    .
```
The time derivative is represented by
```js
(x[i, j] - x0[i, j]) / dt
```
at each point $`(i, j)`$ and the Laplacian is discretised using the 5-point stencil 
```js
N * N * (x[i-1, j] + x[i+1, j] + x[i, j-1] + x[i, j+1] - 4 * x[i, j])
```

The update rule for the quantity `x` is a stable method whereby the values of `x[i, j]` are chosen which, when diffused backwards, yield the initial `x0[i, j]`.
This produces a linear system which is solved with Gauss&ndash;Seidel.

### Advection
The advection step solves the advection equation
```math
    \frac{\partial x}{\partial t} + \boldsymbol{u} \cdot \nabla x = 0
    ,
```
which in turn implies that 
```math
    \frac{\mathrm{d}}{\mathrm{d} t} \tilde{x}(q(t), t) = 0
    ,
```
where $`\tilde{x}(q, t)`$ represents the quantity $`x(q)`$ at time $`t`$ and position $`q`$, and $`q(t)`$ represents the position of a "particle" at time $`t`$.
Here, $`q(t)`$ satisfies $`\dot{q}(t) = \boldsymbol{u}(q(t), t)`$.
In essence, we assume that the quantity represented by $`x`$ (either dye density or a component of the velocity in our case) is transported along the path of the velocity. 
This is once again implemented by backtracing: the initial location of a particle arriving at `x[i,j]` is computed, and `x[i,j]` is interpolated from the four nearest values of `x0`.

### Projection
Finally, the projection step done to the velocity field in-between and after the diffusion and advection enforces the incompressibility constraint:
```math
    \nabla \cdot \boldsymbol{u} = 0
    .
``` 

This is done by producing a value of $`\boldsymbol{u}`$ which is divergence free.
We decompose 
```math
    \boldsymbol{u} = \boldsymbol{u}^* + \nabla p
``` 
where $`\boldsymbol{u}^* `$ is divergence free, and enforce
```math
    \boldsymbol{0} 
    = \nabla \cdot \boldsymbol{u}^*
    = \nabla \cdot (\boldsymbol{u} - \nabla p) 
    = \nabla \cdot \boldsymbol{u} - \nabla^2 p
    .
``` 
The value of $`p`$ is selected to solve the Poisson equation 
```math
    \nabla^2 p = \nabla \cdot \boldsymbol{u}
``` 
and the velocity field is corrected
```math
    \boldsymbol{u}^* = \boldsymbol{u} - \nabla p
    .
``` 

In practice, the field $`\nabla \cdot \boldsymbol{u} `$ is approximated by
```js
(u[i+1, j] - u[i-1, j] + v[i, j+1] - v[i, j-1]) * N / 2
```
where `u` is the horizontal component of the velocity and `v` is the vertical.
Then the backtracking method with Gauss&ndash;Seidel is used to solve the Poisson equation and the gradient field of the solution is used to correct the velocity.



