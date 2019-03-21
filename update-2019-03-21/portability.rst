Portability
===========

CUDA
----

We revived old CUDA code from two years ago, and started integration with QEX.
In the process we are improving the robustness of the compile time AST transformations,
as this also benefits other backends, OpenMP and OpenCL.

- Explicit compile time procedure inlining.
- Explicit compile time loop unrolling.
- Automatic compile time reduction of let bindings (implicit struct copies).


OpenMP
------

Xiao-Yong is currently working on integrating OpenMP target offload support in QEX,
keeping the same high-level interface as the CUDA support.
Currently we are testing OpenMP runtime library routines for
manual memory management and scheduling.


OpenCL
------

