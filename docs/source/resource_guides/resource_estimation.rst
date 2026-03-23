===============================
Generalized Resource Estimation
===============================

Overview
--------
This guide is intended for new researchers interested in submitting a NAIRR Pilot proposal who have little experience with HPC or large-scale resources. This guide will help researchers complete the resource estimation section while submitting a NAIRR Pilot proposal.

Learning Outcomes
-----------------
* Translate research compute needs into CPU-hours, GPU-hours, node-hours, RAM, and storage.
* Produce an estimate for scaling code on High Performance Computing Systems.


Reference Links
---------------
Information about currently available resources for **researchers** is at
`NAIRR Pilot: Resource Requests <https://nairrpilot.org/opportunities/allocations>`_,
and for **educators** at
`NAIRR Pilot: Education Call <https://nairrpilot.org/opportunities/education-call>`_.
A step by step request walkthrough is available on YouTube:
`How to request resources <https://www.youtube.com/watch?v=GCTv5OjI1ys&t=184s>`_
with slides linked in the video description.

Getting Started
---------------
1. **Define the scenarios to run**
  List the concrete workloads such as preprocessing, training, evaluation, inference, and post processing. For each workload, note expected run count, inputs and outputs, and per-run resource needs.

2. **Run a small pilot**
  Execute the same code path on a reduced problem that completes quickly. Record wall time, peak GPU and CPU memory, and output volume such as checkpoints and logs. Keep these measurements for citation in the proposal.

  .. note::
     Scratch storage is typically **not backed up** and is often purged on a schedule. Copy pilot outputs and measurements to project storage promptly.

3. **Measure throughput and scaling**
  Track a stable throughput metric such as tokens per second, images per second, steps per second, or records per second. Scale from pilot to full runs by dividing total work by throughput and converting to hours. If possible, run at two scales such as 1 GPU and 2 GPUs and report observed efficiency.

4. **Decide concurrency**
  Choose a realistic number of concurrent jobs based on queue limits and workflow dependencies. Concurrency affects calendar time, not total GPU-hours or CPU-hours. Note any sequential stages such as preprocessing before training.

5. **Convert to allocation units**
  Compute GPU-hours as runs × hours per run × GPUs per run, and CPU core-hours as runs × hours per run × cores per run. Use node-hours when allocation policy is node-based or when memory per node is the constraint.

6. **Add overhead**
  Add a buffer for retries, debugging, profiling, and extra evaluation, often 20 to 30 percent early in a project. If overhead is large, break it into buckets such as profiling runs × runtime × resources. Provide a brief justification for the chosen buffer.

7. **Map to SUs and summarize**
  Convert totals to service units using site-specific SU-per-GPU-hour and SU-per-core-hour factors. Report raw totals and SU totals, plus a rough calendar-time estimate based on concurrency. Detail the storage needs, separating persistent project storage, peak scratch, and any archive.


Worked Example
~~~~~~~~~~~~~~
Below is a short end-to-end example for a researcher planning to fine-tune a 7 billion parameter language model on 50,000 samples.

1. **Workloads identified:** data preprocessing (CPU-only), fine-tuning (multi-GPU), and evaluation (single GPU).
2. **Pilot run:** fine-tuning on 500 samples with 1 GPU completed in 0.5 hours; peak GPU memory was 38 GB; each checkpoint was ~28 GB.
3. **Throughput:** pilot showed ~17 samples per second on 1 GPU, and ~30 samples per second on 2 GPUs (scaling efficiency ~88%).
4. **Full estimate at 2 GPUs:**

   - Preprocessing: 5 runs × 0.5 hours × 16 CPU cores = 40 core-hours, 0 GPU-hours
   - Fine-tuning: 50,000 samples ÷ 30 samples/s = ~1,667 s ≈ 0.46 hours per run; 3 runs × 0.46 hours × 2 GPUs = 2.8 GPU-hours; CPU component: 3 runs × 0.46 hours × 8 cores = 11 core-hours
   - Evaluation: 3 runs × 0.25 hours × 1 GPU = 0.75 GPU-hours; CPU: 3 runs × 0.25 hours × 4 cores = 3 core-hours

5. **Subtotals:** 54 core-hours, 3.55 GPU-hours.
6. **Overhead (25%):** 67.5 core-hours, 4.44 GPU-hours.
7. **SU conversion** (site rates: 1 SU/core-hour, 32 SU/GPU-hour): 67.5 CPU SUs + 142 GPU SUs = **~210 total SUs**.
8. **Storage:** dataset 12 GB (project), 3 checkpoints × 28 GB = 84 GB peak scratch, final model 28 GB archived.

.. note::
   This example is deliberately small-scale. For large training runs the same method applies, but throughput measurements become especially important to avoid order-of-magnitude estimation errors.


Unit Conversions and Formulas
-----------------------------
Definitions for metrics used in the guide:

- **GPU-hours** = wall hours × GPUs per job × number of jobs
- **CPU core-hours** = wall hours × CPU cores per job × number of jobs
- **Node-hours** = wall hours × nodes per job × number of jobs
- **Memory usage** can be reported as peak GB per job and total GB-hours if needed
- **Storage** is typically reported as average TB retained per month plus peak scratch needs


Memory Estimation
-----------------
Memory misestimation is one of the most common reasons HPC jobs fail. A rough framework:

**For deep learning training**, peak GPU memory is driven by three components: model parameters, optimizer states, and activations. As a rule of thumb, mixed-precision training of a model with *P* parameters requires roughly 18–20 bytes per parameter (2 bytes for the FP16 parameters, 4 bytes for the FP32 master copy, 8 bytes for the Adam optimizer states, and additional bytes for gradients and activations). For example, a 7B parameter model requires approximately 7 × 10⁹ × 20 bytes ≈ 140 GB before accounting for activation memory, which varies with batch size and sequence length.

**For CPU-bound workloads**, peak memory is usually determined by the largest in-memory data structure. Measure this during your pilot run with tools such as ``htop``, ``free -h``, or memory-profiling libraries specific to your framework.

When requesting resources, report the **peak memory per job** and choose node types where available RAM exceeds your peak by a comfortable margin (at least 10–20%).


Storage
-------
Plan for the storage classes that will be used and how data flows between them:

- **Project storage** (persistent, shared, moderate speed)
  Long-lived space for datasets, checkpoints, and logs. Backed up and quota-limited.

- **Scratch** (ephemeral, very fast, local)
  Temporary high-performance space used during runs. Purged automatically and not for long-term storage. A common retention window is **~7–30 days**, but policies vary by host site.

- **Archive / long-term storage** (persistent, slower, cost-efficient)
  For data that is rarely accessed.

.. note::
   Scratch is typically **not backed up** and is often purged on a schedule. Plan to copy important data and outputs to project storage regularly.

**Estimating scratch usage:**
A useful rule of thumb for checkpoint-heavy workloads is: peak scratch ≈ checkpoint size × number of checkpoints retained × number of concurrent runs. For example, if each checkpoint is 28 GB, you keep the 3 most recent, and you run 4 jobs concurrently, plan for at least 28 × 3 × 4 = 336 GB of scratch.

**Data transfer time:**
Moving large datasets onto scratch or between storage tiers can consume significant wall time and may count against your allocation. For multi-terabyte datasets, estimate transfer time using your network bandwidth (e.g., 1 TB at 1 GB/s ≈ 17 minutes) and include this in your overall time budget.


Checkpointing Strategy
-----------------------
Checkpointing protects against lost work from preemptions or failures, but checkpoint frequency and size should be planned deliberately:

- **Frequency:** checkpoint at regular intervals (e.g., every N training steps or every hour). More frequent checkpointing reduces lost work on failure but increases I/O overhead and scratch usage.
- **Retention:** keep only the K most recent checkpoints to limit storage. Older ones can be deleted automatically or archived if needed.
- **Size estimation:** for a model with *P* parameters saved in FP32, each checkpoint is approximately *P* × 4 bytes plus optimizer state. Frameworks that save the full optimizer state (common for resumable training) can produce checkpoints 3–5× the model weight size.

Include checkpoint I/O time and storage in your resource estimate.


Interactive Estimator
---------------------
Service Units scale quickly. To get an estimate for your SU requirement, use the following widget. Set the **SU per core-hour** and **SU per GPU-hour** values to match your target host site's allocation policy (these rates are published on each site's documentation).

.. raw:: html

   <div id="nairr-widget" style="font:14px/1.4 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; border:1px solid #e5e7eb; border-radius:12px; padding:16px; max-width:900px;">
     <style>
       #nairr-widget .row{display:grid;grid-template-columns:220px 1fr 90px;gap:10px;align-items:center;margin:10px 0;}
       #nairr-widget input[type=range]{width:100%;}
       #nairr-widget .small{color:#6b7280;font-size:12px}
       #nairr-widget .num{width:80px}
       #nairr-widget .card{display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;margin-top:12px}
       #nairr-widget .box{border:1px solid #e5e7eb;border-radius:10px;padding:10px}
       #nairr-widget h4{margin:12px 0 6px 0;font-size:14px}
       #nairr-widget .big{font-weight:600;font-size:18px}
       #nairr-widget code{background:#f8fafc;padding:0 4px;border-radius:4px}
     </style>

     <div class="row">
       <label><b>Number of runs</b></label>
       <input id="runs" type="range" min="1" max="10000" value="500" oninput="W.upd()">
       <input id="runs_n" class="num" type="number" min="1" max="10000" value="500" oninput="W.sync('runs')">
     </div>

     <div class="row">
       <label><b>Hours per run</b></label>
       <input id="hours" type="range" min="0.1" max="240" step="0.1" value="10" oninput="W.upd()">
       <input id="hours_n" class="num" type="number" min="0.1" step="0.1" value="10" oninput="W.sync('hours')">
     </div>

     <div class="row">
       <label><b>CPU cores per run</b></label>
       <input id="cores" type="range" min="1" max="2048" value="8" oninput="W.upd()">
       <input id="cores_n" class="num" type="number" min="1" max="2048" value="8" oninput="W.sync('cores')">
     </div>

     <div class="row">
       <label><b>GPUs per run</b></label>
       <input id="gpus" type="range" min="0" max="256" value="1" oninput="W.upd()">
       <input id="gpus_n" class="num" type="number" min="0" max="256" value="1" oninput="W.sync('gpus')">
     </div>

     <div class="row">
       <label><b>Overhead (%)</b></label>
       <input id="buffer" type="range" min="0" max="100" value="20" oninput="W.upd()">
       <input id="buffer_n" class="num" type="number" min="0" max="100" value="20" oninput="W.sync('buffer')">
     </div>

     <div class="row">
       <label><b>Concurrent jobs </b><span class="small">(only used for calendar time)</span></label>
       <input id="conc" type="range" min="1" max="256" value="10" oninput="W.upd()">
       <input id="conc_n" class="num" type="number" min="1" max="256" value="10" oninput="W.sync('conc')">
     </div>

     <div class="row">
       <label><b>SU per <code>core-hour</code> </b><span class="small">(based on host site policy)</span></label>
       <input id="su_core" type="range" min="0" max="10" step="0.1" value="1" oninput="W.upd()">
       <input id="su_core_n" class="num" type="number" step="0.1" value="1" oninput="W.sync('su_core')">
     </div>

     <div class="row">
       <label><b>SU per <code>GPU-hour</code> </b></label>
       <input id="su_gpu" type="range" min="0" max="200" step="1" value="32" oninput="W.upd()">
       <input id="su_gpu_n" class="num" type="number" step="1" value="32" oninput="W.sync('su_gpu')">
     </div>

     <div class="card">
       <div class="box">
         <h4>CPU total</h4>
         <div class="big" id="cpu_hours">—</div>
         <div class="small">core-hours = runs × hours/run × cores/run × (1 + overhead)</div>
       </div>
       <div class="box">
         <h4>GPU total</h4>
         <div class="big" id="gpu_hours">—</div>
         <div class="small">GPU-hours = runs × hours/run × GPUs/run × (1 + overhead)</div>
       </div>
       <div class="box">
         <h4>Calendar time (rough)</h4>
         <div class="big" id="wall_days">—</div>
         <div class="small">days ≈ (runs × hours/run) ÷ concurrent jobs ÷ 24</div>
       </div>
     </div>

     <div class="card">
       <div class="box">
         <h4>CPU SUs</h4>
         <div class="big" id="cpu_su">—</div>
         <div class="small">= core-hours × SU/core-hour</div>
       </div>
       <div class="box">
         <h4>GPU SUs</h4>
         <div class="big" id="gpu_su">—</div>
         <div class="small">= GPU-hours × SU/GPU-hour</div>
       </div>
       <div class="box">
         <h4>Total SUs</h4>
         <div class="big" id="total_su">—</div>
         <div class="small">sum of CPU and GPU SUs</div>
       </div>
     </div>

     <script>
       const W = {
         ids:['runs','hours','cores','gpus','buffer','conc','su_core','su_gpu'],
         g(i){return document.getElementById(i)},
         val(i){return parseFloat(this.g(i).value)},
         fmt(n){return (n>=1000)? n.toLocaleString(undefined,{maximumFractionDigits:0}) : n.toLocaleString(undefined,{maximumFractionDigits:2})},
         sync(id){
           const v = this.g(id+'_n').value;
           this.g(id).value = v;
           this.upd();
         },
         upd(){
           this.ids.forEach(id=>{ this.g(id+'_n').value = this.g(id).value });
           const runs=this.val('runs');
           const hours=this.val('hours');
           const cores=this.val('cores');
           const gpus=this.val('gpus');
           const buf=(1+this.val('buffer')/100.0);
           const conc=Math.max(1,this.val('conc'));
           const su_core=this.val('su_core');
           const su_gpu=this.val('su_gpu');

           const cpu_hours = runs * hours * cores * buf;
           const gpu_hours = runs * hours * gpus * buf;

           const wall_days = (runs * hours) / conc / 24.0;

           const cpu_su = cpu_hours * su_core;
           const gpu_su = gpu_hours * su_gpu;
           const total_su = cpu_su + gpu_su;

           this.g('cpu_hours').textContent = this.fmt(cpu_hours);
           this.g('gpu_hours').textContent = this.fmt(gpu_hours);
           this.g('wall_days').textContent = this.fmt(wall_days);
           this.g('cpu_su').textContent = this.fmt(cpu_su);
           this.g('gpu_su').textContent = this.fmt(gpu_su);
           this.g('total_su').textContent = this.fmt(total_su);
         }
       };
       W.upd();
     </script>
   </div>

.. note::
   Set **SU per core-hour** and **SU per GPU-hour** to match your target resource. These rates are published in each host site's allocation documentation.


Host Site Policy
----------------
Site policies and constraints can force you to restructure jobs or revise your estimates. It is recommended to understand the following factors while planning computations:

- **Maximum wall time per job**
- **Maximum GPUs per job**
- **Job count limits and job array limits**
- **Per-user and per-project storage quotas**
- **Scratch purge policy and retention window**
- **Whether preemption is possible on the requested resource**


Overhead: What to Include and How to Estimate
----------------------------------------------
Most projects need a buffer to cover work that is real, expected, and often forgotten in early estimates.
Overhead is any compute and storage consumed outside the ideal production runs.

Common Sources of Overhead
~~~~~~~~~~~~~~~~~~~~~~~~~~
- **Retries and restarts** such as preemptions, node failures, and network interruptions
- **Queue and workflow variability** in case of failed jobs due to walltime limits and scheduler constraints
- **Profiling and performance tuning** — profiling runs can add up
- **Hyperparameter exploration** — even a small grid search can add to total compute cost
- **Data validation and integrity checks** — processing data for modelling and inference is commonly underestimated
- **Unit/integration tests for custom code** — especially if tests are non-trivial or run frequently
- **Rebuild cycles** — some large codebase build times can also add to the overall compute
- **Data transfer** — staging datasets onto scratch or moving results between storage tiers

A Practical Way to Estimate Overhead
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Use one of these approaches:

1) **Rule-of-thumb buffer:**
   Add **20–30%** if the project is early in development or new to HPC workflows.

2) **Component-based overhead:**
   Estimate overhead as the sum of a few explicit buckets, e.g.:

   - tests: [X test runs] × [test runtime] × [resources requested]
   - profiling: [Y profiling runs] × [runtime] × [resources]
   - retries: [Z%] of production runtime
   - hyperparameter search: [N extra runs] × [runtime] × [resources]

3) **Historical rate:**
   If past runs are available, use them to estimate an "overhead factor":

   overhead factor = total actual compute ÷ ideal production compute

   For example, if a previous project consumed 1,300 GPU-hours to produce work that ideally needed 1,000 GPU-hours, the overhead factor is 1.3 (30% overhead).

.. note::
   Unit tests and build-debug loops can meaningfully impact allocations, especially when tests fail and software must be rebuilt or reconfigured. If tests run on every build or before every large run, it is recommended to include them in the overhead estimate.


Recommended Reading
-------------------
The pages below provide a brief guide to estimate resources for different architectures.

* :doc:`CPU Estimate <cpu_estimation>`
* :doc:`GPU Estimate <gpu_estimation>`