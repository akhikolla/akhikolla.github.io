---
layout: post
title: RcppDeepStateTools 
author: Akhila Chowdary Kolla
categories: [Tools]
tags: [RcppDeepStateTools,Tools,AFL,libfuzzer,honggfuzz,Eclipser,Angora,R]
math: true
---

As a part of the RcppDeepState project, we created another Linux specific R package, RcppDeepStateTools fuzz tests Rcpp packages under different external fuzzers. RcppDeepStateTools provides an interface to four external fuzzers :

1. AFL 
2. LibFuzzer 
3. Eclipser
4. HonggFuzz

**Fuzzers:** 
Fuzzers are software programs that provide invalid, unexpected, or random data as inputs to a computer program. These inputs are passed on the code expecting crashes, failures, and memory leaks.

RcppDeepState makes use of the built-in fuzzer in deepstate to generate inputs to test Rcpp packages. It is likely to uncover more bugs than standard sanitizer and debugging tools.

In this blog post, I'll discuss how to use these external fuzzers for running the function-specific test harnesses. To test any Rcpp package using any of these fuzzers is as easy as making a call to deepstate_compile_tools() function.

```R
RcppDeepState::deepstate_compile_tools(path)
```

This function takes the path to the package that you want to test. I'll use the testSAN package in the RcppDeepState testpkgs folder as my default test package.

```R
> path <- "~/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN"
> RcppDeepStateTools::deepstate_pkg_create_<Fuzzer>(path)
```
You choose from the following Fuzzers:

- AFL
- HonggFuzz
- Eclipser
- LibFuzzer

Here choosing any of the options - will do the following:
1. Install the fuzzer.
2. Compile and link the specific fuzzer with the deepstate base library.
3. Compile and Run the Testharnesses.

**NOTE:**The system might ask for superuser permissions to install the dependencies that are needed by the fuzzers.

Choosing a fuzzer would also generate a fuzzer specific makefile for all the test harnesses that were previously created by RcppDeepState for each test package function. 

**AFL:**

American Fuzzy Lop is a brute-force fuzzer that automatically discovers interesting test cases that could explore new paths of the target binary. Using AFL could help us uncover the unexplored paths that are usually not tested by normal brute force fuzzers.

AFL has compile-time instrumentation which means the program inserts the necessary instructions into the program during compilation resulting in better run-time performance. 

If we want to test the package using AFL we have to choose option 1 :

```shell
> RcppDeepStateTools::deepstate_pkg_create_AFL(path)
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
[+] All test cases are processed.

[!] WARNING: Some test cases are huge (412 kB) - see docs/perf_tips.txt!
[!] WARNING: Some test cases look useless. Consider using a smaller set.
[+] Here are some useful stats:

    Test case count: 1 favored, 0 variable, 7 total
       Bitmap range: 248 to 248 bits (average: 248.00 bits)
        Exec timing: 1543 to 1630 us (average: 1586 us)

[+] All set and ready to roll!

```

The afl output looks something like this:

```shell
     American fuzzy lop 2.52b (rcpp_read_out_of_bound_DeepState_TestHar...)

┌─ process timing ─────────────────────────────────────┬─ overall results ─────┐
│        run time : 0 days, 1 hrs, 11 min, 17 sec      │  cycles done : 498    │
│   last new path : none yet (odd, check syntax!)      │  total paths : 7      │
│ last uniq crash : none seen yet                      │ uniq crashes : 0      │
│  last uniq hang : none seen yet                      │   uniq hangs : 0      │
├─ cycle progress ────────────────────┬─ map coverage ─┴───────────────────────┤
│  now processing : 4* (57.14%)       │    map density : 0.38% / 0.38%         │
│ paths timed out : 0 (0.00%)         │ count coverage : 1.00 bits/tuple       │
├─ stage progress ────────────────────┼─ findings in depth ────────────────────┤
│  now trying : splice 13             │ favored paths : 1 (14.29%)             │
│ stage execs : 16/32 (50.00%)        │  new edges on : 1 (14.29%)             │
│ total execs : 2.33M                 │ total crashes : 0 (0 unique)           │
│  exec speed : 430.4/sec             │  total tmouts : 0 (0 unique)           │
├─ fuzzing strategy yields ───────────┴───────────────┬─ path geometry ────────┤
│   bit flips : 0/224, 0/217, 0/203                   │    levels : 1          │
│  byte flips : 0/28, 0/21, 0/7                       │   pending : 0          │
│ arithmetics : 0/1554, 0/28, 0/0                     │  pend fav : 0          │
│  known ints : 0/134, 0/582, 0/308                   │ own finds : 0          │
│  dictionary : 0/0, 0/0, 0/0                         │  imported : n/a        │
│       havoc : 0/897k, 0/1.43M                       │ stability : 100.00%    │
│        trim : 100.00%/147, 0.00%                    ├────────────────────────┘
├─────────────────────────────────────────────────────┘          [cpu000: 71%]


[+] We're done here. Have a nice day!

```

AFL output has the following categories :

**Process timing:**

This section is pretty straight forward:
 
- run time: It gives the time for how long the fuzzer has been running
- last new path: The tools look for new paths and If it finds any it gives the time passed since the last path.
- last uniq crash/last uniq hang: It gives the time since the last crash/hang occured.

These fuzzing jobs are expected to run for days and weeks sometimes to find interesting paths.

**Cycle progress:** 

- Now processing : It shows the id of the test case that the process is currently working on.
- Path timed out: It gives the number of inputs that cause a timeout when used in the program.

**Stage progress:** 

This part gives is important to understand what the fuzzer is doing:

- now trying : This include various fuzzing stages that include calibration,trim,bitflip,interest,extras,havoc,splice,sync
- stage execs: This gives the number of current program executions.
- total execs: The total number of executions so far.
- exec speed: execution speed at which the program executes.
 
**map coverage:** 
- map density:  This gives the proportion of how many inputs we have already covered with the number of inputs that the input corpus can hold. 
- count coverage: This gives the hit count for every branch.


**findings in-depth:** 
- favored paths: This includes the count of paths that take considerably more time to run.
- new edges on The number of test cases that resulted in better edge coverage.
- total crashes/total tmouts : Counter for crashes and timeouts

**Overall results:**

- cycles done: It gives the number of times the fuzzer ran over the interesting test cases generated so far.
- total paths: The number of test paths discovered so far.
- uniq crashes/uniq hangs: Number of test cases that caused crashes/hangs.

When we run a target binary using AFL we get an afl output folder which contains the following directories:
1. crashes - This folder contains all the file inputs that caused crashes on the executable
2. queue - This folder contains all the inputs that are generated by the fuzzer.
3. fuzzer_stats - This file has all statistic info (unique_crashes:0, fu
zzer_pid:6030)
4. plot_data - This file has details like unix_time, cycles_done, cur_path, paths_total, pending_total, pending_favs, map_size, unique_crashes, unique_hangs, max_depth for every input passed.


**HonggFuzz**

HonggFuzz is a security-oriented, multi-process, and multi-threaded fuzzer. This takes care of running multiple copies of the fuzzer and uses all the available CPU cores with a single instance.

When we test the package using hongg fuzz the output looks something like this:

First runs the make file and then executes the testharness:

```shell
> RcppDeepStateTools::deepstate_pkg_create_HonggFuzz(path)
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
The output of the honggfuzz is self-explanatory. It provides the following information:

- Iterations: The number of fuzzing iterations.
- Mode: The mode of the run and can be chosen between static and persistent.
- Target: the executable binary that we are fuzzing.
- Threads: Number of threads and number of CPU's usage.
- Speed: The speed at which each test case is run.
- Crashes: Number of crashes found
- Timeouts: Number of test cases that are timed out
- Corpus Size: The size of the valid and interesting inputs that are discovered so far.
- Cov Update: Time the fuzzer ran for.
- Coverage: Percentage of code coverage for interesting inputs.

When we run the honggfuzz on the binary it creates the following directories in its output folder:

1. crashes - This folder contains all the file inputs that caused crashes on the executable
2. queue - This folder contains all the inputs that are generated by the fuzzer.
3. fuzzer_stats - This file has all statistic info (unique_crashes:0, fu
zzer_pid:6030)
4. plot_data - This file has details like unix_time, cycles_done, cur_path, paths_total, pending_total, pending_favs, map_size, unique_crashes, unique_hangs, max_depth for every input passed.


**Eclipser**

Eclipser is a binary-based fuzz testing tool that uses lightweight instrumentation. This is also known as coverage based fuzzer that allows deepstate to quickly detect harder to reach bugs by covering interesting paths.

```shell
> RcppDeepStateTools::deepstate_pkg_create_Eclipser(path)
[1] "rm -f *.o && make -f /home/akhila/R/RcppDeepState/inst/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/Eclipser.Makefile"
cd /home/akhila/R/RcppDeepState/inst/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound && deepstate-eclipser ./rcpp_read_out_of_bound_DeepState_TestHarness -o eclipser_out --timeout 30 
INFO:deepstate:Setting log level from DEEPSTATE_LOG: 2
INFO:deepstate.core.base:Setting log level from --min_log_level: 2
INFO:deepstate.core.fuzz:Calling pre_exec before fuzzing
WARNING:deepstate.executors.fuzz.eclipser:Eclipser doesn't limit child processes memory.
INFO:deepstate.core.fuzz:Executing command `['dotnet', '/home/akhila/.RcppDeepState/Eclipser/build/Eclipser.dll', 'fuzz', '--program', '/home/akhila/R/RcppDeepState/inst/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness', '--src', 'file', '--fixfilepath', 'eclipser.input', '--initarg', '--input_test_file eclipser.input --abort_on_fail --no_fork --min_log_level 2', '--outputdir', 'eclipser_out/the_fuzzer', '--maxfilelen', '8192', '--timelimit', '30']`
INFO:deepstate.core.fuzz: Using DeepState output.
INFO:deepstate.core.fuzz: Started fuzzer process with PID 11932.
INFO:deepstate.executors.fuzz.eclipser:Performing decoding on testcases and crashes
FUZZ_STATS:deepstate.core.fuzz:unique_crashes:0
FUZZ_STATS:deepstate.core.fuzz:fuzzer_pid:11932
FUZZ_STATS:deepstate.core.fuzz:start_time:1603856530
FUZZ_STATS:deepstate.core.fuzz:------------------------------
INFO:deepstate.executors.fuzz.eclipser:Performing decoding on testcases and crashes
FUZZ_STATS:deepstate.core.fuzz:unique_crashes:0
FUZZ_STATS:deepstate.core.fuzz:fuzzer_pid:11932
FUZZ_STATS:deepstate.core.fuzz:start_time:1603856530
FUZZ_STATS:deepstate.core.fuzz:------------------------------
INFO:deepstate.core.fuzz:Timeout
INFO:deepstate.executors.fuzz.eclipser:Performing decoding on testcases and crashes
FUZZ_STATS:deepstate.core.fuzz:unique_crashes:0
FUZZ_STATS:deepstate.core.fuzz:fuzzer_pid:11932
FUZZ_STATS:deepstate.core.fuzz:start_time:1603856530
FUZZ_STATS:deepstate.core.fuzz:------------------------------
INFO:deepstate.core.fuzz: Killing process 11932 and childs.
INFO:deepstate.core.fuzz:Fuzzer subprocess (PID 11932) exited with `0`
INFO:deepstate.core.fuzz:Fuzzer exec time: 51.86s
INFO:deepstate.core.fuzz:Calling post-exec for fuzzer post-processing
INFO:deepstate.executors.fuzz.eclipser: Performing decoding on test cases and crashes

```

The Eclipser gives some information about crashes, fuzzer id, and Unix time at which fuzzer started. The output folder of Eclipser looks similar to the output folder of Honggfuzz and AFL.


**LibFuzzer:**


LibFuzzer is a coverage driven fuzzing engine.The code coverage information for libFuzzer is provided by LLVM's Sanitizer instrumentation.

```shell
> RcppDeepStateTools::deepstate_pkg_create_LibFuzzer(path)
[1] "rm -f *.o && make -f /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/libfuzz.Makefile"
cd /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound && ./rcpp_read_out_of_bound_DeepState_TestHarness_LF --fuzz --fuzz_save_passing 
DEBUG: INFO: libFuzzer ignores flags that start with '--'

DEBUG: INFO: Seed: 3628062210

DEBUG: INFO: Loaded 1 modules   (1690 inline 8-bit counters): 
DEBUG: 1690 [604720, 604dba), 
DEBUG: 

DEBUG: INFO: Loaded 1 PC tables (1690 PCs): 
DEBUG: 1690 [5b41b8,5bab58), 
DEBUG: 

DEBUG: INFO: -max_len is not provided; libFuzzer will not generate inputs larger than 4096 bytes

input starts
rbound values: 0
input ends
input starts
rbound values: 167772160
input ends
AddressSanitizer:DEADLYSIGNAL
=================================================================
==1642018==ERROR: AddressSanitizer: SEGV on unknown address 0x614028000240 (pc 0x00000059213b bp 0x7ffe444b33d0 sp 0x7ffe444b33a0 T0)
==1642018==The signal is caused by a READ memory access.
    #0 0x59213b in rcpp_read_out_of_bound(int) /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/src/read_out_of_bound.cpp:7:10
    #1 0x55e412 in DeepState_Test_testSAN_deepstate_test_rcpp_read_out_of_bound_test() /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.cpp:21:5
    #2 0x556cd8 in DeepState_Run_testSAN_deepstate_test_rcpp_read_out_of_bound_test() /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness.cpp:10:1
    #3 0x56f6e7 in DeepState_RunTestNoFork (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x56f6e7)
    #4 0x56f4fa in LLVMFuzzerTestOneInput (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x56f4fa)
    #5 0x45f131 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x45f131)
    #6 0x45e875 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x45e875)
    #7 0x461051 in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x461051)
    #8 0x4614f9 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, fuzzer::fuzzer_allocator<fuzzer::SizedFile> >&) (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x4614f9)
    #9 0x4501ce in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x4501ce)
    #10 0x479012 in main (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x479012)
    #11 0x7f7b02b210b2 in __libc_start_main (/usr/lib/x86_64-linux-gnu/libc.so.6+0x270b2)
    #12 0x424f6d in _start (/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/rcpp_read_out_of_bound_DeepState_TestHarness_LF+0x424f6d)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/src/read_out_of_bound.cpp:7:10 in rcpp_read_out_of_bound(int)
==1642018==ABORTING
make: *** [/home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/libfuzz.Makefile:5: /home/akhila/R/x86_64-pc-linux-gnu-library/3.6/RcppDeepState/testpkgs/testSAN/inst/testfiles/rcpp_read_out_of_bound/libfuzzer_rcpp_read_out_of_bound_log] Error 1
```

The Libfuzzer can detect an invalid read memory access with the address sanitizer instrumentation in the above TestHarness. Once there is an error/bug in the memory the fuzzer aborts with a trace of the issues.

Thanks to `Dr.Toby Dylan Hocking` for his support on this project. This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 
