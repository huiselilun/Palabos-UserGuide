# Appendix: partial function/class reference<div id="PFCR"></div>
## Mutable (in-place) operations for simulation setup and other purposes
This appendix presents predefined mutable operations for block-lattices and data fields. Mutable means that the original block-lattice or data field is modified. For analysis and extraction of data, it is most often more appropriate to use non-mutable operations presented in the appendix `Non-mutable operations for data analysis and other purposes`.

Some of these functions accept in their argument list so-called “functionals”, or user-defined objects which behave like functions. These functionals inherit from one of the three following virtual base classes:

`OneCellFunctional3D`

> This functional is used to specify a fully local action to be performed on a subset of cells of a lattice. The action is defined by overriding the virtual function `void OneCellFunctional3D::execute(Cell<T,Descriptor>& cell) const`.

`OneCellIndexedFunctional3D`

> With this functional, a fully local, but space-dependent action is performed on a subset of cells of a lattice. The action is defined by overriding the virtual function `void OneCellIndexedFunctional3D::execute(plint iX, plint iY, plint iZ, Cell<T,Descriptor>& cell) const`.

`DomainFunctional3D`

> This functional is used to define the region on a lattice on which an action is performed. The region is defined by overriding the virtual function `bool DomainFunctional3D::operator() (plint iX, plint iY, plint iZ) const`.

Finally, some functions make use of static functionals which, thanks to template mechanisms, are exempt from an inheritance hierarchy. Their syntax depends on the context. For example, the functional `Function f` of the function `setBoundaryDensity` must define a method of the form `T operator() (plint iX, plint iY, plint iZ)`, which returns a value for the density at each space point.

All operations are applied to a sub-domain of the block, which is either rectangular-shaped when specified through the argument domain of type `DomainXD`, or general when specified through the argument domainFunctional of type `DomainFunctional3D`.

Parameters followed by a star `(*)` are passed by-pointer, while all others are passed by-value or by-reference.

### Operations on the block-lattice
`apply(lattice, domain, oneCellFunctional*)`

> Apply the same operation to all cells of the domain.

`applyIndexed(lattice, domain, oneCellIndexedFunctional*)`

> Apply a cell-dependent operation to all cells of the domain.

`defineDynamics(lattice, domain, dynamics*)`

> Assign a new dynamics object to all cells of the domain. Each cell gets an independent copy of the dynamics object.

`defineDynamics(lattice, boundingBox, domainFunctional*, dynamics*)`

> Assign a new dynamics object to all cells of the domain. Each cell gets an independent copy of the dynamics object. The argument `boundingBox` is of type `BoxXD`, and it must contain the domain described by `domainFunctional`. It is used for efficiency reasons, to avoid evaluating the domain-functional unnecessarily often.

`defineDynamics(lattice, dotList, dynamics*)`

> Assign a new dynamics object to all cells contained in the dot-list. Each cell gets an independent copy of the dynamics object.

`defineDynamics(lattice, boolMask, domain, dynamics*, flag)`

> Assign a new dynamics object to all cells where boolMask evaluates to `flag`. Each cell gets an independent copy of the dynamics object. The flag `flag` is boolean, and `boolMask` is of type `MultiScalaFieldXD<T>`, but is evaluated to Boolean values (smaller than 0.5 means false). If the argument `domain` is not specified, the whole domain of `lattice` is used.

`defineDynamics(lattice, iX, iY, iZ, dynamics*)`

> Assign a new dynamics object to the cell at the position `(iX,iY,iZ)`.

`setBoundaryVelocity(lattice, domain, velocity)`

> Assign the same velocity to all cells inside the domain which implement a Dirichlet boundary condition for the velocity. On all other cells, this function has no effect.

`setBoundaryVelocity(lattice, domain, Function f)`

> Assign as cell-dependent velocity to all cells inside the domain which implement a Dirichlet boundary condition for the velocity. On all other cells, this function has no effect.

`setBoundaryDensity(lattice, domain, rho)`

> Assign as cell-dependent density to all cells inside the domain which implement a Dirichlet boundary condition for the density (or for the pressure). On all other cells, this function has no effect.

`setBoundaryDensity(lattice, domain, Function f)`

> Assign as cell-dependent density to all cells inside the domain which implement a Dirichlet boundary condition for the density (or for the pressure). On all other cells, this function has no effect.

`initializeAtEquilibrium(lattice, domain, rho, velocity)`

> Initialize the cells of the domain at an equilibrium distribution, using the same density and velocity for all of them. The exact form of the equilibrium is defined by the dynamics object residing on the cell at this moment.

`initializeAtEquilibrium(lattice, domain, Function f)`

> Initialize the cells of the domain at an equilibrium distribution, using a cell-dependent density and velocity. The exact form of the equilibrium is defined by the dynamics object residing on the cell at this moment.

`stripeOffDensityOffset(lattice, domain, deltaRho)`

> Decompose the particle populations into macroscopic variables and off-equilibrium parts. Then, subtract the value deltaRho from the density, and recompose the populations.

`setCompositeDynamics(lattice, domain, compositeDynamics*)`

> Assign a composite-dynamics object to all cells of the domain, using the dynamics currently residing on the cell as base dynamics.

`setExternalScalar(lattice, domain, whichScalar, externalScalar)`

> Assign the same constant value to the external scalar `whichScalar` on all cells of the domain.

`setExternalVector(lattice, domain, vectorStartsAt, externalVector)`

> Here, `externalVector` is of type Array<T,d>. Assign the content of this array to assign a value to 2 (2D) or 3 (3D) external scalars on each cell of the domain. This function is often used to initialize the external force term in forced lattice Boltzmann simulations.

### Operations on scalar-fields
`setToConstant(field, domain, value)`

> Initialize all scalars in the domain to a constant value.

`setToFunction(field, domain, Function f)`

> Initialize all scalars in the domain to a value which is a function of space (function `f` takes two (2D) or three (3D) integer coordinate values, and returns T).

`setToCoordinate(field, domain, index)`

> Replace each scalar in the domain by the `index`-component of its space position (x=0, y=1, and z=2 for `index`).

`apply(Function f, field, domain)`

> Apply any function or functional f (which takes T and returns T) to the domain.

### Scalar-fields: A = A operator alpha
`addInPlace(field, double scalar, Box2D domain)`

> Add the scalar value `scalar` to each value of `field` inside the domain.

`subtractInPlace(field, double scalar, Box2D domain)`

> Subtract the scalar value `scalar` from each value of `field` inside the domain.

`multiplyInPlace(field, double scalar, Box2D domain)`

> Multiply each value of `field` by the scalar `scalar` inside the domain.

`divideInPlace(field, double scalar, Box2D domain)`

> Divide each value of `field` by the scalar `scalar` inside the domain.

### Scalar-fields: A = A operator B
`addInPlace(A, B, domain)`

> Compute A = A+B on the domain.

`subtractInPlace(A, B, domain)`

> Compute A = A-B on the domain.

`multiplyInPlace(A, B, domain)`

> Compute A = A*B on the domain.

`divideInPlace(A, B, domain)`

> Compute A = A/B on the domain.

### Operations on tensor-fields
`setToConstant(field, domain, value)`

> Initialize all tensors/vectors in the domain to a constant value (`value` is of type `Array<T,nDim>`).

`setToFunction(field, domain, Function f)`

> Initialize all tensors/vectors in the domain to a value which is a function of space (function `f` takes two (2D) or three (3D) integer coordinate values, and returns `Array<T,nDim>`).

`setToCoordinates(field, domain, index)`

> Replace each vector in the domain by its space position. The tensor-field `field` must hold vectors of dimension 2 (2D) or 3 (3D).

### Tensor-fields: A = A operator B
`addInPlace(A, B, domain)`

> Compute A = A+B, element-wise on the domain.

`subtractInPlace(A, B, domain)`

> Compute A = A-B, element-wise on the domain.

`multiplyInPlace(A, B, domain)`

> Compute A = A*B, element-wise on the domain.

`divideInPlace(A, B, domain)`

> Compute A = A/B, element-wise on the domain.

## Non-mutable operations for data analysis and other purposes
This appendix presents predefined non-mutable operations for block-lattices and data fields. Non-mutable means that the original block-lattice or data field is not modified, and that the result is stored in a new block-lattice or data-field. These operations are often used for data post-processing, and are therefore presented under the name of “data analysis”. For actual computations during a simulation, it is most often more appropriate to use in-place operations presented in the appendix `Mutable (in-place) operations for simulation setup and other purposes`.

The operators are defined internally as data processors, and presented in the user interface through convenience functions, as shown in the following for the function `computeDensity()`:

`std::auto_ptr<MultiScalarField3D<T> > computeDensity(MultiBlockLattice3D<T,Descriptor>& lattice)`

> This function applies the requested operation over the whole domain of `lattice`, and constructs a new scalar-field into which the result is stored. A function of this form is available for each operator.

`std::auto_ptr<MultiScalarField3D<T> > computeDensity(MultiBlockLattice3D<T,Descriptor>& lattice, Box3D domain)`

> In this case, the operation is only applied to the sub-domain domain, and the resulting scalar-field has a size corresponding to `domain`. A function of this form is available for each operator, and this form only is listed in the following to document the operator.

`void computeDensity(MultiBlockLattice3D<T,Descriptor>& lattice, MultiScalarField3D<T>& density, Box3D domain)`

> In this function, no new field is allocated, as the result is stored in the field `density`, which is given to this function by argument. A function of this form is always available, except for operators which return a scalar value such as `computeAverageDensity()`.

More explanations on using the non-mutable operations can be found in Section `Data evaluation`, and the exact syntax of the functions is given in the Palabos reference guide.

### [block-lattice -> scalar]
`computeAverageDensity(lattice, domain)`

> Return the average value of the density, computed over the domain.

`computeAverageRhoBar(lattice, domain)`

> Return the average value of `rhoBar`, computed over the domain. Note that while conceptually, the command `computeAverageDensity()` is equivalent to `Descriptor<T>::fullRho(computeRhoBar())`, the latter is likely to be less affected by round-off errors.

`computeAverageEnergy(lattice, domain)`

> Return the average of the norm-square of the velocity, divided by two.

### [block-lattice -> block-lattice]
`extractSubDomain(lattice, domain)`

Copy the values from the domain `domain` of `lattice` and deposit them in a new lattice of corresponding size.

### [block-lattice -> scalar-field], [block-lattice -> tensor-field]
`computeDensity(lattice, domain)`

> Compute the density `rho` on each cell.

`computeRhoBar(lattice, domain)`

> Compute the value `rhoBar` on each cell.

`computeKineticEnergy(lattice, domain)`

> Compute the kinetic energy (half the squared velocity-norm) on each cell.

`computeVelocityNorm(lattice, domain)`

> Compute the velocity-norm on each cell.

`computeVelocityComponent(lattice, domain, iComponent)`

> Compute the component iComponent of the velocity (`iComponent` ranges from 0 to 1 in 2D, and from 0 to 2 in 3D) on each cell.

`computeVelocity(lattice, domain)`

> Compute all components of the velocity on each cell.

`computeDeviatoricStress(lattice, domain)`

> Compute the off-equilibrium part of the stress tensor `Pi` on each cell. The result is model-dependent and is determined by the dynamics object defined at the cell location. The result is stored in a 3-component (2D) or 6-component (3D) tensor-field (the number of components is reduced because the tensor is symmetric). To know the index of a specific component, use a notation like `SymmetricTensorImpl<T,3>::xz` for the the xy-component of a 3D symmetric tensor.

`computeStrainRateFromStress(lattice, domain)`

Compute the strain rate S=1/2(grad(u)+grad(u)^T), assuming that it is proportional to the deviatoric stress. The result is model-dependent and is determined by the dynamics object defined at the cell location. The result is stored in a 3-component (2D) or 6-component (3D) tensor-field (the number of components is reduced because the tensor is symmetric). To know the index of a specific component, use a notation like `SymmetricTensorImpl<T,3>::xz` for the the xy-component of a 3D symmetric tensor.

`computePopulation(lattice, domain, iPop)`

Extract the population `iPop` on each cell, and deposit the result in a scalar-field. The value `iPop` ranges from 0 to q-1.

### [scalar-field –> scalar]
`computeAverage(scalarField, domain)`

> Return the average of the values in the scalar-field, over the domain.

`computeBoundedAverage(scalarField, domain)`

> Approximate an integral over the domain with second-order accuracy, using the trapezium rule (as compared to `computeAverage()`, the boundary nodes are accounted for with a smaller weight than bulk nodes.

`computeMin(scalarField, domain)`

> Return the minimum of the values in the scalar-field, over the domain (the location of this value is not returned).

`computeMax(scalarField, domain)`

> Return the maximum of the values in the scalar-field, over the domain (the location of this value is not returned).

###[scalar-field -> scalar-field]
`extractSubDomain(scalarField, domain)`
> Copy the values from the domain `domain` of `scalarField` and deposit them in a new scalar-field of corresponding size.

### Scalar-field: B = A operator alpha
`add(scalar, field, domain)`

> Return the result of computing `scalar+field(x,y,z)` on each element of the domain.

`add(field, scalar, domain)`

> Return the result of computing `field(x,y,z)+scalar` on each element of the domain.

`subtract(scalar, field, domain)`

> Return the result of computing `scalar-field(x,y,z)` on each element of the domain.

`subtract(field, scalar, domain)`

> Return the result of computing `field(x,y,z)-scalar` on each element of the domain.

`multiply(scalar, field, domain)`

> Return the result of computing `scalar*field(x,y,z)` on each element of the domain.

`multiply(field, scalar, domain)`

> Return the result of computing `field(x,y,z)*scalar` on each element of the domain.

`divide(scalar, field, domain)`

> Return the result of computing `scalar/field(x,y,z)` on each element of the domain.

`divide(field, scalar, domain)`

> Return the result of computing `field(x,y,z)/scalar` on each element of the domain.

### Scalar-field: C = A operator B
`add(A, B, domain)`

> Return `C=A+B`, applied to every element of the domain.

`subtract(A, B, domain)`

> Return `C=A-B`, applied to every element of the domain.

`multiply(A, B, domain)`

> Return `C=A*B`, applied to every element of the domain.

`divide(A, B, domain)`

> Return `C=A/B`, applied to every element of the domain.

### [tensor-field -> tensor-field]
`extractSubDomain(tensorField, domain)`

> Copy the values from the domain `domain` of `tensorField` and deposit them in a new tensor-field of corresponding size.

`extractComponent(tensorField, domain, iComponent)`

> Copy the component `iComponent` from each cell of `tensorField` into a resulting scalar-field.

`computeVorticity(velocity, domain)` [3D only]

> Compute the vorticity of each cell in a 3D velocity field, and store the result in a 3D vorticity field. The vorticity is evaluated through a central (second-order accurate) finite difference scheme on the bulk of the domain, and through an asymmetric, nearest-neighbor scheme on boundary nodes.

`computeBulkVorticity(velocity, domain)` [3D only]

> Compute the vorticity of each cell in a 3D velocity field, and store the result in a 3D vorticity field. The vorticity is evaluated through a central (second-order accurate) finite difference scheme on all cells. If `domain` includes non-periodic boundary nodes of the simulation, you should rather use `computeVorticity()` to account for boundary effects.

`computeStrainRate(velocity, domain)`

> Compute the strain rate (S=1/2(grad(U)+grad(U)^T) at each cell of a velocity field, and store the result in a 3-component (2D) or 6-component (3D) tensor-field (the number of components is reduced because the tensor is symmetric). To know the index of a specific component, use a notation like `SymmetricTensorImpl<T,3>::xz` for the the xy-component of a 3D symmetric tensor. The Strain-rate is evaluated through a central (second-order accurate) finite difference scheme on the bulk of the domain, and through an asymmetric, nearest-neighbor scheme on boundary nodes.

`computeBulkStrainRate(velocity, domain)`

> Compute the strain rate at each cell of a velocity field, and store the result in a 3-component (2D) or 6-component (3D) tensor-field. The strain-rate is evaluated through a central (second-order accurate) finite difference scheme on all cells. If `domain` includes non-periodic boundary nodes of the simulation, you should rather use `computeStrainRate()` to account for boundary effects.

### [tensor-field -> scalar-field]
`computeNorm(tensorField, domain)`

> Compute the Euclidean norm of each cell in `tensorField`.

`computeNormSqr(tensorField, domain)`

> Compute the square of the Euclidean norm of each cell in `tensorField`.

`computeSymmetricTensorNorm(tensorField, domain)`

> Compute the Euclidean norm of each cell in the symmetric tensor `tensorField`. It is assumed that `tensorField` only stores half the off-diagonal components, due to symmetry.

`computeSymmetricTensorNormSqr(tensorField, domain)`

> Compute the square of the Euclidean norm of each cell in the symmetric tensor `tensorField`. It is assumed that `tensorField` only stores half the off-diagonal components, due to symmetry.

`computeSymmetricTensorTrace(tensorField, domain)`

> Compute the trace of each cell in the symmetric tensor `tensorField`. It is assumed that `tensorField` only stores half the off-diagonal components, due to symmetry.

`computeVorticity(velocity, domain)` [2D only]

> Compute the vorticity of each cell in a 2D velocity field, and store the result in a scalar-field. The vorticity is evaluated through a central (second-order accurate) finite difference scheme on the bulk of the domain, and through an asymmetric, nearest-neighbor scheme on boundary nodes.

`computeBulkVorticity(velocity, domain)` [2D only]

> Compute the vorticity of each cell in a 2D velocity field, and store the result in a scalar-field. The vorticity is evaluated through a central (second-order accurate) finite difference scheme on all cells. If `domain` includes non-periodic boundary nodes of the simulation, you should rather use `computeVorticity()` to account for boundary effects.

### Tensor-field: C = A operator B
`add(A, B, domain)`

> Return `C=A+B`, applied to every element of the domain.

`subtract(A, B, domain)`

> Return `C=A-B`, applied to every element of the domain.

`multiply(A, B, domain)`

> Return `C=A*B`, applied to every element of the domain.

`divide(A, B, domain)`

> Return `C=A/B`, applied to every element of the domain.