# Topic
## Fuzzing!
Fuzzy software for fun and safety

---
# Intro
## Chris Samuelson
## University of Cincinnati; Computer Engineering
## Writing code since ~2004

---
# Background
- I don't have a ton of personal experience with fuzzing
  - I'm not an expert
  - Consider this an open conversation
- I'm coming at this from a C++ perspective
- I will be using `AFL++` in my examples

---
# Objectives of Fuzzing
- Test resilience against unexpected inputs
- Exposing:
  - Security vulnerabilities
  - Reliability problems
---
# Terminology
--
# Fuzzing
From the users perspective: generating random inputs for a program to run, in order to find unhandled edge-cases, and hidden bugs.

--
# Corpus
The corpus is simply the body of inputs your fuzzer uses to track different sequences of coverage it has previously tested on your code.
We don't want to start from the beginning every time.

You will likely need to provide some minimal starting points for the fuzzer to work with.

---
# How does it work?
- Searches for inputs that cause failures
  - Using sanitizers makes these failures much more common
- A fuzzer can use the same methods of coverage detection as your coverage reporting to find inputs that cover different parts of the code
--
# How does it work?
> [!note]
> Thrown exceptions represent an edge case being handled safely
> To avoid exceptions raising false flags during fuzzing,
> `try`/`catch` should always wrap fuzzed library code
--
# How does it work?
## Corpus Minimizing
To my eye, one of the biggest values of using a tool and not rolling your own (generating random input is easy)

---
# Corpus Minimizing
Minimization looks for 2 things:
1. Removing redundant inputs; inputs that provide the same coverage
2. Reducing the size of the remaining inputs.
  - If a `12 Byte` input causes the same failure as a `5 MB`input, I'd rather debug the `12 Byte` one
--
# Corpus Minimizing
This makes test cases that result from fuzzing easier to reproduce, work with, and as a result, debug, and fix

--
# Corpus Minimizing
Always minimize

---
# Tools
- There's several standard tools that I've been able to find
- I'm unsure if there's any community consensus about the 'correct' one
- Google and LLVM seem to be the primary groups making them
- This will vary by platform, and language

--
# Tools
- [HongGFuzz](https://github.com/google/honggfuzz)
- [AFL](https://github.com/google/AFL)/[AFL++](https://github.com/AFLplusplus/AFLplusplus)
- [libfuzzer](https://llvm.org/docs/LibFuzzer.html)

---
# Using a fuzzer
OK, let's do this
- Write fuzz harness
- Build
- Run
- Minimize
---
# Fuzzing harness
> [!TODO] Write this slide, with example harness
---
# Building
In the end, building is straight-forward, and about how you would build normally.

Fuzzers are, in part, a compiler; well, they're a compiler wrapper.
It will set whatever flags you need for the fuzzer to do its job.
```bash
afl-g++-fast -o program main.cpp \<normal flags>
```

--
This means in practice you can override `CC`, and run `conan build` or `cmake -B build` etc.

--
> [!note] sanitizers
> As suggested before, don't forget to turn on the sanitizers you're interested in using.
> You may want multiple fuzz targets with different sanitizers active
---
# Running
Run the fuzz tool on the fuzz targets you've built
I won't cover this in detail; it varies a lot by tool
```bash
afl-fuzz -i corpus -o fuzzed/ -V 60 -- \<fuzzing instrumented executable> @@
```
--
# Runtime
Fuzzing could run infinitely.  Since the input size is unbounded, there's an infinite number of inputs it could try to provide.
So there will likely be a timeout for the fuzz runtime.
Choose a timeout that's useful for your context.

--
# CI
Fuzzing in CI will likely only last a couple minutes
# 24/7
You **probably** want to run fuzzing 24/7.

--
> [!warn] If you don't somebody else will
> As a rule of thumb, even if you aren't fuzzing your code, you can bet somebody else will.
> They will use the results to manipulate and hack your system.
> SO: Fuzz often

---
# Minimization
Minimize the results, once a run is completed.
We only want to add the simplified results to the corpus.

--
```bash
// file minimization

afl-cmin -i fuzzed -o minimized_result -- \<fuzzing instrumented executable> @@

mkdir -p minimized_result

// content minimization

for i in minimized_result/*; do
  afl-tmin -i $i -o minimized_result/`basename $i` -- \<fuzzing instrumented executable> @@
done
```
--
Upload the results to an artifact store, and use them as a starting point for your next run.

---
# Conclusion
Fuzzing is a form of automated testing that can readily find bugs and vulnerabilities in your code that would be very difficult to identify otherwise.

--
# Conclusion
Since It is 'so easy' to find security issues using fuzzing, use it before your adversaries do.
