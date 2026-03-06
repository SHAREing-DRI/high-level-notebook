# <img src='./images/logo.svg' width=90 style="vertical-align:middle" /> SHAREing: High-level performance assessment report

## Setup Details
* Program: `BabelStream`
* Parallel model: `CUDA`
* Dependencies: 
* Compiler: `gcc`
* Compiler flags: `-DNDEBUG -O3 -mcpu=native`
* Problem spec: 

## CPU Analysis
For a basic high-level analysis of CPU performance, we look for the floating-point operation rate compared to the theoretical rate for the hardware.

The hardware capabilities were determined with
```bash
$ likwid-bench -t peakflops -W S0:64kB:1
```

The software was benchmarked with
```bash
$ likwid-perfctr -f -C 0 -g FLOPS_DP ./cuda-stream
```

| Result | MFLOP/s | 
| ----------- | ----------- |
| Hardware | 7204.9 |
| Performance | 2388.9 |

We determine this software to have a CPU score of `0.33`.

## GPU Analysis
For a basic high-level analysis of GPU performance, we look for the average occupancy of the GPU floating-point modules.

## IO Analysis
For a basic high-level analysis of IO performance, we look for the proportion of the runtime spent processing IO requests.

## Intra-node Analysis
For a basic high-level analysis of intra-node performance, we perform a strong scaling by fixing the problem size and increasing core allocation.

## Inter-node Analysis
For a basic high-level analysis of inter-node performance, we perform a weak scaling by increasing problem size linearly with node allocation.

## Summary
