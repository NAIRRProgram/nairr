CPU Estimate
============

High-level summary of steps
---------------------------
* Run a toy example
* Run a scaled-up (but still modest) example
* Determine modifiers

Baseline Run
------------
The first step is to run a **baseline** job to estimate resource usage, typically with a small problem size that finishes in a few minutes. For example, if your program is a Python script ``mult.py`` that multiplies two matrices:

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

You can use this shell command to measure wall time, CPU time, and peak RAM for a Python program:

.. code-block:: bash

   /usr/bin/time -v python mult.py > program_out.log 2> log.out

The file ``program_out.log`` captures your program’s printed output, and ``log.out`` records resource usage. A typical snippet looks like:

.. code-block:: text

   Command being timed: "python mult.py"
   User time (seconds): 110.19
   System time (seconds): 2.05
   Percent of CPU this job got: 693%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:16.19
   Maximum resident set size (kbytes): 1629240

To constrain NumPy’s threading (for a clean baseline), set:

.. code-block:: bash

   export OMP_NUM_THREADS=1
   export MKL_NUM_THREADS=1

(Choose the variable(s) that apply to your NumPy/BLAS installation.)

