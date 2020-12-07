---
title: Random Seed on BNSL package
author: Akhila Chowdary Kolla
categories: [RcppDeepState Results,Packages]
tags: [R,Valgrind,PackagesIssues,deepstate,FuzzerIssues]
math: true
---
Over the past few months, I have been working on testing the Rcpp packages using RcppDeepState in the cluster. When we fuzz test each Rcpp function-specific testharness I have found issues in 137 packages. 

The List of packages that we found issues in can be found (here)[https://akhikolla.github.io/package-specific/root.html].


The number of inputs passed onto the target binary keeps varying depending on the clock speed, number of cores, cache size of the system. The default timer is 2 minutes, depending on the system's speed RcppDeepState generates as many inputs as possible. The inputs generated are usually stored in .crash/.fail/.pass files.

