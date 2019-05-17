# Input/Output
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

This works fine in both serial and parallel programs. Additionally to this, Palabos offers two global objects of name `argc` and `argv` through which the command-line parameters can be accessed from any location in the program. It is generally recommended to use these global objects instead of the main parameters, because they are more intelligent, with automatic type conversions and error handling. Their usage is illustrated in the following code:

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
In this example, the C++ exception mechanism is used to react to errors in the user input. If you prefer not to handle exceptions, you can use the noThrow version of the read function:

bool ok = global::argv(1).readNoThrow(double_argument);
if (!ok) {
    pcout << "Error while reading argument 1." << endl;
}
As an example for the use of command-line parameters, you can also have a look at the programs examples/showCases/rayleighTaylor2D.cpp and rayleighTaylor3D.cpp.