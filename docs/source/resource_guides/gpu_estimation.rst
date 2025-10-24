GPU Estimate
============

Overview
--------
Most modern AI models scale efficiently across multiple GPUs. Estimating GPU
memory (VRAM) and compute time is critical for NAIRR proposals to avoid
out-of-memory (OOM) errors and unnecessary resource waste.

Estimating GPU Memory for Inference
-----------------------------------
A quick rule of thumb for estimating GPU VRAM in GB:

.. math::

   \mathrm{VRAM}_{\text{inference}} \;(\mathrm{GB}) \;\approx\;
   2 \times \mathrm{params}_{(\mathrm{billions})}
   \;+\; 1 \times \mathrm{context\ length}_{(\mathrm{thousands})}

**Example.** For **StableCode** with 3B parameters and 16k context,
VRAM ≈ 6 GB (weights) + 16 GB (overhead) = **~22 GB** total. This fits on an
**A100** or **H100** for inference.

**Notes**
- Assumes fp16; quantization (8-bit, 4-bit) can reduce VRAM further.
- Actual usage depends on framework/runtime and settings (e.g., CUDA graphs, KV cache policy, preallocation).

Estimating GPU Memory for Training
----------------------------------
A practical planning heuristic for transformer models with Adam and mixed precision:

.. math::

   \mathrm{VRAM}_{\text{training}} \;(\mathrm{GB}) \;\approx\;
   40 \times \mathrm{params}_{(\mathrm{billions})}

**Interpretation.** The 40× factor covers model weights, gradients, optimizer
states, activations (with typical checkpointing), and mixed precision.

Minimal Monitoring
------------------------------------
**Peak VRAM (shell):**

.. code-block:: bash

   nvidia-smi --query-gpu=memory.total,memory.used,gpu_name --format=csv -l 2

**PyTorch (in-code snapshot):**

.. code-block:: python

   import torch
   # ... after warmup or inside training loop
   torch.cuda.reset_peak_memory_stats()
   # run a representative step or small loop...
   peak = torch.cuda.max_memory_allocated() / (1024**3)
   print(f"Peak allocated VRAM: {peak:.2f} GB")

Profiling Tools
---------------
NVIDIA-SMI Usage
~~~~~~~~~~~~~~~~
``nvidia-smi`` is available on GPU-enabled nodes and reports per-GPU and per-process
memory and utilization. It’s the fastest way to sanity-check **VRAM usage** and **GPU load**.

**Basic usage**

.. code-block:: bash

   nvidia-smi

**Typical output**

.. code-block:: text

   Wed Oct 15 20:58:25 2025
   +-----------------------------------------------------------------------------------------+
   | NVIDIA-SMI 550.90.12              Driver Version: 550.90.12      CUDA Version: 12.4     |
   |-----------------------------------------+------------------------+----------------------|
   | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
   |                                         |                        |               MIG M. |
   |=========================================+========================+======================|
   |   0  Tesla V100-PCIE-16GB           On  |   00000000:3B:00.0 Off |                  Off |
   | N/A   27C    P0             37W /  250W |   13830MiB /  16384MiB |      76%     Default |
   +-----------------------------------------------------------------------------------------+
   | Processes:                                                                              |
   |  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
   |=========================================================================================|
   |    0   N/A  N/A     27515      C   .../envs/vllm/bin/python                    13818MiB |
   +-----------------------------------------------------------------------------------------+

**What to watch**
- **Memory-Usage** → *used vs total VRAM*; OOM risk if close to 100%
- **GPU-Util** → percent of time kernels keep the GPU busy
- **Processes table** → *which PID / program* is using VRAM

**Watch continuously (refresh every 2s)**

.. code-block:: bash

   nvidia-smi -l 2

**Log to CSV over time (for later plotting)**

.. code-block:: bash

   nvidia-smi \
     --query-gpu=timestamp,index,name,memory.total,memory.used,utilization.gpu \
     --format=csv -l 2 >> gpu_usage.csv

**Per-process view (memory by PID)**

.. code-block:: bash

   nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv

**Find the right node (SLURM clusters)**

.. code-block:: bash

   # Which node is my job on?
   squeue -u $USER
   # SSH to that node to run nvidia-smi there (if your site allows):
   ssh <node-name>

Tips
~~~~
- Sample **after warm-up** to capture steady-state VRAM (JIT/graphs can spike initially).
- Combine with ``/usr/bin/time -v`` to capture **CPU/RAM** alongside GPU stats.
- If VRAM is near capacity, try **smaller batch/sequence**, **activation checkpointing**, or **quantization**.

See Also
--------
* :doc:`Generalized Resource Estimation <resource_estimation>`
* :doc:`CPU Estimate <cpu_estimation>`

References
----------
.. [OSC-GPU] Ohio Supercomputer Center, *HOWTO: Estimating and Profiling GPU Memory Usage for Generative AI*.
   Available at: `https://www.osc.edu/resources/getting_started/howto/howto_estimating_and_profiling_gpu_memory_usage_for_generative_ai
   <https://www.osc.edu/resources/getting_started/howto/howto_estimating_and_profiling_gpu_memory_usage_for_generative_ai>`_
   (accessed October 20, 2025).
