Analysis
========

This is taken from the Analysis Framework document.

Below is a simple example showing the calculation of a staggered meson
propagator starting from a point source.  It also calculates meson
propagators with symmetric shifts at the source and sink.  In this
example, at the sink, the results are summed over time slices, but on
each time slice the sums are done on separate subsets according to the
eight corners of a :math:`2^3` spatial cube,
:math:`(x\,\mathrm{mod}\,2, y\,\mathrm{mod}\,2, z\,\mathrm{mod}\,2)`.

In this example, the gauge field file (USQCD SciDAC format), quark
mass, source-point location, and solver stopping residual (squared)
are provided on the command line, as, for example,

.. code::

  stagMesons lat1.scidac -d:mass:0.001 -d:pt:0,0,0,10 -d:rsq:1e-16

.. code:: Nim

  qexInit()
  # create layout (lo) and gauge field (g) using program arguments
  defaultSetup()
  var src = lo.ColorVector()
  var dest = lo.ColorVector()
  var r = lo.ColorVector()
  var destS = lo.ColorVector()
  threads:
    g.setBC
    g.stagPhase
  var s = newStag(g)
  var m = floatParam("mass", 0.1)  # get mass from command line (default 0.1)
  let nt = lo.physGeom[^1]
  # create some correlator arrays, 8 is for the 8 corners at the sink
  var cl = newSeq[array[8,float]](nt)   # local
  var cx = newSeq[array[8,float]](nt)   # x symShift
  var cy = newSeq[array[8,float]](nt)   # y symShift
  var cz = newSeq[array[8,float]](nt)   # z symShift
  var cs = [cx.addr,cy.addr,cz.addr]

  var pt = intSeqParam("pt", @[0,0,0,0])  # source point
  var t0 = pt[^1]
  for ic in 0..2:  # loop over color for point source
    src.pointSource(pt, ic)
    mysolve(dest, src)
    cl += stagLocalMesons(dest, dest, t0)
    for mu in 0..2:
      symShift(r, g[mu], src, mu)
      mysolve(destS, r)
      symShift(r, g[mu], destS, mu)
      cs[mu][] += stagLocalMesons(dest, r, t0)

  let f = nt.float/lo.physVol.float
  printLocalMesons(cl, f)
  for mu in 0..2:
    printLocalMesons(cs[mu][], f)

  qexFinalize()


Here we give a description of the key parts of the code.

===== ===========
lines Description
----- -----------
1     Initialize QEX.

3     The ``defaultSetup()`` call reads the gauge field file name from
      the command line, gets the lattice dimension from it, and
      creates the layout (``lo``) and gauge field (``g``)
      objects from data in that file.  The ``lo`` and ``g``
      objects are injected into the local scope so they can be used
      later in the code.

4-7   Create some staggered fermion fields (lattice color vectors).

9-10  Set default boundary conditions (antiperiodic) and staggered
      phases (following MILC conventions) for gauge field ``g``.
      This operation multiplies each gauge link by the appropriate
      phase. The code appears inside a ``thread:`` block so that it
      runs with threading.

11    Create a staggered fermion object from gauge field.

12    Set mass value from command line (e.g. ``-d:mass:0.001``).

13    Get number of timeslices (``nt``) from last element of physical
      geometry array (``^1`` indexes the last element of an array).

15-18 Create sequences (a dynamically sized array) of length ``nt``
      with elements of type array of 8 floats (float in Nim is double
      precision).  These will be used for the correlators on each
      timeslice and spatial hypercube corner.

19    Create array of pointers to the ``cx``, ``cy`` and ``cz``
      sequences, for later convenience.

21    Get source point from command line (e.g. ``-d:pt:0,0,0,10``).

22    Get source timeslice from last element of source point.

23    Loop over colors (for :math:`N_c = 3`).

24    Set field ``src`` to a point source at point ``pt`` with color
      ``ic`` with default value of :math:`1.0`.

25    Solves staggered Dirac equation against source ``src``, putting
      result in ``dest``. The procedure ``mysolve`` is a
      user-defined helper routine that gets the appropriate parameters
      (such as required residual norm) from the command line and then
      calls the appropriate solver routine.  An example code is given
      below.

26    Perform contractions (inner product) of 2 fields (in this case they
      are the same field, ``dest``) along timeslices and spatial
      corners and return the result.  The timeslice of the source
      (``t0``) is passed in so that the result can be rotated to
      start relative to ``t0``.

27    Loop over spatial directions.

28    Perform a symmetric shift (parallel transport actually, from both
      forward and backward directions) of source vector, ``src``, in
      direction ``mu``, using gauge field ``g[mu]`` to multiply
      source vector, and store the result in field ``r``.

29    Solve against shifted source ``r`` and store result in ``destS``.

30    Perform a symmetric shift in the direction ``mu`` on
      ``destS`` and store the result in ``r``.

31    Perform contractions between the original point-source quark and
      the symmertic shifted quark.  ``cs[mu]`` gives the pointer to
      the sequence for the ``mu`` direction, and the final
      ``[]`` dereferences that pointer (like ``*`` in C) to give
      the sequence itself (which is then updated by adding in the new
      correlator).

33-36 Calculate the normalization factor and print results.

38    Finalize QEX.
===== ===========

We give an example implementation of the ``mysolve`` routine below.
It is defined as a Nim template so that the code will be explicitly
inlined at the call site (like a macro in C).  This way the code can
use the previously defined staggered object, ``s``, quark mass,
``m``, and residual vector field, ``r``, from the scope where it
is called.

.. code:: Nim

  template mysolve(dest, src) =
    var sp = initSolverParams()
    sp.r2req = floatParam("rsq", 1e-12)
    s.solve(dest, src, m, sp)
    echo "solve iterations: ", sp.finalIterations
    echo "solve seconds: ", sp.seconds
    threads:
      s.D(r, dest, m)
      threadBarrier()
      r := src - r
      threadBarrier()
      echo "final residual norm2: ", r.norm2


An explanation follows:

===== ===========
lines Description
----- -----------
2-3   Create solver parameters structure and set the residual norm squared
      request from the command line (with a default of :math:`10^{-12}`).

4     Run the solver using staggered object ``s``, with mass ``m``
      and solver parameters ``sp``.

8-12  Calculate residual and print norm2.
===== ===========

Below are examples of a few more QEX features that could be of
interest in an analysis campaign.

.. code:: Nim

  var coef = HypCoefs(alpha1:0.4, alpha2:0.5, alpha3:0.5)
  echo "Hyp coefficients: ", coef
  var sg = lo.newGauge
  coef.smear(g, sg, info)

The code above creates a Hyp smearing object and sets a new gauge field
``sg`` with the Hyp smeared gauge field.  ``info`` is a performance
info object used to return information such as number of flops performed.

.. code:: Nim

  var hc = hisqCoefs()
  echo "HISQ coefficients: ", hc
  var fl = lo.newGauge()
  var ll = lo.newGauge()
  hc.smear(g, fl, ll)
  var s = newStag3(fl, ll)

The code above creates a HISQ smearing object and sets two new gauge fields
``fl`` and ``ll`` with the fat and long links of the HISQ smeared
gauge field.
A new staggered object (which supports a 1-link and 3-link stencil)
is then created from the fat and long links.
This staggered object is actually the same type as the one created for
a 1-link only stencil.
Internally the object knows what stencil to use and will apply the correct
one for the Dslash and solver operations.

.. code:: Nim

  var
    rf = newRNGField(RngMilc6, lo, seed)
    eta = lo.ColorVector
  threads:
    case source_type
    of "Z4": eta.z4 rf
    of "Z2": eta.z2 rf
    of "U1": eta.u1 rf
    of "Gauss": eta.gaussian rf

This creates a random field, ``rf``, using the MILC RNG type,
then sets the field ``eta`` with random numbers based on the
distribution type from the input string ``source_type``.

.. code:: Nim

  let dilute_type = strParam("dilute_type", "EO").parseDilution  # EO, CORNER
  ...
  for dl in dilution(dilute_type):
    threads:
      tmps := 0
      threadBarrier()
      for i in tmps.sites(dl):
        if lo.coords[^1][i] == t0:
          tmps{i} := eta{i}

The code above sets a dilution pattern read in from the command line
(only EO and CORNER currently supported) and then loops over the subsets,
``dl``, within the dilution pattern.
The source field ``tmps`` is first set to zero, then it loops over all
sites within the current dilution subset and if the site is on the source
timeslice, ``t0``, it copies the random source, ``eta``,
at that site over to ``tmps``.

There is also now a gradient flow routine

.. code:: Nim

  g.gaugeFlow(steps=6, eps=0.01):
    echo "WFLOW: ", wflowT
    wflowG.printPlaq

This is implemented as a template that inserts the statement block
after every step.  The ``wflowT`` and ``wflowG`` are the
two variables exposed to the arbitrary statement block for the
flow time and the gauge field at this flow time.
