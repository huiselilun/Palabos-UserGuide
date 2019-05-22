# Getting started with Palabos<div id="GettingStarted"></div>
## Supported Compilers
We don’t provide a detailed list of compilers and compiler version numbers which successfully compile Palabos. The bottom line, however, is that we have tested GCC, the Intel compiler, and the Portland Group compiler, which all work fine if you use recent versions.

## Installing and compiling the code under Linux and other Unix-like systems
The most recent version of Palabos can be downloaded at the address [www.palabos.org](www.palabos.org). It is now shown how to compile and run the software under Linux or other Unix-like environment. The library is exempt from external dependencies: all you need is a working compiler environment, including a modern C++ compiler (for example the free GCC compiler), the command make, and a reasonably recent version of the Python interpreter. To get started, download the tarball of the most recent version of Palabos, for example `plb-v1.1r0.tgz`, and move it to the directory in which you wish the code to reside (we will refer to this directory under the name of `$(PALABOS))`. In the following, the code is staying at this location, as there is currently no explicit installation process. Instead, compiled libraries are deposited in the directory `$(PALABOS)/lib/`. Unpack the code, using for example the command `tar xvfz plb-v1.1r0.tgz`.

The library Palabos makes use of an on-demand compilation process. The code is compiled the first time it is used by an end-user application, and then automatically re-used in future, until a new compilation is needed due to a modification of the code or compilation options. To see how this works, change into the directory of one of the example programs, such as `$(PALABOS)/examples/showCases/cylinder2D/`. Type `make` to compile both the `Palabos` library and the example program under standard conditions with the GCC compiler, and execute the program with the command `./cavity2D`. The program simulates the 2D flow around a cylinder; if the software `ImageMagick` is installed on your system, the program saves GIF images in the `tmp/` subdirectory, representing the velocity field at selected time steps.

To use a different compiler, compile for parallel execution, or modify other compilation options, edit the file Makefile in the local directory. The following options are particularly often modified:

| :------- | :---------- |
| debug:	|Turn on this flag to compile in debug mode. The resulting executable is a bit slower but easier to analyze in search for bugs. The recommended default behavior is to turn on debug mode.|
| MPIparallel:	| Turn on this flag to compile for parallel execution.|
| serialCXX:	| Specify the compiler to use for serial programs.|
| parallelCXX:	| Specify the compiler to use for parallel programs.|
| optimFlags:	| Specify the compiler options to use when the flag `optimize` is true.|
| compileFlags:	| Specify additional compiler options, which are for example specific to your hardware environment. Typical options are `-DPLB_BGP` or `-DPLB_MAC_OS_X` when compiling on a BlueGene or on a Mac respectively.|

If MPI is installed on your system (and if you have several cores or processors available), you can try to compile the code in parallel and execute it, say with two threads, through a command of the type `mpirun -np 2 ./cavity2d`. Note that the execution time of the example program is dominated by output operations. **To observe a significant speed improvement in the parallel program, the output operations need first to be commented in the source code.**

