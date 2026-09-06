Project 0 Getting Started
====================

**University of Pennsylvania, CIS 5650: GPU Programming and Architecture, Project 0**

* Shanshan Wu
* Tested on: Windows 11, AMD Ryzen 9 270 @ ~4.0GHz, 32GB RAM, NVIDIA GeForce RTX 5070 Laptop GPU (personal laptop)

### Part 2.1.1: Build and Run CUDA GL Check

**Compute Capability: 12.0** (NVIDIA GeForce RTX 5070 Laptop GPU, Blackwell architecture, `sm_120`).
<br><br>

### Part 2.1.2: Modify the CUDA Project and Take a Screenshot
![](images/modify.png)

The window title confirms the detected device (`[SM 12.0] NVIDIA GeForce RTX 5070 Laptop GPU`), and the solid magenta fill (RGB 255, 0, 128) is exactly the color `kernel.cu` assigns in the `case 12` branch of its compute-capability color table — i.e. the kernel correctly branched on major version 12 for this GPU.
<br><br><br>

### Part 2.1.3: Nsight Debugging
![](images/Nsight_debug.png)

Breakpoint hit at `CTA (12, 0, 0)`, `threadIdx (8, 1, 0)`, with `index = x + y * width = 200 + 1*800 = 1000`, matching the Autos value exactly. Since `threadIdx.y = 1` falls inside the first warp (lanes covering `y = 0` and `y = 1` for a 16-wide block), the yellow arrow correctly lands on the first row (`Thread: (0,0,0)`, the warp's base lane) in the Warp Info window — the specific active thread within that warp's 32-lane strip is at lane 24 (`1*16 + 8`).
<br><br><br>

### Part 2.1.4: Nsight Systems
![](images/Nsight_system.png)

Over 622 captured CPU frames the app averages 17.09 ms/frame (58.51 FPS, target 60 Hz), with 99% of frames under 17.76 ms — close to the vsync target — but there's a single outlier spike up to 206 ms, most likely a one-time stall during startup / first-frame CUDA-GL interop setup rather than steady-state jank.
<br><br><br>

### Part 2.1.5: Nsight Compute
![Summary](images/nc-summary.png)
*Summary*

![Details](images/nc-details.png)
*Details*

Each `createVersionVisualization` launch (grid of 50×50 blocks, 16×16 threads/block = 640,000 threads) takes ~92.8 µs with Compute Throughput ~25% and Memory Throughput ~38% of peak — both under the 60% threshold Nsight Compute flags as a latency issue. Achieved occupancy is excellent (97.7% vs. 100% theoretical), so the SMs have plenty of resident warps; the low throughput instead reflects that each thread only executes a handful of instructions (one switch-case plus a single write to the PBO), so the kernel is latency-bound rather than compute- or bandwidth-bound — there simply isn't enough per-thread work to saturate the pipeline. No quantifiable optimization opportunities were flagged, which is expected for a small verification kernel rather than a performance-critical one.
<br><br><br>

### Part 2.2: Project Instructions - WebGL
![](images/WebGL.png)

WebGL 1 is supported and hardware-accelerated: the unmasked renderer reports `ANGLE (NVIDIA, NVIDIA GeForce RTX 5070 Laptop GPU ...) Direct3D11`, confirming Chrome routes WebGL through ANGLE/D3D11 to the same discrete GPU used for CUDA, rather than falling back to software rendering or the integrated Radeon 780M.
