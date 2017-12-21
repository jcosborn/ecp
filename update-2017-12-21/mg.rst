Multigrid
---------

Initial implementation of restriction and prolongation.

Simple example

.. code:: Nim

  qexInit()
  let latF = [16,16,16,16]
  let loF = newLayout(latF)

  let latC = [8,8,8,8]
  let loC = newLayout(latC, loF.V, loF.rankGeom, loF.innerGeom)

  let b = newMgBlock(loF, loC)

  const nmgv1 = 20
  type
    SMgR1V* = array[nmgv1,SDiracFermionV]
    SMgP1V* = array[nmgv1,SDiracFermionV]
    SLatticeMgR1V* = Field[loF.V,SMgR1V]
    SLatticeMgP1V* = Field[loF.V,SMgP1V]

    SMgColorVector1V* = Color[VectorArray[nmgv1,SComplexV]]
    SLatticeMgVector1V* = Field[loC.V,SMgColorVector1V]

  var rv: SLatticeMgR1V
  var pv: SLatticeMgP1V
  rv.new(loF)
  pv.new(loF)
  let r = newMgRestrictor(b, rv)
  let p = newMgProlongator(b, pv)

  var fv = loF.DiracFermionS()
  var cv: SLatticeMgVector1V
  cv.new(loC)

  r.apply(cv, fv)  # apply restrictor
  p.apply(fv, cv)  # apply prolongator

  qexFinalize()
