Application code
================

Added link smearing (AsqTad, HISQ, nHYP)
--------------------

HISQ example
~~~~~~~~~~~~~~~~~~~~~
.. code:: nim
  threads:
    g.setBC
    g.stagPhase
  var hisq: HisqCoefs
  hisq.init
  var
    fl = lo.newGauge
    ll = lo.newGauge
  hisq.smear(g, fl, ll)
  var s = newStag3(fl, ll)

nHYP example
~~~~~~~~~~~~~~~~~~~~~
.. code:: nim
  var
    info: PerfInfo
    hyp = HypCoefs(alpha1:0.4, alpha2:0.5, alpha3:0.5)
    sg = lo.newGauge
  hyp.smear(g, sg, info)
  threads:
    sg.setBC
    sg.stagPhase
  var s = sg.newStag

Eigensolver development
--------------------
- Ported existing SVD-based eigensolver from Qlua/QOPQDP to QEX
  + Calls external ``LAPACK`` routines
  .. code:: nim
    import hisqev
    type MyOp = object
      s: type(s)
      r: type(r)
      lo: type(lo)
    var op = MyOp(r:r,s:s,lo:lo)
    template rand(op: var MyOp, v: any) =
      gaussian(v, op.r)
    template newVector(op: MyOp): untyped =
      op.lo.ColorVector()
    template apply(op: MyOp, r,v: typed) =
      threadBarrier()
      stagD(op.s.so, r.field, op.s.g, v.field, 0.0)
    template applyAdj(op: MyOp, r,v: typed) =
      threadBarrier()
      stagD(op.s.se, r.field, op.s.g, v.field, 0.0, -1)
    template newRightVec(op: MyOp): untyped = newVector(op).even
    template newLeftVec(op: MyOp): untyped = newVector(op).odd
    var opts: EigOpts
    opts.initOpts
    opts.nev = intParam("nev", 16)
    opts.nvecs = intParam("nvecs", 32)
    opts.rrbs = intParam("rrbs", opts.nvecs)
    opts.relerr = 1e-4
    opts.abserr = 1e-6
    opts.svdits = intParam("svdits", 500)
    opts.maxup = 10
    var ev = hisqev(op, opts)
- Tuned algorithm
- Tested deflation with computed eigenvectors
- PRIMME integration into QEX (optional compile time dependence)
  + External package (managed by ``nimble``) for wrapping PRIMME C interface
  + Still need to manually compile PRIMME C library
- Presented results at Lattice 17
- Tried Chebyshev preconditioning in PRIMME
  + External ``chebyshev`` package

staggered meson propagator
--------------------

An example for vectorized contractions.

Dilution
--------------------

.. code:: nim
  for dl in dilution(diluteType):    # diluteType = dkEvenOdd or dkCorners3D
    # loop over groups of sites for a dilution pattern
    for i in src.sites(dl):
      # do something with the site index ``i`` in dilution pattern ``dl``

Doubling lattice
--------------------

Replicate lattice periodically and double the length of each
dimension. It is a single MPI rank application for showcasing
coexistence of multiple layouts, query of coordinates, access of
single site contents, and read/write of lattice configurations.
