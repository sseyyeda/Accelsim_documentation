
# Understanding Shared Memory and Unified SRAM Partitioning in Modern GPUs

## Question 1: 
**How does the system determine how many CTAs can be run on an SM when using unified SRAM for both shared memory and L1 cache?**

### ✅ Answer:
In modern GPUs (e.g., Volta, Ampere), the shared memory and L1 cache are backed by a unified SRAM per SM. The total SRAM is dynamically **partitioned once at kernel launch time** based on the shared memory needs of the kernel.

- The system knows the **shared memory per CTA** from kernel metadata.
- You (or the runtime) define how much of the unified SRAM is allocated to shared memory.
- Based on this, the scheduler uses the formula:

  ```
  Max CTAs = floor(SRAM reserved for shared memory / shared memory per CTA)
  ```

- This partition does **not change during runtime**. It is static for the entire duration of the kernel.
- The hardware then applies all resource constraints (threads, shared memory, registers, etc.) to determine CTA concurrency.

---

## Question 2:
**How is the partitioning between shared memory and L1 cache decided if it's all backed by the same unified SRAM?**

### ✅ Answer:
The partition between shared memory and L1 cache is set **statically at kernel launch** in one of two ways:

1. **Manually by the programmer**, using the CUDA API:

    ```cpp
    cudaFuncSetAttribute(myKernel,
        cudaFuncAttributePreferredSharedMemoryCarveout,
        cudaSharedmemCarveoutMaxShared);
    ```

    Options include:
    - `cudaSharedmemCarveoutDefault`
    - `cudaSharedmemCarveoutMaxShared`
    - `cudaSharedmemCarveoutMaxL1`

2. **Automatically by the CUDA runtime**, based on the kernel’s requested shared memory. The runtime uses heuristics to choose a carve-out that balances occupancy and cache performance.

### 🔐 Important Notes:
- The partition is **fixed** for the lifetime of the kernel.
- It determines how many CTAs can be scheduled based on the shared memory constraint.
- You can **tune** this partitioning to optimize for either more CTAs or more L1 cache.

---

### 🧠 Summary Table

| What | When | Who decides | Can you change it? |
|------|------|-------------|--------------------|
| Shared/L1 SRAM partitioning | At kernel launch | Programmer or CUDA runtime | Yes |
| Number of CTAs that fit | After SRAM partitioning | Scheduler | No (unless you reduce per-CTA usage) |
| SRAM division changes during kernel? | ❌ Never | — | ❌ |

---
