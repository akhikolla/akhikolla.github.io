---
title: Prototyping Realistic Input Generation
author: Akhila Chowdary Kolla
categories: [Proposal]
tags: [Prototyping,ValgrindTest]
math: true
---

As a part of [R consortium fuzz testing proposal](https://docs.google.com/document/d/15z2diPTJ3MTLe3H9WfHNAdzkn1bYIsLJ6_tpGWRqQBU/edit), we are creating a simple prototype on how to extract the valid inputs for a function from the RcppTestPackage and test it with Valgrind. We also test the `test harness` of the same rcpp method by passing DeepState's randomized inputs and see if Valgrind can detect any errors for those inputs.

To test an rcpp function, we need to extract the valid inputs first. Usually, not all the rcpp methods have their code documented with well-written examples. Instead, we can look for the rcpp function calls in the package and obtain the inputs. These rcpp function calls are found mostly in another R function. The first we need to do is identify those R functions and check for an rcpp function call and save those inputs.

Here we will perform all our tests and analysis on the Rcpp package `ambient`. For example, consider the rcpp function - `gen_cubic2d_c` we will test this function with both valid inputs and DeepState generated random inputs and check for Valgrind errors. First, check the man page of the function for its examples. But when I refer to the man pages there is no documentation for the rcpp function `gen_cubic2d_c`. In this case, we need to follow the below steps to get the inputs. 

**Identify the function call in the package:**

```shell
akhila@akhila-VirtualBox:~/R/packages/ambient$ grep -Rw -e "gen_cubic2d_c"
Binary file src/RcppExports.o matches
src/cubic.cpp:NumericVector gen_cubic2d_c(NumericVector x, NumericVector y, double freq, int seed) {
Binary file src/cubic.o matches
src/RcppExports.cpp:// gen_cubic2d_c
src/RcppExports.cpp:NumericVector gen_cubic2d_c(NumericVector x, NumericVector y, double freq, int seed);
src/RcppExports.cpp:    rcpp_result_gen = Rcpp::wrap(gen_cubic2d_c(x, y, freq, seed));
R/noise-cubic.R:    gen_cubic2d_c(dims$x, dims$y, frequency, seed)
R/RcppExports.R:gen_cubic2d_c <- function(x, y, freq, seed)
```

From above trace we can see that `gen_cubic2d_c` is found in RcppExports.cpp, RcppExports.R and src/cubic.cpp. But the function call to `gen_cubic2d_c` with valid inputs can be found in R/noise-cubic.R.

**Identify the inputs passed:**

Now, let's check the R/noise-cubic.R and find the gen_cubic2d_c function call.

```R
gen_cubic <- function(x, y = NULL, z = NULL, frequency = 1, seed = NULL, ...) {
  dims <- check_dims(x, y, z)
  if (is.null(seed)) seed <- random_seed()
  if (is.null(z)) {
    gen_cubic2d_c(dims$x, dims$y, frequency, seed)
  } else {
    gen_cubic3d_c(dims$x, dims$y, dims$z, frequency, seed)
  }
}
```
From this, we can understand that a call to `gen_cubic` is to be made to invoke `gen_cubic2d_c`. There is documentation available for the function gen_cubic in man/noise_cubic.R.

```R
g.seq <- seq(1, 10, length.out = 1000)
grid <- long_grid(g.seq, g.seq)
grid$noise <- gen_cubic(grid$x, grid$y)
```

Now make a call to gen_cubic() and store the inputs into a list. To make a call to gen_cubic() in noise_cubic.Rd we need to run the following script:

```shell
akhila@akhila-VirtualBox:~/R/packages/ambient$R vanilla -e 'example("noise_cubic",package="ambient")'

```

This will open the R session and will run the gen_cubic() and when these values are printed to the console the inputs are as follows:

```R 
List of 4
 $ x   : num [1:1000000] 1 1 1 1 1 1 1 1 1 1 ...
 $ y   : num [1:1000000] 1 1.01 1.02 1.03 1.04 ...
 $ freq: num 1
 $ seed: int 880336483

```

These stored inputs are the real and valid arguments to the function `gen_cubic2d_c`. The Rscript to make a call to the function with the above inputs looks like this.

**ambient-test.R**

```R
g.seq <- seq(1, 10, length.out = 1000)
grid <- ambient::long_grid(g.seq, g.seq)
valid.args <- list(
  x=grid$x,
  y=grid$y,
  freq=1,
  seed=880336483L)
str(valid.args)
valid.result <- do.call(ambient:::cubic_2d_c, worklist)
str(valid.result)
```

**Output**
```shell
akhila@akhila-VirtualBox:~/R/packages/ambient$ R -d valgrind --vanilla < ambient-test.R
==6585== Memcheck, a memory error detector
==6585== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==6585== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==6585== Command: /home/akhila/lib/R/bin/exec/R --vanilla
==6585== 

R version 3.6.3 (2020-02-29) -- "Holding the Windsock"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> g.seq <- seq(1, 10, length.out = 1000)
> grid <- ambient::long_grid(g.seq, g.seq)
> valid.args <- list(
+   x=grid$x,
+   y=grid$y,
+   freq=1,
+   seed=880336483L)
> str(valid.args)
List of 4
 $ x   : num [1:1000000] 1 1 1 1 1 1 1 1 1 1 ...
 $ y   : num [1:1000000] 1 1.01 1.02 1.03 1.04 ...
 $ freq: num 1
 $ seed: int 880336483
> valid.result <- do.call(ambient:::gen_cubic2d_c, valid.args)
> str(valid.result)
 num [1:1000000] -0.228 -0.227 -0.226 -0.224 -0.223 ...
> 
==6585== 
==6585== HEAP SUMMARY:
==6585==     in use at exit: 76,529,943 bytes in 10,702 blocks
==6585==   total heap usage: 30,343 allocs, 19,641 frees, 115,713,891 bytes allocated
==6585== 
==6585== LEAK SUMMARY:
==6585==    definitely lost: 0 bytes in 0 blocks
==6585==    indirectly lost: 0 bytes in 0 blocks
==6585==      possibly lost: 0 bytes in 0 blocks
==6585==    still reachable: 76,529,943 bytes in 10,702 blocks
==6585==                       of which reachable via heuristic:
==6585==                         newarray           : 4,264 bytes in 1 blocks
==6585==         suppressed: 0 bytes in 0 blocks
==6585== Rerun with --leak-check=full to see details of leaked memory
==6585== 
==6585== For counts of detected and suppressed errors, rerun with: -v
==6585== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

```
 
For these valid inputs we don't see any Valgrind memory leaks or bugs in the code but when we executed the same code with DeepState generated randomized inputs we are able to detect bugs in the rcpp function.

Inputs generated by DeepState for the function `gen_cubic2d_c`:

```R
> str(deepstate.args)
List of 4
 $ x   : num [1:9] Inf Inf NaN -2.15e-100 NaN ...
 $ y   : num [1:7] -Inf NaN NaN -1.20e+297 -4.39e-67 ...
 $ freq: num 2.23e-63
 $ seed: int 777429942
```
**ambient-test.R**

```R
deepstate.args <- list(
  x=c(Inf, Inf, NaN, -2.1518e-100, NaN, -2.9298e+100, NaN, -Inf, 0),
  y=c(-Inf, NaN, NaN, -1.20158e+297, -4.39091e-67, Inf, 0),freq=2.23269e-63,
  seed=777429942L)
str(deepstate.args)
deepstate.result <- do.call(ambient:::gen_cubic2d_c, deepstate.args)
str(deepstate.result)
```

When we add the below code to ambient-test.R and run it with valgrind we can see the following Valgrind error trace.

**Output**

```shell
==6224== Invalid read of size 8
==6224==    at 0x112E6679: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6224==    by 0x112D930A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6224==    by 0x4F3B953: R_doDotCall (dotcode.c:607)
==6224==    by 0x4F3BE85: do_dotcall (dotcode.c:1280)
==6224==    by 0x4F79765: bcEval (eval.c:7099)
==6224==    by 0x4F85D4F: Rf_eval (eval.c:723)
==6224==    by 0x4F87B6E: R_execClosure (eval.c:1888)
==6224==    by 0x4F88936: Rf_applyClosure (eval.c:1814)
==6224==    by 0x4F85F22: Rf_eval (eval.c:846)
==6224==    by 0x4F08C0C: do_docall (coerce.c:2704)
==6224==    by 0x4F79765: bcEval (eval.c:7099)
==6224==    by 0x4F85D4F: Rf_eval (eval.c:723)
==6224==  Address 0x10f96e20 is 2,912 bytes inside a block of size 7,960 alloc`d
==6224==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==6224==    by 0x4FC4B02: GetNewPage (memory.c:946)
==6224==    by 0x4FC6B41: Rf_allocVector3 (memory.c:2784)
==6224==    by 0x5027381: Rf_allocVector (Rinlinedfuns.h:593)
==6224==    by 0x5027381: ReadItem (serialize.c:1932)
==6224==    by 0x502852F: ReadBCConsts (serialize.c:2101)
==6224==    by 0x502852F: ReadBC1 (serialize.c:2118)
==6224==    by 0x50277EC: ReadBC (serialize.c:2129)
==6224==    by 0x50277EC: ReadItem (serialize.c:1966)
==6224==    by 0x5026B39: ReadItem (serialize.c:1873)
==6224==    by 0x5027596: ReadItem (serialize.c:1961)
==6224==    by 0x50288BD: R_Unserialize (serialize.c:2181)
==6224==    by 0x5029C59: R_unserialize (serialize.c:2892)
==6224==    by 0x502A090: do_lazyLoadDBfetch (serialize.c:3196)
==6224==    by 0x4F862D5: Rf_eval (eval.c:830)
==6224== 
> str(deepstate.result)
 num [1:9] NaN NaN NaN NaN NaN ...
> 
> 
==6224== 
==6224== HEAP SUMMARY:
==6224==     in use at exit: 77,286,802 bytes in 10,884 blocks
==6224==   total heap usage: 30,258 allocs, 19,374 frees, 116,140,760 bytes allocated
==6224== 
==6224== LEAK SUMMARY:
==6224==    definitely lost: 0 bytes in 0 blocks
==6224==    indirectly lost: 0 bytes in 0 blocks
==6224==      possibly lost: 0 bytes in 0 blocks
==6224==    still reachable: 77,286,802 bytes in 10,884 blocks
==6224==                       of which reachable via heuristic:
==6224==                         newarray           : 4,264 bytes in 1 blocks
==6224==         suppressed: 0 bytes in 0 blocks
==6224== Rerun with --leak-check=full to see details of leaked memory
==6224== 
==6224== For counts of detected and suppressed errors, rerun with: -v
==6224== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 75 from 1)

```
We see that an InvalidRead error is reported in the program. This happens when the process tries to access the memory location that is outside of the available memory locations. Here the size 8 means that the process was trying to read 8 bytes. It also gives information about memory addresses.


Thanks to [Dr.Toby Dylan Hocking](https://tdhock.github.io/blog/) for his support on the project.
This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 




