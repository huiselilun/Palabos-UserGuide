# Getting started with Palabos<div id="GettingStarted"></div>
## Supported Compilers
We donot provide a detailed list of compilers and compiler version numbers which successfully compile Palabos. The bottom line, however, is that we have tested GCC, the Intel compiler, and the Portland Group compiler, which all work fine if you use recent versions.

## Installing and compiling the code under Linux and other Unix-like systems
The most recent version of Palabos can be downloaded at the address [www.palabos.org](www.palabos.org). It is now shown how to compile and run the software under Linux or other Unix-like environment. The library is exempt from external dependencies: all you need is a working compiler environment, including a modern C++ compiler (for example the free GCC compiler), the command make, and a reasonably recent version of the Python interpreter. To get started, download the tarball of the most recent version of Palabos, for example `plb-v1.1r0.tgz`, and move it to the directory in which you wish the code to reside (we will refer to this directory under the name of `$(PALABOS))`. In the following, the code is staying at this location, as there is currently no explicit installation process. Instead, compiled libraries are deposited in the directory `$(PALABOS)/lib/`. Unpack the code, using for example the command `tar xvfz plb-v1.1r0.tgz`.

The library Palabos makes use of an on-demand compilation process. The code is compiled the first time it is used by an end-user application, and then automatically re-used in future, until a new compilation is needed due to a modification of the code or compilation options. To see how this works, change into the directory of one of the example programs, such as `$(PALABOS)/examples/showCases/cylinder2D/`. Type `make` to compile both the `Palabos` library and the example program under standard conditions with the GCC compiler, and execute the program with the command `./cavity2D`. The program simulates the 2D flow around a cylinder; if the software `ImageMagick` is installed on your system, the program saves GIF images in the `tmp/` subdirectory, representing the velocity field at selected time steps.

To use a different compiler, compile for parallel execution, or modify other compilation options, edit the file Makefile in the local directory. The following options are particularly often modified:

|Options|Function |
|:------- |:---------- |
|debug:|Turn on this flag to compile in debug mode. The resulting executable is a bit slower but easier to analyze in search for bugs. The recommended default behavior is to turn on debug mode.|
|MPIparallel:|Turn on this flag to compile for parallel execution.|
|serialCXX:|Specify the compiler to use for serial programs.|
|parallelCXX:|Specify the compiler to use for parallel programs.|
|optimFlags:|Specify the compiler options to use when the flag `optimize` is true.|
|compileFlags:|Specify additional compiler options, which are for example specific to your hardware environment. Typical options are `-DPLB_BGP` or `-DPLB_MAC_OS_X` when compiling on a BlueGene or on a Mac respectively.|

If MPI is installed on your system (and if you have several cores or processors available), you can try to compile the code in parallel and execute it, say with two threads, through a command of the type `mpirun -np 2 ./cavity2d`. Note that the execution time of the example program is dominated by output operations. **To observe a significant speed improvement in the parallel program, the output operations need first to be commented in the source code.**

## Compilation under Mac OS X
Under Mac OS X, an easy way of getting GCC is to install xcode, after which the procedure is the same as under Linux. Alternatively, you can follow the same procedure as under Windows. **Important**: In either case, you must define the preprocessor macro PLB_MAC_OS_X (set the corresponding line in the Makefile to `compileFlags= -DPLB_MAC_OS_X`).

## Compilation under Windows (and under many other OSes) with the integrated development environment Code::Blocks
The C++ Integrated Development Environment (IDE) Code::Blocks is free, and you can download binaries for various flavors of Windows and Linux, and for Mac OS X (see [codeblocks](www.codeblocks.org)). A configuration file is delivered with the Palabos releases which can be used to compile Palabos with Code::Blocks. This approach was successfully tested under Windows and Linux, and neither Python nor `make` are needed for the compilation process.

Open the project file `$(PALABOS)/codeblocks/Palabos.cbp` under code::blocks, build and run the project. By default, the example program `examples/showCases/cavity2d.cpp` is compiled. To compile another example or your own program, remove the file `cavity2d.cpp` from the list of source files, and replace it by another one. The GIF images and other output files from the program are written in the directory `$(PALABOS)/codeblocks/tmp/`.

Code::Blocks is a cross-compiler environment, and you can in principle use it with just any C++ compiler. In particular, you can use it with the free GCC compiler. This compiler does not even need to be installed separately under Windows if you download the version of Code::Blocks which is directly packaged with the GCC port MinGW.

Note that the Palabos code does not compile with Visual C++, and we donâ€™t know why; any suggestion is appreciated. In the meantime, you should use another compiler, such as GCC, the Intel compiler, or the Portland Group compiler.

As mentioned above, the Palabos download provides a project file for the free IDE Code::Blocks, through which you can easily use any of these compilers (we have only tested GCC/MinGW, though). If you have no C++ compiler installed on your system (other than Visual C++), remember to download the version of Code::Blocks which is packaged with the GCC port MinGW, after which you can directly compile Palabos.

Just as under Linux, you need to install `ImageMagick` if you want Palabos to be able to produce GIF images. If this fails, Palabos will write images in the PPM format, which is not recognized by most of the standard image viewers under Windows. In this case, you have the possibility to install the free image editor `Gimp`, which can view PPM images and convert them to another format.

It should also be reminded that the pathnames in the Palabos examples are written using the Unix convention, with slashes between directory names, while Windows uses backslashes. Under Windows, it is therefore recommended to change lines like

```C++
global::directories().setOutputDir("./tmp/");
```

to something like

```C++
global::directories().setOutputDir(".\\tmp\\");
```

although the programs appear to behave reasonably well even when you donâ€™t do this.

## Compilation on a BlueGene/P
On the BlueGene/P, the compilation procedure is the same as on any Unix-like system. Just remember to define the preprocessor macro `PLB_BGP` (set the corresponding line in the Makefile to `compileFlags= -DPLB_BGP`). Also, note that on the BlueGene, the GCC compiler produces code that is almost as fast as (and sometimes even than) the dedicated XL compiler. Furthermore, GCC compiles much faster.

It must also be pointed out the BlueGene/P uses a Big-Endian representation of numerical values, as opposed to x86 architectures that use Little-Endian. You can therefore not export binary checkpoint files from the BlueGene to an x86 computer. BlueGene-generated VTK files on the other hand can be read on an Intel PC: just replace the string `LittleEndian` to `BigEndian` in your VTK file. You can for example to this with `sed`: `sed -i "s/LittleEndian/BigEndian/g" myOutputFile.vti`. As for the input STL meshes, it is simplest to provide them in an ASCII STL format, which is platform independent. If your mesh happens to be binary STL, you can convert it to ASCII using the program `toAsciiSTL` in the directory `utility/stl`.

## Open-source libraries which are bundled with Palabos
Palabos makes use of a few other open-source libraries. They doenâ€™t need to be explicitly intalled, though, because they are part of the Palabos releases:

[SConstruct](https://www.scons.org/) (Only tested with Linux):

> This Python-based library is used in Palabos to manage the compilation process.

[TinyXML](http://www.grinninglizard.com/tinyxml/) :

> This library is used to read structured user input in XML format.

## Recommended open-source software
Although the Palabos library is self-contained, it makes sense to combine it with other products, especially for the pre- and post-processing of data. A few recommended open-source programs are listed below:

[Paraview](www.paraview.org) (Linux / Mac OS X / Windows):

> Paraview offers a graphical user interface for the VTK library, and is an excellent tool for the visualization of scientific data. The results of Palabos simulations can be saved in a VTK format and then processed with Paraview. Paraview is found in the software repository of some Linux distributions.

[Octave](http://www.gnu.org/software/octave/) (Linux / Mac OS X / Windows):

> This command-line interpreter is very similar to Matlab. It is useful for processing ASCII data produced by Palabos, to create for example plots. Octave can be found in the software repository of most Linux distributions.

[ImageMagick](https://imagemagick.org/index.php) (Linux / Mac OS X / Windows):

> Without need for an external library, Palabos can produce snapshot images in a PPM format. If ImageMagick is installed, these pictures can be automatically converted into the more common GIF format. Furthermore, ImageMagick can be used to produce an animated GIF from a sequence of GIF images.

[Meshlab](http://www.meshlab.net/) (Linux / Mac Os X / Windows):

> Meshlab converts surface mesh files from many different formats into the STL format, which is required by Palabos to set up the geometry. Also, Meshlab is often able to correct mistakes in the STL file, such as, remove zero-area triangles or revert the surface normal.

## The examples directory
The examples directory is divided into the three sub-directories `showCases`, `codesByTopic`, and `tutorial`. While the examples in the directory `showCases` contain full-featured implementation of simple benchmark flows, the examples in directory `codesByTopic` contain more technical code to illustrate a specific programming concept in Palabos. The content of the directory `tutorial` is discussed separately in the Palabos tutorial.

The example programs are listed and commented in the appendix [Appendix: list of example programs](ListofExamplePrograms.md/#LoEP).