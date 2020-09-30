---
title: Prototyping Realistic Input Generation - Exported Functions 
author: Akhila Chowdary Kolla
categories: [Proposal]
tags: [Prototyping,ValgrindTest,ExportedFunctions]
math: true
---

As a part of [R consortium fuzz testing proposal](https://docs.google.com/document/d/15z2diPTJ3MTLe3H9WfHNAdzkn1bYIsLJ6_tpGWRqQBU/edit), we are creating a simple prototype on how to extract the valid inputs for a function from the RcppTestPackage and test it with Valgrind. We also test the `test harness` of the same rcpp method by passing DeepState's randomized inputs and see if Valgrind can detect any errors for those inputs.

In the [previous blog post](https://akhikolla.github.io./posts/Prototyping-testharness/) we learned to extract the valid inputs for functions and test the functions with both valid inputs and DeepState generated random inputs and check for Valgrind errors. In this blog post, we will go one step further and try to find errors in the exported functions in the package. 

Here we will perform all our tests and analysis on the Rcpp package `BNSL`. For example, consider the rcpp function - `BNSL::mi` we will test this function with both valid inputs and DeepState generated random inputs and check for Valgrind errors. First, check the man page of the function for its examples. 

**Run the valid example on the function call:**

```R
n=100

x=rbinom(n,1,0.5); y=rbinom(n,1,0.5); mi(x,y)

z=rbinom(n,1,0.1); y=(x+z)%%2

mi(x,y,proc=1); mi(x,y,2) 

x=rnorm(n); y=rnorm(n); mi(x,y,proc=10)

x=rnorm(n); z=rnorm(n); y=0.9*x+sqrt(1-0.9^2)*z; mi(x,y,proc=10)
``` 

**Run the valid inputs:**

Now, let's run the  examples in R/mi.R
 
```shell
akolla@snaps-computer:~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/extdata/compileAttributes/BNSL$ R vanilla -e 'example("mi",package="BNSL")'
ARGUMENT 'vanilla' __ignored__


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

> example("mi",package="BNSL")
Loading required package: bnlearn
Loading required package: igraph

Attaching package: ‘igraph’
mi> n=100

mi> x=rbinom(n,1,0.5); y=rbinom(n,1,0.5); mi(x,y)
[1] 0

mi> z=rbinom(n,1,0.1); y=(x+z)

mi> mi(x,y); mi(x,y,proc=1); mi(x,y,2) 
[1] 0.4333487
[1] 0.4197742
[1] 0.4391413

mi> x=rnorm(n); y=rnorm(n); mi(x,y,proc=10)
[1] 0

mi> x=rnorm(n); z=rnorm(n); y=0.9*x+sqrt(1-0.9^2)*z; mi(x,y,proc=10)
[1] 0.4602899
> 
> 

```
We don't any issues with running the examples on the `mi` function. Now let's try using Valgrind with it and see if we find any issues with the function.

**Run function under valgrind:**
```shell
akolla@snaps-computer:~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/extdata/compileAttributes/BNSL$ R -d valgrind --vanilla < mi.R
==1365578== Memcheck, a memory error detector
==1365578== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==1365578== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==1365578== Command: /usr/lib/R/bin/exec/R --vanilla
==1365578== 

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

> n=100
> args <- list(
+     x=rnorm(n),
+     y=rnorm(n),
+     proc=10) 
> result <- do.call(BNSL::mi,args)
> str(result)
 num 0
> 
==1365578== 
==1365578== HEAP SUMMARY:
==1365578==     in use at exit: 59,988,472 bytes in 14,200 blocks
==1365578==   total heap usage: 56,854 allocs, 42,654 frees, 104,358,740 bytes allocated
==1365578== 
==1365578== LEAK SUMMARY:
==1365578==    definitely lost: 0 bytes in 0 blocks
==1365578==    indirectly lost: 0 bytes in 0 blocks
==1365578==      possibly lost: 0 bytes in 0 blocks
==1365578==    still reachable: 59,988,472 bytes in 14,200 blocks
==1365578==                       of which reachable via heuristic:
==1365578==                         newarray           : 4,264 bytes in 1 blocks
==1365578==         suppressed: 0 bytes in 0 blocks
==1365578== Rerun with --leak-check=full to see details of leaked memory
==1365578== 
==1365578== For lists of detected and suppressed errors, rerun with: -s
==1365578== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)


```
Running the code under Valgrind didn't show any bugs or memory leaks. Let's go one step ahead and try running the function under deepstate generated inputs.

As we know to run code under deepstate:

1.Create testharness for the function.
2.Compile the testharness
3.Run the testharness
4.Generate the crash files
5.Check the crash files for inputs that can break the code.


Now make a call to `mi()` and store the deepstate inputs that are obtained from the crash files into a list.

```shell
akolla@snaps-computer:~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/extdata/compileAttributes/BNSL$ R -d valgrind --vanilla < mi-test.R
```

This will open the R session and will run the mi() with the deepstate inputs and the output is as follows: 


**mi-test.R**

```R
dargs <- list(
x=c(1.05004e+27,1.50426e+215,9.63399e+201,1.1132e+274,2.83593e-82,4.79375e+179,3.55624e+245,7.37432e+52,1.57214e+46,NaN,9.01655e+209,-6.39404e+163,1.52852e-99,3.95231e+100,2.77922e+44,1.64826e-168,2.11394e+53,1.57803e+156,-1.32126e+233,-6.12072e+299,3.46716e+09,2.02395e-62,NaN,4.63683e-299,-1.94216e+211,3.90791e-115,-1.34423e+132,-1.94344e+19,3.26647e-129,-1.78086e+15,6.11934e-118,-7.07064e+298,-1.36403e-174,-9.23416e+298,-3.07213e+235,-5.93601e-265,-1.83547e-282,-1.76343e-119,NaN,-1.55076e+22,-1.08499e+112,-9.44741e+78,NaN,-5.77067e+285,-2.17325e+240,-0.00164493,2.11009e+165,-1.38652e-69,NaN,6.42513e-282,-1.1697e-54,-180216,-2.13757e-104,-3.50357e+180,3.47957e+87,-2.97102e+10,1.27176e+11,-1.9436e+25,3.67334e-276,2.27839e+231,8.11476e+61,7.118e+228,-4.51193e-132,-1.34696e+22,-2.10614e-19,-6.13223e-133,-1.5923e+294,-4.27302e-105,4.78741e-108,0),
y=c(3.89587e+273,-8.19642e+21,NaN,NaN,NaN),
proc=-784179594)
 
dresult <- do.call(BNSL::mi,dargs)
```

**Output**
```shell
akolla@snaps-computer:~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/extdata/compileAttributes/BNSL$ R -d valgrind --vanilla < mi.R
==1365599== Memcheck, a memory error detector
==1365599== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==1365599== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==1365599== Command: /usr/lib/R/bin/exec/R --vanilla
==1365599== 

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

> dargs <- list(
+ x=c(1.05004e+27,1.50426e+215,9.63399e+201,1.1132e+274,2.83593e-82,4.79375e+179,3.55624e+245,7.37432e+52,1.57214e+46,NaN,9.01655e+209,-6.39404e+163,1.52852e-99,3.95231e+100,2.77922e+44,1.64826e-168,2.11394e+53,1.57803e+156,-1.32126e+233,-6.12072e+299,3.46716e+09,2.02395e-62,NaN,4.63683e-299,-1.94216e+211,3.90791e-115,-1.34423e+132,-1.94344e+19,3.26647e-129,-1.78086e+15,6.11934e-118,-7.07064e+298,-1.36403e-174,-9.23416e+298,-3.07213e+235,-5.93601e-265,-1.83547e-282,-1.76343e-119,NaN,-1.55076e+22,-1.08499e+112,-9.44741e+78,NaN,-5.77067e+285,-2.17325e+240,-0.00164493,2.11009e+165,-1.38652e-69,NaN,6.42513e-282,-1.1697e-54,-180216,-2.13757e-104,-3.50357e+180,3.47957e+87,-2.97102e+10,1.27176e+11,-1.9436e+25,3.67334e-276,2.27839e+231,8.11476e+61,7.118e+228,-4.51193e-132,-1.34696e+22,-2.10614e-19,-6.13223e-133,-1.5923e+294,-4.27302e-105,4.78741e-108,0),
+ y=c(3.89587e+273,-8.19642e+21,NaN,NaN,NaN),
+ proc=-784179594)
> 
> dresult <- do.call(BNSL::mi,dargs)
==1365599== Conditional jump or move depends on uninitialised value(s)
==1365599==    at 0xE2EC89A: operator() (functional_hash.h:249)
==1365599==    by 0xE2EC89A: _M_hash_code (hashtable_policy.h:1292)
==1365599==    by 0xE2EC89A: std::__detail::_Map_base<double, std::pair<double const, int>, std::allocator<std::pair<double const, int> >, std::__detail::_Select1st, std::equal_to<double>, std::hash<double>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<false, false, true>, true>::operator[](double const&) (hashtable_policy.h:695)
==1365599==    by 0xE2ED2CA: operator[] (unordered_map.h:985)
==1365599==    by 0xE2ED2CA: operator() (table.h:34)
==1365599==    by 0xE2ED2CA: for_each<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::iter_base<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::const_iter_traits>, Rcpp::sugar::CountInserter<std::unordered_map<double, int, std::hash<double>, std::equal_to<double>, std::allocator<std::pair<double const, int> > >, double> > (stl_algo.h:3876)
==1365599==    by 0xE2ED2CA: Table (table.h:95)
==1365599==    by 0xE2ED2CA: Rcpp::Vector<13, Rcpp::PreserveStorage> Rcpp::table<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >(Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > > const&) (table.h:127)
==1365599==    by 0xE2E5836: Jeffreys_mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int, int) (mi_cmi.cpp:55)
==1365599==    by 0xE2E8D30: mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int) (mi_cmi.cpp:40)
==1365599==    by 0xE2D0AE1: BNSL_mi (RcppExports.cpp:97)
==1365599==    by 0x493B475: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49716CF: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D60F: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498F41E: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x4990252: Rf_applyClosure (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D729: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49081CC: ??? (in /usr/lib/R/lib/libR.so)
==1365599== 
==1365599== Conditional jump or move depends on uninitialised value(s)
==1365599==    at 0xE2EC8A0: operator() (functional_hash.h:249)
==1365599==    by 0xE2EC8A0: _M_hash_code (hashtable_policy.h:1292)
==1365599==    by 0xE2EC8A0: std::__detail::_Map_base<double, std::pair<double const, int>, std::allocator<std::pair<double const, int> >, std::__detail::_Select1st, std::equal_to<double>, std::hash<double>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<false, false, true>, true>::operator[](double const&) (hashtable_policy.h:695)
==1365599==    by 0xE2ED2CA: operator[] (unordered_map.h:985)
==1365599==    by 0xE2ED2CA: operator() (table.h:34)
==1365599==    by 0xE2ED2CA: for_each<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::iter_base<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::const_iter_traits>, Rcpp::sugar::CountInserter<std::unordered_map<double, int, std::hash<double>, std::equal_to<double>, std::allocator<std::pair<double const, int> > >, double> > (stl_algo.h:3876)
==1365599==    by 0xE2ED2CA: Table (table.h:95)
==1365599==    by 0xE2ED2CA: Rcpp::Vector<13, Rcpp::PreserveStorage> Rcpp::table<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >(Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > > const&) (table.h:127)
==1365599==    by 0xE2E5836: Jeffreys_mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int, int) (mi_cmi.cpp:55)
==1365599==    by 0xE2E8D30: mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int) (mi_cmi.cpp:40)
==1365599==    by 0xE2D0AE1: BNSL_mi (RcppExports.cpp:97)
==1365599==    by 0x493B475: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49716CF: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D60F: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498F41E: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x4990252: Rf_applyClosure (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D729: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49081CC: ??? (in /usr/lib/R/lib/libR.so)
==1365599== 
==1365599== Use of uninitialised value of size 8
==1365599==    at 0xE2EC8B6: _M_find_before_node (hashtable.h:1538)
==1365599==    by 0xE2EC8B6: _M_find_node (hashtable.h:655)
==1365599==    by 0xE2EC8B6: std::__detail::_Map_base<double, std::pair<double const, int>, std::allocator<std::pair<double const, int> >, std::__detail::_Select1st, std::equal_to<double>, std::hash<double>, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<false, false, true>, true>::operator[](double const&) (hashtable_policy.h:697)
==1365599==    by 0xE2ED2CA: operator[] (unordered_map.h:985)
==1365599==    by 0xE2ED2CA: operator() (table.h:34)
==1365599==    by 0xE2ED2CA: for_each<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::iter_base<Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >::const_iter_traits>, Rcpp::sugar::CountInserter<std::unordered_map<double, int, std::hash<double>, std::equal_to<double>, std::allocator<std::pair<double const, int> > >, double> > (stl_algo.h:3876)
==1365599==    by 0xE2ED2CA: Table (table.h:95)
==1365599==    by 0xE2ED2CA: Rcpp::Vector<13, Rcpp::PreserveStorage> Rcpp::table<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >(Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > > const&) (table.h:127)
==1365599==    by 0xE2E5836: Jeffreys_mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int, int) (mi_cmi.cpp:55)
==1365599==    by 0xE2E8D30: mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int) (mi_cmi.cpp:40)
==1365599==    by 0xE2D0AE1: BNSL_mi (RcppExports.cpp:97)
==1365599==    by 0x493B475: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49716CF: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D60F: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498F41E: ??? (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x4990252: Rf_applyClosure (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x498D729: Rf_eval (in /usr/lib/R/lib/libR.so)
==1365599==    by 0x49081CC: ??? (in /usr/lib/R/lib/libR.so)
(discontinued log)...
==1365599== Use of uninitialised value of size 8
==1365599==    at 0x4D39C5F: __mpn_addmul_1 (addmul_1.S:41)
==1365599==    by 0x1FFEFF761F: ???
==1365599==    by 0x4D40798: __printf_fp_l (printf_fp.c:532)
==1365599==    by 0x4D596C7: printf_positional (vfprintf-internal.c:2072)
==1365599==    by 0x4D5AF4C: __vfprintf_internal (vfprintf-internal.c:1733)
==1365599==    by 0x4D70119: __vsnprintf_internal (vsnprintf.c:114)
==1365599==    by 0x4E11E40: __snprintf_chk (snprintf_chk.c:38)
==1365599==    by 0xE2EB0D1: snprintf (stdio2.h:67)
==1365599==    by 0xE2EB0D1: coerce_to_string<14> (r_coerce.h:241)
==1365599==    by 0xE2EB0D1: r_coerce<14, 16> (r_coerce.h:279)
==1365599==    by 0xE2EB0D1: r_coerce<14, 16> (r_coerce.h:273)
==1365599==    by 0xE2EB0D1: operator()<std::pair<double const, int> > (table.h:49)
==1365599==    by 0xE2EB0D1: Rcpp::sugar::Grabber<std::map<double, int, Rcpp::internal::NAComparator<double>, std::allocator<std::pair<double const, int> > >, 14> std::for_each<std::_Rb_tree_const_iterator<std::pair<double const, int> >, Rcpp::sugar::Grabber<std::map<double, int, Rcpp::internal::NAComparator<double>, std::allocator<std::pair<double const, int> > >, 14> >(std::_Rb_tree_const_iterator<std::pair<double const, int> >, std::_Rb_tree_const_iterator<std::pair<double const, int> >, Rcpp::sugar::Grabber<std::map<double, int, Rcpp::internal::NAComparator<double>, std::allocator<std::pair<double const, int> > >, 14>) (stl_algo.h:3876)
==1365599==    by 0xE2ED60E: operator Rcpp::IntegerVector (table.h:106)
==1365599==    by 0xE2ED60E: Rcpp::Vector<13, Rcpp::PreserveStorage> Rcpp::table<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > >(Rcpp::VectorBase<14, true, Rcpp::sugar::Plus_Vector_Vector<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage>, true, Rcpp::sugar::Times_Vector_Primitive<14, true, Rcpp::Vector<14, Rcpp::PreserveStorage> > > > const&) (table.h:127)
==1365599==    by 0xE2E5836: Jeffreys_mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int, int) (mi_cmi.cpp:55)
==1365599==    by 0xE2E8D30: mi(Rcpp::Vector<14, Rcpp::PreserveStorage>, Rcpp::Vector<14, Rcpp::PreserveStorage>, int) (mi_cmi.cpp:40)
==1365599==    by 0xE2D0AE1: BNSL_mi (RcppExports.cpp:97)
==1365599== 
> print(str(dresult))
 num 2.66
NULL
> 
> 
==1365599== 
==1365599== HEAP SUMMARY:
==1365599==     in use at exit: 58,885,992 bytes in 12,798 blocks
==1365599==   total heap usage: 37,927 allocs, 25,129 frees, 102,635,213 bytes allocated
==1365599== 
==1365599== LEAK SUMMARY:
==1365599==    definitely lost: 0 bytes in 0 blocks
==1365599==    indirectly lost: 0 bytes in 0 blocks
==1365599==      possibly lost: 0 bytes in 0 blocks
==1365599==    still reachable: 58,885,992 bytes in 12,798 blocks
==1365599==                       of which reachable via heuristic:
==1365599==                         newarray           : 4,264 bytes in 1 blocks
==1365599==         suppressed: 0 bytes in 0 blocks
==1365599== Rerun with --leak-check=full to see details of leaked memory
==1365599== 
==1365599== Use --track-origins=yes to see where uninitialised values come from
==1365599== For lists of detected and suppressed errors, rerun with: -s
==1365599== ERROR SUMMARY: 31504 errors from 311 contexts (suppressed: 0 from 0)
```
The complete log can be found here [valgrind log mi](https://github.com/akhikolla/RcppDeepStateTools/blob/master/inst/valgrind_log_mi)

We see that 31504 errors are reported in the program from 311 contexts.
Also, we can see issues like :

Conditional jump or move depends on the uninitialized value(s)
Use of an uninitialized value of size 8

This happens when the process tries to access the memory location that is outside of the available memory locations or the uninitialized memory. Here the size 8 means that the process was trying to read 8 bytes. It also gives information about memory addresses.

Thanks to [Dr. Toby Dylan Hocking](https://tdhock.github.io/blog/) for his support on the project.
This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 




