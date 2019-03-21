OpenCL Support
--------------

Initial tests with generating OpenCL kernel strings at compile-time
from Nim code.
Kernels can then be compiled and executed at runtime.

.. code:: Nim

  let s = toOpencl:
    proc vecAdd(a: global[ptr UncheckedArray[float32]],
                b: global[ptr UncheckedArray[float32]],
                c: global[ptr UncheckedArray[float32]],
                n: int32) {.oclKernel.} =
      let id = get_global_id(0)
      if id < n:
        c[id] = a[id] + b[id]


produces the static string at compile-time

.. code:: C

  kernel void vecAdd(global float (*a), global float (*b), global float (*c), int n)
  {
    int id=get_global_id(0);
    if(id<n){
      c[id]=a[id]+b[id];
    }
  }

Since kernels are compiled and linked at runtime, could set parameters (Nc)
at runtime and compile.

