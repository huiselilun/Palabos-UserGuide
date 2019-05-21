# Introduction<div id="Introduction"></div>
## What is Palabos?
The Palabos library is a framework for general-purpose computational fluid dynamics (CFD), with a kernel based on the lattice Boltzmann (LB) method. It is used both as a research and an engineering tool: its programming interface is straightforward and makes it possible to set up fluid flow simulations with relative ease, or, if you are knowledgeable of the lattice Boltzmann method, to extend the library with your own models.

The library’s native programmer interface in written in C++. It has practically no external dependencies (only Posix and MPI) and is therefore extremely easy to deploy on various platforms, including parallel systems like the BlueGene. Additional programmer intefaces are available for the Python and Java programming languages, which make it easier to rapidly prototype and develop CFD applications. There exists currently no graphical user interface; some amount of programming is therefore necessary in order to get an application running. Especially with the Python scripting interface, the developed code can however be pretty informal, and easy to produce with help of the provided examples.

The intended audience for Palabos covers scientists and engineers with a solid background in computational fluid dynamics, and potentially some knowledge in LB modeling. The software is freely available under the terms of an open-source AGPLv3 license, a copy of which is printed in the appendix of this user guide. It is distributed in the hope that it will promote research in the field of LB modeling, and help researchers concentrate on actual physical problems instead of staying stuck in tedious software development. Furthermore, implementing new LB models in Palabos offers a simple means of promoting new models and exchanging the information between research groups.

The Palabos library shows particularly outstanding performances when it comes to the field of high performance computing, or to the simulation of complex fluid flow. In the first case, close-to ideal scaling has been observed with Palabos on machines with tens of thousands of cores. A particularly striking feature here is that the preprocessing of the simulation is integrated into Palabos’ parallel pipeline. While other tools struggle to create a mesh for really large problems, Palabos uses all the available computational power to build its mesh itself, in a matter of seconds. As for complex fluids, the multi-physics framework of Palabos makes it possible to combine a vast range of physical model, and yet maintain the original parallel efficiency. Examples of complex tasks that have been executed with Palabos include for example the simulation of thermal, multi-phase flow through porous media, simulated with pore-scale accuracy.

A substantial amount of development time for Palabos went into formulating a general programming concept for LB simulation which offers an appropriate balance between generality, ease of use, and numerical efficiency. The difficulty of this task can be understood if one considers that the core ingredients of the LB method are guided by physical considerations rather than criteria from numerical analysis. As a consequence, it is straightforward to implement a LB code for a specific physical model, a rectangular-shaped numerical grid, and simple boundary conditions. However, any of the extensions of the code required for the implementation of practical problems in fluid engineering require careful considerations and a solid amount of programming efforts. It is frequent to find LB codes which implement one specific advanced feature (for example parallelism), but then find themselves in a too rigid shape to allow other extensions (for example coupling of two grids for multi-phase flows). To cope with this problem, the various ingredients of Palabos’ software architecture (physical model, underlying lattice, boundary condition, geometric domain, coupling between grids, parallelism) where largely developed as orthogonal concepts. It is for example possible to formulate a variant of the classical BGK collision model, say a MRT or entropic model, without awareness of the advanced software components of Palabos, and automatically obtain a parallel version of the code, or a multi-phase code based on the new model.

A central concept in Palabos are so-called “dynamics objects” which are associated to each fluid cell and determine the nature of the local collision step. Full locality of inter-particle collisions is a key ingredient of most LB models, a fact which is acknowledged in Palabos by promoting raw numerical efficiency for such models. Specific data structures are also available for algorithms that are not strictly local, as for example specific boundary conditions or inter-particle forces in multi-phase models. It is however assumed that these ingredients have a marginal impact on the overall efficiency pattern of the program. They are therefore implemented with reasonable efficiency, but are excluded from the low-level number-crunching strategy of Palabos in favor of higher-level programming constructs. In practice, we observe that through this approach we get a minial penalty on the program performance, but substantially improve the readability of end-user programs.

The grid structure of the simulations is based on a multi-block approach. Each block behaves like a rudimentary LB implementation, while advanced software ingredients such as parallelism and sparse domain implementations are covered through specific interface couplings between blocks.

The C++ code of Palabos makes a massive use of genericicity in its many facets. Basically, generic programming is intended to offer a single code that can serve many purposes. On one hand, the code implements dynamic genericity through by means of object-oriented mechanisms. An example of such a mechanism is provided by the lattices sites which change their behavior during program execution, to distinguish for example between bulk and boundary cells, or to modify the fluid viscosity or the value of a body force dynamically. On the other hand, C++ templates are used to achieve static genericity. As a result, it is sufficient to write a single generic code in 3D, which then runs on the D3Q15, the D3Q19, and the D3Q27 lattice.

## Functionality covered by Palabos
### Currently implemented
In short, the current release of Palabos covers the following ingredients:

Physics:
> Incompressible Navier-Stokes equations, weakly compressible, non-thermal Navier-Stokes equations, flows with body-force term, thermal flows with Boussinesq approximation, single-component multi-phase fluids (Shan/Chen model), multi-component multi-phase fluids (Shan/Chen model, He/Lee model), free surface flows (volume-of-fluid approach), static Smagorinsky model for fluid turbulence.

Basic fluid models:
> BGK (and its “incompressible” counterpart), a given MRT model (and its “incompressible” counterpart), regularized BGK, LW-ACM (and its “incompressible” counterpart), a given entropic model.

Straight-wall boundary conditions:
> Zou/He, Inamuro, Skordos, regularized BC, simple equilibrium, bounce-back, periodic. All boundary conditions work for straight walls with interior/exterior corners, and can be used to implement a Dirichlet or Neumann condition for the velocity or the pressure. The bounce-back condition is also used for curved boundaries, represented by a stair-case shape.

Off-lattice boundary conditions:
> GUO model, and generalized off-lattice boundary condition. Automatic, and massively parallel, voxelization of STL file and instantiation of off-lattice walls.

Particles:
> Massively parallel (billions of particles are no problem on a parallel machine) simulation of passive scalars, or interacting particles.

Grid:
> The implemented grids are D2Q9, D3Q13, D3Q15, D3Q19, and D3Q27. In all cases, the domain is either a regular matrix or a sparse domain, approximated by a multi-grid pattern.

Parallelism:
> All mentioned models and ingredients are parallelized with MPI for shared-memory and distributed-memory platforms, including I/O operations that are implemented in terms of MPI’s Parallel I/O API.

Pre-processing:
> The domain of a simulation can be constructed manually, or automatically from a corresponding STL-file.

Post-processing:
> The code has the ability to save the data in ASCII or binary files or to directly produce GIF images. Furthermore, the data can be saved in VTK format and further post-processed with an appropriate tool. For better efficiency, Palabos can natively post-process data, producing streamlines and iso-surfaces.

Check-pointing:
> At every moment, the state of the simulation can be saved, and loaded at a later point.

### Under development:
The following features have been repeatedly requested by the community and are currently developed:

* Grid refinement.
* Thermal flows using lattices with extended neighborhoods.

## Project
[The FlowKit Ltd. technology company](https://www.flowkit.com/) manages the development of the Palabos code. It carefully reviews each line of contributed code, asserts the quality of the implemented models and algorithms, and promotes industrial and engineering usage of the code. FlowKit Ltd. also provides consulting solutions to help you get started with the code, to implement new models, or to solve specific problems.

[The University of Geneva](http://www.unige.ch/) is the main research partner of the Palabos project. A pioneer in the domain of computational fluid dynamics and lattice Boltzmann modeling, the [Scientific and Parallel Computing Group SPC](http://spc.unige.ch/) at the University of Geneva provides the theoretical background of the Palabos kernel, and continuously develops new models and approaches in the field of complex fluid dynamics.

## Authors
Project supervision and core development are taken care of by Jonas Latt (FlowKit Ltd.+Universite de Geneve, Switzerland). Orestis Malaspinas (FlowKit Ltd.+Universite de Geneve, Switzerland) writes alternative LB collision terms (MRT, entropic), some of the boundary conditions, and coupled physical models (thermal, multi-phase). Free-Surface and multi-phase models are actively developed by Andrea Parmigiani (Universite de Geneve, Switzerland) and by Dimitrios Kontaxakis (FlowKit Ltd, Switzerland). Grid refinement is taken care of by Daniel Lagrava (Universite de Geneve, Switzerland). Many other people have contributed by writing code, thinking about programming concepts, or understanding some of the LB models for which the literature is sparse. We particularly mention, in alphabetical order, the contributions of Fokko Beekhof (Universite de Geneve, Switzerland), Bastien Chopard (Universite de Geneve, Switzerland), Mathias Krause (Rechenzentrum Karlsruhe, Germany), Switzerland), and Bernd Stahl (Universite de Geneve, Switzerland).

## Getting help
Installation instructions and a short overview of the code are provided in the section [Getting started with Palabos](GettingStarted.mc/#GettingStarted). Additionally to the user guide, the following resources are available as a support for Palabos users:

The tutorial:
> It is highly recommended, to get started, to work through Tutorial 1. The tutorial provides both an overview of high-level structures, and information on the way Palabos works internally.

The forum:
> The forum is the right place to post questions which are not answered in the user guide or the FAQ, to exchange with other Palabos users, or to communicate with the Palabos authors.

The automatic code documentation:
> This guide is automatically generated from the source code, using the program Doxygen . It has little didactic value but offers detailed insight into the class hierarchies and the syntax of the function calls.