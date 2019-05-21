# Utilities<div id="Utilities"></div>
## Timer
Timer objects can be used to make time measurements and performance evaluations. They measure time intervals in terms of wall-clock time, with help of the C function `clock()` in serial programs, and with the MPI function `MPI_Wtime()` in parallel programs. A timer object is accessed through a command like

```C++
global::timer("nameOfTimer")
```

where `nameOfTimer` is an arbitrary string of your choice. You can use as many different timer objects as needed, and the timers are automatically created the first time you refer to them. Timer objects are global and can measure time intervals cumulatively. You can therefore start measuring a time interval in one file of the program, and then add up more time from a place in another file, by referring to the timer with the same string argument.

A timer behaves like a stop-watch. When you call the method `global::timer("nameOfTimer").start()`, it starts measuring time. After calling the method `stop()`, the clock stops moving, but proceeds from where it was the next time you call `start()`. To reset the clock to zero, call the method `reset()` or `restart()`. And example for the usage of a timer object is provided in the directory `examples/codesByTopic/smagorinskyModel`.

## Value tracer
Value tracers are most often used to decide when a simulation has reached a steady state. They track the time evolution of a scalar value, such as the average energy, and decide that a steady state is reached when the standard deviation of this quantity, as measured over a fixed time windows, falls below a given threshold value. An example is provided in the directories `examples/showCases/boussinesqThermal2D` and `3D`.