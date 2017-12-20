QEX build system updates
========================

Build system
--------------------

Currently supports two separate system:
1. ``configure`` and ``make``, which only uses ``nim``;
2. ``nimble``, which automatic installs external ``nim`` package dependencies. 

Check the readme_ file for details.

.. _readme: https://github.com/jcosborn/qex/blob/devel/README.md

Configure and make
~~~~~~~~~~~~~~~~~~~~

Under your working directory, run

.. code:: sh
  QIODIR='/path/to/qio' QMPDIR='/path/to/qmp' \
  QUDADIR='/path/to/quda' CUDADIR='/path/to/cuda/lib' \
  relative/path/to/qex/source/configure

Edit ``config.nims`` to suit your system.

.. code:: sh
  make testStagProp

Nimble
~~~~~~~~~~~~~~~~~~~~

Copy ``qex.nimble`` and ``local.nims`` to your working directory, and
edit ``local.nims`` to suit your system.

To build nim files:

.. code:: sh
  nimble make [debug] [FlagsToNim] [Name=Definition] Target [MoreTargets]

``debug`` will make debug build.
``Target`` can be any file name, optionally with partial directory names.
The produced executables will be under ``bin/``.

Examples:

.. code:: sh
  nimble make debug test0
  nimble make example/testStagProp

External dependencies
--------------------

Required packages:
- ``qmp``
- ``qio``

Optional packages:
- ``LAPACK``
- ``PRIMME`` & the ``primme`` nimble package
- ``QUDA`` & ``CUDA``

Automatic ``nimble`` managed packages:
- ``chebyshev``
- ``primme`` (optional)

Continuous integration (Travis CI)
--------------------

- Automatic testing against ``master`` branch of ``nim``
- Compile and run any file whose name starting with a ``t`` under ``tests`` direcotry
- Better reporting and performance regression tests under development

Tests, examples
--------------------

The files under directories, `tests/base`_ and `tests/examples`_,
serve as a collection of introductory learning guides for starting
with QEX.

.. _`tests/base`: https://github.com/jcosborn/qex/tree/devel/tests/base
.. _`tests/examples`: https://github.com/jcosborn/qex/tree/devel/tests/examples
