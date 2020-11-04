---
title: Random Seed on BNSL package
author: Akhila Chowdary Kolla
categories: [RcppDeepState Results,mi seed]
tags: [R,Seed,BNSL,mi]
math: true
---
Over the past few months, I have been working on testing the Rcpp packages using RcppDeepState in the cluster. When we fuzz test each Rcpp function-specific testharness the number of inputs passed onto the target binary keeps varying depending on the clock speed, number of cores, cache size of the system. When I test a package using RcppDeepState, the number of inputs generated in the cluster are comparatively higher than the ones generated in a single-core system. The default timer is 2 minutes. Depending on the system's speed RcppDeepState generates as many inputs as possible. The inputs generated are usually stored in .crash/.fail/.pass files.

RcppDeepState package provides a function where we can provide the random seed and the time (in seconds). Setting the seed and timer argument can control the number of input files to generate by avoiding the generation of a large number of files.

```R
RcppDeepState::deepstate_fuzz_fun_analyze(fun_path,seed,time.limit.seconds)
```
Making a call to deepstate_fuzz_fun_analyze() will limit the amount of time the fuzzer runs.

When I run the deepstate_fuzz_fun_analyze function it generates an output folder. The output folder contains the Valgrind log files, inputs passed on to the binary.
The name of the output folder is a combination of the timer and the seed provided.

Making a call to deepstate_fuzz_fun_seed on the cluster with 12 cores:

```R
> seed_analyze <- RcppDeepState::deepstate_fuzz_fun_analyze("~/RcppDeepStateTest/BNSL/inst/testfiles/mi",1604461988,5)
> seed_analyze
      inputs          logtable
1: <list[3]> <data.table[1x5]>
```
The output of the method is a data table with two columns. 
inputs: list of inputs passed on the binary
logtable: A data table with error trace.
####Inputs :
```R
> seed_analyze$inputs[[1]]
$proc
[1] -251329436

$x
 [1]   7.501136e-92 -5.926908e-154 -6.821346e-278   1.039617e+71  -6.693818e-42
 [6]  1.460757e-179 -5.904605e-179  -2.331613e-33   3.837833e-19  2.930730e-121
[11]   2.336555e+88 -2.308112e-255  3.052736e+288  -5.003846e-24             NA
[16]  6.438480e-217             NA  1.140529e-265 -1.312231e-307  5.126633e+150
[21]  1.587464e-147   4.880654e+31             NA   1.007890e-69  -6.406234e+44
[26]  2.713041e+178  7.026392e+241   9.488017e+50  1.189190e-285   1.137626e+07
[31] -1.462272e-155  -1.219976e-06 -6.637242e+220  2.081497e+133  3.158171e+307
[36]  4.872180e+233   3.759599e-37   9.856752e+91   3.402226e-97  3.537081e-137
[41]   2.316902e+26 -1.211973e+218             NA  -1.152365e+61  4.144492e+195
[46]   1.834769e-94  2.451281e-207   4.555089e+49             NA -2.811718e+160
[51]  -1.072064e-82 -7.785196e+304 -4.245621e-269 -1.858355e-259  4.806128e+175
[56] -7.607835e-278   0.000000e+00

$y
 [1]  8.029829e-278 -1.731409e+212             NA -2.844294e+288             NA
 [6]  1.933690e-151  -3.083875e-95  3.352638e-188  -2.619177e+13  -1.587476e-62
[11]   1.235092e+68  1.025835e-270  -8.437176e+85  1.329739e-281  -1.784651e-96
[16]  1.377834e+299  2.119751e+239             NA   6.685541e-09  -2.368265e-90
[21] -5.754205e-152  4.447042e+273             NA  -1.552687e-16  -4.925611e+33
[26] -1.351617e-103  -2.535115e+92  -4.200715e+11             NA  4.463942e+149
[31]   0.000000e+00
```
#### Logtable :
```R
> seed_analyze$logtable[[1]]
      err.kind                message           file.line
1: InvalidRead Invalid read of size 8 src/mi_cmi.cpp : 57
                                                      address.msg address.trace
1: Address 0x9f36468 is 0 bytes after a block of size 296 alloc'd          <NA>
> 
```
When we run the code on random seed 1604461988 for 5 seconds on a 12 core system gives the following Valgrind error :
Invalid read of size 8 on function mi_cmi.cpp at line number 57.
Let's run the same seed with the same timer on a single-core system.

The following 5 seconds timer generates only one crash file.
I get the same output when I run the testharness on the single-core system.

```R
> seed_analyze_onecore <- deepstate_fuzz_fun_analyze("~/Documents/packages/BNSL/BNSL/inst/testfiles/mi",1604461988,5)
compiling .. 
cd /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi && rm -f *.o && make

clang++ -g -I/home/akhila/lib/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/qs/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/mi_DeepState_TestHarness.cpp -o /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/mi_DeepState_TestHarness.o -c
clang++ -g -o /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/mi_DeepState_TestHarness /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/mi_DeepState_TestHarness.o -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include -I/home/akhila/.RcppDeepState/deepstate-master/build -I/home/akhila/.RcppDeepState/deepstate-master/src/include -L/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -Wl,-rpath=/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -L/home/akhila/lib/R/lib -Wl,-rpath=/home/akhila/lib/R/lib -L/home/akhila/.RcppDeepState/deepstate-master/build -Wl,-rpath=/home/akhila/.RcppDeepState/deepstate-master/build -lR -lRInside -ldeepstate -I/home/akhila/lib/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/.RcppDeepState/deepstate-master/src/include /home/akhila/Documents/packages/BNSL/BNSL/src/*.cpp
running the executable .. 
cd /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi && valgrind --xml=yes --xml-file=/home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/5_1604461988/1604461988_log --tool=memcheck --leak-check=yes --track-origins=yes ./mi_DeepState_TestHarness --seed=1604461988 --timeout=5 --fuzz > /home/akhila/Documents/packages/BNSL/BNSL/inst/testfiles/mi/5_1604461988/seed_valgrind_log_text 2>&1
```
####Output :
```R
> seed_analyze_onecore
   inputs     logtable
1: <list> <data.table>
``` 
####Inputs :
```R
> seed_analyze_onecore$inputs[[1]]
$proc
[1] 322829161

$x
 [1]   7.501136e-92 -5.926908e-154 -6.821346e-278   1.039617e+71  -6.693818e-42  1.460757e-179
 [7] -5.904605e-179  -2.331613e-33   3.837833e-19  2.930730e-121   2.336555e+88 -2.308112e-255
[13]  3.052736e+288            Inf -7.320795e+305  6.438480e-217           -Inf  1.140529e-265
[19] -1.312231e-307  5.126633e+150  1.587464e-147   4.880654e+31  1.697429e+251   1.007890e-69
[25]  -6.406234e+44  2.713041e+178  7.026392e+241   9.488017e+50  1.189190e-285   1.137626e+07
[31]           -Inf  -1.219976e-06            NaN  2.081497e+133  3.158171e+307  4.872180e+233
[37]   3.759599e-37   9.856752e+91   3.402226e-97  3.537081e-137   2.316902e+26 -1.211973e+218
[43]            Inf  -1.152365e+61  4.144492e+195   1.834769e-94  2.451281e-207   4.555089e+49
[49] -5.412640e+293 -2.811718e+160  -1.072064e-82 -7.785196e+304 -4.245621e-269 -1.858355e-259
[55]  4.806128e+175 -7.607835e-278   0.000000e+00

$y
 [1]   9.894608e-87  5.735044e+122  -1.049110e-37   1.743387e+80  1.190855e-275 -1.557622e+272
 [7] -1.015618e+282  2.615049e+183  -2.984232e+40            Inf -2.734239e+206  -2.210497e+02
[13]  5.107710e-292  -1.470213e+70 -1.777809e+180 -2.006614e-228  -7.556225e+97  1.542692e-195
[19] -2.759879e+118 -2.872285e-183 -1.259615e+275   5.282215e+71  1.202453e-175 -4.378386e+184
[25] -5.832857e-170  -8.634863e+96  1.997253e+115 -1.007742e+228  2.339930e-158   3.411637e-03
[31] -9.995202e+104  5.407699e+165 -1.129186e-103  7.051546e+104 -3.292680e+254   2.693190e+99
[37]            Inf  -2.285491e+12 -2.485616e+144  -2.427284e+30  1.333959e-152   4.186244e+65
[43]   3.837861e-65  1.594996e+237 -3.490244e-290 -6.902360e+242   4.754065e-53 -1.234629e+181
[49]  -5.580830e+02  -5.518447e+98  3.350312e-146  1.031690e+137 -2.303768e+294  -4.405587e-09
[55] -7.941940e-143  1.060620e-275             NA   3.586735e+67             NA -5.851526e-183
[61]  4.502878e-295  7.181342e+165  -6.719922e-72  1.197798e-192  -5.749472e+24 -1.192607e+152
[67] -2.110572e+144  -4.995089e+62  2.686353e-189  3.647098e-272 -9.753619e-300  2.842563e+269
[73]  1.259372e-136  3.197378e-121   0.000000e+00
```
####Logtable:

```R

```
