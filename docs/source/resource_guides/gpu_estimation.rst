GPU Estimate
============
Most of the AI models are now scaled run efficiently on multiple GPUs.
So, estimating GPU memory and compute time is important while 
submitting a proposal to NAIRR Pilot program to avoid out-of-memory 
(OOM) errors and unecessary wastage

Esimating GPU Memory for Inference
----------------------------------
Estimated GPU VRAM in GB = 2x model parameters (in billions) + 1x 
context length (in thousands)

For example, for StableCode with 3 billion parameters and 16k 
context length, we estimate 6GB for model weights + 16GB for 
overhead, totaling 22 GB estimated to run inference.  
A model like this should fit on an A100 or H100 for inference.

This estimate assumes fp16 (half-precision). Quantization to lower 
precisions (8-bit, 4-bit, etc) will reduce memory requirements.

Estimating GPU Memory Usage for Training
----------------------------------------
Estimated GPU VRAM in GB = 40x model parameters (in billions)

For example, for LLaMA-3 with 7 billion parameters, we estimate 
minimum 280GB to train it.  This exceeds the VRAM of even a single 
H100 accelerator, requiring distributed training.  
See HOWTO: PyTorch Fully Sharded Data Parallel (FSDP) for more details.

To note: The training estimate assumes transformer-based 
architecture with Adam optimizer using mixed-precision 
(32bit and 16bit weights used) and is extrapolated from results 
here: Microsoft Deepspeed.

Activation checkpointing can reduce the memory demands, at the cost 
of increasing runtime.