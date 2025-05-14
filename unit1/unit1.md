# Unit 1: Digit DP
## Prerequisites
- Read and understand C++ code
- Time complexity and Big O notation
- Intermediate-level understanding of DP

## What is Digit DP?
**Digit DP** (Digit Dynamic Programming) is a powerful technique used to solve problems that involve counting numbers or solving for specific properties in a range of numbers, such as `[L, R]`. Instead of directly iterating over all the numbers in the range, we use dynamic programming (DP) to break down the problem into smaller subproblems. This method dramatically reduces time complexity by leveraging previously computed results.

### Worked Example: USACO 2014 Open, Silver - Problem 3: Odometer

The problem requires us to count how many **interesting numbers** lie in the range `[X, Y]`. An **interesting number** is defined as a number where at least half of its digits are the same.

You can access the problem statement [here](https://usaco.org/index.php?page=viewproblem2&cpid=435).

#### Naive Approach

A naive approach would be to simply iterate over each number in the range `[X, Y]` and check if it is **interesting**. However, this would be inefficient and result in a runtime of `O(N)`, where `N` is the number of elements in the range. This approach is not feasible for large values of `X` and `Y`.

#### Optimized Approach: Digit DP

Instead of iterating through every number in the range, we use **Digit DP** to count the interesting numbers more efficiently.

### Dynamic Programming State Representation

We define our state as:

```c++
dp[pos][cnt][under][zero]
```

- `pos`: The current position (digit) in the number. This is capped at 18, because 10^18 is the upper bound, and there are at most 18 digits.
- `cnt`: A running count of the number of digits that are the same.
- `under`: A flag indicating whether the current number is still under the upper bound.
- `zero`: A flag that indicates if we are considering leading zeros (i.e., if true, we've encountered a non-zero digit).

#### Transition Logic

At each position, we attempt to set the current digit to a value between 0 and 9 and check if the resulting number is "interesting".

1. Let `d` be the digit we're considering for the current position.
2. If `d` is less than the current digit in the upper bound, we set the `under` flag to `true`.
3. If `d` is non-zero, we update the `zero` flag.
4. If `d` is the same as the interesting digit, we increment the `cnt` (count of same digits).
5. If the count `cnt` exceeds `(pos + 1) / 2`, we know this number is **interesting** and we increment the result.

To count the numbers in the range `[X, Y]`, we calculate the difference of the results for the ranges `[0, Y]` and `[0, X)`.

Solution Code
```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
#define endl "\n"
#define sp <<" "<<

// See Dynamic Programming State Representation
// dp[pos][cnt][under][zero]
ll dp[19][42][2][2];
string num;

void clean() {
	for (int i = 0; i < 19; i++) {
		for (int j = 0; j < 42; j++) {
			for (int k = 0; k < 2; k++) {
				for (int l = 0; l < 2; l++) {
					dp[i][j][k][l] = -1;
				}
			}
		} 
	}
}

// See Transition Logic
ll work(int pos, int cnt, bool under, bool zero, int f1, int f2) {
	// We are at the end of the number
	if (pos == num.size()) {
		if (!zero) {
			return 0;
		}

		// This allows our implementation to not need to check
		// for the length of the current number and instead
		// check if it there are more than or equal to half of
		// the digits by checking if it is greater than or
		// equal to 20, which we initialized it to.

		// If the count cnt exceeds (pos + 1) / 2, we know this
		// number is interesting and we increment the result.
		if (f2 != -1) {
			if (cnt == 20) {
				return 1;
			} else {
				return 0;
			}
		}

		if (cnt >= 20) {
			return 1;
		} else {
			return 0;
		}
	}

	// This is what makes DP possible!
	// If we have already calculated the answer to this subproblem,
	// simply return it - no calculations need to be made.
	if (dp[pos][cnt][under][zero] != -1) {
		return dp[pos][cnt][under][zero];
	}

	ll ans = 0;
	for (int i = 0; i <= 9; i++) {
		// Let d be the digit we're considering for the
		// current position.
		int d = num[pos] - '0';

		// If d is less than the current digit in the
		// upper bound, we set the under flag to true.
		if (!under and i > d) {
			break;
		}
		bool n_under = under or (i < d);

		// If d is non-zero, we update the zero flag.
		bool n_zero = zero or i != 0;

		// Discard non-interesting candidates
		if (n_zero and f2 != -1 and i != f1 and i != f2) {
			continue;
		}

		// If d is the same as the interesting digit,
		// we increment the cnt (count of same digits).
		int n_cnt = cnt;
		if (n_zero) {
			if (f1 == i) {
				n_cnt++;
			} else {
				n_cnt--;
			}
		}

		ans += work(pos + 1, n_cnt, n_under, n_zero, f1, f2);
	}

	return dp[pos][cnt][under][zero] = ans;
}

ll solve(ll bound) {
	// Note that we are initializing cnt to 20 instead of 0
	// This is because we want to add to our count if we have
	// an occurrence of our interesting digit, but subtract 
	// if we have an occurrence of any other digit
	// See work() function for more details
	num = to_string(bound);
	ll ans = 0;
	for (int i = 0; i <= 9; i++) {
		clean();
		ans += work(0, 20, false, false, i, -1);
	}

	// Account for duplicate answers when two distinct 
	// digits i, j occupy exactly n/2 digits
	ll dup = 0;
	for (int i = 0; i <= 9; i++) {
		for (int j = 0; j <= 9; j++) {
			clean();
			dup += work(0, 20, false, false, i, j);
		}
	}

	return ans - (dup / 2);
}
 
void solution() {
	ofstream fout ("odometer.out");
	ifstream fin ("odometer.in");

	//
	ll x, y; fin >> x >> y;
	fout << solve(y) - solve(x - 1) << endl;
}

signed main() {

	solution();
}
```

### Key Points:

- **Digit DP** helps break down counting numbers with certain properties over large ranges, which in this case we call **interesting** numbers. You will find that **interesting** numbers can often even go past 18 digits for problems with stricter properties for **interesting** numbers.

#### Performance

This method significantly improves the time complexity from a brute force approach (`O(N)`) to a more efficient `O(d * 2d * 2 * 2)` or `O(d)`, which is manageable for large ranges.

### Conclusion

Digit DP is a fascinating technique that allows us to solve complex counting problems over ranges efficiently. With this approach, we can reduce the complexity of problems that might otherwise be too slow to compute directly. This is a great tool for competitive programming and problems involving large numbers and digit-specific properties.

---

#### Resources:
- [USACO - Odometer](https://usaco.org/index.php?page=viewproblem2&cpid=435)
- [Codeforces - Digit DP Blog](https://codeforces.com/blog/entry/53960)
- [Youtube - Introduction to Digit DP](https://www.youtube.com/watch?v=heUFId6Qd1A)

#### Further Practice:
- [Codeforces - EDU177E](https://codeforces.com/contest/2086/problem/E)