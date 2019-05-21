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
apply(lattice, domain, oneCellFunctional*)
Apply the same operation to all cells of the domain.
applyIndexed(lattice, domain, oneCellIndexedFunctional*)
Apply a cell-dependent operation to all cells of the domain.
defineDynamics(lattice, domain, dynamics*)
Assign a new dynamics object to all cells of the domain. Each cell gets an independent copy of the dynamics object.
defineDynamics(lattice, boundingBox, domainFunctional*, dynamics*)
Assign a new dynamics object to all cells of the domain. Each cell gets an independent copy of the dynamics object. The argument boundingBox is of type BoxXD, and it must contain the domain described by domainFunctional. It is used for efficiency reasons, to avoid evaluating the domain-functional unnecessarily often.
defineDynamics(lattice, dotList, dynamics*)
Assign a new dynamics object to all cells contained in the dot-list. Each cell gets an independent copy of the dynamics object.
defineDynamics(lattice, boolMask, domain, dynamics*, flag)
Assign a new dynamics object to all cells where boolMask evaluates to flag. Each cell gets an independent copy of the dynamics object. The flag flag is boolean, and boolMask is of type MultiScalaFieldXD<T>, but is evaluated to Boolean values (smaller than 0.5 means false). If the argument domain is not specified, the whole domain of lattice is used.
defineDynamics(lattice, iX, iY, iZ, dynamics*)
Assign a new dynamics object to the cell at the position (iX,iY,iZ).
setBoundaryVelocity(lattice, domain, velocity)
Assign the same velocity to all cells inside the domain which implement a Dirichlet boundary condition for the velocity. On all other cells, this function has no effect.
setBoundaryVelocity(lattice, domain, Function f) Assign as cell-dependent velocity to all cells inside the domain which implement a Dirichlet boundary condition for the velocity. On all other cells, this function has no effect.

setBoundaryDensity(lattice, domain, rho)
Assign as cell-dependent density to all cells inside the domain which implement a Dirichlet boundary condition for the density (or for the pressure). On all other cells, this function has no effect.
setBoundaryDensity(lattice, domain, Function f)
Assign as cell-dependent density to all cells inside the domain which implement a Dirichlet boundary condition for the density (or for the pressure). On all other cells, this function has no effect.
initializeAtEquilibrium(lattice, domain, rho, velocity)
Initialize the cells of the domain at an equilibrium distribution, using the same density and velocity for all of them. The exact form of the equilibrium is defined by the dynamics object residing on the cell at this moment.
initializeAtEquilibrium(lattice, domain, Function f)
Initialize the cells of the domain at an equilibrium distribution, using a cell-dependent density and velocity. The exact form of the equilibrium is defined by the dynamics object residing on the cell at this moment.
stripeOffDensityOffset(lattice, domain, deltaRho)
Decompose the particle populations into macroscopic variables and off-equilibrium parts. Then, subtract the value deltaRho from the density, and recompose the populations.
setCompositeDynamics(lattice, domain, compositeDynamics*)
Assign a composite-dynamics object to all cells of the domain, using the dynamics currently residing on the cell as base dynamics.
setExternalScalar(lattice, domain, whichScalar, externalScalar)
Assign the same constant value to the external scalar whichScalar on all cells of the domain.
setExternalVector(lattice, domain, vectorStartsAt, externalVector)
Here, externalVector is of type Array<T,d>. Assign the content of this array to assign a value to 2 (2D) or 3 (3D) external scalars on each cell of the domain. This function is often used to initialize the external force term in forced lattice Boltzmann simulations.