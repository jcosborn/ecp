Sierra/Lassen
=============

LSD analysis
------------

Started using QEX in production on Lassen for LSD 8f nHYP
analysis.

* updated QUDA bindings to the develop version
* fixed the MILC interface in QUDA for custom rank orders
* added saving of local trace (complex field) for disconnected analysis
* tested staggered analysis code (scalar trace) using QEX and
  cross checked with FUEL on Lassen and Mira
* will work on connected analysis next


Current status on Lassen
------------------------

Sierra and Lassen at LLNL have 4 V100 per node,
with 7.8 Tflops DP per GPU.

Measured QUDA naive staggered CG (with mixed DP+HP) performance from QEX
with overheads of the memory layout transformation and data transfer
using the QUDA MILC interface:

* lattice size of :math:`64^3 \times 128`:

  - 8 nodes 32 GPUs: 536 Gflops * 32 = 17.2 Tflops
  - 16 nodes 64 GPUs: 300 Gflops * 64 = 19.2 Tflops
  - 32 nodes 128 GPUs: 204 Gflops * 128 = 26.1 Tflops
  - 64 nodes 256 GPUs: 116 Gflops * 256 = 29.7 Tflops

* lattice size of :math:`96^3 \times 192`:

  - 16 nodes 64 GPUs: 560 Gflops * 64 = 35.8 Tflops
  - 32 nodes 128 GPUs: 391 Gflops * 128 = 50.0 Tflops
  - 64 nodes 256 GPUs: 333 Gflops * 256 = 85.2 Tflops

We found that the mixed DP+HP solver is about 20% faster than DP+SP in
time to solution.

After testing various jsrun parameters and manual numactl bindings,
along with ``PAMI_*`` settings, we found manual numactl binding showed
no improvement over jsrun.

	"From conversations with IBM, I think the decrease in
	performance with manual binding of the NICs and disabling
	striping is perhaps expected at the moment.  This is a moving
	target though, as IBM are working to improve GDR performance
	on Summit (I'm in conversation with them at the moment and
	this a problem actively being worked on)."  - Kate, 2/13/2019
