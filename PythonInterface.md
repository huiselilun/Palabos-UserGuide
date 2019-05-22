# The Palabos-Python interface<div id="PythonInterface"></div>
Important: To get Palabos up and running, it is not necessary to compile the Palabos-Python interface. The Python interface is a useful extension which you will find useful to develop your Palabos applications more rapidly.

Since version 0.7, Palabos is provided with a Python interface which offers simple scripting access to the library. The look-and-feel of the Python interface is very different from the C++ interface, and a separate user’s guide will be provided at some point for this library. In the mean time, this section provides a few instructions for compiling and using the Python interface.

## Compiling the Python interface
For the Palabos-Python interface to work, you must install a list of libraries, and compile Palabos to create the interface. The required libraries are part of all major Linux distributions and are easily installed through the distribution’s package system. When your package manager offers plain and a `-dev` version of the libraries, you must install both of them. The required programs/libraries are:

* Swig
* NumPy
* SciPy
* Matplotlib
* Mpi4py (to install this, you may need the Python package installation system `easy_install`. On Ubuntu, you get this by installing python-setuptools. Then, type `sudo easy_install mpi4py`).

And, of course, you need Python. Optionally, you can install Mayavi2 for better graphics.

To compile the Palabos-Python interface, you need a working MPI installation and a C++ wrapper for MPI (this is usually done by installing either mpich or openmpi through the package manager). Then, follow these steps:

* Find out in which directory the source code of Palabos is located. Define an environment variable `PALABOS_ROOT` that holds this directory name. In a bash shell, you would for example type a command like `export PALABOS_ROOT=/home/joe/palabos-0.7v0/`.
* Change into the sub-directory `pythonic/src` of Palabos. Type `make`
* If the compilation is sucessful, change into the sub-directory `pythonic/examples` and type the name of one of the exmamples to execute it.

Note that the compilation is slow, and can take as much as half an hour. If the compilation fails with a “file-not-found” error message, this might be due to an improperly installed library. You can circumvent the problem by finding the directory in which the file in question is located, and add the directory to the include paths. For this, add the Makefile, and edit the line `includePaths=...` correspondingly.