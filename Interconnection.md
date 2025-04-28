In any modern Network-on-Chip (NoC), communication between cores or chiplets happens via routers connected through links. Just like CPUs use pipelines to move instructions efficiently through execution stages,
NoC routers use pipelines to move packets (or flits) through different processing stages. Thus, the NoC pipeline refers to the sequence of stages that a flit (the unit of network data, a flow control digit) passes through inside a router during its traversal from input port to output port.
Each pipeline stage performs part of the work needed to move the flit forward. The Classic NoC Pipeline Stages at the microarchitectural level are:

![myimage](data/generic_router.png)

1) **Flit buffering (BW)**
2) **Route Compute (RC)**: This stage figures out where the packet should go next, based on the destination address inside the flit header. Typically, this stage uses routing algorithms like XY routing, dimension order routing, etc.
3) **Virtual Channel Allocation (VA)**: This stage decides which VC at the input port the flit can use. In essence, VCs prevent deadlocks and allow mulitple logical flows to share a physical link, thereby reducingthe chance of occurring Head of Line (HoL) blocking inside the crossbar.
In simple terms, after computing the route of the flit, the current router should make sure that the next hop router has available slot to accommodate the current flit. This is the responsibility of this stage.
4) **Switch Allocation (SA)**: This stage grants access to flits who want to enter the crossbar switch. If multiple flits want the same output port at the same time, arbitration decides who wins., and losers stay buffered and retry in the next cycles.
5) **Switch Traversal (ST)**: The step in which the designated flit moves physically across the crossbar from its input port to the output port.
6) **Link Traversal (LT)**: The step in which the designated flit moves onto the outgoing wire/link toward the next router.

Each of these stages typically takes one clock cycle in a simple design. Thus, moving one flit across one hop can take at least 6 cycles in which the first 5 cycles belong to the router itself, and the last cycle belongs to the link.

Modern designs often optimize or collapse stages to reduce latency. For example, VA and SA combined in a single cycle (VA/SA fusion), adding bypass paths for low-contention flits and Speculative switch traversal.
These optimizations aim to cut router pipeline latency to 2–3 cycles or even fewer when conditions allow. Below diagram shows the stages 

![myimage](data/pipeline_diagram.png)

The pipeline schema is based on Wormhole routing, where a packet can be broken down into 2 or more number of flits, and each flit undergoes every stage per cycle. As can bee seen in below pipeline diagram,
for every packet, only head flit undergoes Route Computation and Virtual Channel Allocation stages, and the rest of the flits should wait until the head flit finishes Switch Allocation stage.
So, **Router Computation and Virtual Channel Allocation are performed once per packet**.
![myimage](data/pipelinge_stages.png)
It is very important to note, however, that, stages are dependent to each other in a specific way. For example, VC allocation stage is dependent only the head flit is decoded and the routing computation is finished, and the destination
of the flit is already known by that time. Otherwise, VC allocation cannot perform. If flit decoding and Route Computation is done and the router makes sure there is an empty buffer in the destination VC, then
the current router participate in Switch Allocation/Arbitration stage. And finally, once Switch Arbitration stage is done, then Switch/Crossbar Traversal gets performed. 
Therefore, we can claim that below chain is the critical path for every router with virtual channel based routing architecture.
![myimage](data/critical_path.png)
As further improvement, a mechanism called **Lookahead Routing** is proposed to perform routing computation for the next router. According to below diagram, VC allocation and switch arbitration can be done in parallel.
This mechanism performs well in low and moderate traffic load, where there are enough buffers in the downstream router such that the VC allocation is successful for most of the time, therefore,
VC allocation and Switch arbitration can be done in parallel. 

![myimage](data/3_stage_critical_path.png)