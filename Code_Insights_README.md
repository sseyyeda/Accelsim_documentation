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

In **Accel-Sim**, this calculation is handled by the folloeing function in shader.cc file:

```cpp
unsigned int shader_core_config::max_cta(const kernel_info_t &k) const {
  unsigned threads_per_cta = k.threads_per_cta();
  const class function_info *kernel = k.entry();
  unsigned int padded_cta_size = threads_per_cta;
  if (padded_cta_size % warp_size)
    padded_cta_size = ((padded_cta_size / warp_size) + 1) * (warp_size);

  // Limit by n_threads/shader
  unsigned int result_thread = n_thread_per_shader / padded_cta_size;

  const struct gpgpu_ptx_sim_info *kernel_info = ptx_sim_kernel_info(kernel);

  // Limit by shmem/shader
  unsigned int result_shmem = (unsigned)-1;
  if (kernel_info->smem > 0)
    result_shmem = gpgpu_shmem_size / kernel_info->smem;

  // Limit by register count, rounded up to multiple of 4.
  unsigned int result_regs = (unsigned)-1;
  if (kernel_info->regs > 0)
    result_regs = gpgpu_shader_registers /
                  (padded_cta_size * ((kernel_info->regs + 3) & ~3));

  // Limit by CTA
  unsigned int result_cta = max_cta_per_core;

  unsigned result = result_thread;
  result = gs_min2(result, result_shmem);
  result = gs_min2(result, result_regs);
  result = gs_min2(result, result_cta);

  static const struct gpgpu_ptx_sim_info *last_kinfo = NULL;
  if (last_kinfo !=
      kernel_info) {  // Only print out stats if kernel_info struct changes
    last_kinfo = kernel_info;
    printf("GPGPU-Sim uArch: CTA/core = %u, limited by:", result);
    if (result == result_thread) printf(" threads");
    if (result == result_shmem) printf(" shmem");
    if (result == result_regs) printf(" regs");
    if (result == result_cta) printf(" cta_limit");
    printf("\n");
  }
