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
==6879== Conditional jump or move depends on uninitialised value(s)
==6879==    at 0x1088DC49: FastFloor (FastNoise.cpp:184)
==6879==    by 0x1088DC49: FastNoise::SingleCubic(unsigned char, double, double) const (FastNoise.cpp:1827)
==6879==    by 0x1089F682: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6879==    by 0x1089230A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6879==    by 0x4F35BB5: do_dotcall (dotcode.c:1252)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4F7CBC9: Rf_eval (eval.c:743)
==6879==    by 0x4F047AC: do_docall (coerce.c:2638)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
==6879== Use of uninitialised value of size 8
==6879==    at 0x1088DC82: Index2D_256 (FastNoise.cpp:271)
==6879==    by 0x1088DC82: ValCoord2DFast (FastNoise.cpp:318)
==6879==    by 0x1088DC82: FastNoise::SingleCubic(unsigned char, double, double) const (FastNoise.cpp:1839)
==6879==    by 0x1089F682: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6879==    by 0x1089230A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6879==    by 0x4F35BB5: do_dotcall (dotcode.c:1252)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4F7CBC9: Rf_eval (eval.c:743)
==6879==    by 0x4F047AC: do_docall (coerce.c:2638)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
==6879== Use of uninitialised value of size 8
==6879==    at 0x1088DD17: Index2D_256 (FastNoise.cpp:271)
==6879==    by 0x1088DD17: ValCoord2DFast (FastNoise.cpp:318)
==6879==    by 0x1088DD17: FastNoise::SingleCubic(unsigned char, double, double) const (FastNoise.cpp:1839)
==6879==    by 0x1089F682: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6879==    by 0x1089230A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6879==    by 0x4F35BB5: do_dotcall (dotcode.c:1252)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4F7CBC9: Rf_eval (eval.c:743)
==6879==    by 0x4F047AC: do_docall (coerce.c:2638)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
==6879== Use of uninitialised value of size 8
==6879==    at 0x1088DD1B: Index2D_256 (FastNoise.cpp:271)
==6879==    by 0x1088DD1B: ValCoord2DFast (FastNoise.cpp:318)
==6879==    by 0x1088DD1B: FastNoise::SingleCubic(unsigned char, double, double) const (FastNoise.cpp:1839)
==6879==    by 0x1089F682: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6879==    by 0x1089230A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6879==    by 0x4F35BB5: do_dotcall (dotcode.c:1252)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4F7CBC9: Rf_eval (eval.c:743)
==6879==    by 0x4F047AC: do_docall (coerce.c:2638)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
==6879== Use of uninitialised value of size 8
==6879==    at 0x1088DDF0: Index2D_256 (FastNoise.cpp:271)
==6879==    by 0x1088DDF0: ValCoord2DFast (FastNoise.cpp:318)
==6879==    by 0x1088DDF0: FastNoise::SingleCubic(unsigned char, double, double) const (FastNoise.cpp:1839)
==6879==    by 0x1089F682: gen_cubic2d_c(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, double, int) (cubic.cpp:85)
==6879==    by 0x1089230A: _ambient_gen_cubic2d_c (RcppExports.cpp:59)
==6879==    by 0x4F35BB5: do_dotcall (dotcode.c:1252)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4F7CBC9: Rf_eval (eval.c:743)
==6879==    by 0x4F047AC: do_docall (coerce.c:2638)
==6879==    by 0x4F6FC50: bcEval (eval.c:6765)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
> str(deepstate.result)
==6879== Conditional jump or move depends on uninitialised value(s)
==6879==    at 0x4FA9760: lunary (logic.c:212)
==6879==    by 0x4FA9760: do_logic (logic.c:60)
==6879==    by 0x4F6E78E: bcEval (eval.c:6837)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4FBD622: dispatchMethod.isra.5 (objects.c:436)
==6879==    by 0x4FBDA5F: Rf_usemethod (objects.c:486)
==6879==    by 0x4FBDE21: do_usemethod (objects.c:565)
==6879==    by 0x4F70B71: bcEval (eval.c:6785)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879== 
==6879== Conditional jump or move depends on uninitialised value(s)
==6879==    at 0x5030BC6: logicalSubscript (subscript.c:577)
==6879==    by 0x5033C93: Rf_makeSubscript (subscript.c:1000)
==6879==    by 0x5036F13: VectorSubset (subset.c:193)
==6879==    by 0x5036F13: do_subset_dflt (subset.c:823)
==6879==    by 0x4F6C511: VECSUBSET_PTR (eval.c:5433)
==6879==    by 0x4F6C511: bcEval (eval.c:6973)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879==    by 0x4F7E85E: R_execClosure (eval.c:1780)
==6879==    by 0x4F7F5A2: Rf_applyClosure (eval.c:1706)
==6879==    by 0x4FBD622: dispatchMethod.isra.5 (objects.c:436)
==6879==    by 0x4FBDA5F: Rf_usemethod (objects.c:486)
==6879==    by 0x4FBDE21: do_usemethod (objects.c:565)
==6879==    by 0x4F70B71: bcEval (eval.c:6785)
==6879==    by 0x4F7C9FF: Rf_eval (eval.c:620)
==6879== 
 num [1:9] NaN NaN NaN NaN NaN ... 
==6879== 
==6879== HEAP SUMMARY:
==6879==     in use at exit: 76,559,975 bytes in 10,724 blocks
==6879==   total heap usage: 30,647 allocs, 19,923 frees, 115,945,040 bytes allocated
==6879== 
==6879== LEAK SUMMARY:
==6879==    definitely lost: 0 bytes in 0 blocks
==6879==    indirectly lost: 0 bytes in 0 blocks
==6879==      possibly lost: 0 bytes in 0 blocks
==6879==    still reachable: 76,559,975 bytes in 10,724 blocks
==6879==                       of which reachable via heuristic:
==6879==                         newarray           : 4,264 bytes in 1 blocks
==6879==         suppressed: 0 bytes in 0 blocks
==6879== Rerun with --leak-check=full to see details of leaked memory
==6879== 
==6879== For counts of detected and suppressed errors, rerun with: -v
==6879== Use --track-origins=yes to see where uninitialised values come from
==6879== ERROR SUMMARY: 7 errors from 7 contexts (suppressed: 0 from 0)

```
The valgrind trace in short shows the following errors:

```R
> library(RcppDeepState)
> RcppDeepState::deepstate_logtest("/home/akhila/R/packages/ambient/inst/testfiles/gen_cubic2d_c/gen_cubic2d_c_output/log_9d161c6d257a3a7d8792ee6bd627aaf470a8c05f/valgrind_log")

              kind                                                        msg
1: UninitCondition Conditional jump or move depends on uninitialised value(s)
2: UninitCondition Conditional jump or move depends on uninitialised value(s)
3: UninitCondition Conditional jump or move depends on uninitialised value(s)
4: UninitCondition Conditional jump or move depends on uninitialised value(s)
                 errortrace                address trace
1:  src/FastNoise.cpp : 184 No address trace found    NA
2: src/FastNoise.cpp : 1827 No address trace found    NA
3: src/FastNoise.cpp : 1819 No address trace found    NA
4:       src/cubic.cpp : 85 No address trace found    NA
```

We see that an uninitialized-value use error is reported in the program. This happens when we use a value that hasn't been initialized.
Valgrind complaints about this when the program tries to make use of this memory and it shows a Conditional jump or move depends on uninitialized value(s) message which results in undefined behavior.


Thanks to [Dr.Toby Dylan Hocking](https://tdhock.github.io/blog/) for his support on the project.
This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 




