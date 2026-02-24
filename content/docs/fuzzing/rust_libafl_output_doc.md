
## LibAFL Fuzzer Output Explanation

This document explains the meaning of the fields shown in the LibAFL console output during a fuzzing run. It is intended to help participants interpret what the fuzzer is reporting while they use the tool.

---

## Event Prefixes

### `[UserStats #0]`
Lines starting with `[UserStats #0]` correspond to **UserStats events**, also called **Client Heartbeat events**.  
These events are emitted regularly to provide a snapshot of the current fuzzing campaign state, even if no new bugs or test cases have been found.

### `[Testcase #0]`
Lines starting with `[Testcase #0]` correspond to **Testcase events**.  
These events are emitted whenever the fuzzer discovers a new test case, such as a new coverage-increasing input or a crashing input.

It is normal to occasionally see no new log lines for some time if the fuzzer is not discovering new test cases or crashes.

---

## Common Output Fields

### `run time`
The amount of time the fuzzer has been running since it started.

### `clients`
The number of parallel fuzzing processes (clients) currently running. Each client independently generates and executes test inputs.

### `corpus`
The number of inputs currently stored in the corpus.  
The corpus contains all test cases that have been deemed interesting, typically because they increased coverage or triggered noteworthy behavior.

### `objectives`
The number of inputs that satisfy the configured objective, such as crashes, timeouts, or out-of-memory conditions.  
These inputs are often stored separately from the main corpus.

### `executions`
The total number of times the system under test (SUT) has been executed across all fuzzing clients.

### `exec/sec`
The current execution throughput, measured as executions per second.  
This reflects how fast the fuzzer is able to run test cases against the target.


**Source:**  
https://appsec.guide/docs/fuzzing/c-cpp/libafl/

---
### `stability`
Stability measures how deterministic the target program’s execution is with respect to coverage.

Stability is defined as the percentage of coverage edges that remain consistent when the **same input** is executed multiple times.  
If re-executing the same input always follows the same control-flow path, stability is 100%.

If coverage varies across executions of the same input (for example due to timing effects, randomness, concurrency, or external inputs), some edges are marked as **unstable**.  
Low stability forces the fuzzer to spend extra executions revalidating inputs, slows coverage growth, and can mask real findings

**Source:**  
https://aflplus.plus/docs/faq/

---
### `edges`
Counts the instrumented edges that have been hit at least once so far. It’s shown as covered/total, e.g. 1/100 (1%), meaning 1 control-flow transitions have been observed out of 100 possible. 

### `size_edges`
Shows how many instrumented edges already have a “smallest input” recorded for them. Fuzzer prints it as covered/total, so 1/100 (1%) means 1 edge currently have a known minimal testcase while 100 edges exist overall. Whenever another input reaches the same edge with fewer bytes, fuzzer optionally the stored size shrinks, helping the fuzzer keep shorter reproducers. 
