GPU Estimate
============

Most modern AI models scale efficiently across multiple GPUs. Estimating GPU
memory and compute time is important when submitting a proposal to the
NAIRR Pilot program to avoid out-of-memory (OOM) errors and unnecessary waste.

Estimating GPU Memory for Inference
-----------------------------------
A quick rule of thumb for fp16 (half precision):

.. math::

   \mathrm{VRAM}_{\text{inference}} \;(\mathrm{GB}) \;\approx\;
   2 \times \mathrm{params}_{(\mathrm{billions})}
   \;+\; 1 \times \mathrm{context\ length}_{(\mathrm{thousands})}

Example: for **StableCode** with 3B parameters and 16k context,
estimated VRAM ≈ 6 GB (weights) + 16 GB (overhead) = **~22 GB** for inference.
A model like this should fit on an **A100** or **H100** for inference.

**Notes**
- Assumes fp16; quantization can reduce VRAM further.
- Actual usage depends on framework.

Estimating GPU Memory for Training
----------------------------------
A practical planning heuristic for transformer models with Adam and mixed precision:

.. math::

   \mathrm{VRAM}_{\text{training}} \;(\mathrm{GB}) \;\approx\;
   40 \times \mathrm{params}_{(\mathrm{billions})}

Example: for **LLaMA-3 7B**, VRAM ≈ **~280 GB** to train. This exceeds a single
H100’s VRAM and typically requires **distributed training** (e.g., ZeRO/FSDP).

**Assumptions & Tips**
- Assumes transformer architecture, Adam optimizer, and mixed precision (16/32-bit).
- Heuristic extrapolated from widely reported results (e.g., DeepSpeed).
- **Activation checkpointing** reduces VRAM at the cost of extra compute time.
- For multi-GPU training, consider **PyTorch FSDP/ZeRO** (see HOWTO: Fully
  Sharded Data Parallel (FSDP)) to shard model states across devices.
