```c++
vector<int> zFunc(string s) {
	int n = s.size();
	vector<int> z(n, 0);
	for (int i = 1, l = 0, r = 0; i < n; i++) {
		if (i <= r && i + z[i - l] - 1 < r) {
			z[i] = z[i - l];
		} else {
			z[i] = max(0, r - i + 1);
			while (i + z[i] < n && s[i + z[i]] == s[z[i]]) z[i]++;
		}
		if (i + z[i] - 1 > r) {
			l = i;
			r = i + z[i] - 1;
		}
	}
	return z;
}
```

```c++
#include <bits/stdc++.h>
using namespace std;
const int L = 1e6 + 5;
const int HASH_CNT = 2;
int hashBase[HASH_CNT] = {29, 31}; 
int hashMod[HASH_CNT] = {int(1e9 + 9), 998244353};

struct hashwithstring {
	int ls;
	long long hashPwd[HASH_CNT][L];
	long long hashValue[HASH_CNT][L];
	char s[L];

	void init() {
		ls = 0;
		for (int i = 0; i < HASH_CNT; i++) {
			hashPwd[i][0] = 1;
			hashValue[i][0] = 0;
		}
	}

	hashwithstring() {
		init();
	}

	void add(char a) {
		s[++ls] = a;
		for (int i = 0; i < HASH_CNT; i++) {
			hashPwd[i][ls] = 1ll * hashPwd[i][ls - 1] * hashBase[i] % hashMod[i];
			hashValue[i][ls] = (1ll * hashValue[i][ls - 1] * hashBase[i] + a) % hashMod[i];
		}
	}

	vector<int> get_hash(int l, int r) {
		vector<int> res(HASH_CNT, 0);
		for (int i = 0; i < HASH_CNT; i++) {
			int t = hashValue[i][r] - 1ll * hashValue[i][l - 1] * hashPwd[i][r - l + 1] % hashMod[i];
			t = (t + hashMod[i]) % hashMod[i];
			res[i] = t;
		}
		return res;
	}
};

bool equal(const vector<int>& e1, const vector<int>& e2) {
	assert(e1.size() == e2.size());
	for (int i = 0; i < HASH_CNT; i++) {
		if (e1[i] != e2[i]) {
			return false;
		}
	}
	return true;
}

int n;
hashwithstring s, t;
char str[L];

void solve() {
	t.init();
	int len = strlen(str);
	for (int i = 0; i < len; i++) t.add(str[i]);

	int d = 0;
	for (int i = min(len, s.ls); i >= 1; i--) {
		if (equal(s.get_hash(s.ls - i + 1, s.ls), t.get_hash(1, i) ) ) {
			d = i;
			break;
		}
	} 
	for (int i = d; i < len; i++) s.add(str[i]);
}
```