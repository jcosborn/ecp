QEX framework updates
=====================

Field I/O
--------------------

Added gauge writer.

Random sources
--------------------

Now supports ``gaussian``, ``uniform``, ``u1``, ``z2``, and ``z4``.

.. code:: nim
  var
    lo = newLayout([8,8,8,8])
    r = lo.newRNGField(RngMilc6, seed)
    eta = lo.ColorVector
  eta.z4 r

Introduced a compilation flag, ``-d:FUELCompat``, for reproducibility
test against FUEL.

Portability GPU/CPU
--------------------

- Exploration of shared GPU/CPU array backends
- Using coalesced pointer approach
- Extensive compile time options for memory layout
- Saturates GPU/CPU bandwidth for optimal data sizes
- Presented at `DOE COE Performance Portability Meeting 2017`_
- Not integrated in QEX yet
- Check out the repository_

.. _DOE COE Performance Portability Meeting 2017: http://www.lanl.gov/asc/doe-coe-mtg-2017.php

.. _repository: https://github.com/jcosborn/cudanim

.. include:: spin.rst

New CG interface
--------------------

*Resumable* solvers, as discussed on the Solver call.

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
