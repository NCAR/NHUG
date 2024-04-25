---
title: Energy Accounting on Derecho
description: Derecho's PBS can now show you how much energy your job used.
draft: false
date: 2024-05-01
categories:
  - Energy
  - Derecho
authors:
  - matthews
  - vanderwb
---

As part of CISL's recent submission to the [Green500](https://www.top500.org/lists/green500/) list, we have implemented a PBS hook (plugin) which allows PBS to record cumulative energy consumption data for each job individually. 

<!-- more -->

This works by sampling the energy consumed by each node individually since last boot time at both the beginning of the job and the end of the job. The total difference between these two readings is then stored in PBS after the job ends. As a result, energy data displayed while a job is running is not meaningful but will be correct after the job is finished.

To query this energy data about a recently finished job (currently within the last 72 hours) `qstat -f -x` can be used, for example:

```pre linenums="0"
matthews@derecho1:~> qstat -f -x 4238938 | grep 'x-ncar.*-energy'
    resources_used.x-ncar-cpu-energy = 758
    resources_used.x-ncar-energy = 2527
    resources_used.x-ncar-gpu0-energy = 0
    resources_used.x-ncar-gpu1-energy = 0
    resources_used.x-ncar-gpu2-energy = 0
    resources_used.x-ncar-gpu3-energy = 0
    resources_used.x-ncar-memory-energy = 1131
```

Each of these fields is in Joules (Watt*Seconds) and covers a period of time from immediately before the job script starts to immediately after it ends. Total energy is provided as "x-ncar-energy" and the energy that the system attributed to the CPU, RAM, and individual GPUs are called out separately. Note that these major components do not constitute all of the energy consumed by the node so the total energy will generally be higher than the sum of the components. There also may be a small error introduced due to the delay between when each of these measurements is sampled.

If the job ran on a node which didn't have GPUs (as above), the GPU energy will be listed as 0 Joules. If the job ran on a shared node (the develop queue), it's not possible to completely isolate the energy consumption of your job from other jobs running on that node. So it's recommended to rerun your job on an exclusive use node before trusting this data.

For completed jobs which began after approximately April 2 2024, `qhist` can instead be used. For example, to see all power metrics for a single job:

```
vanderwb@derecho2:~> qhist -j 4246835 -l
4246835.desched1
...
   Node Eng (J)  = 5787
   CPU  Eng (J)  = 691
   RAM  Eng (J)  = 1006
   GPU0 Eng (J)  = 626
   GPU1 Eng (J)  = 610
   GPU2 Eng (J)  = 623
   GPU3 Eng (J)  = 612
...
```

Using typical `qhist` flags, you can also show power metrics for all of your jobs for a particular time period. Here, we display jobs from April 20, 2024, sorted by node energy usage:

```
vanderwb@derecho2:~> qhist -p 20240420 -f '{energy-node}' -s energy-node | head -n 6
Job ID    E-Node(J)
4190307   6005563096
4198428   5612065907
4194550   3745933887
4199951   3336310504
4197478   1259626184
```

It should also be noted that this data includes only the energy consumed by the compute nodes. Energy consumed by the interconnect, control system, cooling, and storage is not included as it would be very difficult to proportion to individual jobs. In CISL's experiments for the Green500 it was observed that the GPU partition consumes approximately 15% more energy measured at the circuit breaker than would be accounted for by this method. Energy dedicated to storage and generating facility chilled water would be additional to that 15% but it does include the part of the interconnect which is local to the racks in question and some of the control system. Nevertheless, this data should be accurate enough for use in comparisons of workloads on a science throughput per Joule basis.

Due to the heterogeneous and shared nature of Casper, this feature is currently only available on Derecho; however, if you would find it useful on Casper, please let us know. 
