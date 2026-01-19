# WebFluid
This is a fluid simulation based on the document [Real-Time Fluid Dynamics for Games](https://graphics.cs.cmu.edu/nsp/course/15-464/Fall09/papers/StamFluidforGames.pdf) by Jos Stam.

The simulation works on a grid of pixels, each representing a 1/N x 1/N square in the square domain.
At each pixel, there is a scalar density, representing the concentration of a dye in that square, and there is a velocity vector, which represents the movement of the fluid.

At each step of the simulation, the source is added (both to the dye matrix and to the velocity matrix), the velocity 'diffuses', 'advects', and then 'projects'.
