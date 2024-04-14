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