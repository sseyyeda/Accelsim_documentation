# Accel-Sim Insights and Exploration

This repository contains my notes, code snippets, and insights gained while exploring the internals of [Accel-Sim](https://github.com/accel-sim/accel-sim-framework), a detailed simulator for modern GPU architectures. The goal is to deepen understanding of GPU microarchitecture behavior, simulation flow, and performance modeling.

---

## 📌 Insight: Maximum Number of CTAs per SM

### What is a CTA?
A **Cooperative Thread Array (CTA)**, also known as a thread block, is the basic unit of execution in CUDA programming. Multiple CTAs can be scheduled on a single Streaming Multiprocessor (SM), depending on the available hardware resources.

### Key Insight

The **maximum number of CTAs** that can be active on a single SM is determined by several resource constraints:

- Total number of **threads per SM**
- Available **shared memory (SMEM)** per SM
- **Register file size**
- **Warp size**
- Hardware cap on **max CTAs per SM**

In **Accel-Sim**, this calculation is handled by the function:

```cpp
max_cta(const struct gpgpu_ptx_sim_info *kernel_info,
        unsigned threads_per_cta,
        unsigned int warp_size,
        unsigned int n_thread_per_shader,
        unsigned int gpgpu_shmem_size,
        unsigned int gpgpu_shader_registers,
        unsigned int max_cta_per_core)
