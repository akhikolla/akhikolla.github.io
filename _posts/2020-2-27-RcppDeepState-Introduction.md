---
layout: post
title: "RcppDeepState"
category: [Introduction,Way to Fuzz Test packages]
tags: [RcppDeepState,R]
--- 
This Blog gives you an understanding of Fuzzing, why it is important and what are the things that you should have prior knowledge about, before starting on DeepState and it’s usage on Rcpp packages. DeepState helps you use both Fuzzers and Symbolic executors to generate random inputs in the backend.
<hr/>

Fuzzing is a way of identifying bugs by passing randomized or unexpected inputs to your program and we monitor the execution of these inputs for crashes or failures or memory leaks. We can perform fuzzing using Fuzzers and Symbolic executors.
<br/>
`Fuzzers Vs Symbolic Executors:`
<br/>
Although both Fuzzers and Symbolic executors pass inputs that break the code, there is a slight difference in the way they both analyze it. Fuzzers help you identify the ways we can break the program whereas Symbolic executors analyse how the program is handling the inputs and what part of the program is getting executed, to be more precise, branching all the possibilities for the provided inputs. In Symbolic execution, we pass symbolic(abstract) values for inputs rather than the actual ones. Symbolic executors generalize testing more like path-based translation.
It would really be interesting to test our code if we have a tool to interface Fuzzers and Symbolic executors.
<hr/>
`Presenting “DeepState” `
<br/>
DeepState is a testing platform where we can test our code with the combined power of fuzzers and Symbolic engines. In DeepState we have access to high quality,complicated fuzzers and various symbolic execution engines. These high-quality fuzzers are really helpful in identifying serious bugs.
Fuzzers detect the bugs in your code mainly using two strategies:
* `Code Coverage:` Fuzzers help us discover the part of code that is not covered for the passed inputs. This information could be beneficial in generating newer inputs accordingly to improve the performance on the fuzzer.
* `Sophisticated Inputs:` Fuzzers can learn from their previous inputs and produce newer inputs that can work better in breaking the code.

<h4><b>Purpose of combining Rcpp with DeepState:</b></h4>
* R/Rcpp doesn’t have a randomized test tool or a way to generate problematic inputs to test the packages. Also integrating the Rcpp code with a symbolic execution engine or a fuzzer could be strenuous. As DeepState can be integrated with C/C++ code this gives us the freedom to test Rcpp packages. 
* RcppDeepState helps us overcome the difficulty in testing our Rcpp packages with randomized/spontaneous inputs to identify subtle bugs in the code. RcppDeepState could be the first-ever solution and an easy way to fuzz test on the C/C++ code in the Rcpp package with the help of automatic DeepState test harnesses. DeepState currently supports five external fuzzers: libFuzzer, AFL, HonggFuzz, Eclipser and Angora.
* RcppDeepState will be a cross-platform R package, distributed on CRAN. It will provide C++ source code with the test harness for DeepState and few R functions for primary datatypes.	
Based on our analysis, it is observed that the following data types seem more often in Rcpp Packages. The table below demonstrates the count of the top 8 data types that appear in packages and functions.
	
```	
 S.No               Datatype    Functions   Packages
 1:      Rcpp::NumericVector       330         154
 2:      Rcpp::NumericMatrix       236         128
 3:                arma::mat  	   208         102
 4:              std::string  	   159          76
 5:    Rcpp::CharacterVector  	   112          51
 6:                      int  	   108          60
 7:      Rcpp::IntegerVector   	    88          37
 8:                   double   	    79          44
```
From the above analysis, we are able to identify the frequent data types and from that, we plan on implementing the following base functions. These functions could help us cover most of the functions and packages(5000 functions, 995 packages) that use Rcpp. These base functions will pass the randomized inputs for specific these data types.
* "RcppDeepState_NumericVector()"  - generates a randomized Numeric Vector. 
* "RcppDeepState_IntegerVector()"  - generates a randomized Integer Vector.
* "RcppDeepState_CharacterVector()" - generates a randomized Character Vector.
* "RcppDeepState_string()" - generates random strings. 
* "RcppDeepState_int()" - generates a random integer value.
* "RcppDeepState_double()" - generates a random Double value.
* "RcppDeepState_arma::mat()" - generates a randomized Armadillo matrix.
* "RcppDeepState_NumericMatrix()" - generates a randomized Numeric matrix.  

Initially, we would like to take an Rcpp Package and pass our randomized inputs generated from new RcppDeepState_IntegerVector() and RcppDeepState_NumericVector() functions and check for any crashes or bugs in this package.

For this purpose, one such package of interest is `binsegRcpp`. It is a package that computes the statistical model for the efficient implementation of binary segmentation. In this package, we will pass the randomized inputs generated from new RcppDeepState_IntegerVector() and RcppDeepState_NumericVector() functions and check for any crashes or bugs.

[binsegRcpp_package](https://github.com/tdhock/binsegRcpp/blob/master/src/rcpp_interface.cpp):
This package consists of two functions. These functions take in NumericVector,IntegerVector as inputs and return a List. 
```R
//[[Rcpp::export]]
Rcpp::List rcpp_binseg_normal(Rcpp::NumericVector, Rcpp::IntegerVector){}
//[[Rcpp::export]]
Rcpp::List rcpp_binseg_normal_cost(Rcpp::NumericVector, Rcpp::IntegerVector){}
```
Is there an easy way to Fuzz test functions in this package?     
I would say yes, that is by using DeepState functions that we created:

`“RcppDeepState_NumericVector()"` – will generate a random Numeric vector. If required you can pass the size of the vector that you wish to generate,and there is an overloaded function that is similar to Rcpp_NumericVector() that takes this size of the vector as an argument Rcpp_NumericVector(size).
Using the RcppDeepState_NumericVector() we throw random Numeric vectors to the first argument in rcpp_binseg_normal() function in our code.
<br/>
`" RcppDeepState_IntegerVector()"` – will generate a random Integer vector, if needed you can pass the size of the Integer vector that you want to generate,and there is an overloaded function that is similar to Rcpp_IntegerVector() that takes this size of the vector as an argument Rcpp_IntegerVector(size).
Using the RcppDeepState_IntegerVector() we throw random Integer vectors to the second argument in rcpp_binseg_normal() function.

<h3><b>Implementing a TestHarness</b></h3>
We need to include the DeepState headers and namespace in the beginning of the program:
```c++

#include <deepstate/DeepState.hpp>
using namespace deepstate;

```
Tests can be defined using the TEST macro. This macro takes two arguments: a unit name and a test name.
For example, the function generate_Birthdays() returns the birthdays of the students from the past 10 years. I am just curious how many people in my class have their birthdays on February 29th. Here my 
unit test name would be `BornOn_feb29` and test name `Generate_Birthdays`.
Another example of Test Name would be `Generate_Birthdays_Palindromic`. In this way, we can generate as many tests as we want.

```c++

TEST(BornOn_feb29, Generate_Birthdays) {
    ...// this test harness checks for birthdays 
    on feb29th
}
TEST(BornOn_feb29, Generate_Birthdays_Palindromic) {
    ...// this test harness checks for birthdays 
    on feb29th and if it is a palindrome
    
}

```
When working with Rcpp packages and function we are required to pass the randomized values to check for crashes or any vulnerabilities. Instead of randomly generating the values it is easy to just use the API to request `DeepState` for a value. 

The idea behind RcppDeepState_NumericVector() is to generate a randomized Numericvector by passing random values to the vector, which can be something like this:

```c++

// generates size of vector
int num_vector_size = 10
//declares NumericVector of num_vector_size
Rcpp::NumericVector NumericRand_vec[num_vector_size];
for(int i = 0 ; i < num_vector_size ; i++){
  NumericRand_vec[i] = DeepState_Double();
}
      
```
The above code generates a NumericVector of size 10 and each element in the vector is assigned to some random value generated by `DeepState_Double()`.

Original code in the function of RcppDeepState_NumericVector() is as follows:

```c++

Rcpp::NumericVector Exceptional_values(){

NumericVector values = NumericVector::create(NA_REAL,R_NaN,R_PosInf,R_NegInf);
 return values;
}
Rcpp::NumericVector RcppDeepState_NumericVector()
{
   int rand_val = DeepState_RandInt();
   int num_vector_size = DeepState_IntInRange(0,rand_val);
   int low_val = DeepState_Double();
   int high_val = DeepState_Double();
   Rcpp::NumericVector NumericRand_vec[num_vector_size];
   for(int i=0; i< num_vector_size; i++){
     OneOf(
[&]{
        NumericRand_vec[i] = DeepState_Double();
   },
[&]{
       if(low_val > high_val)  
        NumericRand_vec[i] = DeepState_DoubleInRange(high_val,low_val);
       else
        NumericRand_vec[i] = DeepState_DoubleInRange(low_val,high_val);
   },
[&]{
         // need this for NA,Nan,Inf,-Inf
        NumericRand_vec[i] = OneOf(Exceptional_values()); 
   }
   );
        return NumericRand_vec;
}

```
This function takes in zero arguments and returns a randomized numeric vector.
Here Exceptional_values() – method is used for passing values like NaN, NA, +Inf, -Inf.

Here `DeepState_RandInt()`- generates a random value which is the size of our Numeric vector.  
<br/>
Then we used `OneOf` to pick one value from specified values, we will discuss `OneOf` in the latter part.  
We use the following DeepState functions in the above code: 

```c++

//to generate random float values
DeepState_Float()
// to generate random Double values
DeepState_Double()
// to generate random integers from low to high 
DeepState_IntInRange(low, high)
// to generate random double values in provided range
DeepState_DoubleInRange(low,high)

```
We can overload the RcppDeepState_NumericVector() by passing the size of the numeric vector to be generated as the input. This will eliminate the random generation of size.

```c++

Rcpp::NumericVector RcppDeepState_NumericVector(int size_of_vector)
{
    for(int i=0; i< size_of_vector; i++){
     OneOf(
[&] {
       NumericRand_vec[i] = DeepState_DoubleInRange(low_val,high_val);
},
[&] {
       // need this for NA,Nan,Inf,-Inf
       NumericRand_vec[i] = OneOf(Exceptional_values()); 
});
         return NumericRand_vec;
}

```
Demonstrating the RcppDeepState_IntegerVector():
This function takes in zero arguments and returns a randomized Integer vector:

```c++
Rcpp::IntegerVector RcppDeepState_IntegerVector(){
int min_val = DeepState_MinInt();
int max_val = DeepState_MaxInt();
int integer_vector_size = DeepState_IntInRange(0,max_val);
Rcpp::IntegerVector IntegerRand_vec[int_vector_size];
for(int i=0; i< integer_vector_size; i++){
 OneOf(
[&] {
     IntegerRand_vec[i] = DeepState_Int();
},
[&] {
     IntegerRand_vec[i] = DeepState_IntInRange(min_val,max_val);
});
return IntegerRand_vec;
}
```
Similar to Numeric Vector we can overload the Integer Vector function as well, by passing size of the integer vector which is as follows:
```c++

Rcpp::IntegerVector RcppDeepState_IntegerVector(int size_of_vector){
for(int i=0; i< size_of_vector; i++){
 OneOf(
[&] {
     IntegerRand_vec[i] = DeepState_Int();
},
[&] {
     IntegerRand_vec[i] = DeepState_IntInRange(min_val,max_val);
});
return IntegerRand_vec;
}

```
Now we define the test harness and pass these randomized inputs as the arguments to the specific Rcpp function. 

```c++
TEST(Random_Set, Ranges) {
   Rcpp::RObject rcpp_result_gen;
   Rcpp::NumericVector data_vec = Rcpp::RcppDeepState_NumericVector();
   Rcpp::IntegerVector max_segments = Rcpp::RcppDeepState_IntegerVector();
   rcpp_result_gen = Rcpp::wrap(rcpp_binseg_normal(data_vec, max_segments));
}

```
Here rcpp_result_gen is of type Robject that stores the result from the rcpp_binseg_normal.
We are calling the newly created RcppDeepState_NumericVector() and RcppDeepState_IntegerVector() functions to get the randomly generated vectors and later we store them in respective type variables and pass them to make a function call. Above function call passes randomly produced deep state code to the binseg_normal(), this function call goes to the function which is present in RcppExports.cpp.
<br/>
<hr/>
`OneOf:` 
DeepState’s OneOf operator allows a user to express that one set of code “chunks” should be executed.OneOf makes use of `[&]` to connect different expressions present in the set and `{}` seperates each expression.

Suppose we have a case where the user passes two input values for add() method and based on the datatype the system decides on what kind of operation to be done on them. 

If a, b are of type integer then system decides to add them
If a, b are of type character then system decides to concatenate them.
If a, b are of type double then system returns a sum of type double.
<br/>
`OneOf` could be of good use when we let the system decide what to choose depending upon the functionality and input types.
```
int add(int a, int b){
return a+b;
}
char* add(char a,char b){
return  a+b;
}
double add(double a,double b){
return a+b;
}
```
`add(a,b)` 
<br/>
When a function call is made, depending on the datatype of the values that specific function is chosen and the logical execution is shown below.
```c++

if (/*a and b are integers/ ) {
     add() //int add(int,int) is called
} else if (/*a and b are char/ ){
     add() //char* add(char,char) is called
else   
add() // double add(double,double) gets called
}

```
Using OneOf lets DeepState automatically transform this into a switch case, OneOf automatically applies swarm testing where one of those options is to be chosen instead of a single value. 
Example:
```c++
TEST(OneOfTest, Basic) {
  int data = DeepState_Int();
  for (int i = 0; i < 10; ++i)
  {
    OneOf(
[&] {
         some_function_call(data%2);
      },
[&] {
         some_other_call(data^2);
      }
 );
}
}
```

The code for DeepState Test harness for Integer and Numeric vector in binseg package can be found here - 
[RcppDeepState](https://github.com/akhikolla/RcppDeepState)
and test harness can be found in [Testharness](https://github.com/akhikolla/RcppDeepState/blob/master/src/TestRcppharness.cpp)

Thanks to `Dr.Toby Dylan Hocking` for his support on this project. This blog is kindly contributed to [R-bloggers](https://www.r-bloggers.com/). 
<hr/>
