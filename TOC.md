# Table of Content
* [Introduction](Introduction.md/#Introduction)
    - What is Palabos?
    - Functionality covered by Palabos
    - Project
    - Authors
    - Getting help
* [Getting started with Palabos](GettingStarted.md/#GettingStarted)
    - Supported Compilers
    - Installing and compiling the code under Linux and other Unix-like systems
    - Compilation under Mac OS X
    - Compilation under Windows (and under many other OSes) with the integrated development environment Code::Blocks
    - Compilation on a BlueGene/P
    - Open-source libraries which are bundled with Palabos
    - Recommended open-source software
    - The examples directory
* [The Palabos-Python interface](PythonInterface.md/#PythonInterface)
    - Compiling the Python interface
    - Example: Compiling the Python interface under Ubuntu Linux
* [The Palabos-Java interface](JavaInterface.md/#JavaInterface)
    - Introduction
    - Installation
* [Programming with Palabos](Programming.md/#Programming)
    - Quick overview: programming guidelines
    - Non-intrusive program development with Palabos
* [Fundamental data types](FundamentalDataTypes.md/#FundamentalDataTypes)
    - The BlockXD data structures
    - Lattice descriptors
    - The dynamics classes
    - Data processors
* [Implemented fluid models](FluidModels.md/#FluidModels)
    - Non-thermal Navier-Stokes equations
    - Thermal flows with Boussinesq approximation
    - Multi-component and multi-phase fluids
    - Free-Surface Flow
    - Large eddy simulations
    - Non-Newtonian fluids
* [Setting up a problem](SettingUp.md/#SettingUp)
    - Attributing dynamics objects
    - Initial values of density and velocity
* [Defining boundary conditions](DefiningBoundaryConditions.md/#DefiningBoundaryConditions)
    - Overview
    - Grid-aligned boundaries
    - Periodic boundaries
    - Bounce-back
    - Bounce-back domain from an STL file
    - Off-lattice boundaries from an STL file
* [Running a simulation](RunningASimulation.md/#RunningASimulation)
    - Time cycles of a Palabos program
    - At which point do you evaluate data?
    - Other important things to do
* [Input/Output](InputOutput.md/#InputOutput)
    - Output streams: writing to the terminal and into files
    - Input streams: reading large data sets from files
    - Accessing command-line parameters
    - Reading user input from an XML file
    - Producing images in 2D and 3D simulations
    - VTK output for post-processing
    - Checkpointing: saving and loading the state of a simulation
* [Data evaluation](DataEvaluation.md/#DataEvaluation)
    - Overview
    - Pipelining data evaluation operators
* [Particles](Particles.md/#Particles)
    - Overview
* [Grid refinement](GridRefinement.md/#GridRefinement)
    - Overview
    - Multi-layer grid refinement
* [Parallelism](Parallelism.md/#Parallelism)
    - Parallel programming approach
    - Controlling the efficiency
* [Data processors for non-local operations and couplings between blocks](DataProcessors.md/#DataProcessors)
    - Using helper functions to avoid explicitly writing data processors
    - Convenience wrappers for local operations
    - Writing data processors (or actually, data-processing functionals)
* [Utilities](Utilities.md/#Utilities)
    - Timer
    - Value tracer
* [Appendix: list of example programs](ListofExamplePrograms.md/#LoEP)
    - Directory examples/showCases
    - Directory examples/codesByTopic
* [Appendix: partial function/class reference](PartialFunctionClassReference.md/#PFCR)
    - Mutable (in-place) operations for simulation setup and other purposes
    - Non-mutable operations for data analysis and other purposes
* [Appendix: Copyright and license agreements](CopyrightLicenseAgreements.md/#CLA)
    - Copyright of the Palabos user guide
    - Copyright and open-source license for Palabos
    - GNU AFFERO GENERAL PUBLIC LICENSE Version 3, 19 November 2007

