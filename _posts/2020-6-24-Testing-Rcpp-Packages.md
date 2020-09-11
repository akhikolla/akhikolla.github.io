---
title: Testing Rcpp packages
author: Akhila Chowdary Kolla
categories: [Testing, Memory checks]
tags: [Testings,R]
math: true
---

**Testing Rcpp packages using DeepState and Valgrind**


Testing Rcpp packages for memory violations can be tricky. 

The C++ code embedded inside an Rcpp function can introduce subtle bugs. These bugs can be hard to detect because they don't throw any warnings or error messages when we compile and run the code but the behavior of the code changes every time we execute it.

R has its own memory manager whereas in C++ there is no built-in garbage collector to take care of its memory so we need to be extra careful in handling the memory and not to introduce any bugs.

The garbage collector needs to keep track of allocations and references. These create overhead in memory, performance, and the complexity of the language. This is the reason C++ is faster and has higher performance compared to other languages but this higher performance comes with a cost. Memory management in C++ is manual and this could result in undesirable behavior or bugs in most of the cases if not handled properly. 

**Subtle Problems in CPP code:**
1. use after free
2. use after delete
3. referencing to a null pointer
4. read from an undeclared/inaccessible memory
5. write to an index out of bounds/inaccesible memory

These errors may not always be detected. When we run any of the code with the above issues sometimes the code might not give any issues or it might give a normal output or it might crash the code, the output is not predictable and might keep changing depending on the working platforms as well.

The default build of R is designed in such a way to pass these memory issues by not throwing errors. Although there are known errors in the code R fails to detect them. 

Here's a simple example of Rcpp function that causes a segfault:

```c++
#include <Rcpp.h>
using namespace std;
// [[Rcpp::export]]
void rcpp_write_to_null(int val) {
  double *ptr = 0;
  *ptr = val;
}

```

Here the error is obvious that we're creating a pointer of type double and assigning it to zero which means we are creating a null pointer. A null pointer is never a valid address location. If we ever try to dereference the null pointer we will get a segmentation fault.

Running the above code gives something like this:

```c++
> rcpp_write_to_null(10)
> *** caught segfault *** 
> address (nil), cause 'memory not mapped'
> Traceback:
> 1:rcpp_write_to_null(10)
> An irrecoverable exception occurred. R is aborting now ...
> Segmentation fault (core dumped)
```

Although R detects that we are trying to access an invalid memory location by throwing a segmentation fault but it is unable to give us the exact useful information, it missed where the error occurred and why R is aborting.

Many other errors might occur when working with Rcpp code, R might not be able to detect them in the first run. 

**Detecting Memory Issues in Rcpp:**

Today we’re going to look at the various tools that can help us detect various memory issues in Rcpp packages: 

**Sanitizers :**

We can use sanitizers to detect various kinds of memory problems in our Rcpp functions. Compiling your Rcpp functions with Address sanitizer can detect memory problems like :

* Use of Deallocated/freed Memory
* Deallocation of Deallocated Memory
* Deallocation of Nonallocated Memory
* Use of Stack Memory after function return
* Use of Out-of-Scope Stack Memory
* Overflow and Underflow of Buffers

Address sanitizer combined with memory sanitizer can detect the use of uninitialized memory as well.

Similarly, Undefined behavior sanitizer detects some other forms of undefined behavior.UBSAN can detect error like 

* Invalid Float Cast
* Division where the divisor is zero
* When a null value is incorrectly passed as an argument
* When a function incorrectly returns a null value
* When a null value is incorrectly assigned to a variable
* The creation of null references and null pointer dereferences
* Invalid pointer casts due to differences in the sizes of types
* Invalid and overflowing shifts,array bounds that aren’t positive.

Using these sanitizers in your build is as easy as adding -fsanitize flag with respective type. For ASAN we use `-fsanitize=address` and for UBSAN it is `-fsanitize=undefined`.

```c++
R_HOME=/home/akhila/lib/R
COMMON_FLAGS= DeepState_TestHarness.o  -I/home/akhila/R/RcppDeepState/include/ -L/usr/lib/R/site-library/RInside/include/lib -Wl,-rpath=/usr/lib/R/site-library/RInside/include/lib -L${R_HOME}/lib -Wl,-rpath=${R_HOME}/lib -L/home/akhila/deepstate/src/lib -Wl,-rpath=/home/akhila/deepstate/src/lib -lR -lRInside -ldeepstate
DeepState_TestHarness : DeepState_TestHarness.o
	 clang++ -g -fsanitize=address -o deallocate_DeepState_TestHarness ${COMMON_FLAGS} ~/R/testpackage/src/*.o

DeepState_TestHarness.o : DeepState_TestHarness.cpp
	 clang++ -g -fsanitize=address -fno-omit-frame-pointer -I${R_HOME}/include -I/home/akhila/deepstate/src/include -I/usr/lib/R/site-library/Rcpp/include -I/usr/lib/R/site-library/RInside/include -I/home/akhila/R/RcppDeepState/inst/include/ DeepState_TestHarness.cpp -o DeepState_TestHarness.o -c

```
Although sanitizers help us detect most of the memory errors in the code it fails to detect a few of them in a default R build. As not all the errors are getting identified in the default R build we will have to run our code with R-devel build which will be discussed in the next segment.

**Significant Builds of R :**

There are certain builds in R which can help us detect these memory and undefined behavior errors. This is because the regular build of R is designed in such a way to neglect these kinds of issues as it is an optimized version.

The easiest way to get these builds is to install them from the docker container. You can refer to [sanitizers](http://dirk.eddelbuettel.com/code/sanitizers.html) it has excellent information on how to use R-devel build with address sanitizers and undefined sanitizers.

The rocker container provides the following builds to test code using ASAN and UBSAN:
ASAN enabled build of R-devel: r-devel-san.
UBSAN enabled build of R-devel : r-devel-ubsan.

**Limitations:**

1. Using docker could be overhead.
2. As we are trying to test code run in default R build changing build of R to detect those errors could lose its purpose and is not always a good idea.

Also, we have rhub() platform which could help us detect any issues in the Rcpp packages, it checks the package in the Rdevel version. 

**Valgrind :**

Valgrind is usually run during the run time unlike sanitizers during compile time. It helps us detect memory errors and leaks in the code. Valgrind works with default build of R, unlike sanitizers. But using Valgrind on the R build could slower its performance. As we run our code with Valgrind we get the stack trace of the error occurred and the reason for the error and its location.

There are certain tools that Valgrind uses to detect these memory errors, one such tool is memcheck used as `--tool=memcheck` and another is leak-check used as `--leak-check=full`.

```c++
valgrind --tool=memcheck --leak-check=yes ./LOPART_interface_DeepState_TestHarness --fuzz
```

As we are working with DeepState in Rcpp packages we need to take care of two things :

1. We have to make sure to create a TestHarness for each function in the package.
2. Also, Make a call to the respective Rcpp function and pass corresponding randomized parameters with the RcppDeepState functions in RcppDeepState.h in the TestHarness.

Once we have the TestHarness created for every function in the Rcpp package, all we have to do is run the testharness with Valgrind and use memcheck and leak check tools to detect any memory leaks in the code.

The process can be done in 3 simple steps:

```c++
# load RcppDeepState library
library(RcppDeepState) 
```

```c++ 
RcppDeepState::deepstate_pkg_create("pathtopackage")
```
Making a call to deepstate_pkg_create function creates TestHarness and their corresponding Makefiles for each function in the package

```c++
RcppDeepState::deepstate_compile_run("pathtopackage")
```

Making a call to deepstate_compile_run function compiles the code with all the required flags and libraries and executes the code with Valgrind. After executing the code we save the results of the Valgrind into a log file.

**Use after Deallocate :**

```c++

#include <Rcpp.h>
using namespace std;
// [[Rcpp::export]]
int rcpp_use_after_deallocate(int val){
  char *x = new char[val];
  delete[] x;
  return x[5];
} 
  
```
Considering input is always positive and here we are trying to use the memory after it is deleted which is a known failure.
When we run this function in R it gives something like this:

```c++
> rcpp_use_after_deallocate(10)
[1] 86
> rcpp_use_after_deallocate(50)
[1] 86
> rcpp_use_after_deallocate(100)
[1] 86

```
If we observe above we are getting the same output for any input although those memory locations are not allocated or initialized. We still get some garbage value instead of an error.

But creating a testharness for this function and passing deepstate values and memory check under Valgrind gives the following error:

```c++

INFO: Starting fuzzing
WARNING: No seed provided; using 1591573220
WARNING: No test specified, defaulting to first test defined (use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test)
size values: 2049359586
==7136== Invalid read of size 1
==7136==    at 0x418604: rcpp_use_after_deallocate(int) (use_after_deallocate.cpp:7)
==7136==    by 0x4082AD: DeepState_Test_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test() (use_after_deallocate_DeepState_TestHarness.cpp:16)
==7136==    by 0x408188: DeepState_Run_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test()- (use_after_deallocate_DeepState_TestHarness.cpp:7)
==7136==    by 0x405D43: DeepState_RunTest.isra.6 (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E27A: DeepState_FuzzOneTestCase (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E38F: DeepState_Fuzz (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40631D: main (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==  Address 0x59e43045 is 5 bytes inside a block of size 2,049,359,586 free'd
==7136==    at 0x4C3173B: operator delete[](void*) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==7136==    by 0x418603: rcpp_use_after_deallocate(int) (use_after_deallocate.cpp:6)
==7136==    by 0x4082AD: DeepState_Test_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test() (use_after_deallocate_DeepState_TestHarness.cpp:16)
==7136==    by 0x408188: DeepState_Run_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test() (use_after_deallocate_DeepState_TestHarness.cpp:7)
==7136==    by 0x405D43: DeepState_RunTest.isra.6 (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E27A: DeepState_FuzzOneTestCase (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E38F: DeepState_Fuzz (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40631D: main (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==  Block was alloc'd at
==7136==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==7136==    by 0x4185F8: rcpp_use_after_deallocate(int) (use_after_deallocate.cpp:5)
==7136==    by 0x4082AD: DeepState_Test_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test() (use_after_deallocate_DeepState_TestHarness.cpp:16)
==7136==    by 0x408188: DeepState_Run_use_after_deallocate_random_datatypes_rcpp_use_after_deallocate_test() (use_after_deallocate_DeepState_TestHarness.cpp:7)
==7136==    by 0x405D43: DeepState_RunTest.isra.6 (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E27A: DeepState_FuzzOneTestCase (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40E38F: DeepState_Fuzz (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136==    by 0x40631D: main (in /home/akhila/R/testUBSAN/inst/include/use_after_deallocate_DeepState_TestHarness)
==7136== 
==7136== 
==7136== HEAP SUMMARY:
==7136==     in use at exit: 50,292,267 bytes in 9,713 blocks
==7136==   total heap usage: 27,722 allocs, 18,009 frees, 2,135,357,440 bytes allocated
==7136== 
==7136== LEAK SUMMARY:
==7136==    definitely lost: 0 bytes in 0 blocks
==7136==    indirectly lost: 0 bytes in 0 blocks
==7136==      possibly lost: 0 bytes in 0 blocks
==7136==    still reachable: 50,292,267 bytes in 9,713 blocks
==7136==                       of which reachable via heuristic:
==7136==                         newarray           : 4,264 bytes in 1 blocks
==7136==         suppressed: 0 bytes in 0 blocks
==7136== Reachable blocks (those to which a pointer was found) are not shown.
==7136== To see them, rerun with: --leak-check=full --show-leak-kinds=all
==7136== 
==7136== For counts of detected and suppressed errors, rerun with: -v
==7136== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

```
If we break down this log file output into steps we get the stack trace: 

> Invalid read of size 1 

Valgrind detects where exactly there is an invalid read, It caught that we attempted to perform an invalid read of size one i.e of x[5] at line 7 in our rcpp_use_after_deallocate function. We can also the see traceback of the rcpp function call in the TestHarness at line 16 (use_after_deallocate_DeepState_TestHarness.cpp:16)

> Address 0x59e43045 is 5 bytes inside a block of size 2,049,359,586 free'd:

It also gives information about the number of bytes that are free'd with operator delete[] at line 6 in rcpp_use_after_deallocate(int)

> Block was alloc'd: 

It gives information about the array created using operator new[] at line 5 in rcpp_use_after_deallocate(int)

**Read out of bounds :**

```c++
#include <Rcpp.h>
using namespace std;
// [[Rcpp::export]]
int rcpp_read_out_of_bound(int val){
  int *stack_array = new int[val];
  return stack_array[val+100];
  	
} 
```
Here we are trying to read a value from an array index which is out of bound to stack_array. Assuming the input is greater than zero when we run the code in R we get something like :

```c++
> testcase::rcpp_read_out_of_bound(10)
[1] 296227088
> testcase::rcpp_read_out_of_bound(1000)
[1] 5
> testcase::rcpp_read_out_of_bound(52)
[1] 796026227
> testcase::rcpp_read_out_of_bound(1)
[1] 32723

```
We are trying to access the value at an index that doesn't exist but R fails to detect that. When we call the function we get different outputs mostly garbage values for every input instead of R detecting the error.

When we run the code using Valgrind it throws the error as below:

```c++
INFO: Starting fuzzing
WARNING: No seed provided; using 1591571071
WARNING: No test specified, defaulting to first test defined (read_out_of_bound_random_datatypes_rcpp_read_out_of_bound_test)
sizeofarray values: 1454661619
==6735== Invalid read of size 4
==6735==    at 0x4185D3: rcpp_read_out_of_bound(int) (read_out_of_bound.cpp:9)
==6735==    by 0x4082AD: DeepState_Test_read_out_of_bound_random_datatypes_rcpp_read_out_of_bound_test() (read_out_of_bound_DeepState_TestHarness.cpp:16)
==6735==    by 0x408188: DeepState_Run_read_out_of_bound_random_datatypes_rcpp_read_out_of_bound_test() (read_out_of_bound_DeepState_TestHarness.cpp:7)
==6735==    by 0x405D43: DeepState_RunTest.isra.6 (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40E27A: DeepState_FuzzOneTestCase (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40E38F: DeepState_Fuzz (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40631D: main (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==  Address 0x1b4b5b19c is 5,818,646,876 bytes inside a block of size 5,818,650,528 in arena "client"
==6735== 
==6735== 
==6735== HEAP SUMMARY:
==6735==     in use at exit: 5,868,938,743 bytes in 9,714 blocks
==6735==   total heap usage: 27,722 allocs, 18,008 frees, 5,904,644,330 bytes allocated
==6735== 
==6735== 5,818,646,476 bytes in 1 blocks are possibly lost in loss record 1,306 of 1,306
==6735==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==6735==    by 0x4185D2: rcpp_read_out_of_bound(int) (read_out_of_bound.cpp:8)
==6735==    by 0x4082AD: DeepState_Test_read_out_of_bound_random_datatypes_rcpp_read_out_of_bound_test() (read_out_of_bound_DeepState_TestHarness.cpp:16)
==6735==    by 0x408188: DeepState_Run_read_out_of_bound_random_datatypes_rcpp_read_out_of_bound_test() (read_out_of_bound_DeepState_TestHarness.cpp:7)
==6735==    by 0x405D43: DeepState_RunTest.isra.6 (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40E27A: DeepState_FuzzOneTestCase (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40E38F: DeepState_Fuzz (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735==    by 0x40631D: main (in /home/akhila/R/testUBSAN/inst/include/read_out_of_bound_DeepState_TestHarness)
==6735== 
==6735== LEAK SUMMARY:
==6735==    definitely lost: 0 bytes in 0 blocks
==6735==    indirectly lost: 0 bytes in 0 blocks
==6735==      possibly lost: 5,818,646,476 bytes in 1 blocks
==6735==    still reachable: 50,292,267 bytes in 9,713 blocks
==6735==                       of which reachable via heuristic:
==6735==                         newarray           : 4,264 bytes in 1 blocks
==6735==         suppressed: 0 bytes in 0 blocks
==6735== Reachable blocks (those to which a pointer was found) are not shown.
==6735== To see them, rerun with: --leak-check=full --show-leak-kinds=all
==6735== 
==6735== For counts of detected and suppressed errors, rerun with: -v
==6735== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```
If we analyze the stack trace :

> Invalid read of size 4

It say there is an invalid read of size 4 as we are trying to access an integer index its size is 4, at line 9 in rcpp_read_out_of_bound(int) (read_out_of_bound.cpp:9). It also gives us the line number from where this rcpp_read_out_of_bound is called from in DeepState_TestHarness.
(read_out_of_bound_DeepState_TestHarness.cpp:16).

Valgrind can also detect errors like write out of bound, use after free, use of a zero-sized array, use of uninitialized values, use of null pointers. In the same way, Valgrind gives the stack trace for every error that it caught. This kind of information(a type of error, location of error) is really helpful in debugging the code. Once we have all the information it could be very easy for the user to rectify the error. 

Now RcppDeepState supports testing your Rcpp packages in Travis. It is as easy as making a call to 
 
```c++
library(RcppDeepState)
RcppDeepState::deepstate_ci_setup(pathtopackage)
```

This function creates a .travis.yml file for your Rcpp package if it doesn't exist with the necessary r_packages and environment variables which makes it easy for any package developers to use RcppDeepState on Travis-CI.
The Most recent build for [RcppDeepState](https://travis-ci.org/github/akhikolla/RcppDeepState/builds/704036361). 

It also generates a test-rcppdeepstate.R in the test directory and inside that test file we make a call to deepstate_pkg_create(pkg path) to create TestHarness for the rcpp functions in the package and deepstate_compile_run(pkg path) will compile and run the code and returns a list of error messages along with line numbers. 

Also, if the list returned is empty it means your Rcpp package doesn't have any bugs. Yayyy!!!

I have used deepstate_ci_setup() to set up package LOPART for testing with RcppDeepState on Travis. This function creates the .travis.yml and test-rcppdeepstate.R with necessary functionalities. 

Once the changes are pushed to the user's GitHub Travis build runs and executes the functions. If all the tests pass in the Travis build your package doesn't have any crashes/bugs. The recent Travis build of [LOPART](https://travis-ci.org/github/akhikolla/LOPART/builds/704049717)

Thanks to [Dr.Toby Dylan Hocking](https://tdhock.github.io/blog/) for his support on the project.
