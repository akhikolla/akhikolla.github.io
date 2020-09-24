---
title: Prototyping test harness
author: Akhila Chowdary Kolla
categories: [Proposal]
tags: [Prototyping,ValgrindTest]
math: true
---


As a part of (R consortium fuzz testing proposal)[https://docs.google.com/document/d/15z2diPTJ3MTLe3H9WfHNAdzkn1bYIsLJ6_tpGWRqQBU/edit], we are creating a simple prototype on how to extract the valid inputs for a function from the RcppTestPackage and test it with Valgrind. We also test the `test harness` of the same rcpp method by passing DeepState's randomized inputs and see if Valgrind can detect any errors for those inputs.



