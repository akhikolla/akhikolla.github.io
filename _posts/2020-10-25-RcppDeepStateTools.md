---
title: RcppDeepStateTools 
author: Akhila Chowdary Kolla
categories: [Tools]
tags: [RcppDeepStateTools,Tools,AFL,libfuzzer,honggfuzz,Eclipser,Angora]
math: true
---

As a part of the RcppDeepState project, we created another Linux specific R package, RcppDeepStateTools fuzz tests Rcpp packages under different external fuzzers. RcppDeepStateTools provides an interface to  five external fuzzers :

1. AFL 
2. LibFuzzer 
3. Angora
4. HonggFuzz
5. Eclipser

In this blog post, I'll discuss how to use these external fuzzers for running the DeepState function-specific test harnesses. 
To test any Rcpp package using any of these fuzzers is as easy as making a call to deepstate_compile_tools() function.

```R
RcppDeepState::deepstate_compile_tools(path)
```

This function takes the path to the package that you want to test. I'll use the testSAN package in the RcppDeepState testpkgs folder as my default test package.

```R
> path <- "~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN"
> RcppDeepStateTools::deepstate_compile_tools(path)
Please choose an option to select the Fuzzer:
1 - AFL
2 - LibFuzzer
3 - Eclipser
4 - HonggFuzz
```

Here choosing any of the options - will do the following:
1. Install the fuzzer.
2. Compile and link the specific fuzzer with the deepstate base library.
3. Compile and Run the Testharnesses.


Choosing a fuzzer would also generate a fuzzer specific makefile for the testharness that was previously created by RcppDeepState for each test package function. 

If I want to test the package using AFL I have to choose 1 as my option :

```shell
> RcppDeepStateTools::deepstate_compile_tools(path)
Please choose an option to select the Fuzzer:
1 - AFL
2 - LibFuzzer
3 - Eclipser
4 - HonggFuzz
1
[1] "rm -f *.o && make -f /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/AFL.Makefile"
/home/akhila/.RcppDeepState/afl-2.52b/afl-clang++ -g -I/usr/share/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/qs/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.cpp -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.afl.o -c
afl-cc 2.52b by <lcamtuf@google.com>
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 533 locations (64-bit, non-hardened mode, ratio 100%).
/home/akhila/.RcppDeepState/afl-2.52b/afl-clang++ -g -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.afl /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.afl.o -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include -I/home/akhila/.RcppDeepState/deepstate-master/build_afl -I/home/akhila/.RcppDeepState/deepstate-master/src/include -L/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -Wl,-rpath=/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -L/usr/share/R/lib -Wl,-rpath=/usr/share/R/lib -L/home/akhila/.RcppDeepState/deepstate-master/build_afl -Wl,-rpath=/home/akhila/.RcppDeepState/deepstate-master/build_afl -lR -lRInside -ldeepstate_AFL -I/usr/share/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/.RcppDeepState/deepstate-master/src/include /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/src/*.cpp
afl-cc 2.52b by <lcamtuf@google.com>
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 1364 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
afl-as 2.52b by <lcamtuf@google.com>
[+] Instrumented 25 locations (64-bit, non-hardened mode, ratio 100%).
cd /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound && /home/akhila/.RcppDeepState/afl-2.52b/afl-fuzz -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/afl_rcpp_read_out_of_bound_output -m 150 -t 2000 -i ~/.RcppDeepState/deepstate-master/build_afl/ -- /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.afl --fuzz_save_passing 
afl-fuzz 2.52b by <lcamtuf@google.com>
[+] You have 12 CPU cores and 2 runnable tasks (utilization: 17%).
[+] Try parallel jobs - see docs/parallel_fuzzing.txt.
[*] Checking CPU core loadout...
[+] Found a free CPU core, binding to #0.
[*] Checking core_pattern...
[*] Checking CPU scaling governor...
[*] Setting up output directories...
[*] Scanning '/home/akhila/.RcppDeepState/deepstate-master/build_afl/'...
[+] No auto-generated dictionary tokens to reuse.
[*] Creating hard links for all input files...
[*] Validating target binary...
[*] Attempting dry run with 'id:000000,orig:CMakeCache.txt'...
[*] Spinning up the fork server...
[+] All right - fork server is up.
    len = 14294, map size = 248, exec speed = 1579 us
[*] Attempting dry run with 'id:000001,orig:Makefile'...
    len = 27177, map size = 248, exec speed = 1551 us
[!] WARNING: No new instrumentation output, test case may be useless.
[*] Attempting dry run with 'id:000002,orig:cmake_install.cmake'...
    len = 3243, map size = 248, exec speed = 1543 us
[!] WARNING: No new instrumentation output, test case may be useless.
[*] Attempting dry run with 'id:000003,orig:libdeepstate.a'...
    len = 422642, map size = 248, exec speed = 1610 us
[!] WARNING: No new instrumentation output, test case may be useless.
[*] Attempting dry run with 'id:000004,orig:libdeepstate32.a'...
    len = 299086, map size = 248, exec speed = 1630 us
[!] WARNING: No new instrumentation output, test case may be useless.
[*] Attempting dry run with 'id:000005,orig:libdeepstate_AFL.a'...
    len = 422642, map size = 248, exec speed = 1630 us
[!] WARNING: No new instrumentation output, test case may be useless.
[*] Attempting dry run with 'id:000006,orig:setup.py'...
    len = 2281, map size = 248, exec speed = 1560 us
[!] WARNING: No new instrumentation output, test case may be useless.
[+] All test cases processed.

[!] WARNING: Some test cases are huge (412 kB) - see docs/perf_tips.txt!
[!] WARNING: Some test cases look useless. Consider using a smaller set.
[+] Here are some useful stats:

    Test case count : 1 favored, 0 variable, 7 total
       Bitmap range : 248 to 248 bits (average: 248.00 bits)
        Exec timing : 1543 to 1630 us (average: 1586 us)

[+] All set and ready to roll!

```

The afl output looks something like this:
![Test Image 4](https://github.com/akhikolla/akhikolla.github.io/tree/master/assets/img/sample/afl.png)




