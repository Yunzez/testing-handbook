# Fuzzing

Fuzzing represents a dynamic testing method that inputs malformed or unpredictable data to a system to detect security issues, bugs, or system failures. The concept of fuzz testing dates back over 30 years. Since then, fuzzing has evolved from purely generating random input (blackbox fuzzing) to generating inputs based on feedback gathered while executing the tested program (graybox fuzzing). 

## Terminology

* **SUT/target:** The System Under Test (SUT) is the piece of software that is being tested. 
* **Fuzzing/Fuzzer:**  Fuzz testing, also known as fuzzing, is an automated software testing method that supplies a SUT with invalid, unexpected, or random data as inputs. The program that implements the fuzzing algorithm is called a fuzzer.
* **Test case:** A test case is the concrete input that is given to the testing harness. Usually, it is a string or bitstring. A test input can also be encoded in a more complex format like an abstract syntax tree.
* **harness:** A harness handles the test setup for a given SUT. The harness wraps the software and initializes it such that it is ready for executing test cases. A harness integrates a SUT into a testing environment.
* **Corpus:**: The evolving set of test cases. During a fuzzing campaign, the corpus usually grows as new test cases are discovered.
* **Fuzz test:** A fuzz test consists of a fuzzing harness and the SUT. You might refer to a compiled binary that includes the harness and SUT as a fuzz test.
* **Fuzzing campaign:** A fuzzing campaign is an execution of the fuzzer. A fuzzing campaign starts when the fuzzer starts testing and stops when the fuzzing procedure is stopped.
* **Code coverage:** A metric to measure the degree to which the source code of a program is executed.
{.no-bullet-point-list}


The first step when fuzzing a software package is assessing the fuzz-worthy targets in the package. This is because fuzzing works particularly well with certain code structures and can be more involved with other structures. For instance, fuzzing a parser is straightforward: fuzzing is easy to set up, and finding bugs is very likely. 

Before we introduce typical fuzzing setups, we first want to explain today's default fuzzing strategy: mutation-based evolutionary fuzzing.

## The default fuzzing algorithm is mutation-based and evolutionary

The original [AFL](https://lcamtuf.coredump.cx/afl/) fuzzer employed a fuzzing algorithm inspired by evolutionary algorithms. This algorithm is the de facto algorithm for fuzzers.

The basic idea is to maintain a population of test cases in a **corpus**. Each test case within the corpus has a certain fitness—analogous to the biological theory. This can be determined by using a coverage metric to denote the fitness of a test case. An evolutionary algorithm then schedules fit test cases and applies mutations to produce offspring. Each new mutated test case is executed, and a fitness is assigned. The idea is to only allow offspring that are beneficial in terms of fitness to survive. 


## Introduction to fuzzers

Every fuzzing setup consists of an instrumented System Under Test (SUT), the fuzzing harness, and the fuzzer runtime. A runtime for the instrumentation may also be required. For example, the optional AddressSanitizer (ASan) instrumentation adds a runtime that is used to detect memory corruption bugs like [heap-buffer overflows](https://en.wikipedia.org/wiki/Heap_overflow) more reliably. The following figure shows the standard fuzzing setup.

![alt text](./intro.svg "Title")
The general fuzzing scenario consists of the developer writing a harness for a SUT. After starting a fuzzing campaign, the fuzzer runtime generates random test cases that are sent to the harness. The harness then executes the SUT, which could lead to the discovery of bugs and crashes. Instrumentation runtime and the instrumentation added to the SUT are generally optional, even though most fuzzers instrument the SUT code and add a runtime.



**SUT (System Under Test):** This is the code you want to test. To create a fuzzing build of your SUT, you need to control how the application's code is compiled and linked. The following figure shows a very simple SUT that serves as a running example throughout this chapter of the Testing Handbook.

**Example code with a bug that causes an abort. The `check_buf` function aborts for the input "abc"**.
```Rust
use std::process;

fn check_buf(buf: &[u8]) {
    if buf.len() > 0 && buf[0] == b'a' {
        if buf.len() > 1 && buf[1] == b'b' {
            if buf.len() > 2 && buf[2] == b'c' {
                process::abort();
            }
        }
    }
}

fn main() {
    let buffer: &[u8] = b"123";
    check_buf(buffer);
}
```


**Harness:** The harness is the entrypoint for your fuzz test. The fuzzer calls this function with random —or carefully mutated—data.

```Rust
fn harness(data: &[u8]) {
    check_buf(data);
}
```

There is no standard in the Rust ecosystem for the function signature of a harness. To remain generic, let's pretend a Rust fuzzing harness is a void function that receives a byte array with random data of Rust type `&[u8]`.

**Fuzzer runtime:** The fuzzing loop is implemented here. This unit also provides the main function for the fuzzer. The fuzzing runtime parses fuzzing options, executes the harness, collects feedback, and manages the fuzzer state. The runtime is provided by the fuzzing project you use, such as libFuzzer or AFL++. Any runtime that is required for collecting feedback through instrumentations is implemented in the fuzzer runtime.
