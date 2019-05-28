# Implemented fluid models<div id="FluidModels"></div>
## Non-thermal Navier-Stokes equations
There exists a bunch of LB models which solve the dynamics of a compressible fluid but in which, for various reasons, the modeling of the energy equation is wrong. These models are sometimes used for small-Mach-number compressible flows with negligible temperature variations, but most often, they are simply used to solve the incompressible Navier-Stokes equations. In this case, the Mach number is kept low to damp unwanted compressibility effects, and the model is used as a so-called quasi-compressible solver.

Unless it is specified differently, all presented models can be used with the D2Q9, D3Q15, D3Q19, and D3Q27 lattices, using the corresponding descriptors `DdQqDescriptor`.

If you use a model with external force term, replace the lattice descriptor `DdQqDescriptor` by `ForcedDdQqDescriptor`.

It should be mentioned that among Palabos developers an uncertainty reigns over the entropic model (it might be fully valid on D2Q9 and D3Q27 only).

### Single-relaxation-time BGK
This is so to say the mother of all LB models: it is widely used, simple to implement and surprisingly versatile.

- Dynamics for plain BGK: BGKdynamics
- Dynamics with external force:	GuoExternalForceBGKdynamics, ShanChenExternalForceBGKdynamics, HeExternalForceBGKdynamics
- Reference: chen-98
- Example: `codesByTopic/navierStokesModels`

### Incompressible BGK
In standard BGK, the equilibrium term is multiplied by the fluid density. In the so-called incompressible model, the density only appears in the constant term (the one without velocity), and the velocity is defined to be proportional to momentum (it is not divided by rho). This model is computationally a little bit cheaper than BGK, because it avoids the division by rho for the computation of the velocity. Furthermore, the compressibility error of a simulation as compared to an exact solution of the incompressible Navier-Stokes equations is reduced for steady flows, as it scales like the cube of the Mach number, as opposed to the square Mach number in standard BGK. In unsteady flows, the error remains proportional to the square-Mach-number, though.

- Dynamics: IncBGKdynamics
- Reference: *missing*
- Example: `codesByTopic/navierStokesModels`

### Regularized BGK
Before the collision step, the particle populations are projected onto a Hermite base with polynomials up to second order. While this keeps the dynamics of the Navier-Stokes equations intact, it improves the numerical stability by imposing additional stability on the particle populations.

There exists a generic version for the regularized collision of type `CompositeDynamics`, which can be used to regularized any LB model. Additionally, a specific class is implemented for the regularized BGK model, for efficiency reasons.

- Generic Dynamics: RLBdynamics
- Dynamics for BGK: RegularizedBGKdynamics
- Reference: latt-06
- Example: `codesByTopic/navierStokesModels`

### Multiple-relaxation-time with linear stability optimization
In multiple-relaxation-time models, the linear collision operator is recast into the space of velocity moments, and their corresponding relaxation parameters are individually adjusted, either to modify the physics of the model, or to improve numerical stability. In the model implemented here, specific parameters are used for the non-physical ghost modes to improve the stability.

- Dynamics: MRTdynamics
- Descriptors: MRTD2Q9Descriptor, MRTD3Q19Descriptor
- Reference: dhumieres-02
- Example: `codesByTopic/navierStokesModels`

### Entropic model
In the absence of boundaries, this model is unconditionally stable because it guarantees the existence of an H-function which, just like in nature, is non-decreasing.

- Dynamics for plain model: EntropicDynamics
- Dynamics with external force: ForcedEntropicDynamics
- Reference: *missing*
- Example: `codesByTopic/navierStokesModels`

## Thermal flows with Boussinesq approximation
In the Boussinesq approximation, a fluid is modeled as incompressible, and the influence of the temperature is visible only through a body-force term, representing buoyancy effects. The temperature field obeys an advection-diffusion equation.

The coupling between the temperature field and the fluid solver is implemented by means of a data processor. For the fluid, you can use any of the non-thermal Navier-Stokes models with external force term.

### Advection-diffusion equation for the temperature
- Dynamics: AdvectionDiffusionBGKdynamics
- Descriptors: AdvectionDiffusionD2Q5Descriptor, AdvectionDiffusionD3Q7Descriptor
- Reference: guo-02c

### Coupling between fluid and temperature
- Data Processor: BoussinesqThermalProcessorXD
- Examples: `showCases/boussinesqThermal2D` and `showCases/boussinesqThermal3D`
- Reference: guo-02c

## Multi-component and multi-phase fluids
The widely used Shan/Chen models for immiscible multi-component fluids and for single-component multi-phase fluids are implemented in Palabos, in 2D and in 3D. Note that a lot of information about the theory and practical aspects of these models can be found in the book [Sukop-and-Thorne]<sup>[1](#myfootnote1)</sup>.

In both models, an inter-particle force term needs to be computed which is not entirely local. This force term is therefore computed by a data processor. This data processor needs to know the density and the velocity on a given lattice node (in the multi-component model, these macroscopic variables must be known for all components). To avoid computing these variables twice, as they need also to be known for the subsequent, normal collision step, they are written by the data processor to an external scalar of the cell, and then re-used by the dynamics object for the collision. It is therefore necessary to use a lattice descriptor which provides space for these external scalars, such as `ShanChenD2Q9Descriptor`, or `ForcedShanChenD2Q9Descriptor` if additionally to the inter-particle potential there is an external body force. In 3D, the same descriptors are available for the D3Q19 lattice (and other descriptors are easy to formulate). To be sure to read the macroscopic variables from external scalars during the collision step, use the classes `ExternalMomentBGKdynamics` or `ExternalMomentRegularizedBGKdynamics` for the fluid model.

Note that in both models, the effect of the external body force term is automatically accounted for if an external force is defined in the lattice descriptor. The force term is then added by the Shan/Chen data processor by means of a momentum correction to the flow velocity. Therefore, you don’t need to use a specific collision model with force term like `GuoExternalForceBGKdynamics`. A plain BGK dynamics, or regularized BGK dynamics, is fine.

Palabos also implements the 3D He/Lee multi-phase fluid model, based on a two populations. This model enforces incompressibility by solving a pressure evolution equation. While it is more complicated than the Shan/Chen model from an algorithmic standpoint, it has the ability to simulate larger density and viscosity differences between the two phases, and eliminates spurious velocities to a certain extent.

### Multi-component Shan/Chen model
In this model, each component is simulated on a separate block-lattice, and the components are coupled through a Shan/Chen data processor. You can couple as many components as you wish.

- Data Processor: ShanChenMultiComponentProcessorXD
- Examples: `showCases/multiComponent2d` and `showCases/multiComponent3d`
- Reference: shan-chen-93

### Single-component Shan/Chen multi-phase model
In this model, the Shan/Chen data processor takes an additional argument which defines the shape of the inter-particle potential. The following potentials are defined (see the file `src/multiPhysics/interparticlePotential.h` for details):

- PsiIsRho()                ![](http://latex.codecogs.com/gif.latex?\\Psi=\\rho
- PsiShanChen93(rho0)	    ![](http://latex.codecogs.com/gif.latex?\\Psi = \\rho_0\\left(1-e^{-\\rho/\\rho_0} \\right)
- PsiShanChen94(Psi0,rho0)	![](http://latex.codecogs.com/gif.latex?\\Psi = \\Psi_0\\, e^{-\\rho_0/\\rho}
- PsiQian95(rho0,g)	        ![](http://latex.codecogs.com/gif.latex?\\Psi = g\\,\\rho_0^2\\, \\rho^2 / (2(\\rho_0+\\rho)^2)

Please remark that in this model, the fluid density must be carefully adapted to the amplitude of the interaction force in order to enter the critical regime in which phase separation occurs.

- Data Processor: ShanChenSingleComponentProcessorXD
- Examples: `codesByTopic/shanChenMultiPhase`
- Reference: han-chen-93

## Free-Surface Flow
In a free-surface flow, the viscosity of one of the two phases is virtually infinite. This phase is therefore neglected, and a surface-tracking approach is applied to the remaining phase. Palabos uses a volume-of-fluid like approach to trace the free surface.

As an example, a dam break application is provided in `examples/showCases/breakingDam3d`.

## Large eddy simulations

### Static Smagorinsky model
In the Smagorinsky model, it is assumed that the subgrid scales have the effect of a viscosity correction which is proportional to the norm of the strain-rte tensor at the level of the filtered scales, *:math: nu = nu_0 + nu_T*. The formula for the turbulent viscosity correction nuT is

![](http://latex.codecogs.com/gif.latex?\\nu_T = C^2 \\left|S\\right|

where *C* is the Smagorinsky constant, and the tensor-norm of the strain rate is defined as ![](http://latex.codecogs.com/gif.latex?\\left|S\\right|=\\sqrt{S:S} (attention: there is no factor 1/2 inside the square-root, as it can be found in other definitions). The value of the Smagorinsky constant depends on the physics of the problem, and usually varies between 0.1 and 0.2 far from boundaries. This model is called static because the value of the Smagorinsky constant is imposed and does not change in time.

In the Palabos implementation, the strain-rate is computed from the stress tensor ![](http://latex.codecogs.com/gif.latex?\\Pi. It is remarked that the relationship between *S* and ![](http://latex.codecogs.com/gif.latex?\\Pi contains depends on the relaxation time ![](http://latex.codecogs.com/gif.latex?\\tau, and therefore on the viscosity nu. The formula for the turbulent viscosity nuT is therefore implicit, but is of second-order only and can therefore be solved explicitly.

Given that the stress tensor ![](http://latex.codecogs.com/gif.latex?\\Pi can be computed from local variables on a cell, the Smagorinsky model is entirely local and is implemented in Palabos through a dynamics class. As so often, there are several implementations for this model. The generic one, based on composite-dynamics objects, adds a Smagorinsky viscosity correction to any existing dynamics during runtime. The specific ones are written explicitly for the BGK and for the regularized BGK models, and are therefore somewhat more efficient. In each case, the value of the viscosity is modified before the execution of the usual collision step. Therefore, if during a simulation you access the relaxation parameter ![](http://latex.codecogs.com/gif.latex?\\omega at a cell, for example through a function call `lattice.get(x,y).getOmega()`, you get the relaxation parameter related to the effective viscosity ![](http://latex.codecogs.com/gif.latex?\\nu, and not the “molecular-scale viscosity” ![](http://latex.codecogs.com/gif.latex?\\nu_0.

The value of the Smagorinsky constant must be provided to these dynamics classes as a constructor argument. It is important to note that the dynamics object of each cell can have a different value of the Smagorinsky constant: this “constant” can be space dependent. This is useful for example to model boundary layers, where the Smagorinsky constant drops to zero. The most convenient way to create a simulation with space-dependent Smagorinsky constant is to assign the Smagorinsky dynamics to each cell of the lattice from within a data processor which is aware of the desired value of the *C* as a function of space.

- Generic Dynamics: SmagorinskyDynamics
- Dynamics for BGK: SmagorinskyBGKdynamics
- Dynamics for Regularized BGK: SmagorinskyRegularizedDynamics
- Reference: missing
- Example: `codesByTopic/smagorinskyModel`

## Non-Newtonian fluids
### Carreau model
In the Carreau model, the value of the viscosity is adjusted, just like in the Smagorinsky model, depending on the value of the local strain-rate. The constitutive equation is given by

![](http://latex.codecogs.com/gif.latex?\\nu=\\nu_0*(1+(\\lambda*\\left|S\\right|)^2)^{(n-1)/2}

where ![](http://latex.codecogs.com/gif.latex?\\lambda and *n* are given real parameters. As for the Smagorinsky model, the strain rate *S* is computed from the stress tensor ![](http://latex.codecogs.com/gif.latex?\\Pi, and the resulting equation is implicit because the *S* vs. ![](http://latex.codecogs.com/gif.latex?\\Pi relation is dependent on the relaxation time ![](http://latex.codecogs.com/gif.latex?\\tau. In the present case however, there is no explicit solution, and the equation is solved at each time step through a fixed-point iteration (which in this case converges very quickly and is therefore more efficient than a gradient-based solution method).

In this implementation, the parameters ![](http://latex.codecogs.com/gif.latex?\\nu_0, ![](http://latex.codecogs.com/gif.latex?\\lambda and *n* cannot be space-dependent, and they are set through a function call to the global singleton CarreauParameters. for example:

```C++
global::CarreauParameters().setNu0(nu0);
global::CarreauParameters().setLambda(1.);
global::CarreauParameters().setExponent(0.3);
```

- Dynamics:	 CarreauDynamics
- Reference: missing
- Example: `codesByTopic/nonNewtonian`

----------------
<a name="myfootnote">1</a>
[Sukop-and-Thorne]	Michael C. Sukop and Daniel T. Thorne (2006), Lattice Boltzmann Modeling; an Introduction for Geoscientists and Engineers. Springer-Verlag Berlin/Heidelberg.