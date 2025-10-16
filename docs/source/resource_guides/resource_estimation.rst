===============================
Generalized Resource Estimation
===============================

Overview
========
Welcome! This guide is intended for new researchers interested in 
submitting a NAIRR Pilot proposal and has little experience with 
HPC or large-scale resources. 

Learning Outcomes
-----------------
* Translate a research task into CPU-hours, GPU-hours, node-hours, 
  RAM and scratch & long-term storage.
* Produce a back-of-the-envelope estimate from small pilot timings.

Key Words
----------
* CPU core-hour = 1 CPU core used for 1 hour
* GPU-hour = 1 GPU used for 1 hour
* GB-hour = memory in GB × hours used
* TB-month = storage used averaged over a month
* SU (Service Unit) = accounting unit that maps to time on a 
  specific resource.
  An example of SU units: Jetstream2 GPU virtual machines (VMs) 
  cost 2x vCPUs. G3.large VM costs 32 SUs per hour. NAIRR pilot can 
  request 128,000 SUs (2000 GPU-hours)

NAIRR
=====
This material resource is to guide you to estimate the compute 
resources you might need while requesting HPC or large-scale 
resources from National Artificial Intelligence Research Resource 
(NAIRR) Pilot program.

NAIRR Pilot connects U.S. researchers and educators to computational,
data, and training resources needed to advance AI research. This
program works through shared Federal and private compute 
infrastructure to birdge the gap between researchers and educators
to AI resources. More information reagaring NAIRR and how to 
acknowledge them can be found at: https://nairrpilot.org/about.

The information regarding which all resources are available for researchers
can be found at: https://nairrpilot.org/opportunities/allocations 
and the same for educators can be found at 
https://nairrpilot.org/opportunities/education-call. A detailed 
video on instructions to request for resources is published in a
youtube video at: https://www.youtube.com/watch?v=GCTv5OjI1ys&t=184s,
with link to the slide used shared in the description.


Add your content using ``reStructuredText`` syntax. See the
`reStructuredText <https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html>`_
documentation for details.


.. toctree::
    :maxdepth: 2
    :name: mastertoc
    :caption: Table of Contents

    gpu_estimation
    cpu_estimation

.. meta::
   :description: The documentation of NAIRR Pilot resource estimation
   :keywords: NAIRR, documentation, CPU, GPU, resource
