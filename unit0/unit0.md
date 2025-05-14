# Unit 0
## Prerequisites
- Read and understand C++ code
- Time complexity and Big O notation

## Introduction to Dynamic Programming
### What is Dynamic Programming?
**Dynamic Programming** (DP) is a problem-solving technique that can be used on problems that can be broken down into subproblems. More formally, we can apply dynamic programming when the *optimal substructure* and *overlapping substructure* properties hold true.

In layman terms, this means we can save time and space (a.k.a. memory) by reusing previous computations (recurrences). To further understand this, let's go through a worked example, Fibonacci.

### Worked Example: Fibonacci
Fibonacci is famous sequence that is built on a recursive formula, which allows for it to easily be optimized with DP:
```
F(0) = 1
F(1) = 1
F(i) = F(i-1) + F(i-2)
```

If we naively implement this via a recursive function, it results in a time complexity of O(2<sup>n</sup>), where n represents the nth Fibonacci number.
```c++
long long fib(int i) {
    if (i == 0 or i == 1) {
        return 1LL;
    } else {
        return fib(i - 1) + fib(i - 2);
    }
}
```

We can improve our implementation by using dynamic programming. There are generally two approaches to DP, *memoization* (top-down) and *tabulation* (bottom-up). The first utilizes recursion, while the second utilizes iteration. Iteration is typically faster than recursion, so it is preferred when possible.

Here's code that will generate up to the nth fibonacci number on every call. Then, to find any fibonacci number, we can simply access it via index from the *fib* vector.

```c++
vector<long long> fib = {1, 1};
void generate(int mx) {
    if (mx < 2 || mx < fib.size()) return;
    int prevSize = fib.size();
    fib.resize(mx+1);
    for (int i = prevSize; i <= mx; i++) {
        fib[i] = fib[i-1] + fib[i-2];
    }
}
```

A quick glance might not differentiate the two implementations immediately, but on closer inspection, the time and space complexity are evidently linear O(n).

Note that if we all we require is the nth fibonacci number, then there exist further optimizations: *space optimization*, with time O(n), space O(1); and *matrix exponentation*, with time O(log(n)), space O(log(n)).

### States and Transitions
A common way to think about dynamic programming is through the idea of *states* and *transitions*. A state is defined as the current state of a subproblem. In this case, it's the `i`-th fibonacci number. A transition is defined as the transition between two (or more) states, or more succinctly, the operation(s) that allow for a state to be calculated from other states. In this case, the `i`-th fibonacci number can be calculated from the `i-1` and `i-2` states.
