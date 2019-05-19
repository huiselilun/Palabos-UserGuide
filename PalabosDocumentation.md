* Introduction
    - What is Palabos?
    - Functionality covered by Palabos
    - Project
    - Authors
    - Getting help
* Getting started with Palabos
    - Supported Compilers
    - Installing and compiling the code under Linux and other Unix-like systems
    - Compilation under Mac OS X
    - Compilation under Windows (and under many other OSes) with the integrated development environment Code::Blocks
    - Compilation on a BlueGene/P
    - Open-source libraries which are bundled with Palabos
    - Recommended open-source software
    - The examples directory
* The Palabos-Python interface
    - Compiling the Python interface
    - Example: Compiling the Python interface under Ubuntu Linux
* The Palabos-Java interface
    - Introduction
    - Installation
* Programming with Palabos
    - Quick overview: programming guidelines
    - Non-intrusive program development with Palabos
* Fundamental data types
    - The BlockXD data structures
    - Lattice descriptors
    - The dynamics classes
    - Data processors
* Implemented fluid models
    - Non-thermal Navier-Stokes equations
    - Thermal flows with Boussinesq approximation
    - Multi-component and multi-phase fluids
    - Free-Surface Flow
    - Large eddy simulations
    - Non-Newtonian fluids
* Setting up a problem
    - Attributing dynamics objects
    - Initial values of density and velocity
* Defining boundary conditions
    - Overview
    - Grid-aligned boundaries
    - Periodic boundaries
    - Bounce-back
    - Bounce-back domain from an STL file
    - Off-lattice boundaries from an STL file
* Running a simulation
    - Time cycles of a Palabos program
    - At which point do you evaluate data?
    - Other important things to do
* [Input/Output](Input-Output.md/#InputOutput)
    - Output streams: writing to the terminal and into files
    - Input streams: reading large data sets from files
    - Accessing command-line parameters
    - Reading user input from an XML file
    - Producing images in 2D and 3D simulations
    - VTK output for post-processing
    - Checkpointing: saving and loading the state of a simulation
* Data evaluation
    - Overview
    - Pipelining data evaluation operators
* Particles
    - Overview
* Grid refinement
    - Overview
    - Multi-layer grid refinement
* Parallelism
    - Parallel programming approach
    - Controlling the efficiency
* Data processors for non-local operations and couplings between blocks
    - Using helper functions to avoid explicitly writing data processors
    - Convenience wrappers for local operations
    - Writing data processors (or actually, data-processing functionals)
* Utilities
    - Timer
    - Value tracer
* Appendix: list of example programs
    - Directory examples/showCases
    - Directory examples/codesByTopic
* Appendix: partial function/class reference
    - Mutable (in-place) operations for simulation setup and other purposes
    - Non-mutable operations for data analysis and other purposes
* Appendix: Copyright and license agreements
    - Copyright of the Palabos user guide
    - Copyright and open-source license for Palabos
    - GNU AFFERO GENERAL PUBLIC LICENSE Version 3, 19 November 2007