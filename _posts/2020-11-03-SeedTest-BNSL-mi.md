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
> seed_analyze <- deepstate_fuzz_fun_analyze("~/RcppDeepStateTest/BNSL/inst/testfiles/mi",1604505428,5)
running the executable .. 
cd /home/akhila/RcppDeepStateTest/BNSL/inst/testfiles/mi && valgrind --xml=yes --xml-file=/home/akhila/RcppDeepStateTest/BNSL/inst/testfiles/mi/5_1604505428/1604505428_log --tool=memcheck --leak-check=yes --track-origins=yes ./mi_DeepState_TestHarness --seed=1604505428 --timeout=5 --fuzz > /home/akhila/RcppDeepStateTest/BNSL/inst/testfiles/mi/5_1604505428/seed_valgrind_log_text 2>&1
```
####Output :

```R
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
[1] -19761251

$x
 [1] -8.213736e-268 -1.951914e+111 -9.649061e+204  5.483429e-282 -5.254999e+138
 [6]   1.045533e-24 -3.494727e+224  1.615706e-259             NA -6.122974e+263
[11] -9.058724e-124   1.234785e+39 -6.111462e-282             NA  4.098092e-244
[16]   6.243492e-15  7.166968e+145  2.216562e-289  5.401894e+301   7.074301e-83
[21]   1.614506e-06   7.276571e-91 -1.848420e-262  4.080779e-261  8.245177e-141
[26] -4.667568e+297 -8.396755e-120  -1.528634e+65   1.256631e-53  5.962137e+130
[31]   3.429086e+88 -1.522745e+144  1.272269e+122  2.775626e+299 -1.306130e+303
[36] -8.402301e+206  6.906340e-159  -8.118336e+06             NA  -3.902448e-94
[41]  3.706050e-145 -1.057885e+173  3.247081e-159 -7.658977e+127  1.693851e+259
[46]  1.392144e-269 -7.915335e-116 -9.418790e+169 -5.420730e+268   4.083569e+44
[51]   1.881020e-59  1.844175e-265  -1.150161e+30  7.938846e+261  -1.039836e-02
[56] -8.750542e-238  3.986502e+148  1.945617e-176  -1.307638e+71 -6.730065e-215
[61]  3.305767e+287 -2.410883e+241  5.535715e+255 -5.280081e+226  2.130304e-247
[66]  1.456126e-267  1.534800e-128 -1.756331e-123   4.660655e+92             NA
[71]   5.351347e-05  2.744170e+135   2.006996e+85 -1.794635e+153   4.968092e+43
[76]  -9.867509e+86 -4.357863e+272   3.973519e-86   5.242205e-20  2.716399e-253
[81]   0.000000e+00

$y
 [1] -4.062348e-197             NA             NA  4.214413e-277  4.367095e+225
 [6] -1.643402e-274  4.829940e-133  9.947815e+147 -1.669293e-287  -3.515790e-53
[11]  -8.549773e-30  3.888799e+205 -2.035246e-143             NA  -4.788236e-09
[16]  -4.798404e-95  1.759833e-241 -1.718525e-135 -6.977067e+286 -2.336878e+198
[21]   3.050028e+65  -3.317124e-62  1.298565e+304   1.027234e-12  1.048329e-186
[26]             NA  7.691019e-173 -6.669731e+156  1.856961e+172   5.269763e-26
[31] -4.962290e-107  9.547934e+136  5.718846e+156 -2.778251e+122  -9.292919e-64
[36]  -3.215966e+91 -3.462424e-252  7.858450e+127 -2.618490e-263  1.491577e-264
[41]   5.545080e+72  6.482595e-274   7.474529e+41  -2.647484e-29 -7.688922e-277
[46] -4.910448e-257  1.047750e+187  1.556882e-296 -7.804792e+268  1.983118e-207
[51] -8.672210e+201  4.650370e+105 -3.736430e-227  -1.637330e+40 -1.512451e+152
[56]  1.396793e+194 -4.433286e-186   1.605388e+03             NA  2.607847e-176
[61]   0.000000e+00

```
#### Logtable :

```R
> seed_analyze$logtable[[1]]
      err.kind                message           file.line
1: InvalidRead Invalid read of size 8 src/mi_cmi.cpp : 57
                                                      address.msg address.trace
1: Address 0x83bf0b8 is 0 bytes after a block of size 536 alloc'd          <NA> 

```
When we run the code on random seed 1604461988 for 5 seconds on a 12 core system gives the following Valgrind error :
Invalid read of size 8 on function mi_cmi.cpp at line number 57.
Let's run the same seed with the same timer on a single-core system.

The following 5 seconds timer generates only one crash file.
I get the same output when I run the testharness on the six-core system.

```R
> seed_analyze <- deepstate_fuzz_fun_analyze(path,1604505428,5)
running the executable .. 
cd /home/akolla/extdata/packages/BNSL/inst/testfiles/mi && valgrind --xml=yes --xml-file=/home/akolla/extdata/packages/BNSL/inst/testfiles/mi/5_1604505428/1604505428_log --tool=memcheck --leak-check=yes --track-origins=yes ./mi_DeepState_TestHarness --seed=1604505428 --timeout=5 --fuzz > /home/akolla/extdata/packages/BNSL/inst/testfiles/mi/5_1604505428/seed_valgrind_log_text 2>&1
```
####Output :
```R
> seed_analyze
      inputs          logtable
1: <list[3]> <data.table[1x5]>

``` 
####Inputs :
```R
> seed_analyze$inputs[[1]]
$proc
[1] -1490887255

$x
 [1] -8.213736e-268 -1.951914e+111 -9.649061e+204  5.483429e-282 -5.254999e+138
 [6]           -Inf -3.494727e+224  1.615706e-259  9.814026e+151 -6.122974e+263
[11] -9.058724e-124   1.234785e+39 -6.111462e-282 -8.484119e-261  4.098092e-244
[16]   6.243492e-15  7.166968e+145  2.216562e-289  5.401894e+301   7.074301e-83
[21]   1.614506e-06   7.276571e-91 -1.848420e-262  4.080779e-261  8.245177e-141
[26] -4.667568e+297 -8.396755e-120  -1.528634e+65   1.256631e-53  5.962137e+130
[31]            Inf -1.522745e+144  1.272269e+122  2.775626e+299 -1.306130e+303
[36] -8.402301e+206  6.906340e-159  -8.118336e+06            NaN  -3.902448e-94
[41]  3.706050e-145 -1.057885e+173  3.247081e-159 -7.658977e+127  1.693851e+259
[46]  1.392144e-269 -7.915335e-116 -9.418790e+169 -5.420730e+268   4.083569e+44
[51]   1.881020e-59  1.844175e-265  -1.150161e+30  7.938846e+261  -1.039836e-02
[56] -8.750542e-238  3.986502e+148  1.945617e-176  -1.307638e+71 -6.730065e-215
[61]  3.305767e+287 -2.410883e+241  5.535715e+255 -5.280081e+226  2.130304e-247
[66]  1.456126e-267  1.534800e-128 -1.756331e-123   4.660655e+92            Inf
[71]   5.351347e-05  2.744170e+135   2.006996e+85             NA   4.968092e+43
[76]  -9.867509e+86 -4.357863e+272   3.973519e-86   5.242205e-20  2.716399e-253
[81]   0.000000e+00

$y
 [1]   6.742224e-54   2.264369e-75  2.552010e+103 -1.634306e-293 -3.580926e-165
 [6] -1.184564e+263 -4.295887e-290  -1.584849e+06   6.878235e+06            Inf
[11]            NaN  5.146838e+287 -4.904249e+226   4.395615e+79  1.618476e-264
[16] -3.580926e-165  2.370920e+250  5.806345e+280 -1.718164e-268  9.788555e-196
[21] -3.580926e-165  4.075043e-281 -9.659011e-171   0.000000e+00
```
####Logtable:

```R
> seed_analyze$logtable[[1]]
      err.kind                message           file.line
1: InvalidRead Invalid read of size 8 src/mi_cmi.cpp : 57
                                                      address.msg address.trace
1: Address 0xaa492a0 is 0 bytes after a block of size 240 alloc'd          <NA>

```

The output produced on both the systems is similar and shows that the issue is with line 57 in mi_cmi.cpp file.
