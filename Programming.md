# Programming with Palabos<div id="Programming"></div>

## Quick overview: programming guidelines
C++ is an exceptionally free language which imposes little restrictions on programming style and coding structure. It is therefore not sufficient to know the language C++ to start working with a library like Palabos; you also need to get familiar with the particular algorithmic choices and programming guidelines which were issued for the library. Once you know the rules, you’ll understand more precisely what the code does and avoid the many pitfalls of programming such as memory leaks, inconsistent parallel program execution, unexpected behind-the-scene effects of object-oriented programming, and more. The user’s guide tries to get you familiarized progressively with these concepts. Whenever a new concept is introduced which requires a certain amount of discipline from the programmer to guarantee correct results, the importance of this concept is highlighted in form of a rule:

**Rule for this topic**: You’ve been warned: follow this rule or your program’s result is undefined.

Some of the most fundamental programming rules in Palabos are reviewed in this section, in order to clarify aspects of the code that appear puzzling at a first reading.

### Basic data types
In Palabos, the signed and unsigned integer data types `int` and `unsigned int` are systematically replaced by `plint` and `pluint` (with only a few exceptions). The reason for this is based on 64-bit compatibility of the code and is easy to understand. A 32-bit signed integer can count from -2 Billion to +2 Billion, whereas a 64-bit integer goes far beyond this. The 32-bit version is therefore often insufficient in simulations with data sets larger than 2 GB. Errors in relation with the 32-bit vs. 64-bit issue are highly unpleasant, and believe me, you don’t want them. Under this light, guess what’s the size of the type `int` on a 64-bit platform? For some obscure reason, the answer is platform dependent, but most current 64-bit architectures made the choice of the data type `int` being 32-bit. The solution to this problem are the Palabos data types `plint` and `pluint` which are 32-bit on a 32-bit machine, and 64-bit on 64-bit machine. In conclusion, always prefer the integer data types `plint` and `pluint` over `int` and `unsigned int`, unless you have specific reasons in favor of `int` and you are very, very certain that your reason is valid.

Another cherished C++ object which you must abandon is the output stream buffer `cout`. In an MPI-parallel program each thread writes to the terminal through `cout`. To avoid this and observe a consistent behavior between sequential and parallel programs, use the buffer `pcout` instead. Similarly, replace `cerr` by `pcerr` and `clog` by `pclog`. For file output, replace `ofstream` by `plb_ofstream`.

### Memory management
As C++ has no garbage collector, objects allocated dynamically with the operator `new` must be disposed of manually with the operator `delete`. Keeping track of these objects and de-allocating them at the right moment is an unpleasant burden imposed by this language. Unlike other libraries, Palabos does not include an external garbage collector or similar construct to work around the problem. Instead, discipline is the solution, and you’ll avoid problems by respecting a few rules.

In general, when a function in Palabos takes a pointer to an object as a parameter, this means the function (or the object, in case of a class method) assumes responsibility over the object. You’re not responsible for its deletion any more. As a matter of fact, you are not even *allowed* to delete it or, by all means, do anything with it (because you don’t know at which point it is deleted). Reversely, when you get a pointer returned to you from a function, you become the owner of the object pointed to. Do what you need to do with it, and don’t forget to delete it afterward.

Issues around the question of who deletes an object pointed to by many instances are most often avoided in Palabos through a one-pointer-per-object approach. Whenever a second instance needs to access an object which is already in use, it creates its own copy. To make such copies possible, all Palabos classes (or at least those which use inheritance) have a method `clone`:

```C++
A* object1 = new A;
A* object2 = object1->clone();
```

You need to implement this method whenever writing a class which inherits from Palabos types. Luckily enough, the method `clone` has always exactly the same shape:

```C++
A* A::clone() const {
    return new A(*this);
}
```

There’s unfortunately no mechanism in C++ for doing this automatically, but you’ll soon be in the habit of writing it without noticing. As a side remark, it is not advised to define the method `clone` by different means than the one suggested here, for the sake of uniformity and readability of the code. Some classes will require that you write a copy constructor in order for this form of the method `clone` to be valid. However, if you write this type of classes, it is assumed that you know what you are doing. And that you knew you had to define the copy constructor.

### Arrays
The right choice of a data type to store arrays of values depends basically on the size of the data. Large data sets, like the particle populations of a lattice Boltzmann simulation, are of course stored in one of Palabos’ BlockLattice types. These specialized and parallelized data types are after all the reason why you started using Palabos in the first place. The syntax for instantiating such a type looks as follows:

```C++
// Instantiate a nx*ny*nz D3Q19 lattice with double-precision
//   floating point numbers.
MultiBlockLattice<double,D3Q19descriptor> lattice(nx,ny,nz);

// Instantiate a nx*ny*nz matrix of double-precision floating
//   point scalars.
MultiScalarField<double> field(nx,ny,nz);
```

Small, non-parallelized arrays are needed for example to store parameters of a simulation, or a couple of results which require little memory. The standard library of C++ offers the data type vector for the instantiation of dynamically sized value arrays. The vector type is efficient, elegant and easy to use:

```C++
plint numspecies = 5;
// Instantiate a vector of double-precision floating point
//   numbers with 5 elements.
vector<double> viscosities(numSpecies);
viscosities[3] = 0.63;
```

It is strongly recommended to prefer the type `vector` over old-style dynamical arrays allocated with the `new []` operator, to reduce the risk of programming errors and to improve the readability of the code.

For very small, non-parallelized and fixed-sized arrays, Palabos defines the type `Array`. It is commonly used to store small physical vectors, such as, a single velocity value:

```C++
// Instantiate a fixed-size array of type double and with
//   3 elements. The values can be initialized directly in
//   the constructor for arrays of size 2 and 3.
Array<double,3> velocity(0., 0., 0.5);
velocity[0] = 0.1;
```

Again, the type `Array` should be preferred over the fixed-size array inherited in C++ from the language C (`double velocity[3]`). From an efficiency point of view, both choices are equivalent, as the type `Array` is defined on top of the C-array through a template mechanism. It offers however specific advantages. In particular, it performs range checks when Palabos is compiled in debugging mode, a feature which can substantially reduce the time spent in debugging.

### Velocity and Density
Velocity and mass density are the two most recurrent macroscopic variables in lattice Boltzmann. It is therefore surprising that variables with names like `rho` or `u` rarely occur inside the Palabos code (nothing prevents you from perusing them in end-user code, though). Instead, internal Palabos structures most often work with the variables `rhoBar` and `j`. The variable `j` is nothing else than the first-order velocity moment which, for most models, is identical with the fluid momentum: `j = rho * u`. The definition of the variable `rhoBar` on the other hand can be customized. By default, it is defined as `rhoBar=rho-1` and is used to improve the numerical accuracy of the method. Indeed, as the density is often close to 1, you gain significant digits in the representation of floating point variables by representing a quantity fluctuating around 0 rather than 1.

Without getting into details, here are a few rules for switching between the `rho / u` and the `rhoBar / j` representation:

```C++
// Use custom definitions in lattice descriptor to compute rho from rhoBar
rho = Descriptor<T>::fullRho(rhoBar);

// Use custom definitions in lattice descriptor to compute rhoBar from rho
rhoBar = Descriptor<T>::rhoBar(rho);

// Momentum is equal to density times velocity (component-wise)
j[iD] = rho * u[iD];

// Velocity is equal to inverse-density times momentum (component-wise)
u[iD] = 1./rho * j[iD];

// Inverse-density can be computed right away from rhoBar. Depending on
//   the custom definitions in the descriptor, this is substantially more
//   efficient than computing 1./rho, because the Taylor expansion of the
//   inverse-function is truncated at an appropriate position.
u[iD] = Descriptor<T>::invRho(rhoBar) * j[iD];
```

### Parallelism
The library Palabos uses a single-program-multiple-data (SPMD) approach to parallelism, no matter whether a shared-memory or distributed-memory platform is used. This means that your program looks the same when you write it for sequential or parallel execution (at least, it should look the same, if you are a little careful). In a distributed-memory environment (“a cluster”), multiple instances of the same program are started in distinct threads. In that case, Palabos’ distributed objects (`MultiBlockLattice`, `MultiScalarField`, and `MultiTensorField`) are parallelized, i.e. the amount of memory they allocate whithin each thread is divided by the total number of threads. All other data is, in general, duplicated over all threads. There are a few exceptions to this rule, though, which are highlighted in the user’s guide. For example, the default behavior of reduction operations (such as computing the average density of a block lattice) is to store the result in the main thread only, for efficiency reasons.

## Non-intrusive program development with Palabos<div id="NIntrusive"></div>
For physicists and scientific programmers with little experience in software engineering, collaborative programming often means exchanging code snippets by e-mail and integrating them manually into existing code. In this approach, every programmer possesses an entirely independent version of the code, and adjusts its components to fit a specific task.

This section tries to convince you to not adopt this approach with Palabos, and to use the code of this project like a library instead of a code template which is adjusted to your local needs. There are several reasons why such a non-intrusive way of working with Palabos is more efficient in the short and the long run. First of all, the Palabos source code is quite large, with more than 50‘000 lines of code, and starting to modify the code as a whole rapidly degenerates into a very time consuming task. A second, even more compelling reason is that, as soon as you modify your local version of Palabos, your copy becomes independent of the main version. It becomes difficult or even impossible to take profit of subsequent Palabos releases, with bug fixes, efficiency improvement, and new features. Furthermore, you loose the possibility to easily exchange code snippets with colleagues, because you’re somehow forced to shuffle around full copies of the Palabos source all the time. Compare this with the relative simplicity of exchanging just two code files based on the official Palabos version, one of which implements a new dynamics class for your own LB model, and the other a sample application.

Imagine that you are producing advanced 3D graphics based on the library OpenGL. You would certainly think about how to formulate your problem in terms of the interface offered by OpenGL, instead of modifying the source of OpenGL itself right? Think of Palabos in the same way, and you’re guaranteed to save a lot of time, even though you are forced comply with a certain amount of decisions which were taken for the Palabos project.

### Extending without modifying
Imagine you want to write a LB model similar to BGK, with small differences. The first thing to do is to check out section `Implemented fluid models` and make sure it is not yet implemented in Palabos. If it’s not, you’ll probably want to view the files `src/basicDynamics/isothermalDynamics.h` and `.hh` and discover that the class `BGKdynamics` inherits from `IsoThermalBulkDynamics` and implements the three methods `clone()`, `collide()`, and `computeEquilibrium()`. From the preceding discussion, you understand that the next step is not to modify the implementation of `collide()` in BGKdynamics, because (among many other reasons) you would then loose the ability to use both the original BGK dynamics and your new model at the same time in an application. Instead, create two new files `myNewModel.h` and `myNewModel.hh`, containing a class `MyNewDynamics` which inherits from `IsoThermalBulkDynamics` and defines the same three methods as `BGKdynamics`, adapted to your case. This implementation is clean: it does not conflict with updates to newer Palabos releases, and you can share it with colleagues by exchanging just two files.

Your new files can be located in any local directory, independent of the Palabos directory tree. Just make sure to add this directory under the keyword `includePaths` in the Makefile, so that it can be found during the compilation process.

### Which parts of Palabos are extensible?
There are, roughly speaking, three ways to extend Palabos: (1) **Write a new dynamics class**, (2) **Write a new data processor**, and (3) **Write a new lattice descriptor**. The dynamics class is used to define a new local collision step, whereas a new lattice descriptor means a new lattice topology (discrete velocities, weights, etc.), and the data processor is for everything else. Technically speaking, you can also adapt Palabos’ behavior by writing a custom class for floating point numbers, but this is a less common thing to do. Once you have written new dynamics classes, data processors, and/or lattice descriptors, you might of course also want to write convenience code such as wrapper functions which automatically deliver the right data processor for a given task, or which automatically add the data processor to a block-lattice. As an example of such convenience code, consider the `OnLatticeBoundaryConditionXD` classes which automatically add dynamics objects, data processors, or both to a block-lattice to create a boundary condition.

All other parts of Palabos are not extensible. It is for example a bad idea to write a new class similar to the `MultiBlockLatticeXD`, or to re-invent the `MultiScalarFieldXD`. Nothing prevents you from doing so, of course, but redefining such a class breaks the concept of modularity in Palabos. For short, a custom implementation of a data processor is able to take profit of future innovation in Palabos, such as new approaches to parallelism, whereas a user-invented version of a `MultiBlockLatticeXD` is difficult to write and not supported by subsequent Palabos releases.

The good news is that the extensible parts of Palabos allow a lot of flexibility. Take as an example the implementation of the Shan/Chen model for the 2D and the 3D implementation of single-component multi-phase and immiscible multi-component fluids. This was simply achieved by writing a new lattice descriptor and a new data processor, without modification to any structural parts of Palabos. And, the size of the resulting source code makes up for as little as one percent of the whole project.

Sometimes, the choices made in Palabos may seem a little restrictive, though. A problem appeared for example when we implemented the thermal fluid model with Boussinesq approximation. In this case, the temperature field follows an advection-diffusion equation which is solved by means of a LB model with linear equilibrium. From the previous discussion, it is clear that this simply requires the definition of a new dynamics class which implements the collision step of the advection-diffusion equation. The problem is that the order-0 moment of the particle populations is always called “rho” in Palabos (this is imposed by the interface of dynamics classes), while the order-0 moment of the temperature model is, well, the temperature. At this point, we had to overcome our strong sense of discipline and order, accept that the temperature is called “rho” at two or three places inside the code, add a few comments to make sure other programmers are not confused by this, and wrap the final product into convenience functions which call the temperature temperature. Programming is always an exercise in compromise, and it appears that avoiding to re-write hundreds of thousands of lines of code is worth the small heresy of calling the temperature “rho” on a few occasions.