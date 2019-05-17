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

```C++
% Analysis of the data in Octave
% Assign manually the dimensions:
nx = 100;
ny = 100;
load energy.dat;
energy = reshape(energy, nx, ny);
imagesc(energy);
```