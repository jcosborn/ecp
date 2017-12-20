QEX framework updates
=====================

Random sources
--------------------

Supports ``gaussian``, ``uniform``, ``u1``, ``z2``, and ``z4``.

.. code:: nim
  var
    lo = newLayout([8,8,8,8])
    r = lo.newRNGField(RngMilc6, seed)
    eta = lo.ColorVecto
  eta.z4 r

Quda integration
--------------------

Calling Quda inverter (currently only staggered fermion) works for multi GPU / multi node.

.. code:: nim
  var s = g.newStag
  s.solve(dest, src, m, resid)    # By default will call Quda solver if compiled in.
  s.solve(dest, src, m, resid, cpuonly = true)    # Always use CPU.

where ``g`` is a gauge object.

Profiling
--------------------

.. code:: nim
  tic("Timing Label")    # Begin measuring time duration for a label
  tic()    # Begin measuring time duration without a label (record file & line number)
  DoSomething()
  toc()    # End measuring time duration from the last ``tic()`` call
  toc("Timing Label")   # End measuring time duration for the specific label

IO (parallel)
--------------------

Calls ``QIO`` and use ``QIO_PARALLEL``.

Portability GPU/CPU
--------------------

- Exploration of shared GPU/CPU arrays backends
- Extensive compile time options for memory layout
- Saturates GPU/CPU bandwidth for optimal data sizes
- Not integrated in QEX yet
- Check out the repository_

.. _repository: https://github.com/jcosborn/cudanim

Compatibility, reproducibility with FUEL
--------------------

Introduced a compilation flag, ``-d:FUELCompat``, for reproducibility test against FUEL.

new CG interface
--------------------

.. code:: nim
  var cg = newCgState(x=v2, b=v1)
  cg.solve(oa, sp)    # The default solve

  v2 := 0
  cg.reset
  sp.maxits = 0
  while cg.r2 > cg.r2stop:   # Solve with finer control
    sp.maxits += 10
    cg.solve(oa, sp)
    let c = cg.x.norm2
    echo cg.iterations, ": ", c


Spin & wilson dslash
--------------------

.. include:: spin.rst
