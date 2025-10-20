CPU Estimate
============

High-Level Summary of Steps
---------------------------
* Run a toy example
* Run a scaled-up (but still modest) example
* Determine scaling modifiers (e.g., problem size, number of cores)

Baseline Run
------------
The first step is to run a **baseline** job to estimate resource usage, typically with a small problem size that completes in a few minutes.  
For example, if your program is a Python script ``mult.py`` that multiplies two matrices:

.. code-block:: python
   :linenos:

   import numpy as np
   N = 2**13
   a = np.random.rand(N, N)
   b = np.random.rand(N, N)
   # multiply two matrices
   c = a @ b
   print("Multiplying matrices of size", N)

Things to consider:
* How large are your inputs, and where are they stored on the cluster?
* How large are your outputs, and where will you save them?
* Where will you write your log and error files?

You can use this shell command to measure wall time, CPU time, and peak RAM for your Python program:

.. code-block:: bash

   /usr/bin/time -v python mult.py > program_out.log 2> log.out

The file ``program_out.log`` captures your program’s printed output, while ``log.out`` records resource usage metrics.  
A typical snippet looks like this:

.. code-block:: text

   Command being timed: "python mult.py"
   User time (seconds): 110.19
   System time (seconds): 2.05
   Percent of CPU this job got: 693%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:16.19
   Maximum resident set size (kbytes): 1629240

Limiting Threads
----------------
To get a consistent baseline on a single core, restrict NumPy’s internal threading:

.. code-block:: bash

   export OMP_NUM_THREADS=1
   export MKL_NUM_THREADS=1

Choose the variable(s) that apply to your NumPy or BLAS installation.  
Once you have your baseline, you can scale up by varying ``OMP_NUM_THREADS`` or switching to MPI for distributed computation.


Worked Example A — CPU Data Prep
--------------------------------
**Pilot:** 10k images in 12 min on 8 threads; peak RAM = 6 GB.  
**Full data:** 10M images ⇒ scale ×1000 ⇒ 12,000 min on 8 threads ⇒ **1,600 core-hours**.  
Run 10 jobs in parallel (to finish sooner): still **1,600 core-hours** total.  
Add 20% buffer ⇒ **1,920 core-hours**.  
**Storage:** inputs 4 TB, outputs 6 TB, logs+artifacts +1 TB ⇒ **11 TB**; retain 2 months ⇒ **22 TB-months**.

See Also
--------
* :doc:`Generalized Resource Estimation <resource_estimation>`
* :doc:`GPU Estimate <gpu_estimation>`
