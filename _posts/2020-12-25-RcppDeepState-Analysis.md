---
title: RcppDeepState Analysis
author: Akhila Chowdary Kolla
categories: [RcppDeepState Results,Packages]
tags: [R,Valgrind,PackagesIssues,deepstate,FuzzerIssues]
math: true
---

Over the past few months, I have been working on testing the Rcpp packages using RcppDeepState in the cluster. When we fuzz test each Rcpp function-specific testharness I have found issues in 134 packages. 

Refer the [web page](https://akhikolla.github.io./packages-folders/root.html) to check if your Rcpp package has any subtle bugs. This web page lists the packages with Issues in exported functions and unexported functions.


