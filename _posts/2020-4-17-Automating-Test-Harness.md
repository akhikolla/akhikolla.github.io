---
layout: post
title: Automating the TestHarness
author: Akhila Chowdary Kolla
categories: [RcppDeepState,Automate Test Harness]
tags: [TestHarness,R]
math: true
---

**RcppDeepState testharness for binsegRcpp package using RInisde**

In this blog, I’ll be explaining the process of automating the testharness and integrating Rcpp with deepstate. This blog is a continuation of my previous blog explaining the idea of working with deepstate in R.

If you want to know more about embedding a C program with R you can refer to [Embedding_R]( https://tdhock.github.io/blog/2020/embedded-R/). Also, if you want to dig deeper and link functions of the Rcpp package with a main function in an external C++ file using RInside you can check [binsegRcpp_inside_C++_code](https://tdhock.github.io/blog/2020/binsegRcppInside/).

**Integrating deepstate with Rcpp using RInside**

To use RInside in any program first we need to create an Instance of RInside class, which represents an embedded R interpreter in that particular C/C++ program. RInside package is designed in such a way to make it easier to embed R in a C++ class.
Using RInside in a C++ code follow 2 easy steps:

***Including the RInside header*** 

```c++
#include <RInside.h>
```
***Creating an Instance of R*** 
```c++
RInside R(argc,argv);
```
Here argc, argv are arguments of the main function. As we don’t have a main function in a testharness, we can just declare argc,argv, and make a call to R. Deepstate runs the test harness with a main function automatically at runtime.

***Generation of TestHarness***

I created a function that generates testharness based on the information taken from the RcppExports file in the package.

`RcppDeepState::deepstate_harness_create("package_path")` is a function that takes path of the package and generates a testharness for each function declared in the package.  

`deepstate_harness_create()` has the script to filter the RcppExports for function prototypes, Number of functions, Number of arguments for each function, and their names in the package.It also generates corresponding makefiles for the testharnesses.

For example, if you want to run this function on the `binsegRcpp` package, use below line of code.

```c++
//Here ~/R/binsegRcpp path of binsegRcpp 
deepstate_harness_create("~/R/binsegRcpp") 
```
This function makes a call to following functions

```c++
deepstate_harness_create(package_path){

//gives list of functions and their arguments in the package
binseg.list <- get_fun_body(package_path) 

//gives the prototypes of the functions in the package
  prototypes <-get_prototype_of_functions(package_path)
 
 //script to generate Testharness
 
 //creates make files for the generated testharness
 create_makefile(generatedTestharnessname)
}
```
The obtained lists from the functions are filtered and placed accordingly in the Testharness.You can refer to the function [deepstate_harness_create](https://github.com/akhikolla/RcppDeepState/blob/master/R/pkg_harness_create.R) here to understand in detail.

In RcppDeepState, we have created the RcppDeepState.h header file in RcppDeepState/inst/include where all of the RcppDeepState_* function definitions are present, usually, header files placed in inst/include are considered to be in the root directory and can be accessed easily.
Link to [RcppDeepState/inst/include](https://github.com/akhikolla/RcppDeepState/tree/master/inst/include) 

In the header file, we have the member functions to generate randomized vectors for that particular datatype. There is a detailed description of generating these random functions in my [previous blog](https://akhikolla.github.io/posts/RcppDeepState-Introduction/)

**What happens inside the testharness?**

First, It will include two header files one is `RInside.h` which provides all the necessary functions to embed R with C++ and the second is `RcppDeepState.h` as it has the randomized vector definitions which will be used in the testharness.

```c++
#include <RInside.h>
#include <RcppDeepState.h> 
```
binsegRcpp package has two functions with the following prototypes and both the functions accept two input arguments: an IntegerVector and a NumericVector.  

```R
Rcpp::List rcpp_binseg_normal(Rcpp::NumericVector, Rcpp::IntegerVector);
Rcpp::List rcpp_binseg_normal_cost(Rcpp::NumericVector, Rcpp::IntegerVector);
```
As we are generating the testharness for rcpp_binseg_normal. It will declare the prototype of this function in the testharness for the compiler to recognize it. 

TEST macro is always written inside a Testharness. This TEST macro will take the unit test name and test name as arguments. For each function in the package binsegRcpp, a testharness is created.

Let's see an example of testharness for rcpp_binseg_normal() (one of the functions in binsegRcpp).

For rcpp_binseg_normal(), binseg_normal_DeepState_TestHarness.cpp is the name of the testharness.
The unit test name is binseg_normal_name_randomdatatypes which is the first argument for the TEST macro, The test name is binseg_normal_name_test which is the second argument for the TEST macro

```c++
TEST (binseg_normal_randomdatatypes,rcpp_binseg_normal_test){
//test harness code
}
```
Inside the test macro, it will declare two variables `argc,argv`.
```c++
int argc;
char **argv; 
```
Now it will create an instance of RInside by passing the above-declared variables as arguments which will help to embed R with C++.
```c++
RInside R(argv,argc);
```
Now as we need to test rcpp_binseg_normal we need two data vectors. These data vectors are randomly generated by making a call to the randomized vector functions of `RcppDeepState.h` header file.

The below code explains how those function calls are being made.
A call to RcppDeepState_NumericVector is made and it returns a random numericvector whose values are stored in data_vec, and a call to RcppDeepState_IntegerVector is made and it returns a random IntegerVector whose values are stored in max_segments.

```R
Rcpp::NumericVector data_vec = RcppDeepState_NumericVector();
Rcpp::IntegerVector max_segments = RcppDeepState_IntegerVector();
```
Now, these data_vec, max_segments with randomized values are passed to rcpp_binseg_normal.The below code makes a call to the rcpp_binseg_normal function in binsegRcpp package.
 
```R
rcpp_binseg_normal(data_vec, max_segments);
```

**binseg_normal_DeepState_TestHarness.cpp**

```c++
//including the necessary headers
#include <RInside.h>
#include <RcppDeepState.h> 

//declaring the prototype of the function for the compiler to recognize it. 
Rcpp::List rcpp_binseg_normal(const Rcpp::NumericVector data_vec, const Rcpp::IntegerVector max_segments);

//defining the test macro
TEST (binseg_normal_randomdatatypes,rcpp_binseg_normal_test)
{
//declare commandline arguments as there is no main function at this time in the code
int argc;
char **argv; 

//calling instance of R
RInside R(argv,argc);

//making a call to randomized vector methods of class RcppDeepState and storing the return values
Rcpp::NumericVector data_vec =RcppDeepState_NumericVector();
Rcpp::IntegerVector max_segments =RcppDeepState_IntegerVector();

//making a call to rcpp_binseg_normal passing those random values
rcpp_binseg_normal(data_vec, max_segments);
}
```
For the code to compile first we need to install the binsegRcpp package in the system. After the installation, binsegRcpp/src has the compiled object files which are used during the compilation of testharness.You can find [binsegRcpp_package](https://github.com/tdhock/binsegRcpp) here.
 
To compile the test harness of rcpp_binseg_normal(), we require a Makefile. The same deepstate_harness_create() will make a call to create_makefile() function which creates the respective makefile for the testharness. In our case, it created `rcpp_binseg_normal.Makefile` 

The below snippet shows what happens when we run the code without any fuzzer.
if we run the code without any fuzzer it will execute the test case with default values for the respective datatypes.
Example: The default value of Integer is 0  and Numeric is 0.0

```c++
akhila@akhila-VirtualBox:~/R$ rm -f *.o && make -f binsegnormal.Makefile binseg_normal_DeepState_TestHarness
clang++ -I/home/akhila/lib/R/include -I/home/akhila/deepstate/src/include -I/usr/lib/R/site-library/Rcpp/include -I/usr/lib/R/site-library/RInside/include -I/home/akhila/R/RcppDeepState/inst/include/ binseg_normal_DeepState_TestHarness.cpp -o binseg_normal_DeepState_TestHarness.o -c
clang++ -o binseg_normal_DeepState_TestHarness binseg_normal_DeepState_TestHarness.o -I/home/akhila/R/RcppDeepState/include/ -L/usr/lib/R/site-library/RInside/include/lib -Wl,-rpath=/usr/lib/R/site-library/RInside/include/lib -L/home/akhila/lib/R/lib -Wl,-rpath=/home/akhila/lib/R/lib -L/home/akhila/deepstate/src/lib -Wl,-rpath=/home/akhila/deepstate/src/lib -lR -lRInside -ldeepstate /home/akhila/R/binsegRcpp/src/*.o
./binseg_normal_DeepState_TestHarness
TRACE: Running: binseg_normal_randomdatatypes_rcpp_binseg_normal_test from binseg_normal_DeepState_TestHarness.cpp(8)
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(15): before declaration
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(21): after assignment
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 0 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 1 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 2 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 3 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 4 rand_numeric vector: 0.000000
TRACE: Passed: binseg_normal_randomdatatypes_rcpp_binseg_normal_test
```
Breaking down the lines in the Makefile, the compilation has 3 steps

**Compiling step:** 

```c++
clang++ -I/home/akhila/lib/R/include -I/home/akhila/deepstate/src/include -I/usr/lib/R/site-library/Rcpp/include -I/usr/lib/R/site-library/RInside/include -I/home/akhila/R/RcppDeepState/inst/include/ binseg_normal_DeepState_TestHarness.cpp -o binseg_normal_DeepState_TestHarness.o -c
```
Here we include all the libraries that are necessary to compile the code, COMMON_FLAGS have list of libraries that are to be included at the time of compiling the code

```c++
COMMON_FLAGS=binseg_normal_DeepState_TestHarness.o  -I/home/akhila/R/RcppDeepState/inst/include/ -L/usr/lib/R/site-library/RInside/include/lib -Wl,-rpath=/usr/lib/R/site-library/RInside/include/lib -L${R_HOME}/lib -Wl,-rpath=${R_HOME}/lib -L/home/akhila/deepstate/src/lib -Wl,-rpath=/home/akhila/deepstate/src/lib -lR -lRInside -ldeepstate
```

**Linking step:**

We link the compiled testharness in the above step with the object file of the functions as 
follows:

```c++
clang++ -o binseg_normal_DeepState_TestHarness binseg_normal_DeepState_TestHarness.o -I/home/akhila/R/RcppDeepState/inst/include/ -L/usr/lib/R/site-library/RInside/include/lib -Wl,-rpath=/usr/lib/R/site-library/RInside/include/lib -L/home/akhila/lib/R/lib -Wl,-rpath=/home/akhila/lib/R/lib -L/home/akhila/deepstate/src/lib -Wl,-rpath=/home/akhila/deepstate/src/lib -lR -lRInside -ldeepstate /home/akhila/R/binsegRcpp/src/*.o
```
**Running the Code:**

In the rcpp_binsegnormal.Makefile we didn’t specify the executable to run with fuzzers, resulting in default values being passed on the function. If we want to run the executable with fuzzer all we have to do is add the `--fuzz` argument to the executable. 

```c++
./binseg_normal_DeepState_TestHarness –fuzz
```
Using `—fuzz` argument during the runtime helps us get the randomized fuzzing inputs to be passed on to the executable.
If you want to see those fuzzing inputs on the console you can just add `–min_level_log 0` argument.
The `min_log_level` argument controls the way inputs are displayed on the console while fuzzing.

Running the executable with `fuzz` and `min_level_log` arguments :
```c++
akhila@akhila-VirtualBox:~/R$ ./binseg_normal_DeepState_TestHarness --fuzz --min_log_level 0
INFO: Starting fuzzing
WARNING: No seed provided; using 1588038306
WARNING: No test specified, defaulting to first test defined (binseg_normal_randomdatatypes_rcpp_binseg_normal_test)
TRACE: Running: binseg_normal_randomdatatypes_rcpp_binseg_normal_test from binseg_normal_DeepState_TestHarness.cpp(8)
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(15): before declaration
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(21): after assignment
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 0 rand_numeric vector: -12563812741723733078733884073387044629005491060075677258726713056417780995798020336653792386553526424619637830649583857521090658230996187146232755069113565923454666543942553438502052028584808990707067614737351070834911295195278791062329372854128548541690368660149302893900452908812991244328340473551832219648.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 1 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 2 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23):
 index: 3 rand_numeric vector: -59112322608599703552.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(23): 
index: 4 rand_numeric vector: 0.000000
TRACE: Passed: binseg_normal_randomdatatypes_rcpp_binseg_normal_test 
```
TestHarness is created, compiled, and executed the same for rcpp_binseg_normal_cost() in the package.Similarly the same is carried for every function present in that package.

After running the executable with fuzz we get the count of passed and failed testcases:
```c++
INFO: Done fuzzing! Ran 166 tests (1 tests/second) with 87 failed/79 passed/0 abandoned tests
```
Here 166 testcases are ran but only 79 of them passed and 87 failed.

Failing of binsegRcpp::rcpp_binseg_normal() include following conditions:
rcpp_binseg_normal throws errors in 3 cases. All of these errors are particularized previously in the function.

**Case 1:**
if size of data_vec is less than 1 - it throws NO_DATA error.

```c++
TRACE: Running: binseg_normal_randomdatatypes_rcpp_binseg_normal_test from binseg_normal_DeepState_TestHarness.cpp(7)
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(13): size of numeric vector 0
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(26): size of integer vector 7
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 0 rand_Integer vector: 0
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 1 rand_Integer vector: 0
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 2 rand_Integer vector: 1
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 3 rand_Integer vector: 5
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 4 rand_Integer vector: 1
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 5 rand_Integer vector: 5
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 6 rand_Integer vector: 1
terminate called after throwing an instance of 'Rcpp::exception'
  what():  no data
ERROR: Failed: binseg_normal_randomdatatypes_rcpp_binseg_normal_test
```
Here the first line represents the trace of running the unit test binseg_normal_randomdatatypes for the test name binseg_normal_test from the binseg_normal_DeepState_TestHarness.cpp file.
The next line gives the size of data_vec. The next 8 lines give the size of the max_segments and the values assigned to it.
As the size of data_vec is zero system throws `Rcpp:: exception - no data`

**Case 2:**
When deepstate passes max_segments value less than 1 - it throws NO_SEGMENTS error.
```c++
TRACE: Running: binseg_normal_randomdatatypes_rcpp_binseg_normal_test from binseg_normal_DeepState_TestHarness.cpp(7)
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(13): size of numeric vector 3
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(19): index: 0 rand_numeric vector: 0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(19): index: 1 rand_numeric vector: -0.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(19): index: 2 rand_numeric vector: -549597172537956305361476352992032084967034120685391253548321674087287542206259282924505133616777038420141334167021856546111699614438625352540376419326680499753137951174060529473856821415788016811851287442002018219512838717832056589714313327622732747428243813670070517760.000000
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(26): size of integer vector 1
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 0 rand_Integer vector: 0
terminate called after throwing an instance of 'Rcpp::exception'
  what():  no segments
ERROR: Failed: binseg_normal_randomdatatypes_rcpp_binseg_normal_test
```
Here the first line represents the trace of running the unit test binseg_normal_randomdatatypes for the test name binseg_normal_test from the binseg_normal_DeepState_TestHarness.cpp file.
The next 6 lines give the size of data_vec and values present in it.
The next line gives the size of the max_segments and the values assigned to it.
Here we encountered `Rcpp::Exception no segments` error because max_segments values are less than one

**Case 3:**
when deepstate passes size of data_vec is less than the value in max_segments - it throws TOO_MANY_SEGMENTS error.
```c++
TRACE: Running: binseg_normal_randomdatatypes_rcpp_binseg_normal_test from binseg_normal_DeepState_TestHarness.cpp(7)
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(13): size of numeric vector 2
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(19): index: 0 rand_numeric vector: 6485702193556522171471109734302530991973990400.000000
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(19): index: 1 rand_numeric vector: 0.000000
INFO: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(26): size of integer vector 2
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 0 rand_Integer vector: 3
TRACE: /home/akhila/R/RcppDeepState/inst/include/RcppDeepState.h(32): index: 1 rand_Integer vector: 4
terminate called after throwing an instance of 'Rcpp::exception'
  what():  too many segments
ERROR: Failed: binseg_normal_randomdatatypes_rcpp_binseg_normal_test
```
Here the first line represents the trace of running the unit test binseg_normal_randomdatatypes for the test name binseg_normal_test from the binseg_normal_DeepState_TestHarness.cpp file.
The next 3 lines give the size of data_vec and values present in it.
The next 3 lines gives the size of the max_segments and the values assigned to it.
Here we encountered `Rcpp::Exception too many segments` error because max_segments value is greater than the size of data_vec.

Considering the above conditions, for rcpp_binseg_normal to run without throwing any expected errors we can handle those errors with the help of try and catch block.

These expected errors can be handled by placing a try-catch block around the function call rcpp_binseg_normal() inside the testharness. While running the above code block when the Rcpp::exception is thrown the catcher recognizes and handles it as std::exception, which is the base class for all the exceptions.

```c++
try{
rcpp_binseg_normal(data_vec, max_segments);
}
catch(std::exception& e){
cout << "Exception Handled" << endl;
}
```
The above code handles the exceptions generated by the testharness which will continue the execution of the testharness without failing it.
 
Thanks to [Dr.Toby Dylan Hocking](https://tdhock.github.io/blog/) for his support.
This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 
