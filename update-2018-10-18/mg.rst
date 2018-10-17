Multigrid Support
=================

Currently focused on developing multigrid solver in QEX.

Test code is now running for Wilson.

Supports even/odd preconditioning and mixed precision.

Comparing small tests on :math:`8^4` lattice to old QOPQDP code
to verify functionality.

Not currently splitting chirality, but small tests seem to work
as well as QOPQDP with split chirality.  Need to evaluate on larger
lattices.
Will also add split chirality as option.

Testing using Lanczos-based SVD for setup.  So far gives similar
results to repeated inversions.

Only testing 2-level method with emulated coarse operator.

Plan to finish testing components and develop a flexible code
to explore variations.  Will use to test and tune algorithm
on larger lattices.

Once code is working well for Wilson, will add support for Staggered.


The base types for the Wilson MG nullspace (restrictors and prolongators)
are

.. code:: Nim

  type
    WmgVectorsV*[N:static[int]] = array[N,SDiracFermionV]
    LatticeWmgVectorsV*[N:static[int]] = Field[VLEN,WmgVectorsV[N]]

On each site, there is an ``array[N,SDiracFermionV]``, where ``N`` is the
number of nullspace vectors and ``SDiracFermionV`` is a single precision
Dirac Fermion with a Simd vector as the inner-most element.

``LatticeWmgVectorsV`` defines a lattice Field of these where ``VLEN`` is
the Simd vector length.

These Field types are then contained in a ``MgTransfer`` object, which can
be used as either a restrictor or prolongator.

.. code:: Nim

  type
    MgTransfer*[VF,VC: static[int]; T] = object
      mgb*: MgBlock[VF,VC]
      v*: Field[VF,T]

``MgBlock`` is an object that contains the mappings between the fine
and coarse lattices used for restrictions and prolongations.
``VF`` and ``VC`` are the Simd vector lengths of the fine and coarse lattices.

Creating these objects looks something like

.. code:: Nim

  # loF is the existing fine layout
  # latC is the coarse lattice size
  let loC = newLayout(latC, loF.V, loF.rankGeom, loF.innerGeom)
  let mgb1 = newMgBlock(loF, loC)
  const nmgv1 {.intDefine.} = 48
  var rv,pv: LatticeWmgVectorsV[nmgv1]
  rv.new(loF)
  pv.new(loF)
  var r = newMgTransfer(mgb1, rv)
  var p = newMgTransfer(mgb1, pv)

Using these transfer objects, the MG preconditioner can be written as

.. code:: Nim

  proc preconditioner*(oa: var opArgsVc; z: var Field; gs: GcrState) =
    oa.r.restrict(oa.cb, gs.r)
    oa.csolve2(oa.cx, oa.cb)
    oa.p.prolong(z, oa.cx)

    oa.smoother(z, gs.r)

Code is being used to test correctness and explore variations.
Many allocations are being done inside preconditioner, need to be moved
out to initialization phase.
After that, the code should be ready to test at scale.
