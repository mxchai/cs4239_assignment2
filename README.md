# Using LLVM to check for pointer arithmetic done on non-array variables

We are trying to do an analysis on pointer arithmetic on non-array objects. This is the [link](https://www.securecoding.cert.org/confluence/display/c/ARR37-C.+Do+not+add+or+subtract+an+integer+to+a+pointer+to+a+non-array+object) for the SEI CERT C standard. 

This is related to software security because C struct members are not guaranteed to be contiguous. Hence, there might be an exploit where a program iterating through a struct of function pointers (via pointer arithmetic) and executing them ends up executing the an attacker’s injected functions.

# Rough algorithm sketch

**Perform 1st pass through the LLVM IR, map all virtual registers to their types**
* Data structure: an unordered map 
  * Key will be virtual register name, Value will be its LLVM type
* Keep a look out for struct pointers
* Keep a lookout for getelementptr: it doesn’t actually “get” the element pointer, but rather it adds to the address of the pointer
  * One exception where it really is used for “getting” pointers is when you supply an addition argument of 0 (so you don’t actually add anything to the address)

**Perform 2nd pass through LLVM IR**
* Keep a look out for getelementptr (because that’s the only way LLVM IR does pointer arithmetic)
* Check the arguments’ type against our hashmap
* If it’s an array, ignore it
  * If it’s non-array, we found ourselves a bug
    * Structs
    * And basically every single other type that’s not an array 
  * When we encounter a bug, print out to the screen. Or print all of them at the end, whichever way works.

# Helpful information

## Reading C++ and LLVM source code

* Use `errs()` to print out debug messages. Using `cout` will not print at the
correct place because `errs()` prints to `stderr`, and `stdout` output only appears
after `stderr` messages.
* Everything in `llvm` namespace has a `.dump()` method, which prints out what
they to `stderr`. Pretty useful for debugging.
* `llvm::Instruction` is a derived class of `llvm::Value`, which has a
`getName()` function that returns a `StringRef`. You can call `.str()` on a
`StringRef` to retrieve the contents as a `std::string`
* In our code, `I` has type `llvm::Instruction`

## Links

The links to the source code are much more useful, since we are using `llvm-3.4.2`, but the generated documentation is only for `llvm-6.0.0`. The official documentation links are still included here because the inheritance graphs could be helpful.

* How to interpret LLVM documentation's Inheritance Graphs: [Link](http://users.elis.ugent.be/~jvcleemp/LLVM-2.4-doxygen/graph_legend.html)
* [Instruction.cpp](https://github.com/llvm-mirror/llvm/blob/release_34/lib/IR/Instruction.cpp)
* [Instructions.cpp](https://github.com/llvm-mirror/llvm/blob/release_34/include/llvm/IR/Instructions.h)
* [Value.cpp](https://github.com/llvm-mirror/llvm/blob/release_34/lib/IR/Value.cpp)
* [GetElementPtrInst](http://llvm.org/doxygen/classllvm_1_1GetElementPtrInst.html)
* [LoadInst](http://llvm.org/doxygen/classllvm_1_1LoadInst.html)
* [StoreInst](http://llvm.org/doxygen/classllvm_1_1StoreInst.html)
* [AllocaInst](http://llvm.org/doxygen/classllvm_1_1AllocaInst.html)
