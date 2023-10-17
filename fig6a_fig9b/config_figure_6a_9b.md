# Configuration for figure 3, 4
This document describe how to configurate a machine to reproduce the result in figure 6a and 9b. 

## Hardware
These are the hardware used to produce the figures:

| Hardware | Description |
| -------- | ----------- |
| CPU | 2x Intel Xeon 6430 CPU |
| DRAM | DDR5 4800 MT/s (see below for DIMM configurations)|
| CXL devices | PCIe attached CXL devices |

### BIOS configuration
* Hyperthreading -- disabled
* Sub-NUMA clustering -- **enabled**
* CXL-Type3 legacy mode -- enabled (check with your CXL device vendor)

### DRAM population
```
- Socket 0
    - DIMM 1 <Inserted>
    - DIMM 2 <Inserted>
    ...
    - DIMM 7 <Inserted>
- Socket 1
    - DIMM 1 <Inserted>
    - DIMM 2 <Not used>
    ...
    - DIMM 7 <Not used>
```

### Inserting CXL device
```
    NUMA node 0 --- NUMA node 1
        |
        |
        |
     CXL device
```

## Software
### YCSB
In some cases, the locally built YCSB performs better than the release verison. Please clone the YCSB repository:
```
$ git clone https://github.com/brianfrankcooper/YCSB.git
```
and use the `./bin/ycsb` or `./bin/ycsb.sh` for testing.

After setting up YCSB, please place `run_redis.sh`, `figure_6a.sh` and `figure_9b.sh` withn first level of the repository folder. 

Additionally, please copy `workload*` configuration files in the `workloads` folder to the `workloads` folder in the YCSB repository. These workloads maps workload(a) to workload(r), (b) -> (s) ..., with a working set size of about 10 GB.

### Redis
Please follow this [guide](https://redis.io/docs/getting-started/installation/install-redis-on-linux/) to install Redis.

### Patched Linux Kernel:
We applied the N:M interleaving [patch](https://lore.kernel.org/linux-mm/YqD0%2FtzFwXvJ1gK6@cmpxchg.org/T/) to a Linux Kernel 5.19. We make the following modification to allow a more find-grain tuning of the memory ratio:
  + The patch added a tunable parameter (numa\_tier\_interleave) in `vm_table` in `kernel/sysctl.c`
  + In our case, we use two parameters to control the top and bot ratio independetly
    * `numa_tier_interleave_top` for top tier
    * `numa_tier_interleave_bot` for bot tier
  + The rest of the patch is applied without any modification


### System configuration
The `util_scripts/config_all.sh` is executed before each test within the testing scripts.

It does the following configurations:
* Lock CPU frequency 
* Disable frequency boosting
* Disable hyperthreading in software
* Disable NUMA balancing
* Stop `numad` if it exists

On Xeon 6430, the freqeuncy is locked to 2100MHz. The TSC freqeuncy on our machine is also 2100MHz. Please adjust the `util_scripts/lock_cpu_freq.sh` if this is tested on a different platform. The result may be significantly different if the frequency is not adjusted to the max frequency of the CPU.
