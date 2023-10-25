

# Memory model litmus tests as kvm-unit-tests

In this guide, we explain how to use kvm-unit-tests in combination
with litmus7 to generate and run litmus tests as tiny operating
systems.

## Background

### Memory model litmus tests

A litmus test is a small program designed to exercise a certain
behavior. Traditionally, litmus tests have been used to demonstrate
and exercise the memory model of parallel computing systems. Litmus
tests are often described in assembly or pseudo-assembly code and
require tools like [litmus7](http://diy.inria.fr/doc/litmus.html) to
genererate executable programs and run litmus tests on real hardware.

### Why kvm-unit-tests

litmus7 uses kvm-unit-tests to encapsulate litmus tests and generate
executables in the form of tiny operating system. Inside this tiny
operating system, the litmus test can control parameters of the
execution that a user space application cannot. For example, virtual
address translation and catch/handle exceptions.

## Building and running litmus kvm-unit-tests

litmus7 is a tool that given a litmus test will generate C source
code. The generated C source code is compiled and linked with
kvm-unit-tests to generate a flat file.

## Prerequisites

litmus7 is part of the herdtools7 toolsuite. See
https://github.com/herd/herdtools7/blob/master/INSTALL.md for
instructions of how to install herdtools7.

In addition to herdtools7, this guide assumes that you have a copy of
kvm-unit-tests and you have already configured and built the default
tests.

## Building a test

In this guide, we use MP (Message Passing), a popular litmus test
which demonstrates the communication pattern between a sender and a
receiver process of a single message `x`. The variable `y` is the flag
that the sender send only after the message is ready.

```
AArch64 MP
"PodWW Rfe PodRR Fre"
Generator=diyone7 (version 7.56)
Prefetch=0:x=F,0:y=W,1:y=F,1:x=T
Com=Rf Fr
Orig=PodWW Rfe PodRR Fre
{
0:X1=x; 0:X3=y;
1:X1=y; 1:X3=x;
}
P0          | P1          ;
MOV W0,#1   | LDR W0,[X1] ;
STR W0,[X1] | LDR W2,[X3] ;
MOV W2,#1   |             ;
STR W2,[X3] |             ;
exists (1:X0=1 /\ 1:X2=0)
```

Note: [diy7](http://diy.inria.fr/doc/gen.html) and
[diyone7]](http://diy.inria.fr/doc/gen.html) from the herdtools7
toolsuite can systematically generate litmus tests. For example to
generate the MP:

   diyone7 -arch AArch64 PodWW Rfe PodRR Fre -name MP

Assuming the file MP.litmus contains the test and TOP_DIR is the top
directory of kvm-unit-tests, we can use litmus7 to generate
the C sources:
```
   litmus7 -mach kvm-aarch64 -variant precise -o ${TOP_DIR}/litmus_tests MP.litmus
```

To build the test:

   cd litmus-tests && make && cd -

This will build the test in the file `litmus-tests/MP.flat` which you
can run like any other test:

    ./arm/run litmus-tests/MP.flat -smp 4

NOTE: Currently litmus7 makes the assumption that it will run with 4
cores.

# References

https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/expanding-memory-model-tools-system-level-architecture
https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/running-litmus-tests-on-hardware-litmus7
http://diy.inria.fr/doc/litmus.html
https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/generate-litmus-tests-automatically-diy7-tool
http://diy.inria.fr/doc/gen.html