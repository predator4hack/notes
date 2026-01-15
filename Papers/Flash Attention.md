
Volatile memory: loses data when power is removed

HBM: High bandwidth external memory for throughput
SRAM(Static Random Access Memory): Fast on-chip cache for low latency

|Memory Type|Description|Example Use|
|---|---|---|
|**SRAM**|Very fast but expensive. Keeps data as long as power is on.|CPU/GPU cache|
|**DRAM**|Slower but higher capacity and cheaper. Needs constant refreshing.|Main memory, GPU VRAM (HBM, DDR, GDDR)|

All **SRAM** and **DRAM (including HBM)** are **volatile memories**.

## How It Relates to GPUs and Fast Attention

Now, in the **GPU + Fast Attention** context:
### 🏗️ GPU Memory Hierarchy
1. **Registers / SRAM (On-chip)**
    - Closest to GPU cores (streaming multiprocessors).
    - Tiny but _blazing fast_ — stores intermediate computations like attention weights or matrix tiles.
    - Bandwidth in **terabytes/sec** range, but capacity is only a few **MB per GPU**.
2. **HBM (Off-chip DRAM)**
    - Sits outside the GPU die but connected via **high-bandwidth interfaces (1024-bit+)**.
    - Used to hold large model weights, activations, and attention matrices.
    - Bandwidth ~ **1–3 TB/s**, capacity in **tens of GBs**, but latency higher than on-chip SRAM.
3. **Host DRAM (CPU memory)** and **Disk (non-volatile)**
    - Even slower, used for staging data when GPU memory runs out.





