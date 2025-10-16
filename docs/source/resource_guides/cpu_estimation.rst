CPU Estimate
============

High-level summary of steps
---------------------------
* Run toy example
* Run scaled up (but still modest) example
* Determine modifiers

Baseline Run
------------
First step is to do a baseline run for your code to estimate the 
resource usage, typically for a small system size system that 
finishes the program in a few minutes. For example, if you program 
is a python code ``mult.py`` to multiply two matrices is 

.. code-block:: python
    :linenos:
    
    import numpy as np
    N = 2**13
    a = np.random.rand(N,N)
    b = np.random.rand(N,N)
    #multiply two matrices
    c = a @ b
    print("Multiplying matrices of size ", N)

You need to take into consideration certain things
* How large is your input, where are you loading them in the cluster?
* How large is your output, where are you saving them?
* Where do you save your log file and error file?

You can use this bash script to measure run time, wall time, 
peak RAM, disk usage and input/output (I/O) time for a python program.


.. code-block:: sh

    /usr/bin/time -v python mult.py > program_out.log 2> log.out

The ``program_out.log`` saves the output from your program and 
logfile ``log.out`` saves the resource used by your program. 
The typical logfile contains

Running on i5-8250U
4 cores 8 threads. Numpy using 8 threads


.. code-block:: sh
    :linenos:

    Command being timed: "python mult.py"
    User time (seconds): 110.19
    System time (seconds): 2.05
    Percent of CPU this job got: 693%
    Elapsed (wall clock) time (h:mm:ss or m:ss): 0:16.19
    Maximum resident set size (kbytes): 1629240
    Average resident set size (kbytes): 0

Setting export ``OMP_NUM_THREADS=1`` or ``export MKL_NUM_THREADS=1``
depending on your numpy insatllation limits the number of threads 
used by numpy to 1