===============================
Generalized Resource Estimation
===============================

.. raw:: html

   <p style="font-size:14px; color:#444;">
   <b>Authors:</b> Abhin Suresh (University of Delaware), Kevin Bhimani (University of Delaware), Evan Jaffe (Ohio Supercomputing Center)
   </p>

Overview
--------
Welcome! This guide is intended for new researchers interested in
submitting a NAIRR Pilot proposal who have little experience with
HPC or large-scale resources.

Learning Outcomes
-----------------
* Translate a research task into CPU-hours, GPU-hours, node-hours,
  RAM, and scratch & long-term storage.
* Produce an estimate for scaling your code on High Performance Computing Systems
* Generate metrics that can be used for NAIRR pilot project.

NAIRR
-----
The **National Artificial Intelligence Research Resource (NAIRR) Pilot** program connects U.S. researchers and educators to computational,
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

Getting Started
---------------
A well-written proposal includes a detailed breakdown of the resources needed.
To get started, the following steps can be useful:

1. **Define the scenarios** you will actually run to understand data preparation, model train, inferrence, post processing, storage needs.

2. **Run a small pilot** that finishes in minutes to collect timing and resource data.

3. **Note throughput carefully** to understand your program’s computational behavior.

4. **Decide concurrency** — how many jobs will run in parallel.

5. **Convert to GPU-hours, core-hours, or node-hours**, then map to SUs for your target resource.

6. **Add a 20–30% overhead** for retries, queue variability, and profiling.

7. **Map to SUs** resource estimates based on your resource results.


Unit Conversions & Formulas (at a glance)
-----------------------------------------
- **GPU-hours** = hours × GPUs/job × jobs
- **Core-hours** = hours × cores/job × jobs
- **Node-hours** = hours × nodes/job × jobs
- **GB-hour (memory)** = peak GB × hours
- **TB-month (storage)** = average TB retained per month

Storage & Data Movement
-----------------------
Plan for the storage classes you’ll use and how data flows between different storages:

- **Project storage** (persistent, shared, moderate speed)  
  Long-lived space for datasets, checkpoints, and logs. Backed up and quota-limited.

- **Scratch** (ephemeral, very fast, local)  
  Temporary high-performance space used during runs. Purged automatically and not for long term storage.

- **Archive / long-term storage** (persistent, slower, cost-efficient)  
  For the data you rarely access

Interactive Estimator
---------------------

The SUs scale quickly. To get an estimate for the SU requirement, consider using the following widget.

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
       <input id="runs" type="range" min="1" max="500" value="10" oninput="W.upd()">
       <input id="runs_n" class="num" type="number" min="1" max="500" value="10" oninput="W.sync('runs')">
     </div>

     <div class="row">
       <label><b>Hours per run</b></label>
       <input id="hours" type="range" min="0.1" max="200" step="0.1" value="2" oninput="W.upd()">
       <input id="hours_n" class="num" type="number" min="0.1" step="0.1" value="2" oninput="W.sync('hours')">
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
       <label><b>Overhead (%)</b> <span class="small"></span></label>
       <input id="buffer" type="range" min="0" max="100" value="30" oninput="W.upd()">
       <input id="buffer_n" class="num" type="number" min="0" max="100" value="30" oninput="W.sync('buffer')">
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
       <label><b>SU per <code>GPU-hour</code> </b><span class="small"></span></label>
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
           // keep range <-> number in sync
           this.g(id).value = this.g(id+'_n').value;
           this.upd();
         },
         upd(){
           // keep number boxes synced from ranges
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
   Set **SU/core-hour** and **SU/GPU-hour** to match the resource you plan to request.

See Also
--------
The pages below provides a brief guide to estimate resources for different architectures.

* :doc:`CPU Estimate <cpu_estimation>`
* :doc:`GPU Estimate <gpu_estimation>`
