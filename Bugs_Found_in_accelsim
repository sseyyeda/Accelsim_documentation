## Issues Observed in Accel-Sim

### **1. L1 Cache Handling Problem**

The way the Accel-Sim code handles the L1 cache is problematic.  
I reported the bug on GitHub, and you can find their response here:  
[https://github.com/accel-sim/accel-sim-framework/issues/422#issue-3017973431](https://github.com/accel-sim/accel-sim-framework/issues/422#issue-3017973431)

---

### **2. `max_cta()` Function Behavior**

In the `max_cta()` function (see [`Maximum_Number_CTAs_per_SM.md`](./Maximum_Number_CTAs_per_SM.md) for more details), the code checks what limits the number of concurrent CTAs. However, it apparently only checks the first kernel, and the rest of the kernels are decided based on that first one!  

This was observed because each time `max_cta()` is called, it should print a message like:

```c
printf("GPGPU-Sim uArch: CTA/core = %u, limited by:", result);
