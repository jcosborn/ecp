Expression Optimization
=======================

Working on new Field framework that can optimize shifts inside expressions.

Example code can handle expressions such as

.. code:: Nim

   r := 2*v + 3*(m1*v) + 4*(m2*s1.shift(v)) + 5*(m3*s2.shift(m4*v))

optimizes to

.. code:: Nim

   s1.shiftStartComms(v)
   s2.shiftStartComms(m4*v)
   for i in r:
     r[i] := 2*v[i] + 3*(m1[i]*v[i])
     if s1.isInterior(i):
       r[i] += 4*(m2[i]*s1.indexInterior(v, i)
     if s2.isInterior(i):
       r[i] += 5*(m3[i]*(
                         s2.indexInterior(m1,i)

) + 5*(m3*s2.shift(m4*v))
