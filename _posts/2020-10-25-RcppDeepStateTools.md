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

**AFL:**

American Fuzzy Lop is a brute-force fuzzer that automatically discovers interesting testcases that could explore new paths of the target binary. Using AFL could helps us uncover the unexplored paths that usually not tested by normal brute force fuzzers.

If we want to test the package using AFL we have to choose option 1 :

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

<img src="https://github.com/akhikolla/akhikolla.github.io/blob/master/assets/img/sample/afl.png"/>


When we run a target binary using AFL we get an afl output folder which contains the following directories:
1. crashes - This folder contains all the file inputs that caused crashes on the executable
2. queue - This folder contains all the inputs that are generated by the fuzzer.
3. fuzzer_stats - This file hhas all statistic info (unique_crashes:0, fuzzer_pid:6030)
4. plot_data - This file has details like unix_time, cycles_done, cur_path, paths_total, pending_total, pending_favs, map_size, unique_crashes, unique_hangs, max_depth for every input passed.


**HonggFuzz**

When we test the package using hongg fuzz the output looks something like this:

First runs the make file and then executes the testharness:

```shell
> RcppDeepStateTools::deepstate_compile_tools(path)
Please choose an option to select the Fuzzer:
1 - AFL
2 - LibFuzzer
3 - Eclipser
4 - HonggFuzz
4
[1] "rm -f *.o && make -f /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/Hongg.Makefile"
/home/akhila/.RcppDeepState/honggfuzz/hfuzz_cc/hfuzz-clang++ -g -I/usr/share/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/qs/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.cpp -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.hongg.o -c
/home/akhila/.RcppDeepState/honggfuzz/hfuzz_cc/hfuzz-clang++ -g -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.hongg /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.hongg.o -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/include -I/home/akhila/.RcppDeepState/deepstate-master/build_honggfuzz -I/home/akhila/.RcppDeepState/deepstate-master/src/include -L/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -Wl,-rpath=/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RInside/lib -L/usr/share/R/lib -Wl,-rpath=/usr/share/R/lib -L/home/akhila/.RcppDeepState/deepstate-master/build_honggfuzz -Wl,-rpath=/home/akhila/.RcppDeepState/deepstate-master/build_honggfuzz -lR -lRInside -ldeepstate_HFUZZ -I/usr/share/R/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/Rcpp/include -I/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppArmadillo/include -I/home/akhila/.RcppDeepState/deepstate-master/src/include /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/src/*.cpp
cd /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound && /home/akhila/.RcppDeepState/honggfuzz/honggfuzz -t 2000 -i /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_output -o /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/hongg_rcpp_read_out_of_bound_output -x -- /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.hongg ___FILE___ --fuzz_save_passing 
Start time:'2020-10-28.00.12.14' bin:'/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.hongg', input:'/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_output', output:'/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/hongg_rcpp_read_out_of_bound_output', persistent:false, stdin:false, mutation_rate:5, timeout:2000, max_runs:0, threads:6, minimize:false, git_commit:af84d98b2e5ddf1dca6c81a23a8e579e83af7139

------------------------[  0 days 00 hrs 11 mins 04 secs ]----------------------
  Iterations : 519174 [519.17k]
        Mode : Static
      Target : /home/akhila/R/x86_64-pc-linux-g.....FILE___ --fuzz_save_passing
     Threads : 6, CPUs: 12, CPU%: 690% [57%/CPU]
       Speed : 805/sec [avg: 781]
     Crashes : 0 [unique: 0, blacklist: 0, verified: 0]
    Timeouts : 0 [2000 sec]
 Corpus Size : 0, max: 8192 bytes, init: 86885 files
  Cov Update : 0 days 00 hrs 11 mins 04 secs ago
    Coverage : [none]
---------------------------------- [ LOGS ] ------------------/ honggfuzz 2.3 /-

```



