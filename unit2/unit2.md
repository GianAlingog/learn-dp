# Unit 2: Bitmask DP and SOS DP
## Prerequisites
- Read and understand C++ code
- Time complexity and Big O notation
- Intermediate-level understanding of DP
- Bitmasks and subsets

## What is Bitmask DP?

**Bitmask DP** is a dynamic programming technique used to solve problems where we need to explore subsets or combinations of elements, typically represented as bits. In Bitmask DP, a bitmask is used to represent a subset of elements, where each bit corresponds to the inclusion or exclusion of an element in the subset. The key advantage of Bitmask DP is that it allows us to compactly represent and operate on subsets, often reducing the problem complexity significantly.

## Sum over Subsets DP (SOS DP) Optimization

**SOS DP** is an optimization used with **Bitmask DP** to calculate sums over subsets efficiently. In problems involving bitmasking, we often need to perform operations over all subsets of a set, which can be computationally expensive. SOS DP allows us to calculate the sums over subsets without explicitly iterating over each possible subset, thus significantly improving the time complexity. It avoids the need to explicitly calculate the sum for each individual subset.

SOS DP works by combining results of smaller subsets and building up to larger subsets, often by processing the subsets incrementally.

### Worked Example: USACO 2025 Gold, Problem 3: Friendship Editing

The problem asks us to minimize the number of edits required to ensure that a group of friends are all connected in some way. Specifically, we are given a set of friendships, and we need to edit the friendships so that the group becomes connected. We use Bitmask DP to efficiently explore the different ways in which we can connect all the friends, given their initial relationships.

You can access the problem statement [here](https://usaco.org/index.php?page=viewproblem2&cpid=1499).

#### Naive Approach

A brute force approach would involve trying to connect each pair of friends iteratively and checking if all of them are connected. This approach has exponential time complexity because of the number of potential friend connections, which would make it too inefficient.

#### Optimized Approach: Bitmask DP

Using **Bitmask DP**, we can represent subsets of friends as bitmasks and iterate through the different subsets to compute the minimal number of edits required to connect all the friends.

### Dynamic Programming State Representation

We define our state as:

```cpp
dp[mask]
```

Where:
- `mask` represents a subset of friends, with each bit indicating whether a particular friend is included in the subset.
- `dp[mask]` stores the minimal number of edits required to make the subset of friends represented by `mask` connected.

#### Transition Logic

To update the DP table, we consider all possible subsets of friends and check if they can be connected with fewer edits by examining each possible transition. For each mask, we attempt to remove a friend or add a new one and compute the minimal number of edits required.

We calculate the number of flips (i.e., edits) needed to convert a given subset into a fully connected group. This is done by checking the adjacency matrix of friendships for the current subset and using the minimum of previous results to update the current state.

### Solution Code

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
#define endl " "
#define sp <<" "<<

const int INF = 1e9 + 7;

// See Transition Logic
// Adjacency Matrix
int adj[16][16];

// See Dynamic Programming State Representation
// dp[mask]
int dp[1 << 16];
int n, m;

// Stores flips from original graph
// Let the bitmask represent which friends are connected
// flip[mask] represents the edit distance from the
// original graph
int flip[1 << 16];

void solution() {
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int u, v; cin >> u >> v; u--, v--;
        adj[u][v] = adj[v][u] = 1;
    }

    // For each of the 2^n subsets, count how many flips
    // (or edits) are needed for this subset
    // This has a time complexity of O(n^2 * 2^n)
    for (int mask = 0; mask < (1 << n); i++) {
        for (int i = 0; i < n; i++) {
            for (int j = i+1; j < n; j++) {
                if (mask & (1 << i) and mask & (1 << j)) {
                    if (adj[i][j]) flip[mask]++;
                    else flip[mask]--;
                }
            }
        }
    }

    // Max edges in a graph: n * (n-1) / 2
    // Initial edge count: m
    // Max unset edges in out graph:
    dp[0] = n * (n-1) / 2 - m;
    for (int mask = 1; mask < (1 << n); mask++) {
        dp[mask] = INF;

        // Compute the state for this subset based on
        // a combination of its subsets
        // Particularly, given a submask, it takes
        // its complement to form mask and adds
        // its flips (or edit distance) to form
        // the mask we are interested in
        for (int sub = mask; sub; sub = (sub-1) & mask) {
            dp[mask] = min(dp[mask], dp[mask & ~sub] + flip[sub]);
        }
    }

    // (1 << n) - 1 = all 1's of length n in binary
    // This takes the minimum amount of edits for
    // all friends to be connected
    cout << dp[(1 << n) - 1] << endl;
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);

    solution();
}
```

---

### Key Points:
- **Bitmask DP** allows efficient computation over subsets of elements using bit manipulation, storing results in a DP table for minimal computation.
- **SOS DP** optimizes Bitmask DP by reducing the need to explicitly iterate over all subsets, thereby speeding up computations that involve subset sums or other related operations.
- Both techniques are crucial for solving combinatorial problems with large input sizes and are commonly used in competitive programming.

#### Resources:
- [USACO - Friendship Editing](https://usaco.org/index.php?page=viewproblem2&cpid=1499)
- [USACO Guide - Bitmask DP](https://usaco.guide/gold/dp-bitmasks?lang=cpp)
- [USACO Guide - SOS DP](https://usaco.guide/plat/dp-sos?lang=cpp)

#### Further Practice:
- [CSES - SOS Bit Problem](https://cses.fi/problemset/task/1654)
- [Codeforces - Bits and Pieces](https://codeforces.com/contest/1208/problem/F)
