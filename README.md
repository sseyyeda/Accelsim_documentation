# Accel-Sim Insights and Exploration

This repository contains my notes, code snippets, and insights gained while exploring the internals of [Accel-Sim](https://github.com/accel-sim/accel-sim-framework), a detailed simulator for modern GPU architectures. The goal is to deepen understanding of GPU microarchitecture behavior, simulation flow, and performance modeling.

---
**Maximum_Number_CTAs_per_SM.md**: provides an overview of the limiting factors that prevent a high number of active concurrent CTAs on an SM, and explains why GPU occupancy might be low.

**L1_sharedmem_cache_demystifying.md**: Elaborates the dynamics and interplay between shared memory and L1 cache capacity in GPU architecture. 

**Unified_SRAM_Partitioning_l1_sharedmem.md** :Understanding Shared Memory and Unified SRAM Partitioning in Modern GPUs
**Bugs_Found_in_accelsim.md** :In this file, we document the possible bugs found in Accel-Sim.
