# Data evaluation<div id="DataEvaluation"></div>
## Overview
The variables simulated in a lattice Boltzmann program, the particle populations, contrast with the quantities a typical fluid engineer is interested in, the macroscopic variables pressure, velocity, and others. Obviously, one needs to somehow convert the data before providing it to a standard post-processing tool. Many functions for conversion, evaluation, or transformation of data are provided in Palabos, as listed in `Appendix Non-mutable operations for data analysis and other purposes`. In the present section, the function `computeVelocity` is discussed as an example, as the other functions work in a very similar way. These functions are defined for the 2D and the 3D case, and they work with `atomic-blocks` and `multi-blocks`. For the sake of illustration, the following codes refer to the 3D multi-block case.

The function `computeVelocity` is provided in three versions:

```C++
// Version 1
void computeVelocity(MultiBlockLattice3D<T,Descriptor>& lattice,
                     MultiTensorField3D<T,Descriptor<T>::d>& velocity, Box3D domain);

// Version 2
std::auto_ptr<MultiTensorField3D<T,3> >
    computeVelocity(MultiBlockLattice3D<T,Descriptor>& lattice, Box3D domain);

// Version 3
std::auto_ptr<MultiTensorField3D<T,3> >
    computeVelocity(MultiBlockLattice3D<T,Descriptor>& lattice);
```

In the first version, the velocity is computed on a sub-domain of a block-lattice, and the result is written to a corresponding sub-domain of a three-component tensor-field. The block-lattice and the tensor-field don’t need to have the same size, nor do they need to have the same internal block arrangement. If the domain exceeds the dimensions of either the block-lattice or the tensor-field, the domain is trimmed correspondingly.

In the second version, which is much more often used in practice, a tensor-field with the size of `domain` is automatically created and returned from the function. The auto-pointer keyword used in the return value of the function refers to a class of smart-pointers offered by the C++ standard library. From the user’s point of view, it is practically the same as a pointer. This means that you treat an object of type `std::auto_ptr<MultiTensorField3D<T,3> >` in the same way as you would treat an object of type `MultiTensorField3D<T,3>*`. The big difference between the two is that the auto-pointer has an automatic memory management mechanism, and that you never need to call the operator delete on this type of pointers. If the return value of the function were a raw pointer, you wouldn’t be able to pipeline different operators, as shown in the next section, because you’d never get an explicit pointer to them and therefore couldn’t delete them.

The following code shows a typical use case of the function `computeVelocity`:

```C++
pcout << *computeVelocity(lattice, domain) << endl;
```

Here, the computed velocity values are immediately redirected to the terminal. The star in front of the function call is used to dereference the pointer to the velocity field. At the end of this program line, the memory for the velocity field is automatically de-allocated, and does not need to be disposed of explicitly.

The third version is a pure convenience function in which the argument `domain` is replaced by the full domain of the lattice, `lattice.getBoundingBox()`.

## Pipelining data evaluation operators
The return value of a function like `computeVelocity` can directly be reused as the argument of other data evaluation operators. In this way, it is possible to construct complex expressions. For example, the function `computeAverage` in the previous section could be replaced by a computation of the velocity field, followed by the computation of the norm-square of each element, a division by two, and finally a computation of the average value:

```C++
pcout << "The value "
      << *computeAverageEnergy(lattice) << " is the same as "
      << computeAverage (
                 *multiply ( 0.5,
                         *computeNormSqr (
                                 *computeVelocity(lattice) ) ) )
      << endl;
```

All the functions used in this example are listed in the Appendix `Appendix: partial function/class reference`. More examples of the evaluation of data, constructions of scalar fields, and combination of data evaluation operators are provided in the directory `examples/codesByTopic/scalarField`.