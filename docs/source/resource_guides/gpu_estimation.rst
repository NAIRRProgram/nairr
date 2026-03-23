GPU Estimate
============

Overview
--------
Most modern AI models scale efficiently across multiple GPUs. Estimating GPU
memory (VRAM) and compute time is critical for NAIRR proposals to avoid
out-of-memory (OOM) errors and unnecessary resource waste.

Estimating GPU Memory for Inference
-----------------------------------
In an autoregressive transformer, each new token attends to all previous tokens.
To avoid recomputing attention for previous tokens at each step,
the model stores the **Keys** and **Values** for every layer and attention head.
This stored data is called the **KV cache**. Caching drastically reduces
computational time from :math:`O(T^2)` to :math:`O(T)` at the cost of **VRAM**.

.. math::
    \textrm{KV-cache} = 2 \times L \times H \times d \times T \times b,

where the factor 2 comes from storing both keys and values, :math:`L` is the number of
transformer layers, :math:`H` is the number of attention heads, :math:`d` is the dimension of each
head, :math:`T` is the context length, and :math:`b` is bytes per element (FP16 = 2).

With typical values of these parameters :math:`(L=32, H=32, d=128, b=2)`,
the **VRAM** from the KV cache is over **0.5 GB** for a context length of one thousand tokens.
For the purpose of resource estimation, we can treat **1 GB per thousand tokens**
(equivalently, 0.001 GB per token) as a **conservative upper bound**.

The runtime overhead — comprising the *CUDA context*, *cuBLAS/cuDNN workspaces*, and
*kernel launch buffers* — can vary from **300 MB** to **1 GB** per process.

A quick rule of thumb for estimating GPU VRAM in GB:

.. math::
   \begin{align}
   \mathrm{VRAM}_{\text{inference}} \;(\mathrm{GB}) \;\approx\; &
   \underbrace{2 \times \mathrm{params}_{(\mathrm{B})}}_{\text{weight VRAM}}
   \;+\;
   \underbrace{1 \times \mathrm{context}_{(\mathrm{k\ tokens})}}_{\text{KV-cache VRAM}} \\
   &+\;\underbrace{0.15 \times (\text{weight VRAM} + \text{KV-cache VRAM}) + 1}_{\text{runtime overhead}}
   \end{align}

where **weight VRAM** is the first term (:math:`2 \times \mathrm{params}_\mathrm{B}`) and
**KV-cache VRAM** is the second term (:math:`1 \times \mathrm{context}_\mathrm{k}`).
The overhead is modelled as 15% of the combined weight and cache footprint plus a 1 GB constant
for the CUDA context and workspace allocations.

**Example.** For **StableCode** with 3B parameters and 16k context,
VRAM ≈ 6 GB (weights) + 16 GB (KV-cache) + (3.3 + 1) GB (overhead) = **~26 GB** total. This fits on an
**A100**, **H100**, or 32 GB **V100** for inference.

.. note::
    - For inference, context length is often the major VRAM bottleneck — consider dropping old tokens or using a sliding window.
    - Actual usage depends on the framework, runtime, and settings (e.g., CUDA graphs, KV-cache eviction policy, preallocation).

.. raw:: html

    <div id="vram_calc" style="font:14px/1.4 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; border:1px solid #e5e7eb; padding:16px; border-radius:12px; max-width:900px;">
      <h3>Inference VRAM Requirement Estimator</h3>
        <style>
           #vram_calc .row{display:grid;grid-template-columns:220px 1fr 90px;gap:10px;align-items:center;margin:10px 0;}
           #vram_calc .row-select{display: grid; grid-template-columns: 220px 1fr; gap: 10px; align-items: center;}
           #vram_calc input[type=range]{width:100%;}
           #vram_calc .small{color:#6b7280;font-size:12px}
           #vram_calc .num{width:80px}
           #vram_calc .card{display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;margin-top:12px}
           #vram_calc .box{border:1px solid #e5e7eb;border-radius:10px;padding:10px}
           #vram_calc h4{margin:12px 0 6px 0;font-size:14px}
           #vram_calc .big{font-weight:600;font-size:18px}
           #vram_calc code{background:#f8fafc;padding:0 4px;border-radius:4px}
        </style>

      <div class="row">
        <label>Model size (billion parameters):</label>
        <input type="range" id="params" min="1" max="200" step="1" value="7" oninput="updateVRAM()">
        <input type="text" id="params_val" value="7 B" readonly>
      </div>

      <div class="row">
        <label>Context length (tokens):</label>
        <input type="range" id="context" min="1" max="32" step="1" value="4" oninput="updateVRAM()">
        <input type="text" id="context_val" value="4 k" readonly>
      </div>

      <div class="row-select">
        <label for="quant">Quantization:</label>
        <select id="quant" onchange="updateVRAM()">
          <option value="16">FP16</option>
          <option value="32">FP32</option>
          <option value="8">INT8</option>
          <option value="4">INT4</option>
        </select>
      </div>

      <div class="card">
        <div class="box">
          <h4>Estimated VRAM</h4>
          <div class="big" id="vram">—</div>
          <div class="small">VRAM ≈ params(B) × bytes/param + KV-cache + overhead</div>
        </div>
      </div>

      <script>
        function updateVRAM() {
          var params = parseFloat(document.getElementById('params').value);
          var ctx_k = parseInt(document.getElementById('context').value);
          var q = parseInt(document.getElementById('quant').value);

          // derived values
          var ctx = ctx_k * 1000;              // actual tokens
          var bytes_per_param = q / 8;

          // model weights (GB)
          var weight_vram = params * bytes_per_param;

          // KV cache: ~1 GB per 1k tokens (0.001 GB per token)
          var kv_vram = ctx * 0.001;

          // runtime overhead: 15% of (weights + KV) + 1 GB constant
          var overhead = 0.15 * (weight_vram + kv_vram) + 1;

          // total VRAM
          var total_vram = weight_vram + kv_vram + overhead;

          // UI updates
          document.getElementById('params_val').value = params + " B";
          document.getElementById('context_val').value = ctx_k + " k";
          document.getElementById('vram').innerText = total_vram.toFixed(1) + " GB";
        }
        updateVRAM();
      </script>
    </div>


A Small Code to Test Memory Consumption and Performance
--------------------------------------------------------
Install the relevant packages (PyTorch, Transformers, etc.) and
run on a machine with GPU access. You can also watch GPU usage with
``watch -n 0.5 nvidia-smi``.

.. code-block:: python

    import torch, time
    from transformers import AutoTokenizer, AutoModelForCausalLM

    MODEL = "gpt2-large"
    DTYPE = torch.float16
    DEVICE = "cuda"
    max_new_tokens = 256

    prompt = "Explain KV cache in one paragraph."
    # download or load the correct tokenizer for the model
    tokenizer = AutoTokenizer.from_pretrained(MODEL, use_fast=True)
    # load model weights and move to GPU
    model = AutoModelForCausalLM.from_pretrained(MODEL, dtype=DTYPE).to(DEVICE)

    # tokenize input as tensors and move to GPU
    inputs = tokenizer(prompt, return_tensors="pt").to(DEVICE)
    # warm-up run to initialize CUDA kernels and allocate buffers
    with torch.inference_mode():
        _ = model.generate(**inputs, max_new_tokens=32, do_sample=False)

    # wait for GPU to finish the process
    torch.cuda.synchronize()
    # reset previous peak memory stats
    torch.cuda.reset_peak_memory_stats()

    # start timing
    t0 = time.perf_counter()
    with torch.inference_mode():
        out = model.generate(**inputs, max_new_tokens=max_new_tokens,
                            do_sample=False)

    torch.cuda.synchronize()
    t1 = time.perf_counter()

    new_tokens = out.shape[-1] - inputs["input_ids"].shape[-1]

    tok_per_s = new_tokens / (t1 - t0)
    peak_alloc = torch.cuda.max_memory_allocated() / 1024**3
    peak_reserved = torch.cuda.max_memory_reserved() / 1024**3

    print(f"Generated tokens: {new_tokens}")
    print(f"Time: {t1 - t0:.3f} s")
    print(f"Throughput: {tok_per_s:.2f} tokens/s")
    print(f"Peak allocated: {peak_alloc:.2f} GB")
    print(f"Peak reserved:  {peak_reserved:.2f} GB")

Where **Peak allocated** is the actual memory used by tensors — a
good approximation of real usage. **Peak reserved** is the memory
reserved by PyTorch's caching allocator, which may be higher because
the allocator keeps freed blocks for reuse. For **small models**, GPU memory is often dominated
by **fixed overhead** (CUDA context, libraries) rather than the model itself, which is why
``nvidia-smi`` may report higher VRAM usage than ``peak_allocated`` alone.

On a machine with an NVIDIA GeForce 1050 Ti with Max-Q Design, the code
produced:

.. code-block:: text

    Generated tokens: 256
    Time: 17.774 s
    Throughput: 14.40 tokens/s
    Peak allocated: 1.53 GB
    Peak reserved:  1.67 GB

while ``nvidia-smi`` showed a maximum memory usage of 1.8 GB.

Estimating GPU Memory for Training
-----------------------------------
A practical planning heuristic for transformer models trained with Adam and mixed precision:

.. math::

   \mathrm{VRAM}_{\text{training}} \;(\mathrm{GB}) \;\approx\;
   40 \times \mathrm{params}_{(\mathrm{billions})}

**Breakdown of the 40× factor.** For a model with *P* billion parameters trained in
mixed precision with the Adam optimizer, the major VRAM consumers are:

- **FP16 weights + FP32 master copy:** ~6 GB per billion params (2 + 4 bytes/param)
- **Gradients (FP16):** ~2 GB per billion params
- **Adam optimizer states (FP32 momentum + variance):** ~8 GB per billion params
- **Activations** (with typical gradient checkpointing): ~20–24 GB per billion params, depending on batch size and sequence length

Together these total roughly 36–40 GB per billion parameters. The 40× figure is a
conservative upper bound that absorbs minor sources like temporary buffers and fragmentation.

**Example.** A 7B parameter model requires approximately 7 × 40 = **280 GB** of VRAM for training,
which would need at least 4× A100 80 GB GPUs with model parallelism, or 8× A100 40 GB GPUs.

.. note::
    - Activation checkpointing (trading compute for memory) can reduce the activations
      component significantly at the cost of ~30% more compute time.
    - Techniques like ZeRO (DeepSpeed), FSDP (PyTorch), or tensor parallelism can distribute
      optimizer states and gradients across GPUs, reducing per-GPU VRAM.
    - The 40× heuristic assumes a reasonable batch size. Very large batch sizes will
      increase activation memory beyond this estimate.


Minimal Monitoring
------------------
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
memory and utilization. It is the fastest way to sanity-check **VRAM usage** and **GPU load**.

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

**What to watch:**

- **Memory-Usage** → used vs. total VRAM; OOM risk if approaching 100%.
- **GPU-Util** → percent of time kernels keep the GPU busy.
- **Processes table** → which PID and program is consuming VRAM.

**Watch continuously (refresh every 0.5 s)**

.. code-block:: bash

   watch -n 0.5 nvidia-smi

**Log to CSV over time (for later plotting)**

.. code-block:: bash

   timeout 60s nvidia-smi --query-gpu=timestamp,power.draw,memory.used,temperature.gpu \
    --format=csv,nounits -l 1 > gpu_usage.csv


**Per-process view (memory by PID)**

.. code-block:: bash

    timeout 60s nvidia-smi --query-compute-apps=pid,process_name,used_memory \
    --format=csv,nounits -l 1 > gpu_usage.csv

**Find the right node (SLURM clusters)**

.. code-block:: bash

   # Which node is my job on?
   squeue -u $USER
   # SSH to that node to run nvidia-smi there (if your site allows):
   ssh <node-name>

Tips
~~~~
- Sample **after warm-up** to capture steady-state VRAM (JIT compilation and CUDA graphs can cause initial spikes).
- Combine with ``/usr/bin/time -v`` to capture **CPU and RAM** alongside GPU stats.
- If VRAM is near capacity, try a **smaller batch or sequence length**, **activation checkpointing**, or **quantization**.

Recommended Reading
-------------------
* :doc:`Generalized Resource Estimation <resource_estimation>`
* :doc:`CPU Estimate <cpu_estimation>`

References
----------
.. [OSC-GPU] Ohio Supercomputer Center, *HOWTO: Estimating and Profiling GPU Memory Usage for Generative AI*.
   Available at: `https://www.osc.edu/resources/getting_started/howto/howto_estimating_and_profiling_gpu_memory_usage_for_generative_ai
   <https://www.osc.edu/resources/getting_started/howto/howto_estimating_and_profiling_gpu_memory_usage_for_generative_ai>`_
   (accessed October 20, 2025).