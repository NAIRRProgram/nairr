===============================
Generalized Resource Estimation
===============================

Overview
--------
Welcome! This guide is intended for new researchers interested in
submitting a NAIRR Pilot proposal who have little experience with
HPC or large-scale resources.

Learning Outcomes
-----------------
* Translate a research task into CPU-hours, GPU-hours, node-hours,
  RAM, and scratch & long-term storage.
* Produce a back-of-the-envelope estimate from small pilot timings.

Key Terms
---------
* **CPU core-hour** = 1 CPU core used for 1 hour
* **GPU-hour** = 1 GPU used for 1 hour
* **GB-hour** = memory in GB × hours used
* **TB-month** = storage used averaged over a month
* **SU (Service Unit)** = accounting unit that maps to time on a
  specific resource.
  * Example: Jetstream2 GPU virtual machines (VMs) may be billed relative to vCPU rate; e.g.,
    a ``g3.large`` flavor is commonly cited as ~32 SUs/hour. NAIRR Pilot
    requests can request ~128,000 SUs ≈ ~2,000 GPU-hours on Jetstream2

NAIRR
-----
This guide helps you estimate the compute resources you might need when
requesting HPC or large-scale resources from the **National Artificial
Intelligence Research Resource (NAIRR) Pilot** program.

NAIRR Pilot connects U.S. researchers and educators to computational,
data, and training resources needed to advance AI research. The program
leverages shared federal and private compute infrastructure to bridge
the gap between researchers/educators and AI resources. More information
is available at the `NAIRR Pilot program page <https://nairrpilot.org/about>`_.

Information about currently available resources for **researchers** is at
`NAIRR Pilot: Resource Requests <https://nairrpilot.org/opportunities/allocations>`_,
and for **educators** at
`NAIRR Pilot: Education Call <https://nairrpilot.org/opportunities/education-call>`_.
A step-by-step request walkthrough is available on YouTube:
`How to request resources <https://www.youtube.com/watch?v=GCTv5OjI1ys&t=184s>`_
(with slides linked in the video description).
