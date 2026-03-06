# <img src='./images/logo.svg' width=90 style="vertical-align:middle" /> SHAREing: High-level performance assessment guide

This document provides a guide to the SHAREing high-level performance assement methodology and report template. We provide some command examples and tooling recommendations, but there are undoubtedly more that can provide the same functionality for different systems than we have considered.

You will probably already have noticed that not all these sections are applicable to all programs. We suggest that you might take one of two approaches: include only those sections usefully applicable to your program, or recompile your program to the other of CPU and GPU than it normally does. As this performance assessment methodology is geared towards HPC codes intended to run on supercomputers i.e. with at least intra-node and likely inter-node parallelism, we expect that most codes will 

## Setup Details
This section of the report feeds directly from the pre-assessment workflow. Indeed, if this report is packaged as part of a full assessment, you may wish to remove this section, as all information included may already be in the pre-assessment section. However, it is included here as it is useful to gather together this information prior to doing the high-level assessment, and you may find you have several varieties of setup if you are comparing a code across several architectures.

## CPU Analysis
This section of the report covers CPU utilisation and performance. For the high-level report, we restrict this to floating-point performance, measured by the rate of floating point operations (FLOPS/s). In our examples, we have usually used `likwid` to do this. Information about `likwid` can be found at https://hpc.fau.de/research/tools/likwid/.

For an arbitrary `benchmark` program using double-precision floating point, we can analyse as follows. First, in order to get the peak rate, we want to analyse with all the data in the L1 cache, so we determine how big that cache is and halve it in order to be sure. 
```shell
$ gcc -O3 -march=native benchmark.c -o benchmark
$ likwid-bench -t peakflops -W S0:64kB:1 | grep Flops
Number of Flops:        8388608000
MFlops/s:               7428.79
$ likwid-perfctr -f -C 0 -g FLOPS_DP ./benchmark | grep MFLOP/s
|     DP [MFLOP/s]     |  2458.1529 |
|   AVX DP [MFLOP/s]   |          0 |
```
We can then determine our CPU score as `score=2458.1/7428.8=0.3309` (to 4 significant figures).



## GPU Analysis
For a basic high-level analysis of GPU performance, we look for the average occupancy of the GPU floating-point modules.

## IO Analysis
For a basic high-level analysis of IO performance, we look for the proportion of the runtime spent processing IO requests.

## Intra-node Analysis
For a basic high-level analysis of intra-node performance, we perform a strong scaling by fixing the problem size and increasing core allocation.

## Inter-node Analysis
For a basic high-level analysis of inter-node performance, we perform a weak scaling by increasing problem size linearly with node allocation.

## Summary
