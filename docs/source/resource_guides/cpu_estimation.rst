CPU Estimate
============

High-Level Summary of Steps
---------------------------
- Run a toy example
- Run a scaled-up (but still modest) example
- Determine scaling modifiers (e.g., problem size, number of cores)

Baseline Run
------------
Example Python Code
~~~~~~~~~~~~~~~~~~~
The first step is to run a **baseline** job to estimate resource 
usage, typically with a small problem size that completes in a few 
minutes. For example, if your program is a Python script ``mult.py`` 
that multiplies two matrices:

.. code-block:: python
   :linenos:

   import numpy as np
   N = 2**13
   a = np.random.rand(N, N)
   b = np.random.rand(N, N)
   # multiply two matrices
   c = a @ b
   print("Multiplying matrices of size", N)

Extracting Infromation From Shell Command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You can use this shell command to measure wall time, CPU time, and 
peak RAM for your Python program:

.. code-block:: bash

   /usr/bin/time -v python mult.py > program_out.log 2> log.out

The file ``program_out.log`` captures your program’s printed output, 
while ``log.out`` records resource usage metrics. The relavent parts 
of the ``log.out`` in while running the job on intel i7-1185G7 while
setting number of *threads* to **1**

.. code-block:: text

   Command being timed: "python mult.py"
   User time (seconds): 27.52
   System time (seconds): 1.16
   Percent of CPU this job got: 99%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:28.79
   Maximum resident set size (kbytes): 1630324

and while setting the number of *threads* to **4**

.. code-block:: text
    
   Command being timed: "python mult.py"
   User time (seconds): 59.21
   System time (seconds): 1.39
   Percent of CPU this job got: 365%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:16.60
   Maximum resident set size (kbytes): 1631708

Here, the output parameters corresponds to:

- Wall clock time: elaspsed time for running your code
- User time: time CPU spend executing your calculation
- System time: time taken by the system on waiting on I/O
- Peak RAM: the maximum amount of RAM consumed by your program. This
  typically sets the lower bound on the availbale RAM for the node
  you request
- if outputs are saved in the ``output_folder``, then the command
  ``du -sh /path-to-output/output_folder`` after running the program 
  gives you the estimated storage requirements

+------------+---------------+--------------+---------------+--------------------+
| Array Size | Cores/Threads | Wall time(s) | Peak RAM (GB) | Percent of CPU (%) |
+============+===============+==============+===============+====================+
|  8192      |       1       |  28.79       |  1.63         |  99                |
+------------+---------------+--------------+---------------+--------------------+
|  8192      |       4       |  16.60       |  1.63         |  365               |
+------------+---------------+--------------+---------------+--------------------+
|  8192      |       8       |  12.47       |  1.63         |  686               |
+------------+---------------+--------------+---------------+--------------------+

.. note::
    
    Independent of CPU utalization, you job will cosume the CPU core
    hours based on number of CPUs you reqested. Its is your job to
    maximize the efficiency of the program to avoid wastage

Limiting Threads
----------------
To get a consistent baseline on a single core, restrict NumPy’s 
internal threading using:

.. code-block:: bash

   export OMP_NUM_THREADS=1
   export MKL_NUM_THREADS=1

Set the environment variable that apply to your NumPy's BLAS installation.
Regular numpy installation comes with OpenBLAS, unless it configured
to use MKL. You can check the NumPy linkage by executing the command

.. code-block:: python

    import numpy as np
    np.show_config()

You can also verify the number of threads being used using the command
``htop`` in your Shell. Once you have your baseline run without any errors, 
you can scale up by varying the associated environment varibale in 
the above case of python matrix multiplication, or varying number of 
processes in an MPI run for distributed computation.

.. code-block:: shell

    miprun -np 4 my_mpi_program

.. note::

    For the Python matrix multiplication example above, it 
    might be faster to run on single thread comapred to multiple 
    threads unless you have large engough matrix.

.. note::

    When you have both mpi and mulithreading, it is a good idea to 
    set the number of mpi processes :math:`\times` number of threads 
    to not exceed the number of available/requested CPU cores

List of Things Missing from /usr/bin/time
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Disk usage growth as a function of time
- RAM consumption change as a function of time
- GPU usage, check :doc:`GPU Estimate <gpu_estimation>`
- Concurrent processes/thread breakdown


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
