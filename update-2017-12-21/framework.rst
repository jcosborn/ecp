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

portability GPU/CPU arrays (not integrated in QEX yet)
--------------------

compatibility, reproducibility with FUEL (-d:FUELCompat)
--------------------

optimizeAst (temporary let)
--------------------

new CG interface
--------------------

Spin & wilson dslash
--------------------

.. include:: spin.rst
