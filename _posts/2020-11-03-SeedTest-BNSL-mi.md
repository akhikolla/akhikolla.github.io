---
title: Random Seed on BNSL package
author: Akhila Chowdary Kolla
categories: [RcppDeepState Results,mi seed]
tags: [R,Seed,BNSL,mi]
math: true
---
Over the past few months, I have been working on testing the Rcpp packages using RcppDeepState in the cluster. When we fuzz test each Rcpp function-specific Testharness the number of inputs that are passed onto the target binary keeps varying depending on the clock speed, number of cores, cache size of the system. When I test a package using RcppDeepState, the number of inputs that are generated in the cluster are comparatively higher than the ones that are generated in a single-core system. The default timer is set to 2 minutes. Depending on the system speed the inputs RcppDeepState generates as many inputs as possible. The inputs generated are stored in .crash/.fail/.pass files.

RcppDeepState package provides a function where we can provide the random seed and the time (in seconds). Setting the seed and timer argument we can control the number of input files to generate avoiding the generation of large number of files.

```R
RcppDeepState::deepstate_fuzz_fun_seed(fun_path,seed,time.limit.seconds)
```
Making a call to deepstate_fuzz_fun_seed() will limit the time the fuzzer is run.
When I run the following code on the cluster:

RcppDeepState::deepstate_fuzz_fun_seed("~/RcppDeepStateTest/BNSL/inst/testfiles/mi",1604461988,5)
```R
> deepstate_fuzz_fun_seed("~/RcppDeepStateTest/BNSL/inst/testfiles/mi",1604461988,5)
      err.kind                message           file.line
1: InvalidRead Invalid read of size 8 src/mi_cmi.cpp : 57
                                                      address.msg address.trace
1: Address 0x9f36468 is 0 bytes after a block of size 296 alloc'd          <NA>
> 

```
The code when run on random seed 1604461988 for 5 seconds I get the following valgrind error.
There is an invalid read of size 8 on the function mi_cmi.cpp at line number 57.
The following 5 seconds timer generates only one crash file.
I get the same output when I run the testharness on the single core system.


