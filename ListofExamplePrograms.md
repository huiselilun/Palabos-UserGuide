# Appendix: list of example programs<div id="LoEP"></div>
The directory `examples/tutorials` is treated separately in the Palabos tutorial.

Please note: the only purpose of these example codes is to teach the use of Palabos, and in many cases, we have not validated the results from a scientific standpoint. It is your responsibility to understand and possibly adapt the code if you use them in your research.

## Directory examples/showCases
This directory contains full-featured (but basic) programs for the simulation of well-known benchmark problems.

### aneurysm:
This is the most full-featured example, and is therefore not the best one to pick to get started with Palabos. It illustrates crucial concepts for advanced Palabos simulations:

* Reading a mesh in STL format, voxelizing the domain, and generating an off-lattice boundary condition.

* Copying the result of the simulation from a coarse to a finer grid. In this example, this technique is used to accelerate convergence to a steady state (converge first on the coarse mesh, and use this as an initial condition on the fine mesh).

* Reading user input parameters from an XML file.

* Various types of output, including surface data (wall-shear-stress).

### boussinesqThermal2d:
A fluid constrained between a hot bottom wall (no-slip for the velocity) and a cold top wall (no-slip for the velocity). The lateral walls are periodic. Under the influence of gravity, convection rolls are formed. Thermal effects are modeled by means of a Boussinesq approximation: the fluid is incompressible, and the influence of the temperature is visible only through a body-force term, representing buoyancy effects. The temperature field obeys an advection-diffusion equation.

The simulation is first created in a fully symmetric manner. The symmetry is therefore not spontaneously broken; while the temperature drops linearly between the hot and and cold wall, the convection rolls fail to appear at this point. In a second stage, a random noise is added to trigger the instability.

This application is technically a bit more advanced than the other ones, because it illustrates the concept of data processors. In the present case, they are used to create the initial condition, and to trigger the instability.

Techniques illustrated in this code:

* Coupling between a Navier-Stokes and an Advection-Diffusion simulation to simulate a thermal fluid with Boussinesq approximation.

* Use of value-tracer classes to decide when a simulation has reached a steady state.

### boussinesqThermal3d:
A fluid constrained between a hot bottom wall (no-slip for the velocity) and a cold top wall (no-slip for the velocity). The lateral walls are periodic. Under the influence of gravity, convection rolls are formed. Thermal effects are modeled by means of a Boussinesq approximation: the fluid is incompressible, and the influence of the temperature is visible only through a body-force term, representing buoyancy effects. The temperature field obeys an advection-diffusion equation.

The simulation is first created in a fully symmetric manner. The symmetry is therefore not spontaneously broken; while the temperature drops linearly between the hot and and cold wall, the convection rolls fail to appear at this point. In a second stage, a random noise is added to trigger the instability.

This application is technically a bit more advanced than the other ones, because it illustrates the concept of data processors. In the present case, they are used to create the initial condition, and to trigger the instability.

Techniques illustrated in this code:

* Coupling between a Navier-Stokes and an Advection-Diffusion simulation to simulate a thermal fluid with Boussinesq approximation.

* Use of value-tracer classes to decide when a simulation has reached a steady state.

### breakingDam3d:
In this example, a free-surface flow is implemented that simulates a dam break problem, with a down-stream collision of the water against an obstacle. The example illustrates:

* How to set up a free surface flow problem (two-phase problem in which the effect of one phase is neglected).

* How to describe the geometry of the flow in the free-surface case.

* How to export data, and in particular, how to write an STL file describing the shape of the free surface.

### cavity2d:
Flow in a lid-driven 2D cavity. The cavity is square and has no-slip walls, except for the top wall which is driven to the right with a constant velocity. The benchmark is challenging because of the velocity discontinuities on corner nodes. The code on the other hand is very simple. It could for example be used as a first example, to get familiar with Palabos.

Techniques illustrated in this code:

* Use of timer object to benchmark the code.

* Data analysis: computation of velocity, velocity-norm, and vorticity.

* Creation of GIF images and VTK files from 2D simulation data.

### cavity3d:
Flow in a diagonally lid-driven 3D cavity. In this 3D analog of the 2D cavity, the top-lid is driven with constant velocity in a direction parallel to one of the two diagonals. The benchmark is challenging because of the velocity discontinuities on corner nodes. The code on the other hand is very simple. It could for example be used as a first 3D example, to get familiar with Palabos.

Techniques illustrated in this code:

* Use of timer object to benchmark the code.

* Data analysis: computation of velocity, velocity-norm, and vorticity.

### collidingBubbles3d:
Collision between two bubbles projected against each other, in 3D. The application makes use of the He/Lee multi-phase fluid model.

Techniques illustrated in this code:

* Coupling between lattices and various scalar- and tensor-fields, with the He/Lee algorithm for multi-phase fluids.

* Straightforward initialization of scalar- and tensor-field values through data processing functionals.

### cylinder2d:
Flow around a 2D cylinder inside a channel, with the creation of a von Karman vortex street. This example makes use of bounce-back nodes to describe the shape of the cylinder. The outlet is modeled through a Neumann (zero velocity-gradient) condition.

Techniques illustrated in this code:

* Use of a domain functional to instantiate bounce-back nodes on specific locations, from an analytical prescription.

* Instantiation of a Neumann velocity condition for the outlet.

* Accessing the internal statistics to compute quantities like the average kinetic energy and the average density without computational effort.

### multiComponent2d:
Rayleigh-Taylor instability occurring with a heavy fluid residing initially on top of a light fluid, in 2D. The application makes use of the Shan/Chen multi-component model for multi-phase fluids.

Techniques illustrated in this code:

* Coupling between two Navier-Stokes lattices with the Shan/Chen algorithm to simulate an immiscible multi-component fluid.

* Simple creation of space-dependent boundary conditions with a one-cell indexed functional.

### multiComponent3d:
Rayleigh-Taylor instability occurring with a heavy fluid residing initially on top of a light fluid, in 3D. The application makes use of the Shan/Chen multi-component model for multi-phase fluids.

Techniques illustrated in this code:

* Coupling between two Navier-Stokes lattices with the Shan/Chen algorithm to simulate an immiscible multi-component fluid.

* Simple creation of space-dependent boundary conditions with a one-cell indexed functional.

### poiseuille:
Implementation of a stationary, pressure-driven 2D channel flow, and comparison with the analytical Poiseuille profile. The velocity is initialized to zero, and converges only slowly to the expected parabola. This application illustrates a full production cycle in a CFD application, ranging from the creation of a geometry and definition of boundary conditions over the program execution to the evaluation of results and production of instantaneous graphical snapshots. From a technical standpoint, this showcase is not trivial: it implements for example hybrid velocity/pressure boundaries, and uses an analytical profile to set up the boundary and initial conditions, and to compute the error. As a first Palabos example, you might prefer to look at a more straightforward code, such as cavity2d.

Techniques illustrated in this code:

* Definition of functionals (or function objects) to create space-dependent initial and boundary conditions.

* Comparison of the computed velocity field with an analytical formula, and computation the root-mean-square error.

* Definition of velocity or pressure boundaries, or hybrid cases using a velocity condition on some parts of the boundary, and a pressure condition on other parts.

### rectangularChannel3d:
Extension of the Poiseuille flow to 3D, in a channel with rectangular cross section. This flow is stationary, and the numerical result is compared to an analytical formula.

### womersley:
Implementation of a pulsatile, force-driven 2D channel flow, and comparison with the analytical Womersley profile. The external force varies in time as a simple cosine; the resulting velocity profile is far from trivial, and is described by the Womersley solution.

Techniques illustrated in this code:

* Usage of an external force field (which can be space-dependent; in this example it is however space-independent, but time dependent).

* Implementation of periodic boundaries in one direction.

## Directory examples/codesByTopic
### particlesInTube
3D steady flow simulation in a tube. Passive scalar particles are injected, and written into an ASCII file for visualization.

### bounceBack
Techniques illustrated in this directory:

* `computeDrag.cpp`: Computation of drag and lift forces acting on a bounce-back obstacle, by evaluating the momentum exchange.

### boundaryCondition
Techniques illustrated in this directory:

* `neumannOutlets.cpp`: Two different ways of implementing a Neumann-type outlet. (1) Obatin a zero velocity-gradient by copying the velocity from the previous site and use it for the Dirichlet boundary condition. (2) Copy all unknown particle populations from the previous site.

### dataAnalysis
Techniques illustrated in this directory:

* `cavity3d.cpp`: Compare (efficient) internal statistics with manually computed values and conclude that they are equal.

### dotList
* `cylinder2d.cpp`: Use a data processor based on a dot-list instead of the usual boxed data-processor to create a complex domain.

### couplings
Techniques illustrated in this directory:

* `coupleVelocityField.cpp`: Write a data processor to create a coupling between a scalar-field and a block-lattice. In this example, the values of the scalar-field are used to setup the boundary condition in the block-lattice.

### include
This directory aggregates common functionalists used by various example codes.

### io
Techniques illustrated in this directory:

* `checkPointing.cpp`: Save the state of the simulation to proceed at a later time, after an interruption of the program.

* `loadGeometry.cpp`: Read a boolean mask from an ASCII file and set up bounce-back nodes, based on the values of this bool-mask. Such an approach is useful for complex geometries, such as for example porous media.

* `useIOstream.cpp`: Using output stream to write simulation results in a formatted way into files or to the terminal.

* `userInput.cpp`: Getting input parameters from the command line or from an input file.

### multiBlock
Techniques illustrated in this directory:

* `manualBlockCavity2d.cpp`: Use all parameters to the multi-block-lattice constructor to create the distribution of blocks explicitly.

* `manualBlockCavity3d.cpp`: Use all parameters to the multi-block-lattice constructor to create the distribution of blocks explicitly.

* `manualBlockPoiseuille2d.cpp`: Use all parameters to the multi-block-lattice constructor to create the distribution of blocks explicitly.

### navierStokesModels
Techniques illustrated in this directory:

* `allModels2d.cpp`: Lists all models available for the incompressible Navier-Stokes equations: BGK, regularized, MRT, entropic.

### nonNewtonian
Techniques illustrated in this directory:

* `carreauPoiseuille.cpp`: Use the non-Newtonian Carreau model in a 2D channel.

### scalarField
Techniques illustrated in this directory:

* `scalarField2d.cpp`: Perform array-based operations on 2D scalar-fields. In this example, the values of the scalar-field are initialized to a space-dependent sine wave.

* `scalarField3d.cpp`: Perform array-based operations on 3D scalar-fields. In this example, the values of the scalar-field are initialized to a space-dependent sine wave.

### smagorinskyModel
Techniques illustrated in this directory:

* `smagorinskyCavity3D.cpp`: LES simulation in a lid-driven cavity with a static Smaogrinsky model. Note that the result is not trustworthy, because the boundary layers are not modeled appropriately.