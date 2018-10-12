HMC
===


Quasi-Newton HMC
----------------

Quasi-Newton HMC code has been developed by Xiao-Yong for Critical Slowing
Down project.  It is currently being tested on U(1) Gauge Theory.

The code implements a BFGS ensemble HMC algorithm using QEX.

It also calculates eigenvalues of the mass matrix using PRIMME.

The MD framework is available as a separate Nimble package.


MD Framework
------------

We have a new nimble package for molecular dynamics evolution, MDevolve_.
The following is an example, which is a snippet took from `puregaugehmc.nim`_,
showing the usage.

.. code:: Nim

  proc mdt(t:float) =
    threads:
      for i in 0..<g.len:
        for e in g[i]:
          let etpg = exp(t*p[i][e])*g[i][e]
          g[i][e] := etpg
  proc mdv(t:float) =
    f.gaugeforce2(g, gc)
    threads:
      for i in 0..<f.len:
        for e in f[i]:
          let tf = (-t)*f[i][e]
          p[i][e] += tf
  
  # For FGYin11
  proc fgv(t:float) =
    f.gaugeforce2(g, gc)
    threads:
      for i in 0..<g.len:
        for e in g[i]:
          let etfg = exp((-t)*f[i][e])*g[i][e]
          g[i][e] := etfg
  var gg = lo.newgauge
  proc fgsave =
    threads:
      for i in 0..<g.len:
        gg[i] := g[i]
  proc fgload =
    threads:
      for i in 0..<g.len:
        g[i] := gg[i]

  let
    # H = mkOmelyan4MN5FV(steps = steps, V = mdv, T = mdt)
    H = mkFGYin11(steps = steps, V = mdv, T = mdt, Vfg = fgv,
      save = fgsave(), load = fgload())

  H.evolve tau

In this part of the code, we define ``mdt`` and ``mdv``
as the usual ``T`` and ``V`` update, respectively.
The procedure ``fgv``, which updates the gauge field directly
using the force, is created specifically for the
force gradient update using the approximation
proposed by Yin & Mawhinney, 2011.
The helper procedure ``fgsave`` and ``fgload`` is used to
save the gauge field temporarily and to load it back.
All of these procedure operate on the momentum field, ``p``,
and the gauge field, ``g``, which are defined in the same
scope of where these procedures are defined.
Since Nim allows procedure definition inside a local scope,
we can define ``p`` and ``g`` in a local scope.

The call to ``mkFGYin11`` creates an object that knows how
to integrate the system in the number of steps, ``steps``,
where each step performs the simplest form of the force gradient
integrator ``V T G T V``.
This integration is then performed by calling ``H.evolve tau``,
which evolve the system along a trajectory length ``tau``.

An object that does not use force gradient update can be created
similarly, by calling, as an example,
``mkOmelyan4MN5FV(steps = steps, V = mdv, T = mdt)``.

The strength of the MDevolve_ package is that it allows easy
construction of recursive integrators.

.. code:: Nim

  let
    VK = mkLeapFrog(updateVFh, updateK, 64)
    Vl0VK = mkOmelyan4MN4FP(VK.evolve, updateVFl0, 1)
    H = mkOmelyan4MN5FV(updateVFl1, Vl0VK.evolve)

The above creates an object that uses 4MN5FV as the outer
integrator, which uses leapfrog and 4MN4FP as its two
updater.  A two level FGYin11 can be created similarly.

.. code:: Nim

  let
    VK = mkFGYin11(updateVFh, updateK, updateVfgFh, (xsave.save s.x), (s.x.load xsave), 2)
    Vl0VK = mkFGYin11(updateVFl0, VK.evolve, updateVfgFl0, (xsave.save s.x), (s.x.load xsave), 2)
    H = mkFGYin11(updateVFl1, Vl0VK.evolve, updateVfgFl1, (xsave.save s.x), (s.x.load xsave))

The two arguments to ``mkFGYin11``, ``save`` and ``load``,
accept expressions, and here we used expressions instead
of creating more procedures.

In addition to the recursive construction, the package
also allows creating shared evolution, as follows.

.. code:: Nim

  var
    VK = mkLeapFrog(updateVFh, updateK, 3, shared = 1)
    Vl0K = mkLeapFrog(updateVFl0, updateK, 2, shared = 1)
    Vl1K = mkLeapFrog(updateVFl1, updateK, 1, shared = 1)
    H = mkSharedEvolution(VK, Vl0K, Vl1K)

When integrating along a trajectory, this shared evolution
will call ``updateVFh``, ``updateVFl0``, and ``updateVFl1``,
independently according to the schedule set by the respective
object, ``VK``, ``Vl0K``, and ``Vl1K``.  Along the way, any
call to ``updateK`` would be fused as one without duplication,
with its step size changed automatically, as it is shared
among the three integrators.

You can find more examples in the test file, `test1.nim`_.


.. _Mdevolve: https://github.com/jxy/MDevolve

.. _`puregaugehmc.nim`: https://github.com/jcosborn/qex/blob/devel/src/examples/puregaugehmc.nim

.. _`test1.nim`: https://github.com/jxy/MDevolve/blob/master/tests/test1.nim
