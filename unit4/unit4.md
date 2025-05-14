# Unit 4
## Tree DP
### What is Tree DP?
**Tree DP** is used to perform Dynamic Programming on a tree. A tree is a unique graph which contains `n` vertices and `n-1` edges. Notably, there are no cycles in a tree, and each pair of connected vertices only have one edge connecting them. DP becomes useful on a tree when we reduce problems to subproblems with a vertex and its children.

To help visualize this concept, let's work through two ways to go about tree DP, top-down and bottom-up.

#### Top Down
Calculating vertex depth in a rooted tree is one natural way of using tree DP. Let the root vertex have a depth of `0`, then while performing a depth first search set its children to have their depth to be `depth + 1`.

#### Bottom Up
Calculating subtree size in a rooted tree

### Worked Example: APIO 2018 - Problem 3: Duathlon
<a href="https://oj.uz/problem/view/APIO18_duathlon">Problem Statement</a>

This problem asks that we find all triplets `(S, C, F)` such that it forms a unique triplet on the tree, which has the property that there is a simple path from S to F that contains C. With this in mind, we can simplify the problem to counting the amount of nodes between an arbitrary S and F.

To learn about Tree DP, we will focus on Subtasks 4-5, which state that the graph is acyclic. This implies that the resulting graph is a tree or a collection of trees.

#### Naive Approach
We can implement this by fixing our starting point S as the root of the tree, then traversing the tree with a Depth-First Search, keeping track of the depth of each node as we go. With the way we've framed the problem, and given that the depth of the current root is 1, we can add `(depth - 2)` to answer for each node if its depth is greater than 2. This is because the distance to our root S and a candidate descendant F is defined by their difference in depth. We subtract `2` to account for the fact that we only want to count how many ways we can place C in between them. Repeat this process by re-rooting the tree at each vertex for a time complexity of `O(n^2)`.

### Solution Code
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef long double ld;

#define endl "\n"
#define sp <<" "<<
#define REP(i, a, b) for(ll i = a; i < b; i++)
#define fast_io() ios_base::sync_with_stdio(false); cin.tie(NULL)

ll ans = 0;

// Perform a Depth-First Search to find the depths of each node given a root
// Note that depth-finding in itself is DP!
void dfs(int u, vector<vector<int>> &adj, vector<int> &depth, int p = -1) {
    for (auto &v : adj[u]) {
        if (v != p) {
            depth[v] = depth[u] + 1;
            dfs(v, adj, depth, u);
        }
    }

    if (depth[u] > 2) {
        ans += (depth[u] - 2);
    }
}

void solution() {
    int n, m; cin >> n >> m;
    vector<vector<int>> adj(n);
    REP(i, 0, m) {
        int u, v; cin >> u >> v; --u, --v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // Iterate through every possible node and set that as the root
    // of the tree
    // Sum up all their answers
    vector<int> depth(n);
    REP(i, 0, n) {
        fill(all(depth), 1);
        dfs(i, adj, depth);
    }

    cout << ans << endl;

    return;
}

signed main() {
    fast_io();

    solution();

    return 0;
}
```

### Optimized approach: All Roots Tree DP
**All Roots Tree DP** is an optimization for Tree DP problems, which given specific circumstances, it is possible to derive the solution for a subtree rooted at V from the solution for a subtree at U, for which U is V's parent vertex.

### Dynamic Programming State Representation

We define our state as:

```c++
void dfs(int u, ll prev, ll subp, int p)
```

- `u` is the current vertex that we are working on
- `prev` represents the solution to the tree rooted at the parent P
- `subp` represents the subtree size left over
- `p` is the parent of the current vertex

#### Transition Logic
In the case of our problem, because we only care about the sum of depths, its possible to rederive the solution for a given root, given the solution for its parent. Envision how the sum of depths change as we shift the root from U to V. The sum increases by the subtree size of V, not including the subtree size of U; and it will decrease by the sutreee size of V.

### Solution Code
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef long double ld;

#define endl "\n"
#define sp <<" "<<
#define REP(i, a, b) for(ll i = a; i < b; i++)
#define fast_io() ios_base::sync_with_stdio(false); cin.tie(NULL)

int root = 0;
ll ans = 0, base = 0;
vector<vector<int>> adj;
vector<int> depth, subt;
vector<bool> vis;

// Peform a depth-first search to find the
// depths of each node given a root
void dep(int u, int p = -1) {
    vis[u] = true;
    for (auto &v : adj[u]) {
        if (v != p) {
            depth[v] = depth[u] + 1;
            dep(v, u);
        }
    }

    if (depth[u] > 2) {
        base += (depth[u] - 2);
    }
}

// Perform a depth-first search to find the
// subtree sizes of each node given a root
void sub(int u, int p = -1) {
    for (auto &v : adj[u]) {
        if (v != p) {
            sub(v, u);
            subt[u] += subt[v];
        }
    }
}

// See Transition Logic
// Perform a depth-first search to calculate the
// answer to each re-rooted tree based on the
// answer of the original root

// See Dynamic Programming State Representation
void dfs(int u, ll prev = 0, ll subp = 0, int p = -1) {
    if (u == root) {
        // If we are at the current root, start the traversal
        // by passing the base answer
        ans += base;

        for (auto &v : adj[u]) {
            if (v != p) {
                dfs(v, base, subt[u] - subt[v], u);
            }
        }
    } else {
        ll curr = prev + subp - subt[u];

        ans += curr;
        for (auto &v : adj[u]) {
            if (v != p) {
                dfs(v, curr, subp + subt[u] - subt[v], u);
            }
        }
    }
}

void solution() {
    int n, m; cin >> n >> m;
    adj.assign(n, vector<int>());
    depth.assign(n, 1);
    subt.assign(n, 1);
    vis.assign(n, false);

    REP(i, 0, m) {
        int u, v; cin >> u >> v; --u, --v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    REP(i, 0, n) {
        // Note that our tree isn't necessarily connected
        // We handle each connected component separately,
        // assigning an arbitrary root, performing the
        // base answer calculation, then generalizing
        // for all roots in that connected component
        if (!vis[i]) {
            base = 0;
            root = i;
            dep(i), sub(i), dfs(i);
        }
    }

    cout << ans << endl;

    return;
}

signed main() {
    fast_io();

    solution();

    return 0;
}
```

### Key Points:
- **Tree DP** allows us to calculate subproblems for each vertex, either by utilizing a top-down or bottom-up approach (parent->children or children->parent relation).
- **All Roots Tree DP** allows us to calculate the answer for what it would be on a tree with an arbitrarily chosen root, then generalize this base solution to find that to find the answer for the tree rooted at every other vertex. 

#### Performance
This method significantly improves the time complexity from a brute force approach (`O(N^2)`) to a more efficient `O(N + N + N)` or `O(N)`, which is manageable for larger trees.

---

#### Resources:
- [APIO - Duathlon](https://oj.uz/problem/view/APIO18_duathlon)
- [USACO Guide - Tree DP](https://usaco.guide/gold/dp-trees?lang=cpp)
- [USACO Guide - All Roots Tree DP](https://usaco.guide/gold/all-roots?lang=cpp)

#### Further Practice:
- [Codeforces - Jeremy Bearimy](https://codeforces.com/contest/1280/problem/C)
- [Codeforces - Miss Punyverse](https://codeforces.com/contest/problem/1280/D)
