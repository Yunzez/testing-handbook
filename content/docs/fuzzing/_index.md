# Fuzzing

Fuzzing is an automated testing technique where a program (the *fuzzer*) runs a piece of code many times with different inputs, trying to trigger crashes or unexpected behavior that might reveal bugs or security vulnerabilities.

Instead of a developer manually writing many test cases, a fuzzer:

- Automatically generates or mutates inputs.
- Runs the code under test with those inputs.
- Watches for crashes and keeps track of which inputs explore new behavior.
  
## Terminology

* **SUT/target:** The System Under Test (SUT) is the piece of software that is being tested. 
* **Fuzzer:**  Fuzz testing, also known as fuzzing, is an automated software testing method that supplies a SUT with invalid, unexpected, or random data as inputs. The program that implements the fuzzing algorithm is called a fuzzer.
* **Test case:** A test case is the concrete input that is given to the testing harness.
* **harness:**  A small wrapper function that:
  - Accepts input from the fuzzer.
  - Converts it into the right argument types.
  - Calls the target function (or functions) you want to fuzz.
  - In this study, the IDE extension **auto-generates** harnesses for you. You can open and edit them like normal Rust code.
* **Fuzz test:** A fuzz test consists of a fuzzing harness and the SUT. You might refer to a compiled binary that includes the harness and SUT as a fuzz test.
* **Fuzzing campaign:** One run of the fuzzer. When you click the fuzzing run button in the Extension, you start a campaign: the fuzzer generates inputs, runs the harness+SUT, and records results.
* **Code coverage:** A metric to measure the degree to which the source code of a program is executed.

You do not need to know the exact formulas or internals; we mostly care about how you interpret and use these concepts while working with the tool.


The first step when fuzzing a software package is assessing the fuzz-worthy targets in the package. This is because fuzzing works particularly well with certain code structures and can be more involved with other structures. For instance, fuzzing a parser is straightforward: fuzzing is easy to set up, and finding bugs is very likely. 

Before we introduce typical fuzzing setups, we first want to explain today's default fuzzing strategy: mutation-based evolutionary fuzzing.

## The default fuzzing algorithm is mutation-based and evolutionary

Many modern fuzzers use a **mutation-based, coverage-guided** strategy:

1. Start with one or more initial inputs (a *corpus*).  
2. Run the harness+SUT on these inputs and measure code coverage.  
3. Take inputs that explore new code paths and mutate them.  
4. Run the new inputs, keep the ones that seem interesting (e.g., increase coverage or cause crashes).  
5. Repeat this loop during the fuzzing campaign.

You can think of it as a feedback loop: inputs that lead to new or surprising behavior are kept and used to generate more inputs.

## Typical fuzzing setup

Every fuzzing setup consists of an instrumented System Under Test (SUT), the fuzzing harness, and the fuzzer runtime. A runtime for the instrumentation may also be required. For example, the optional AddressSanitizer (ASan) instrumentation adds a runtime that is used to detect memory corruption bugs like [heap-buffer overflows](https://en.wikipedia.org/wiki/Heap_overflow) more reliably. The following figure shows the standard fuzzing setup.

A fuzzing run usually connects three main pieces:

```text
   +-----------------+       +-----------------+       +-----------------+
   |  Fuzzer runtime | ----> |    Harness      | ----> |   SUT / target  |
   | (libFuzzer, etc)|       | (wrapper fn)    |       |  (your library) |
   +-----------------+       +-----------------+       +-----------------+
            ^                        |                          |
            |                        |                          |
            +------------------------+--------------------------+
                         crashes, coverage, logs
```

![alt text](./intro.svg "Title")
The general fuzzing scenario consists of the developer writing a harness for a SUT. After starting a fuzzing campaign, the fuzzer runtime generates random test cases that are sent to the harness. The harness then executes the SUT, which could lead to the discovery of bugs and crashes. Instrumentation runtime and the instrumentation added to the SUT are generally optional, even though most fuzzers instrument the SUT code and add a runtime.



**SUT (System Under Test):** This is the code you want to test. To create a fuzzing build of your SUT, you need to control how the application's code is compiled and linked. The following figure shows a very simple SUT that serves as a running example throughout this chapter of the Testing Handbook.

## 5. A tiny example
Here is a small example of code with a bug and a matching harness, adapted from common fuzzing tutorials.

**Example code with a bug that causes an abort. The `check_buf` function aborts for the input "abc"**.
```Rust
use std::process;

fn check_buf(buf: &[u8]) {
    if buf.len() > 0 && buf[0] == b'a' {
        if buf.len() > 1 && buf[1] == b'b' {
            if buf.len() > 2 && buf[2] == b'c' {
                process::abort(); // I AM BUG
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

The fuzzer’s job is to automatically discover an input like `b"abc"` that makes `check_buf` abort, without a human having to think of that specific combination.

In your tasks, you will not be writing these harnesses from scratch—the IDE extension will generate them. You will focus on:

- Choosing targets.  
- Inspecting and (optionally) editing harnesses.  
- Running fuzzing campaigns from the IDE.  
- Interpreting the output (crashes, coverage, other fields) to decide what to do next.  

That is all the background you need for the study. You are not expected to know or remember the details of fuzzing internals; we are interested in how you understand and use the tool while working with realistic code.
