Object wrapper types
--------------------

Added support for custom tensor types.
Currently supports `Color`, `Spin` as wrappers

.. code:: Nim

   type
     Color*[T] = object
       fColor*: T
     Spin*[T] = object
       fSpin*: T

Dirac fermion can be defined as

.. code:: Nim

   const nc = 3
   const ns = 4
   type
     Complex* = ComplexType[float32]
     ColorVector* = Color[VectorArray[nc,Complex]]
     DiracFermion* = Spin[VectorArray[ns,ColorVector]]

and ColorMatrix can be defined as

.. code:: Nim

   type
     ColorMatrix* = Color[MatrixArray[nc,nc,Complex]]

ColorMatrix knows nothing about Spin.  We support operations mixing
tensor types such as

.. code:: Nim

   var
     m: ColorMatrix
     d: DiracFermion
     d2: DiracFermion

   d2 := m * d

by adding appropriate overloads

.. code:: Nim

   template `*`*(x: Color, y: Spin): untyped =
     asSpin(asScalar(x) * y[])

   template `*`*(x: Color, y: Color): untyped =
     asColor(x[] * y[])

Also updating framework to use these basic mathematical wrapper types

.. code:: Nim

   AsField

   AsMatrix
   AsVector

   AsComplex
   AsImaginary

   AsScalar

   AsSimd

Base mathematical framework deals with these types.

Other physical types (Color, Spin, Tensor, ...) built on top of
base mathematical layer.

Spin matrices
--------------------

First pass at implementing Spin projection

.. code:: Nim

   const
     z0 = newComplex( 0.0,  0.0)
     z1 = newComplex( 1.0,  0.0)
     zi = newComplex( 0.0,  1.0)
     n1 = newComplex(-1.0,  0.0)
     ni = newComplex( 0.0, -1.0)

     spprojmat1p* = p([[ z1, z0, z0, zi ],
	               [ z0, z1, zi, z0 ]])

   h := spprojmat1p * d

Spin and Color loops explicitly unrolled.
Allowed C compiler to optimize away unnecessary addition and multiplication
by 0.
Gives reasonable runtime performance, but leads to slow compile times.

Currently implementing projectors by hand

.. code:: Nim

   template spproj1p*(xx: Spin): untyped =
     let x = xx
     let v0 = x[0] + I(x[3])
     let v1 = x[1] + I(x[2])
     spinVector[type(v0)](2,[v0,v1])

Have working Wilson Dslash using these projectors and reconstructors.

Eventually plan to have special operator overloads for constant matrices
that performs arithmetic at compile time.

Optimization
--------------------

New code uses lots of ``let``'s.
Can lead to unnecessary copies.
C compilers could optimize it away, but in practice don't always.

Introduced macro `optimizeAst` which goes through final code and
removes unnecessary copies.
Can make a large difference in performance for larger objects.
