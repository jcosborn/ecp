Communications and I/O
======================

Talked with Robert more at the 2019 ECP meeting about their analysis.
Requires transferring data from full lattice to a lattice with only a
subset of timeslices.  Also requires reading a lattice subset
(timeslice) from file (not in ILDG format).

QEX has a general communications layer (originally written in C, then
converted to Nim), which allow arbitrary communications maps (similar
to QDP/C).

However, the full capability wasn't exposed in Nim since only lattice
shifts were initially needed.  Code was also more complicated than
necessary due to being originally in C.
Decided to write new general communications layer instead of using
existing code.


General gather
--------------

.. code:: Nim

  let c = getComm()
  let nranks = c.commsize

  var rl = newSeq[RecvList](0)
  for i in 0..9:
    let destIndex = i.int32
    let srcRank = (i mod nranks).int32
    let srcIndex = (i xor 1).int32
    rl.add RecvList(didx: destIndex, srank: srcRank, sidx: srcIndex)

  let gm = c.makeGatherMap(rl)
  c.gather(gm, elementSize, srcPtr, dstPtr)


Scatter is currently implemented as a "reverse gather," i.e. a
gather, but swapping source and destination.

.. code:: Nim

  c.reverseGather(gm, elementSize, srcPtr, dstPtr)
  # performs gather on reversed mapping

Can be used to move data between lattices

.. code:: Nim

  var lo = newLayout(lat, 1)  # inner vector length = 1
  var lo2 = newLayout(lat2, 1)

  var cv = lo.ColorVector1()
  var cv2 = lo2.ColorVector1()

  var rl = newSeq[RecvList](0)
  var x = newSeq[int32](lat.len)
  for i in 0..<lo2.nSites:
    lo2.coord(x, i)
    let ri = lo.rankIndex(x)
    rl.add RecvList(didx: i.int32, srank: ri.rank.int32, sidx: ri.index.int32)


General parallel I/O
--------------------

General writer for hypercubic subset

.. code:: Nim

  var wm = lo.setupWrite(size, offset, ioranks)  # create write map
  # size: dimensions of subgrid
  # offset: offsets of subgrid in lattice
  # ioranks: sequence of ranks to be used as I/O nodes

  var pw = openCreate(filename)  # creates file and returns parallel writer
  pw.writeSingle("<header>")
  pw.write(f, wm)                # writes field 'f' using write map
  pw.writeSingle("<footer>")
  pw.close()

Parallel reader functions similarly.

Currently general I/O only works with non-vectorized field layouts.

Being used to read QDP++ "map object disk" format.
