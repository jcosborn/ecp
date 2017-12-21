QEX build system updates
========================

Build system
--------------------

Currently supports two separate systems:
- ``configure`` and ``make`` scripts, which use ``Nim`` underneath
- ``nimble``, which automatically installs external ``Nimble``
  package dependencies

Check out the readme_ file and Nimble_ documentation for details.

.. _readme: https://github.com/jcosborn/qex/blob/devel/README.md

.. _Nimble: https://github.com/nim-lang/nimble

Configure and make
~~~~~~~~~~~~~~~~~~~~

Under your working directory, run

.. code:: sh
  QIODIR='/path/to/qio' QMPDIR='/path/to/qmp' \
  QUDADIR='/path/to/quda' CUDADIR='/path/to/cuda/lib' \
  relative/path/to/qex/source/configure

Edit ``config.nims`` to suit your system.

.. code:: sh
  make test/examples/testStagProp

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
  nimble make examples/testStagProp

Fixed a few issues with ``Nimble`` to allow it to work in multi-user
and shared home directory environments.


External dependencies
--------------------

Currently required packages:
- ``QMP``
- ``QIO``

Optional packages:
- ``LAPACK``
- ``PRIMME`` and the primme_ Nimble package
- ``QUDA`` and ``CUDA``

Automatic ``Nimble`` managed packages:
- chebyshev_
- primme_ (optional)

.. _primme: https://github.com/jxy/primme

.. _chebyshev: https://github.com/jxy/chebyshev


Documentation, examples and testing
--------------------

Added continuous integration (Travis CI)
- Automatic testing against ``devel`` branch

Better reporting and performance regression tests under development

Will start integrating examples, documentation and tests once new
``runnableExamples`` and ``:test:`` are available
- can use examples, tutorials and inline documentation as unit tests
