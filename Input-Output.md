<div id="InputOutput"></div># Input/Output
## Output streams: writing to the terminal and into files
In Palabos, never use the usual objects `cout`, `cerr`, or `clog` , or even worse, the ignominious `printf` to print anything to the terminal. These symbols are replaced by the corresponding parallelizable versions `pcout`, `pcerr`, and `pclog`. They have the same syntax and the same behavior as their traditional counterparts, but additionally, they offer the expected behavior in a parallel program: if you print a message from a program parallelized on a hundred cores, the message is printed only once, and not a hundred times. You can use them to print strings, numbers, or even extracts from a lattice, scalar-field, or tensor-field to the screen:

```C++
pcout << "The average energy is " << computeAverageEnergy(lattice) << endl;
pcout << "And here are some values of the energy, with 10-digit accuracy:" << endl;
pcout << setprecision(10)
      << *computeEnergy(lattice, Box2D(0,10, 0,10)) << endl;
```

In the same way, data can be streamed into an ASCII file (an example for the usage of output file-streams is also provided in the file `examples/codesByTopic/io/useIOstream.cpp`). In this case, the standard `ofstream` is replaced by an `plb_ofstream` as follows:

```C++
plb_ofstream ofile("energy.dat");
ofile << setprecision(10) << *computeEnergy(lattice) << endl;
```

This can be useful to subsequently load the data into an environment for interactive data analysis like Octave or Matlab:

```Matlab
% Analysis of the data in Octave
% Assign manually the dimensions:
nx = 100;
ny = 100;
load energy.dat;
energy = reshape(energy, nx, ny);
imagesc(energy);
```

## Input streams: reading large data sets from files
Input files are handled in the same way as output files in the previous section, through objects of type `plb_ifstream`. The file `energy.dat` written in the previous code can for example be streamed back into a matrix through the command:

```C++
plb_ifstream ifile("energy.dat");
if (ifile.is_open()) {
    // It is your responsibility to create the matrix with the right dimensions
    //    nx-by-ny, compatible with the size of the data in "energy.dat".
    MultiScalarField2D<T> energy(nx,ny);
    ifile >> energy;
}
```

An important point to remember is that the scalar-field into which the data is streamed must be first constructed manually with the right dimensions, in order to avoid memory corruption.

This approach works however only if the data is streamed into a Palabos object. If you wish to stream data from a file into plain C++ data types, for example when accessing user-defined parameters, you may not use the `plb_ifstream` data type as illustrated above, because it leads to an unexpected behavior in parallel programs. In this case, there exist two possible workarounds. The first, which is the recommended approach, is to require that the user creates parameter files in an XML format, which then are parsed using Palabos’ automatic XML parsing facilities, as explained below in sections `input-from-xml`. Doing so has many advantages, as it leads to short, well readable code, with automatic type conversion of the user input, and automatic error handling in case of erroneous input data. Furthermore, parameter files in XML format are well structured and to a large extent self-documenting.

If for some reason you are unwilling to use XML files for user input, you can still use plain `plb_ifstream` files, but you need to manually broadcast the data to all processors in order to get a working parallel program which is compatible with the data-parallel programming concept of Palabos. To illustrate this, let us consider a situation in which the dimensions `nx` and `ny` of the matrix are written at the beginning of the file `energy.dat` and are read into the program in order to construct the scalar-field automatically with the right dimensions:

```C++
plb_ifstream ifile("energy.dat");
if (ifile.is_open()) {
    plint nx, ny;
    ifile >> nx >> ny;
    // Broadcast the input data to all processors; otherwise, the behavior
    //   of the program is undefined.
    global::mpi().bCast(&nx, 1);
    global::mpi().bCast(&ny, 1);
    MultiScalarField2D<T> energy(nx,ny);
    ifile >> energy;
}
```

This way of handling input values is illustrated in the code `examples/codesByTopic/io/manualUserInput.cpp`. Please remark that the MPI-related code also compiles in non-parallel programs, in which it simply has no effect. It is therefore good to write them in all cases, to guarantee that the program works in parallel without modification.

There is no object `xcin` for interactive input in Palabos; user input must be received either through an input file or directly from the command line.

## Accessing command-line parameters
As in usual C++ programs, command-line parameters can be accessed through the `argc` and `argv` parameters to the main function:

```C++
int main(int argc, char* argv[])
```

This works fine in both serial and parallel programs. Additionally to this, Palabos offers two global objects of name `argc` and `argv` through which the command-line parameters can be accessed from any location in the program. It is generally recommended to use these global objects instead of the `main` parameters, because they are more intelligent, with automatic type conversions and error handling. Their usage is illustrated in the following code:

```C++
int main(int argc, char* argv[]) {
    // Never forget to initialize Palabos, in every program.
    plbInit(&argc, &argv);

    pcout << "The number of input arguments is: "
          << global::argc() << endl;

    double double_argument;
    plint  plint_argument;
    string string_argument;

    try {
        global::argv(1).read(double_argument);
        global::argv(2).read(plint_argument);
        global::argv(3).read(string_argument);
    }
    // React to errors, either if there are less than 3 arguments,
    //   or if one of the arguments fails to convert to the expected
    //   data type.
    catch(PlbIOException& exception) {
        // Print the corresponding error message.
        pcout << exception.what() << endl;
        // Terminate the program.
        exit(1);
    }

    // ... Execute the main program ...
}
```

In this example, the C++ exception mechanism is used to react to errors in the user input. If you prefer not to handle exceptions, you can use the `noThrow` version of the `read` function:

```C++
bool ok = global::argv(1).readNoThrow(double_argument);
if (!ok) {
    pcout << "Error while reading argument 1." << endl;
}
```

As an example for the use of command-line parameters, you can also have a look at the programs `examples/showCases/rayleighTaylor2D.cpp` and `rayleighTaylor3D.cpp`.

## Reading user input from an XML file

Palabos offers a way to read structured input data from an XML file. This functionality works both in serial and parallel programs. The data is however stored in plain, non-parallel data structures, which are duplicated over all threads of a parallel program. XML files should therefore be used only for small data sets, such as, the parameters needed to set up a simulation. Large data sets, such as, the content of a lattice, should instead be read into a Palabos data structures, as explained in section `input-streams`, so the data is distributed over the parallel machine.

The XML data is parsed with help of the open-source library `TinyXML`. This library needs not be installed manually, as it is packaged with the Palabos releases.

Palabos can read any well formed XML document, with three restrictions:

> * Document Type Definitions (DTDs) or the eXtensible Stylesheet Language (XSL) cannot be parsed.
> * Attributes are not recognized (but a tag is still properly parsed, even if attributes are present). For example, the value of the attribute `id` in the following XML tab cannot be accessed in Palabos: `<someTag id="5">`.
> * A given tag name can be used only once. If it is used multiple times, only the last occurrence will be accessed in Palabos.

These restrictions are made to simplify the syntax of the XML parser in Palabos, and because they don’t restrict the generality of the input format. The following is a typical input file which could be used with Palabos:

XML file `myInput.xml`:

```XML
<?xml version="1.0" ?>
<!-- Configuration parameters for my simulation -->

<Geometry>
    <inputFile> /home/user/data/inputFile.dat </inputFile>
    <size>
    <!-- Number of lattice nodes in each space direction. -->
        <nx> 10 </nx>
        <ny> 20 </ny>
        <nz> 30 </nz>
    </size>
    <viscosity>
    <!-- Viscosity in lattice units. -->
        0.023
    </viscosity>
    <umax>
    <!-- Maximum velocity in lattice units. -->
        0.01
    </umax>
</Geometry>

<Inlet>
    <!-- Use a velocity Dirichlet boundary condition. -->
    <kind> Dirichlet </kind>
    <!-- Velocity profile values on the inlet, in
         dimensionless units. -->
    <values> 0 0.1 0.2 0.3 0.4 0.4 0.3 0.2 0.1 0 </values>
</Inlet>
```

And here’s how they would be parsed in a Palabos program:

```C++
try {
    // Open the XMl file.
    XMLreader xmlFile("myInput.xml");

    // It's good policy to flush the content of the XML
    //   file to the terminal, so that in future, when
    //   you check the program's output, you remember
    //   which parameters where used.
    pcout << "CONFIGURATION" << endl;
    pcout << "=============" << endl << endl;
    xmlFile.print(0);

    string inputFile;
    xmlFile["Geometry"]["inputFile"].read(inputFile);

    plint nx, ny, nz;
    xmlFile["Geometry"]["size"]["nx"].read(nx);
    xmlFile["Geometry"]["size"]["ny"].read(ny);
    xmlFile["Geometry"]["size"]["nz"].read(nz);

    T viscosity, uMax;
    xmlFile["Geometry"]["viscosity"].read(viscosity);
    xmlFile["Geometry"]["umax"].read(uMax);

    string inletKind;
    xmlFile["Inlet"]["kind"].read(inletKind);
    vector<T> inletValues;
    xmlFile["Inlet"]["values"].read(inletValues);
}
catch (PlbIOException& exception) {
    pcout << exception.what() << endl;
    return -1;
}
```

It is particularly noted that one can read full data arrays from a single tag, as shown with the the last tag in the above example, which is read into the vector `inletValues`.

## Producing images in 2D and 3D simulations

It is possible to produce images from the entire domain, a sub-domain, or a slice of a scalar-field. By default, Palabos writes images in the PPM format, an ASCII format which is easily produced without the need for an external graphics library. If the package `ImageMagick` (and in particular, the command-line tool `convert` contained in this package) is installed, it is also possible to write images in the more standard GIF format. This works as well under Linux as under Windows. Most examples in the directory `examples/showCases` illustrate how to write GIF images.

As a first step, you need to create an object of type `ImageWriter`, as in the following example:

```C++
ImageWriter<T> imageWriter("leeloo");
```

The parameter string stands for the colormap used to map the scalar variables to a three-component RGB color scheme. The available default maps are `earth`, `water`, `air`, `fire`, and `leeloo`.

The next step is to write a PPM or GIF image, either with a colormap adjusted to a fixed range of color values, or with a scaled range which is automatically adjusted to fit the minimum and maximum value of the data. With the GIF format, the image can also be rescaled to a different size:

```C++
// Write a PPM image with a color map adapted to the range [minValue, maxValue].
imageWriter.writePpm(scalarField, "myImage", minValue, maxValue);

// Write a PPM image with an automatically scaled color map.
imageWriter.writeScaledPpm(scalarField, "myImage");

// Write a GIF image with a color map adapted to the range [minValue, maxValue].
imageWriter.writeGif(scalarField, "myImage", minValue, maxValue);

// Write a GIF image with an automatically scaled color map.
imageWriter.writeScaledGif(scalarField, "myImage");

// Write a GIF image with an automatically scaled color map, and rescale it in
//  order to fit in a siyeX-by-sizeY box (the aspect ratio is preserved, though).
imageWriter.writeScaledGif(scalarField, "myImage", sizeX, sizeY);
```

It is also extremely useful to use this approach to produce snapshots from a 3D simulation. In this case, you need to extract a slice from the 3D data, in form of a 3D Box with one degenerate dimension (a dimension which has a one-cell-width):

```C++
Box3D slice(0,nx-1, 0,ny-1, zValue, zValue);
ImageWriter<T>("earth").writeScaledGif(*computeVelocity(lattice, slice));
```

## VTK output for post-processing

All data from a block-lattice, a scalar-field, or a tensor-field can be written into a file in a `VTK` data format. From there, it can be further post-processed by a scientific data visualization tool such as `Paraview`.

The VTK format used in Palabos is based on a lossless binary representation of the data by means of the Base64 format (it’s an ASCII-based binary format, the same which is probably used by your e-mail program to encode images in the e-mail). It is often an overkill to use a format which preserves the full numerical accuracy of your simulated data, because post-processing operations often require less numerical precision than CFD simulations. The least you can do to save some memory is to convert the data from double-precision do single-precision arithmetics, as shown in the following 3D example:

```C++
// Evaluate the discretization parameters, in order to write the data
//    in dimensionless units into the VTK file.
T dx = parameters.getDeltaX();
T dt = parameters.getDeltaT();

// Create a VTK data object and indicate the cell width through the
//   parameter dx. This object is of the same type as the simulation, type T.
VtkImageOutput3D<T> vtkOut("simulationData.dat", dx);

// Add a 3D tensor-field to the VTK file, representing the velocity, and rescale
//   with the units of a velocity, dx/dt. Explicitly convert the data to single-
//   precision floats in order to save storage space.
vtkOut.writeData<3,float>(*computeVelocity(lattice), "velocity", dx/dt);

// Add another 3D tensor-field for the vorticity, again as floats.
vtkOut.writeData<3,float>(*computeVorticity(*computeVelocity(lattice)), "vorticity", 1./dt);

// To end-with add a scalar-field for the density.
vtkOut.writeData<1,float>(*computeDensity(lattice), "density", 1.);
```

You can add as many blocks as you wish to the VTK file, and each of them can have an arbitrary number of components (in the example above, 3-component tensor-fields and a 1-component scalar-field were used).

Similarly, 2D data can be written in a VTK format by using an object of type VtkImageOutput2D (an example is provided in the directory `examples/showCases/poiseuille/`. A tool like Paraview is however not always the best choice for the analysis of 2D data. You are often better advised to flush the data into a text file, as explained in Section `output-streams`, and then visualize it with Octave or Matlab (or Python or R or SciLab or OO-Spreadsheet, or whatever your favorite data interpreter is).

## Checkpointing: saving and loading the state of a simulation

While the results of a simulation can be written into a VTK file with the help of a `VtkImageOutputXD` object, the reverse procedure is not implemented: it is not possible to read VTK data into an Palabos data structure. If you wish to save the state of the system in order to resume the simulation at a later time, use the function saveBinaryBlock, as illustrated for example in the file `examples/codesByTopic/io/checkPointing.cpp`:

```C++
saveBinaryBlock(lattice, "checkpoint.dat");
```

and load it later on with the function loadBinaryBlock:

```C++
loadBinaryBlock(lattice, "checkpoint.dat");
```

With this procedure, you can only write data one block at a time. If the state of a simulation is represented by various blocks (e.g. in a multi-component fluid which uses a few block-lattices), each of them needs to be saved in a separate file. It is currently not possible to save structural data (dynamics objects and data processors). When loading the state of a simulation, the context, such as boundary conditions, must therefore be recreated before loading the data.