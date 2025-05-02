# Unit 1
## Digit DP
### What is Digit DP?
**Digit DP** is used to find the amount of numbers in a range [L, R] that hold a certain property.

### Worked Example: USACO 2014 Open, Silver - Problem 3: Odometer
<a href="https://usaco.org/index.php?page=viewproblem2&cpid=435">Problem Statement</a>

The problem requires us to count how many interesting numbers lie in the range [X, Y]. An interesting number is defined to be a number with at least half of its digits being the same.

A naive solution would be to loop through all numbers from X to Y, and checking on each iteration, if the number is interesting. However, it's quite clear that this would be too slow, given that the runtime is O(N), where N is defined to be the absolute value difference of X and Y.

We can instead approach this problem with Dynamic Programming, specifically Digit DP. Let's jump right into what it looks like, so we can get a better understanding.

Let our state be `dp[pos][cnt][under][zero]`. There's a lot to unpack here, so let me break down each part:
- pos - current position in our number (this is capped by 18, as 10<sup>18</sup> is the upper bound; there are 18 digits at most)
- cnt - running count of the number of digits that are the same
- under - upper-bound control (more on this later)
- zero - leading zero (note that true denotes that we have a non-zero digit)

At each position, we will try setting it to all numbers 0 through 9, and checking if this results in an interesting number. Particularly, we will try  The transition is as follows:
- Let d be the digit we are setting at the current position
- If d is less than the pos-th digit of our upper bound, then we set under to true.
- If d is non-zero, set zero to true
- If d is equal to our interesting digit, increment cnt by 1
- If cnt is greater than (pos + 1) / 2, then we know that this number is interesting and we can increment our answer by 1

One last observation we need to make is that instead of handling the range [X, Y] we can instead take the difference of the ranges[0, Y] and [0, X).

Solution Code (credits USACO Guide):
```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
#define endl "\n"
#define sp <<" "<<

ll dp[19][42][2][2];
string num;

void clean() {
	for (int i = 0; i < 19; i++) {
		for (int j = 0; j < 50; j++) {
			for (int k = 0; k < 2; k++) {
				for (int l = 0; l < 2; l++) {
					dp[i][j][k][l] = -1;
				}
			}
		} 
	}
}

ll work(int pos, int cnt, bool under, bool zero, int f1, int f2) {
	if (pos == num.size()) {
		if (!zero) {
			return 0;
		}

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

	if (dp[pos][cnt][under][zero] != -1) {
		return dp[pos][cnt][under][zero];
	}

	ll ans = 0;
	for (int i = 0; i <= 9; i++) {
		int dig = num[pos] - '0';

		if (!under and i > dig) {
			break;
		}

		bool n_under = under or (i < dig);
		bool n_zero = zero or i != 0;

		if (n_zero and f2 != -1 and i != f1 and i != f2) {
			continue;
		}

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
	num = to_string(bound);
	ll ans = 0;
	for (int i = 0; i <= 9; i++) {
		clean();
		ans += work(0, 20, false, false, i, -1);
	}

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

	ll x, y; fin >> x >> y;
	fout << solve(y) - solve(x - 1) << endl;
}

signed main() {

	solution();
}
```