# Data processors for non-local operations and couplings between blocks<div id="DataProcessors"></div>
Dynamics classes and data processors are the nuts and bolts of a Palabos program. Conceptually speaking, you can view dynamics classes as an implementation of a raw “lattice Boltzmann paradigm”, whereas data processors are rather a realization of a more general data-parallel multi-block paradigm. Dynamics classes are (relatively) easy, because lattice Boltzmann is easy, as long as there are no boundaries, refined grids, parallel programs, or any other advanced structural ingredients. Data processors can be a little bit harder to understand, because they have the difficult task to solve “everything else” which is not taken charge of by dynamics classes. They implement all non-local ingredients of a model, they execute operations on scalar-fields and tensor-fields, and they create couplings between all types of blocks. And, just like dynamics objects, they handle reduction operations, and they must be efficient and inherently parallelizable.

The importance of data processors is particularly obvious when you are working with scalar-fields and tensor-fields. Unlike block-lattices, these data fields do not have intelligent cells with a local reference to a dynamics object; data processors are therefore the only efficient way of performing collective operations on them. As an example, consider a field of type `TensorField3D<T,3>` that represents a velocity field (each element is a velocity vector of type `Array<T,3>`). Such a field is for example obtained from a LB simulation by calling the function `computeVelocity`. It might be interesting to compute space-derivatives of the velocity through finite differences. The only morally right way of doing this is through a data processor, because it is the only approach which is parallelizable, scalable, efficient, and forward-compatible with future versions of Palabos. This is exactly what Palabos does when you call one of the functions like `computeVorticity()` or `computeStrainRate()` (see the appendix `[tensor-field -> tensor-field]`). Note, once more, that you could also evaluate the finite difference scheme by writing a simple loop over the space indices of the tensor-field. This would produce the correct result in serial and in parallel, and it would even be pretty efficient in serial, but it is not parallelizable (in parallel, efficiency is lost instead of gained).

All in all, it should have become clear that while data processors are powerful objects, they also need to address a certain amount of complexity and are therefore conceptually less simple than other components of Palabos. Consequently, data-processors are most certainly the part of Palabos’ user’s interface which is toughest to understand, and this section is filled with ad-hoc rule which you just need to learn at some point. To alleviate this thing a bit, the section starts with two relatively simple topics which teach you how to solve many tasks in Palabos through the available helper functions and avoid explicitly writing data processors in many cases. The remainder of this section contains many links to existing implementations in Palabos, and you are strongly encouraged to actually have a look at these examples to assimilate the theoretical concepts.

## Using helper functions to avoid explicitly writing data processors
Data processors can be (arbitrarily) split into three categories, according to the use which is made out of them. The first category is about setting up a simulation, assigning dynamics objects to a sub-domain of the lattice, initializing the populations, and so on. These methods are explained in the sections `Initial values of density and velocity` and `Defining boundary conditions`, and the functions are listed in the appendix `Mutable (in-place) operations for simulation setup and other purposes`. The second category embraces data processors which are added to a lattice to implement a physical model. Many models are predefined, as presented in section `Implemented fluid models`. Finally, the last category is for the evaluation of the data, and for the execution of short post-processing task, as in the above example of the computation of a velocity gradient. Examples are found in the section `Data evaluation`, and the available functions are listed in the appendix `Non-mutable operations for data analysis and other purposes`.

## Convenience wrappers for local operations
Imagine that you have to perform a local initialization task on a block-lattice, i.e. a task for which you don’t need to access the value of surrounding neighbors, and for which the available Palabos functions are insufficient. As a lightweight alternative to writing a classical data-processing functional, you can implement a class which inherits from `OneCellFunctionalXD` and implements the virtual method (here in 2D)

```C++
virtual void execute(Cell<T,Descriptor>& cell) const;
```

In the body of this method, simply perform the desired action on the argument cell. If the action depends on the space position, you can instead inherit from `OneCellIndexedFunctionalXD` and implement the method (again illustrated for the 2D case)

```C++
virtual void execute(plint iX, plint iY, Cell<T,Descriptor>& cell) const;
```

An instance of this one-cell functional is then applied to the lattice through a function call like

```C++
// Case of a plain one-cell functional.
applyIndexed(lattice, domain, new MyOneCellFunctional<T,Descriptor>);
// Case of an indexed one-cell functional.
applyIndexed(lattice, domain, new MyOneCellIndexedFunctional<T,Descriptor>);
```

This method is used in the example program located in `examples/showCases/multiComponent2d`. Here, a customized initialization process is required to get access to the external scalars of a lattice for a thermal simulation.

These one-cell functionals are less general than usual data processing functionals, because they cannot be used for non-local operations. Furthermore, they tend to be numerically somewhat less efficient, because Palabos needs to perform a virtual function call to the method execute of the one-cell functional on each cell of the domain. However, this loss of efficiency is usually completely negligible during the initialization stage, where it is important to have a code which scales on a parallel machine, but not to optimize the code for a gain of a micro-second. In this case you should prefer a code which, as proposed by the one-cell functionals, is shorter and easier to read.

## Writing data processors (or actually, data-processing functionals)
A common way to execute an operation on a matrix is to write some sort of loop over all elements of the matrix, or at least over a given sub-domain, and to execute the operation on each cell. If the memory of the matrix is subdivided into smaller components, as it is the case for Palabos’ multi-blocks, and these components are distributed over the nodes of a parallel machine, then your loop needs also to be subdivided into corresponding smaller loops. The purpose of the data processors in Palabos is to perform this subdivision automatically for you. It then provides the coordinates of the subdivided domains, and requires from you to execute the operation on these domains, instead of the original full domain.

As a developer of a data processor, you’re almost always in touch with so-called data-processing functionals which provide a simplified interface by performing a part of the repetitive tasks behind the scenes. There exist many different types of data-processing functionals, as listed in the next section. For the sake of illustration, we consider now the case of a data-processor which acts on a single 2D block-lattice or multi-block-lattice, and which does not perform any data reduction (it returns no value). Let’s say that the aim of the operation is to exchange the value of `f[1]` and `f[5]` on a given amount on lattice cells. This could (and should, for the sake of code clarity) be done with the simple one-cell-functional introduced in Section `Convenience wrappers for local operations`, but we want to do the real thing now, and get to the core of data functionals.

The data-processing functional for this job must inherit from `BoxProcessingFunctional2D_L` (the L indicates that the data processor acts on a single lattice), and implement, among other methods described below, the virtual method process:

```C++
template<typename T, template<typename U> class Descriptor>
class Invert_1_5_Functional2D : public BoxProcessingFunctional2D_L<T,Descriptor> {
    public:
    // ... implement other important methods here.
    void process(Box2D domain, BlockLattice2D<T,Descriptor>& lattice)
    {
        for (plint iX=domain.x0; iX<=domain.x1; ++iX) {
            for (plint iY=domain.y0; iY<=domain.y1; ++iY) {
                Cell<T,Descriptor>& cell = lattice.get(iX,iY);
                // Use the function swap from the C++ STL to swap the two values.
                std::swap(cell[1], cell[5]);
            }
        }
    }
};
```

The first argument of the function `process` corresponds to domain computed by Palabos after sub-dividing the original domain to fit to the small components of the original block. The second argument is corresponds to the lattice on which the operation is to be performed. This argument is always an atomic-block (i.e. it is always a block-lattice and never a multi-block-lattice), because at the point of the function call to `process`, Palabos has already subdivided the original block and is accessing its internal, atomic sub-blocks. If you compare this to the procedure of writing a one-cell-functional as shown in Section `Convenience wrappers for local operations`, you will see that the additional work you need to do in the present case is to write yourself the loop over the space indices of the domain. Having to write out these loops by hand all the time is tiring, especially when you write many data processors, and it is error-prone. But it is the cost to pay for optimal efficiency, and in the field of computational physics, efficiency counts just a bit more than in other domains of software engineering and must be valued, unfortunately, against elegance from time to time.

Another advantage against one-cell-functionals is the possibility to implement non-local operations. In the following example, the value of `f[1]` is swapped with `f[5]` on the right neighboring cell:

```C++
void process(Box2D domain, BlockLattice2D<T,Descriptor>& lattice)
{
    for (plint iX=domain.x0; iX<=domain.x1; ++iX) {
        for (plint iY=domain.y0; iY<=domain.y1; ++iY) {
            Cell<T,Descriptor>& cell = lattice.get(iX,iY);
            Cell<T,Descriptor>& partner = lattice.get(iX+1,iY);
            std::swap(cell[1], partner[5]);
        }
    }
}
```

You can do this without risking to access cells outside the range of the lattice if you respect two rules:

1. On nearest-neighbor lattices (D2Q9, D3Q19, etc.), you can be non-local by one cell but no more (you may write `lattice.get(iX+1,iY)` but not `lattice.get(iX+2,iY))`. On a lattice with extended neighborhood you can also extend the distance at which you access neighboring cells in a data processor. The amount of allowed non-locality is determined by the constant `Descriptor<T>::vicinity`.

2. Non-local operations are not allowed in data processors which act on the communication envelope (Section `The methods you need to override` explains what this means).

To conclude this sub-section, let’s summarize the things you are allowed to do in a data processors, and the things you are not allowed to. You are allowed to access the cells in the provided range (plus the nearest neighbors, or a few more neighbors according to the lattice topology), to read them and to modify them. The operation which you perform can be space dependent, but this space dependency must be generic and cannot depend on the specific coordinates of the argument `domain` provided to the function `process`. This is an extremely important point which we shall line out as the Rule 0 of data processors:

**Rule 0 of data processors::** A data processor must always be written in such a way that executing the data processor on a given domain has the same effect as splitting the domain into two sub-domains, and then executing the data processor consecutively on each of these sub-domains.

In practice, this means that you are not allowed to make any logical decision based on the parameters `x0`, `x1`, `y0`, or `y1` of the argument `domain`, or directly based on the indices `iX` or `iY`. Instead, these local indices must first be converted to global ones, independent of the sub-division of the data processor, as explained in Section `Absolute and relative position`.

### Categories of data-processing functionals
Depending on the type of blocks on which a data processor is applied, there exist different types of processing functionals, as listed below:

```C++
Class BoxProcessingFunctionalXD<T>
void processGenericBlocks(BoxXD domain, std::vector<AtomicBlockXD<T>*> atomicBlocks);
```

This class is practically never used. It is the fall-back option when everything else fails. It handles an arbitrary number of blocks of arbitrary type, which were casted to the generic type `AtomicBlockXD`. Before use, you need to cast them back to their real type.

```C++
Class LatticeBoxProcessingFunctionalXD<T,Descriptor>
void process(BoxXD domain, std::vector<BlockLatticeXD<T,Descriptor>*> lattices);
```

Use this class to process an arbitrary number of block-lattices, and potentially create a coupling between them. This data-processing functional is for example used to define a coupling between an arbitrary number of lattice for the Shan/Chen multi-component model defined in the files `src/multiPhysics/shanChenProcessorsXD.h` and `.hh`. This type of data-processing functional is not very frequently used either, as the two-block versions listed below are more appropriate in most cases.

```C++
Class ScalarFieldBoxProcessingFunctionalXD<T>
void process(BoxXD domain, std::vector<ScalarFieldXD<T>*> fields);
```

Same as above, applied to scalar-fields.

```C++
Class TensorFieldBoxProcessingFunctionalXD<T,nDim>
void process(BoxXD domain, std::vector<TensorFieldXD<T,nDim>*> fields);
```

Same as above, applied to tensor-fields.

```C++
Class BoxProcessingFunctionalXD_L<T,Descriptor>
void process(BoxXD domain, BlockLatticeXD<T,Descriptor>& lattice);
```

Data processor acting on a single lattice.

```C++
Class BoxProcessingFunctionalXD_S<T>
void process(BoxXD domain, ScalarFieldXD<T>& field);
```

Data processor acting on a single scalar-field.

```C++
Class BoxProcessingFunctionalXD_T<T,nDim>
void process(BoxXD domain, TensorFieldXD<T,nDim>& field);
```

Data processor acting on a single tensor-field.

```C++
Class BoxProcessingFunctionalXD_LL<T,Descriptor1,Descriptor2>
void process(BoxXD domain, BlockLatticeXD<T,Descriptor1>& lattice1, BlockLatticeXD<T,Descriptor2>& lattice2);
```

Data processor for processing and/or coupling two lattices with potentially different descriptors. Similarly, there is an `SS` version for two scalar-fields, and a `TT` version for two tensor-fields with potentially different dimensionality nDim.

```C++
Class BoxProcessingFunctionalXD_LS<T,Descriptor>
void process(BoxXD domain, BlockLatticeXD<T,Descriptor>& lattice, ScalarFieldXD<T>& field);
```

Data processor for processing and/or coupling a lattice and a scalar-field. Similarly, there is an `LT` and an `ST` version for the lattice-tensor and the scalar-tensor case.

For each of these processing functionals, there exists a “reductive” version (e.g. `ReductiveBoxProcessingFuncionalXD_L`) for the case that the data processor performs a reduction operation and returns a value.

### The methods you need to override
Additionally to the method `process`, a data-processing functional must override three methods. The use of these three methods is now illustrated for the example of class `Invert_1_5_Functional2D` introduced at the beginning of this section:

```C++
BlockDomain::DomainT appliesTo() const
{
    return BlockDomain::bulk;
}

void getModificationPattern(std::vector<bool>& isWritten) const
{
    isWritten[0] = true;
}

Invert_1_5_Functional2D<T,Descriptor>* clone() const
{
    return new Invert_1_5_Functional2D<T,Descriptor>(*this);
}
```

To start with, you need to provide the method `clone()` which is paradigmatic in Palabos (see Section `Programming with Palabos`). Next, you need to tell Palabos which of the blocks treated by the data processor are being modified. In the present case, there is only one block. In the general case, the size of the vector `isWritten` is equal to the number of involved blocks, and you must assign a flag `true/false` to each of them. Among others, this information is exploited by Palabos to decide whether an inter-process communication for the block is needed after execution of the data processor.

The third method, method `appliesTo` is used to decide whether the data processor acts only on the actual domain of the simulation (`BlockDomain::bulk`) or if it also includes the communication envelopes (`BlockDomain::bulkAndEnvelope`). Let’s remember that the atomic-blocks which are the components of a multi-block are extended by a single cell layer (or a multiple cell-layer for extended lattices) to incorporate communication between blocks. This envelope overlaps with the bulk of another atomic-block, and the information is duplicated between the corresponding bulk and envelope cells. It is this envelope which makes it possible to implement a non-local data processor without incurring the danger of accessing out-of-range data. Normally, it is sufficient to execute a data processor on the bulk of the atomic blocks, and it is better to do so, in order to avoid the restrictions listed below when using the envelopes. This is sufficient, because a communication is automatically initiated between the envelopes and the bulk of neighboring blocks to update the values in the envelope if needed. Including the envelope is only needed if (1) you assign a new dynamics object to some or all of the cells (as it is done when you call the function `defineDynamics`), or (2) if you modify the internal state of the dynamics object (as it is done when you call the function `defineVelocity` to assign a new velocity value on Dirichlet boundary nodes). In these cases, including the envelope is necessary, because the nature and the content of dynamics objects is not transferred during the communication step between atomic-blocks. The only information which is transferred is the cell data (the particle populations and the external scalars).

If you decide to include the envelope into the application area of the data processor, you must however respect the two following rules. Otherwise, undefined behavior shall arise.

1. The data processor must be entirely local, because there are no additional envelopes available to cope with non-local data access.

2. The data processor can have write access to at most one of the involved blocks (the vector `isWritten` returned from the method `getModificationPattern()` can have the value true at most at one place).

### Absolute and relative position

The coordinates `iX` and `iY` used in the space loop of a data processor are pretty useless for anything else than the execution of the loop, because they represent local variables of an atomic-block, which is itself situated at a random position inside the overall multi-block. To make decision depending on a space position, the local coordinates must therefore first be converted to global ones:

```C++
// Access the position of the atomic-block inside the multi-block.
Dot2D relativePosition = lattice.getLocation();

// Convert local coordinates to global ones.
plint globalX = iX + relativePosition.x;
plint globaly = iY + relativePosition.y;   /// It is different between here and website. I think it is a mistake on the website.
```

An example is provided in the directory `examples/showCases/boussinesqThermal2d/`. This conversion is a bit awkward, and this is again a good reason to use the one-cell functionals presented in Section Convenience wrappers for local operations, which do the job automatically for you.

Similarly, if you execute a data processor on more than just one block, the relative coordinates are not necessarily the same in all involved blocks. If you measure things in global coordinates, then the argument domain of the method process always overlaps with all of the involved blocks. This is something which is guaranteed by the algorithm implemented in Palabos. However, all multi-blocks on which the data processor is applied are not necessarily working with the same internal data distribution, and have potentially a different interpretation of local coordinates. The argument domain of the method process is always provided as local coordinates of the first atomic-block. To get at the coordinates of the other blocks, a corresponding conversion must be applied:

```C++
Dot2D offset_0_1 = computeRelativeDisplacement(lattice0, lattice1);
Dot2D offset_0_2 = computeRelativeDisplacement(lattice0, lattice2);

plint iX1 = iX + offset_0_1.x;
plint iY1 = iY + offset_0_1.y;
plint iX2 = iX + offset_0_2.x;
plint iY2 = iY + offset_0_2.y;
```

Again, this process is illustrated in the example in `examples/showCases/boussinesqThermal2d/`. This displacement needs to be computed if any of the following conditions is verified (if you are unsure, it is best to compute the displacement by default):

1. The multi-blocks on which the data processor is applied don’t have the same data distribution, because they were constructed differently.

2. The multi-blocks on which the data processor is applied don’t have the same data distribution, because they don’t have the same size. This is the case for all functions like `computeVelocity`, which computes the velocity on a sub-domain of the lattice. It uses a data-processor which acts on the original lattice (which is big) and the velocity field (which can be smaller because it has the size of the sub-domain).

3. The data processor includes the envelope. In this case, a relative displacement stems from the fact that bulk nodes are coupled with envelope nodes from a different atomic-block. This is one more reason why it is generally better not to include the envelope it the application domain of a data processor.

## Executing, integrating, and wrapping up data-processing functionals
There are basically two ways of using a data processor. In the first case, the processor is executed just once, on one or more blocks, through a call to the function `executeDataProcessor`. In the second case, the processor is added to a block through a call to the function `addInternalProcessor`, and then adopts the role of an internal data processor. An internal data processor is part of the block and can be executed as many times as wished by calling the method `executeInternalProcessors` of this block. This approach is typically chosen when the data processing step is part of the algorithm of the fluid solver. As examples, consider the non-local parts of boundary conditions, the coupling between components in a multi-component fluid, or the coupling between the fluid and the temperature field in a thermal code with Boussinesq approximation. In a block-lattice, internal processors have a special role, because the method `executeInternalProcessors` is automatically invoked at the end of the method `collideAndStream()` and of the method `stream()`. This behavior is based on the assumption that `collideAndStream()` represents a full lattice Boltzmann iteration cycle, and `stream()`, if used, stands at the end of such a cycle. The internal processors are therefore considered to be part of a lattice Boltzmann iteration and are executed at the very end, after the collision and the streaming step.

For convenience, the function call to `executeDataProcessor` and to `addInternalProcessor` was redefined for each type of data-processing functional introduced in Section `Categories of data-processing functionals`, and the new functions are called `applyProcessingFunctional` and `integrateProcessingFunctional` respectively. To execute for example a data-processing functional of type `BoxProcessingFunctional2D_LS` on the whole domain of a given lattice and scalar field (they can be either of type multi-block or atomic-block), the function call to use has the form

```C++
applyProcessingFunctional (
    new MyFunctional<T,Descriptor>, lattice.getBoundingBox(),
    lattice, scalarField );
```

All predefined data-processing functionals in Palabos are additionally wrapped in a convenience function, in order to simplify the syntax. For example, one of the three versions of the function `computeVelocityNorm` for 2D fields is defined in the file `src/multiBlock/multiDataAnalysis2D.hh` as follows:

```C++
template<typename T, template<typename U> class Descriptor>
void computeVelocity( MultiBlockLattice2D<T,Descriptor>& lattice,
                      MultiTensorField2D<T,Descriptor<T>::d>& velocity,
                      Box2D domain )
{
    applyProcessingFunctional (
            new BoxVelocityFunctional2D<T,Descriptor>, domain, lattice, velocity );
}
```

## Execution order of internal data processors
There are different ways to control the order in which internal data processors are executed in the function call `executeInternalProcessors()`. First of all, each data processor is attributed to a processor level, and these processor levels are traversed in increasing order, starting with level 0. By default, all internal processors are attributed to level 0, but you have the possibility to put them into any other level, specified as the last, optional parameter of the function `addInternalProcessor` or `integrateProcessingFunctional`. Inside a processor level, the data processors are executed in the order in which they were added to the block. Additionally to imposing an order of execution, the attribution of data processors to a given level has an influence on the communication pattern inside multi-blocks. As a matter of fact, communication is not immediately performed after the execution of a data processor with write access, but only when switching from one level to the next. In this way, all MPI communication required for by the data processors within one level is bundled and executed more efficiently. To clarify the situation, let us write down the details of one iteration cycle of a block-lattice which has data processors at level 0 and at level 1 and automatically executes them at the end of the function call `collideAndStream`:

1. Execute the local collision, followed by a streaming step.

2. Execute the data processors at level 0. No communication has been made so far. Therefore, the data processors at this level have only a restricted ability to perform non-local operations, because the cell data in the communication envelopes is erroneous.

3. Execute a communication between the atomic-blocks of the block-lattice to update the envelopes. If any other, external blocks (lattice, scalar-field or tensor-field) were modified by any of the data processors at level 0, update the envelopes in these blocks as well.

4. Execute the data processors at level 1.

5. If the block-lattice or any other, external blocks were modified by any of the data processors at level1, update the envelopes correspondingly.

Although this behavior may seem a bit complicated, it leads to an intuitive behavior of the program and offers a general way to control the execution of data processors. It should be specially emphasized that if a data processor B depends on data produced previously by another data processor A, you must make sure that a proper causality relation between A and B is implemented. In all cases, B must be executed after A. Additionally, if B is non-local (and therefore accesses data on the envelopes) and A is a bulk-only data-processor, it is required that a communication step is executed between the execution of A and B. Therefore, A and B must be defined on different processor levels.

If you execute data processors manually, you can choose to execute only the processors of a given level, by indicating the level as an optional parameter of the method `executeInternalProcessors(plint level)`. It should also be mentioned that a processor level can have a negative value. The advantage of a negative processor level is that it is not executed automatically through the default function call `executeInternalProcessors()`. It can only be executed manually through the call `executeInternalProcessors(plint level)`. It makes sense to exploit this behavior for data processors which are executed often, but not at every iteration step. Calling `applyProcessingFunctional` each time would be somewhat less efficient, because an overhead is incurred by the decomposition of the data processor over internal atomic-blocks.