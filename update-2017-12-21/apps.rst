Application code
================

QUDA integration
--------------------

Calling QUDA solver (currently only staggered fermion) works for
multi node and multi GPU.

.. code:: nim
  var s = g.newStag  # g is a gauge field
  s.solve(dest, src, m, resid)    # By default will call QUDA solver if compiled in
  s.solve(dest, src, m, resid, cpuonly = true)    # Always use CPU


Staggered analysis
--------------------

Added link smearing (AsqTad, HISQ, nHYP)

Dilution iterators

.. code:: nim
  for dl in dilution(diluteType):    # diluteType = dkEvenOdd or dkCorners3D
    # loop over groups of sites for a dilution pattern
    for i in src.sites(dl):
      # do something with the site index ``i`` in dilution pattern ``dl``

Used to port LSD 8f scalar trace analysis code from FUEL to QEX.


Doubling lattice
--------------------

Wrote simple application to read gauge field,
replicate the lattice periodically (doubling each dimension)
and write it out.
Used for providing initial lattice for larger ensemble.
Currently only runs on single rank.

Used to test multiple lattices and indexing routines.


Eigensolver development
-----------------------

- Ported existing SVD-based eigensolver from Qlua/QOPQDP to QEX
  + Calls external ``LAPACK`` routines
- Tuned algorithm and tested on BG/Q
- Tested deflation with computed eigenvectors
- PRIMME integration into QEX (optional compile time dependence)
  + External package (managed by ``Nimble``) for wrapping PRIMME C interface
  + Still need to manually compile PRIMME C library
- Presented results at `Lattice 17`_
- Tried Chebyshev preconditioning in PRIMME
  + External ``chebyshev`` package

.. _Lattice 17: https://makondo.ugr.es/event/0/session/97/contribution/409


.. include:: mg.rst
