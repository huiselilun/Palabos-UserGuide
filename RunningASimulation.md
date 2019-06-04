# Running a simulation<div id="RunningASimulation"></div>
## Time cycles of a Palabos program
The lattice Boltzmann method (or at least, the “classical” lattice Boltzmann method as it is implemented in Palabos) consists of an explicit solver. With a single iteration step, the state of the fluid evolves from a time `t` to the next time `t+dt`. The following discussion is led in the units of the lattice, with `dt=1`. One time iteration of this kind takes the following form in Palabos:

1. To start with, all fluid variables are defined at time `t`. The particle populations are in pre-collision state (also called “incoming populations”).
2. The collision operator is applied to all cells. They are now in post-collision state (also called “outgoing variables”).
3. The streaming operator is applied to the lattice.
4. All data processors are executed, to perform non-local operations or couplings between lattices.
5. The populations are now again in pre-collision state, but at time `t+1`.

The collision step (Step 2) is executed by invoking the method `lattice.collide()`, and the streaming step (Step 3) is called with the method `lattice.stream()`. These two methods can also be combined into a single call to `lattice.collideAndStream()`. On most hardware platforms, the `collideAndStream()` version is computationally more efficient, because it is executed by traveling through the memory of the lattice only once.

The data processors can be executed manually by calling the method `lattice.executeInternalProcessors()`, as explained in Section `Executing, integrating, and wrapping up data-processing functionals`. This is however rarely done, because the data processors are also executed automatically at the end of the function `stream()` and the function `collideAndStream()`. If you’d like to call `stream()` or `collideAndStream()` without having the data processors executed, for example for debugging a program, you can use the domain-version of these functions, for example `lattice.collideAndStream(lattice.getBoundingBox())`.

In Palabos, data processors are always executed after the collision-streaming cycle. Consequently, non-local operations are always executed after the collision-streaming cycle, at a moment where the populations are in a pre-collision state (incoming populations). This bears no loss of generality, though, because executing an operation before the collision of time `t` is equivalent to executing it after the streaming of time `t-1`, right? The only problem arises with the initial condition, as it is most often desired to have the data processors executed exactly once at the very beginning, right after setting up the initial condition. This is achieved by calling the method `lattice.initialize()` right before starting the first iteration step. This method executes the data processors once, and performs an internal communication step inside the block to guarantee that its internal state is consistent.

## At which point do you evaluate data?
To monitor the evolution of a program, it is useful evaluate some hydrodynamic quantities from time to time, such as the average energy:

```C++
pcout << computeAverageEnergy(lattice) << endl;
```

It is recommended to compute hydrodynamic variables always only when the system is in pre-collision state (incoming populations). While this distinction not really matters for the conserved variables density and velocity (they are equal in the pre- and post-collision state), it is important for the non-conserved variables such as the stress tensor. Non-conserved velocity moments can be related to hydrodynamic variables only when they are computed in the pre-collision state. Note that if you use the method `collideAndStream()`, there is no risk for doing things wrong, because you have no access to post-collision variables anyway.

Normally, the computation of hydrodynamic variables like the average energy in the example above is performed right after the call to the method `collideAndStream()`, because at this point all hydrodynamic variables are well-defined, and correspond to the same moment in time (between collision and streaming the velocity is well defined, but the strain-rate is not). An exception is made for the internal statistics of lattice. Internal statistics are automatically computed without any impact on performance (at least not in serial program; for the parallel case, see the discussion in Section `Controlling the efficiency`), as a side-effect of executing the collision step. They are however evaluated for the fluid variables at time `t`, during the collision-streaming cycle which carries the system from time `t` to time `t+1`. It is therefore usual to access the internal statistics after the call to the method `collideAndStream()`. Computing the average energy as in the example above before collision-and-streaming produces the same result as accessing the average energy from internal statistics, as in the example below, after collision-and-streaming:

```C++
pcout << getStoredAverageEnergy(lattice) << endl;
```

## Other important things to do
Numbers are often difficult to interpret. It is therefore useful to regularly produce images in your program, so that you can monitor the state of simulation, identify problems as soon as possible, and re-run the program when needed. Section `Producing images in 2D and 3D simulations` explains how to do this.

Finally, it is always good to save the state of a simulation from time to time, in order not to loose everything when the computer crashes, and in order to be able to recover the data if you forgot to produce a crucial output file for post-processing. This is explained in Section `Checkpointing: saving and loading the state of a simulation`.