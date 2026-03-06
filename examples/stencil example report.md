# <img src='./images/logo.svg' width=90 style="vertical-align:middle" /> SHAREing: High-level performance assessment report

## Setup Details
* Program: `stencil-2d`
* Parallel model: `OpenMP`
* Dependencies: 
* Compiler: `g++`
* Compiler flags: `-O3 -march=native -fopenmp`
* Problem spec: 

## CPU Analysis
For a basic high-level analysis of CPU performance, we look for the floating-point operation rate compared to the theoretical rate for the hardware.

The hardware capabilities were determined with
```shell
$ likwid-bench -t peakflops -W S0:64kB:1
```

The code was benchmarked with
```shell
$ likwid-perfctr -f -C 0 -g FLOPS_DP ./cuda-stream
```

These yielded a hardware peak performance of `8800.5` MFLOP/s and an actual performance of `5492.5` MFLOP/s. We hence determine this code to have a CPU score of `0.6242`.

## GPU Analysis
For a basic high-level analysis of GPU performance, we look for the average occupancy of the GPU floating-point modules.

This section is not applicable to OpenMP, but we compiled a CUDA version and measured that.

The occupancy was determined with
```shell
$ ncu --csv --metrics sm__warps_active.avg.pct_of_peak_sustained_active --print-fp --log-file stencil-2d-cuda.metrics.csv ./stencil-2d-cuda
```

Over the 10 iterations run by default, the average occupancy was `93.78%`.

## IO Analysis
For a basic high-level analysis of IO performance, we look for the proportion of the runtime spent processing IO requests.

As this is a purely synthetic test code, the IO time is 0. For the purposes of this example, we have created a program that will read in a bunch of random data, increase each byte by a bit, and write it out again. The execution is overall timed by wrapping in `clock_gettime(CLOCK_MONOTONIC, &time)` and taking the difference between start and end.

We used `darshan` to profile IO time. In order for this to work we need to set up darshan to profile programs which aren't running using MPI:
```shell
$ export DARSHAN_ENABLE_NONMPI=1
```

We can now profile our `sample_io` program with
```shell
$ env DARSHAN_LOGFILE=sample_read.darshan LD_PRELOAD=libdarshan.so ./sample_io
Elapsed: 2.257649s (2257649.4460 µs), for size 1073741812
```

The easiest way to get the results out is
```shell
$ darshan-parser --total sample_read.darshan | grep "F_.\{4,5\}_TIME:"
total_STDIO_F_META_TIME: 0.254716
total_STDIO_F_WRITE_TIME: 0.495494
total_STDIO_F_READ_TIME: 1.122406
```

We can now calculate our IO utilisation ratio as `(1.122+0.495+0.255)/2.258=0.829`, and the IO score as `1-0.829=0.171`.

In reality, `stencil-2d` doesn't do any file IO so gets an IO score of `1.000`.

## Intra-node Analysis
For a basic high-level analysis of intra-node performance, we perform a strong scaling by fixing the problem size and increasing core allocation.

For this code, we tested with core counts in powers of 2 from 1 to 64.

| Thread count | Time (s) |
| - | - |
| 1 | 29.995 |
| 2 | 18.230 |
| 4 | 10.740 |
| 8 | 10.330 |
| 16 | 9.1307 |
| 32 | 8.5401 |
| 64 | 7.4589 |

By fitting a decaying exponential `a e^(-b t) + c` then doing `1-2b`, we normalise an intra-node efficiency score. 
This code gets an intra-node score of `0.639`.

## Inter-node Analysis
For a basic high-level analysis of inter-node performance, we perform a weak scaling by increasing problem size linearly with node allocation.

## Summary


| Result | MFLOP/s | 
| ----------- | ----------- |
| CPU | 0.624 |
| GPU | 0.938 |
| IO | 1.000 |
| Intra-node | ? |