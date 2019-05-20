# Parallelism<div id="Parallelism"></div>

## Parallel programming approach
Programs written with Palabos can be automatically parallelized, at least if they are following the recommendations provided in this user’s guide. Parallelization is performed with the message-passing paradigm of the MPI library. This approach works fine on distributed-memory platforms (e.g. clusters) as well as on shared-memory platforms (SMP or multi-core architectures). As a matter of fact, due to the high availability of dual-core or many-core processors, it is recommended to compile Palabos programs practically always in parallel, even during the development phase. The significant speedup obtained from exploiting all cores of a machines leads to an improved efficiency during code development and program testing cycle.

To achieve parallelism with programs which have the look and feel of serial applications, Palabos distinguishes between two classes of data. Data which is spatially distributed, such as the block-lattices, the scalar- or the tensor-fields, are handled through a data-parallel paradigm. The data space is partitioned into smaller regions that are distributed over the nodes or cores of a parallel machine. Other data types which require a small amount of storage space are duplicated on every node. All native C++ data types, as well as the data types of the C++ STL, and small data-types of Palabos like the `Array<T,nDim>` are automatically duplicated, by virtue of the Single-Program-Multiple-Data model of MPI. These types should be used to handle scalar values, such as the parameters of the simulation, or integral values over the solution (e.g. the average energy).

Finally, it is pointed out once more that only the multi-block structures (multi-block lattice, multi-scalar field, and multi-tensor field) are data parallel, while their atomic counterparts are duplicated overall cores if they are created in end-user code. For this reason, it is important to always work with multi-block structures in order to get parallel performance improvements.

## Controlling the efficiency
### Rules for efficient parallel programs
The Rule 0 for efficient program implementations in Palabos has been emphasized and repeated many times in the user’s guide, because it is extremely important.

**Rule 0 for efficient program implementations::** An end-user application should never contain a loop which runs over the indices of a multi-block. It is OK to access the cells of a multi-block occasionally, through their array-like interface, to evaluate or modify data. But it is NOT OK to access them repeatedly from within a space loop, because this would be catastrophically slow in a parallel programs. Space loops must appear inside data processors and nowhere else.

Once this rule is respected, most other ingredients leading to an efficient parallel program are simple common sense. For example, input/output operations are relatively slow in parallel programs, because of the way they are currently implemented (see Section bottlenecks). You should therefore make sure that the program doesn’t output data too often. Many of the programs provided in the directory examples are for example pretty slow in parallel, because they frequently produce images to illustrate a point. If you try to benchmark their parallel performance, you should therefore comment out the output operations.

In many cases, it is also possible to improve the efficiency of a program substantially by avoiding collective MPI operations, such as reductions. The lattice Boltzmann algorithms don’t actually need reduction operations to execute their time iterations. Reductions are just executed out of convenience at every time step, to evaluate the internal statistics (the average energy and the average density). These internal statistics are extremely useful in serial programs, because they are computed at practically no charge, but they can virtually ruin the performance in parallel programs. This is due to the fact that a reduction acts like a synchronization barrier, a fact which is observed to have a negative impact on performance, especially on cluster-like parallel machines. It is therefore possible to turn off the internal statistics (and thus, the reduction operations), through a function call

```C++
lattice.toggleInternalStatistics(false);
```

If you’d just like to know the internal statistics from time to time, say, once in twenty time iterations, it is possible to turn them on just for a short moment,

```C++
lattice.toggleInternalStatistics(true);
```

then, execute the time iteration:

```C++
lattice.collideAndStream();
```

and, finally, access the statistics and turn them off again:

```C++
T rho = getStoredAverageDensity<T>(lattice);
lattice.toggleInternalStatistics(false);  /// The value in website is true. I think it is a mistake.
```

### How many processors to use?

When a program is parallelized on few cores, it is common for the parallel efficiency to be nearly optimal. Each time the number of cores is doubled, the execution speed is almost doubled as well. As the number of cores if further increased, the speed up curve starts flattening, and at some point it’s not worth to add any more cores. It is common to speak about the efficiency of the program as the execution time on one core, divided by the execution time on a given number n of cores, and divided by `n`. If the program runs at, say, 75% efficiency, you can argue that it works reasonably well, because parallelization is never optimal. If on the other hand the efficiency drops below 50%, it is reasonable to demand that you reduce the number of cores, as you are just wasting the resources of your institute.

Without getting into the details of the theory of parallelism, let us just mention that the optimal number of cores to use for a problem depends on many parameters. The parallel efficiency tends to be better for simulations on large domains, and it is favorably influenced by a good interconnecting network (actually, by a high ratio of communication speed over the computation speed of a core). In practice, the optimal number of cores must be determined empirically for a given problem and a given parallel machine. A value which tends to be easy to remember is the size of the sub-domain associated to each core when the program runs in an optimal regime. On a given home-grown cluster, the example program `cavity3d` was for example observed once to run at 75% efficiency when the per-core domain size (the total domain size divided by the number of cores) was approximately 20-by-20-by-20. Since then, this machine is associated with the “20-cube-rule”, by which it is easy to rapidly evaluate the optimal number of cores to be used for a given problem size.

### Bottlenecks in the current implementation

While intense efforts where invested to obtain an excellent parallel efficiency in Palabos, there is still room for improvement. Currently, one of the most severe bottlenecks appears to be given by input/output operations. For both input and output, the data is always transferred through one of the cores. These operations are therefore non-parallel: they are not faster, or only marginally faster on many cores than on a single core. While this is something one can live with on smaller parallel machines with a few hundred cores, it tends to become problematic on thousand-core hardware were input/output operations can dominate the overall execution time of a program. Parallel input/output is therefore one of the most important features which needs to be implemented in a near future.