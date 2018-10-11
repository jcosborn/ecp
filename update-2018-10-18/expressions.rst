QEX Expression Optimization
===========================

Working on new Field framework that can optimize shifts inside expressions.

Example code can handle expressions such as

.. code:: Nim

   r := 2*v + 3*(m1*v) + 4*(m2*S1.shift(v)) + 5*(m3*S2.shift(m4*v))

expands to something like

.. code:: Nim

  S1.startComms(v)
  S2.startComms(m4*v)

  for i in r:
    r[i] := 2*v[i] + 3*(m1[i]*v[i])
    if S1.isInterior(i):
      r[i] += 4*( m2[i]*S1.indexInterior(v,i) )
    if S2.isInterior(i):
      r[i] += 5*( m3[i]*( S2.indexInterior(m4,i) * S2.indexInterior(v,i) ) )

  S1.peqFinishComms(r, 4*(m2*S1) )
  S2.peqFinishComms(r, 5*(m3*S2) )

``S2.startComms(m4*v)`` gets expanded to something like

.. code:: Nim

  for i in S2.sendBoundary:
    S2.sendBuf[i] := m4[i]*v[i]
  S2.start

and ``S2.peqFinishComms(r, 5*(m3*S2) )`` becomes like

.. code:: Nim

  S2.wait
  for i in S2.recvBoundary:
    r[i] += 5*( m3[i] * S2.indexBoundary(i) )

Test framework also optimizes site-level expressions

.. code:: Nim

  r := m1*v1 + m2*v2

becomes

.. code:: Nim

  for i in 0 ..< r.len:
    var t1 = sum(j, m1[i,j]*v1[j])  # sum is just pseudocode
    var t2 = sum(j, m2[i,j]*v2[j])  # real code writes this as typical loop
    r[i] := t1 + t2

..
    var t1 = m1[i,0] * v1[0]
    for j in 1 ..< m1.ncols:
      t1 += m[i,j] * v1[j]
    var t2 = m1[2,0] * v2[0]
    for j in 1 ..< m2.ncols:
      t2 += m[i,j] * v2[j]

and this

.. code:: Nim

  r := m1 * ( m2 * v )

gets split apart, effectively becoming

.. code:: Nim

  let t = m2 * v
  r := m1 * t


Expression Optimization status
------------------------------

Still just a prototype.  Currently missing subsets (easy), and nested
shifts (more difficult, but follows the same patterns).

Not in active development now due to other priorities.
