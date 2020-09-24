---
title: Prototyping test harness
author: Akhila Chowdary Kolla
categories: [Proposal]
tags: [Prototyping,ValgrindTest]
math: true
---

As a part of (R consortium fuzz testing proposal)[https://docs.google.com/document/d/15z2diPTJ3MTLe3H9WfHNAdzkn1bYIsLJ6_tpGWRqQBU/edit], we are creating a simple prototype on how to extract the valid inputs for a function from the RcppTestPackage and test it with Valgrind. We also test the `test harness` of the same rcpp method by passing DeepState's randomized inputs and see if Valgrind can detect any errors for those inputs.

To test an rcpp function, we need to extract the valid inputs first. Usually, not all the rcpp methods have their code documented with well-written examples. Instead, we can look for the rcpp function calls in the package and obtain the inputs. These rcpp function calls are found mostly in another R function. The first we need to do is identify those R functions and check for an rcpp function call and save those inputs.

Here we will perform all our tests and analysis on the Rcpp package `ambient`. For example, consider the rcpp function - `gen_cubic2d_c` we will test this function with both valid inputs and DeepState generated random inputs and check for Valgrind errors. First, check the man page of the function for its examples. But when I refer to the man pages there is no documentation for the rcpp function `gen_cubic2d_c`. In this case, we need to follow the below steps to get the inputs. 

* Identify the function call in the package:

```c++
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

* Identify the inputs passed:

Now lets check the R/noise-cubic.R and find the gen_cubic2d_c function call.







 


