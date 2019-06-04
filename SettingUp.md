# Setting up a problem<div id="SettingUp"></div>
## Attributing dynamics objects
When constructing a new block-lattice, you must decide what type of collision is going to be executed on each of its cells, by assigning them a dynamics object. To avoid bugs related to cells without dynamics objects, the constructor of a block-lattice assigns a default-value for the dynamics to all cells, the so-called background dynamics:

```C++
// Construct a new block-lattice with a background-dynamics of type BGK.
MultiBlockLattice3D<T,DESCRIPTOR> lattice( nx, ny, nz,
                                           new BGKdynamics<T,DESCRIPTOR>(omega) );
```

After this, the dynamics of each cell can be redefined in order to adjust the behavior of the cells locally. The background dynamics differs from usual dynamics objects in an important way. To obtain memory savings, the constructor of the block-lattice creates only one instance of the dynamics object, and all cells refer to the same instance. If you modify a dynamics object on one cell, it changes everywhere. As an example, consider the relaxation parameter `omega`, which is stored in the dynamics, not in the cell. A function call like

```C++
lattice.get(0,0,0).getDynamics().setOmega(newOmega);
```

would affect the value of the relaxation parameter on each cell of the lattice. Note that this particular behavior of the background dynamics, having several cells referring to the same object, cannot be reproduced by other means than constructing a new block-lattice. All other functions which override the background-dynamics on given cells with a new dynamics object are defined in such a way as to create an independent copy of the dynamics object for each cell.

If the reference semantics of the background dynamics is contrary to your needs, you can re-define all dynamics objects right after constructing a block-lattice, to guarantee that all cells have an independent copy:

```C++
// Override background-dynamics to guarantee and independent per-cell
//   copy of the dynamics object.
defineDynamics( lattice, lattice.getBoundingBox(),
                new BGKdynamics<T,DESCRIPTOR> );
```

There exist different variants of the function `defineDynamics` which you can use to adjust the dynamics of a group of cells (their syntax can be found in the Appendix `Operations on the block-lattice`):

One-cell version:
- Assign a new dynamics to just one cell.

- Example: `tutorial/tutorial2/tutorial2_3.cpp`

BoxXD version:
- Assign a new dynamics to all cells within a rectangular domain.

- Example: `showCases/multiComponent2d/rayleighTaylor2D.cpp`, or the `3d` example.

DotListXD version:
- Assign a new dynamics to several cells, listed individually in a dot-list structured.

- Example: `codesByTopic/dotList/cylinder2d.cpp`

Domain-functional version:
- Provide an analytical function which indicates the coordinates of cells which get a new dynamics object.

- Example: `showCases/cylinder2d/cylinder2d.cpp`

Bool-mask version:
- Specify the location of cells which get a new dynamics object through a Boolean mask, represented through a scalar-field.

- Example: `codesByTopic/io/loadGeometry.cpp`

## Initial values of density and velocity
It is quite common to assign an initial value of velocity and density to a lattice by initializing all cells to an equilibrium distribution with the chosen density and velocity value. While this approach is not sufficient for time-dependent benchmark problems which depend critically on the initial state, it is most often sufficient to get a simulation started with reasonable initial values. The function `initializeAtEquilibrium` (see Appendix `Operations on the block-lattice`) is provided to initialize the cells within a rectangular-shaped domain at an equilibrium distribution. It comes in two flavors: one with a constant density and velocity within the domain (see the example `examples/showCases/cavity2d` or `3d`), and one with a space-dependent value of these macroscopic values, specified through a user-defined function (see the example `examples/showCases/poiseuille`).

To initialize cells in a different way, it is simplest to apply a custom operator to the cells with the help of one-cell functionals and indexed one-cell functionals introduced in Section `Convenience wrappers for local operations`.