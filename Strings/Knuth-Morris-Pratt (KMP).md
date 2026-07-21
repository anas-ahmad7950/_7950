Yields a prefix function (failure function) that is used to find the number of occurrences of a pattern $pt$ in a larger string $s$ (+ occurrences positions), get the frequency of each prefix in $pt$ relative to itself or other string (+ get unique prefixes of $pt$ relative to itself) and get the longest palindrome prefix/suffix.

* Failure Function: $O(n)$
* KMP: $O(n + m)$
Template 1
```c++
vector<int> FailureFunction(string &pt) {
    int m = pt.size();
    vector<int> lpp(m);
    lpp[0] = 0;
    for (int i = 1, k = 0; i < m; i++) {
        while (k > 0 && pt[i] != pt[k]) k = lpp[k - 1];
        lpp[i] = (pt[i] == pt[k] ? ++k : 0);
    }
    return lpp;
}

int KMP(string &s, string &pt) {
    int n = s.size();
    int m = pt.size();
    vector<int> lpp = FailureFunction(pt);
    
    int ret = 0;
    for (int i = 0, k = 0; i < n; i++) {
        while (k > 0 && s[i] != pt[k]) k = lpp[k - 1];
        
        if (s[i] == pt[k]) k++;
        
        if (k == m) {
            ret++;
            // cout << i - m + 1 << '\n';
            k = lpp[k - 1];
        }
    }
    return ret;
}

vector<vector<int>> kmp_automaton(string pt) {  
    pt.push_back('#');  
    int m = pt.size();  
    vector<int> lpp = FailureFunction(pt);  
  
    vector<vector<int>> instant(m, vector<int>(26));  
    for (int k = 0; k < m; k++) {  
        for (int ch = 0; ch < 26; ch++) {  
            if (k > 0 && pt[k] - 'a' != ch) instant[k][ch] = instant[lpp[k - 1]][ch];  
            else instant[k][ch] = k + (pt[k] - 'a' == ch);  
        }  
    }  
    return instant;  
}
```

### KMP TRICKS

**Longest Palindrome Prefix/Suffix**
Given a string $s$ of length $n$, we can find the longest palindrome suffix of $s$ by creating a string $t$ which equals to $rev(s)@s$ where $@$ doesn't appear in the input. This utilizes the fact that a string is palindrome if it remains the same when reversing it, therefore, the longest palindrome suffix will equal to $f(sz(t) - 1)$.

In case of obtaining the longest palindrome prefix, string $t$ will become $s@rev(s)$ and the rest will remain the same. 
```c++
	string s;
    cin >> s;
    
    string rev = s;
    reverse(rev.begin(), rev.end());
    string t = rev + "@" + s; // t = rev(s)@s -> longest palindrome suffix
    // t = s@rev(s) -> longest palindrome prefix
    
    vector<int> lpp = FailureFunction(t);
    cout << lpp[t.size() - 1] << '\n';
    
    // Minimum number of characters to add in order to make the string palindrome = n - max(longest palindrome prefix, longest palindrome suffix)
```

___

**Minimum Repetition Cycle**
Given a string $s$ of length $n$, we can find the smallest prefix size such that $s$ equals to the concatenation of that prefix maximum number of times or report that there no repetition cycle in string $s$.

let $s = \text{abcabcabcabc}$, then its the failure function will be $f(i) = [0, 0, 0, 1, 2, 3, 4, 5, 6, 7, 8 , 9]$. We can think in this problem block-wise, since $f(11) = 9$, therefore the first 9 characters equals to the last 9 characters. So we can assume that each $\text{block size} = 3$, more formally:
$$\text{block size} = sz(s) - f(sz(s) - 1)$$
If $sz(s) \bmod \text{block size} = 0$, then string $s$ has a repetition cycle with minimum size equals to $blocksize$. Why is that true? Lets return to our example. Lets denote the first $3$ characters of $s$ with $A$, the second $3$ with $B$, the third $3$ with $C$ and the fourth $3$ with $D$. Due to the fact that $f(11) = 9$, then the first $3$ blocks equals to the last $3$ blocks, in other words: $ABC = BCD$, which implies that $A = B = C = D$.
```c++
string s;  
cin >> s;  
int n = s.size();  
vector<int> lpp = FailureFunction(s);  
  
int blk_sz = n - lpp[n - 1];  
if (n % blk_sz == 0) { cout << blk_sz << ' ' << s.substr(0, blk_sz) << '\n'; }  
else { cout << "No Repetition Cycle\n"; }
```

___

**Prefixes Frequency in a String**
For a string $s$ of length $n$, we can count the frequency of each prefix $s_i$ in that string.
We can utilize the fact that each prefix fails to another smaller prefix. So, as a base case each prefix has a frequency of $1$, and then for each prefix size $i$ from $n$ down to $1$, we add the frequency of $i$ to the frequency of the smaller prefix that $p_i$ fails to.
```c++
string s; cin >> s;  
int n = s.size();  
  
vector<int> frq(n + 1), lpp = FailureFunction(s);  
for (int i = 1; i <= n; i++) frq[i]++;  
for (int i = n; i; i--) frq[lpp[i - 1]] += frq[i];

// f(index) = proper prefix size
```

If we are given two strings $s$ and $pt$ of lengths $n$ and $m$ respectively, we can count the frequency of each prefix of $pt$ in string $s$ by creating a string $t = pt@s$, obtaining its failure function $f(i)$, then we set our base case by looping on string $t$ from index $m + 1$ to $n + m$ and increasing the frequency of the smaller prefix that $t[0 \dots i]$ fails to by $1$, and finally, we apply the same logic of propagation as before.
```c++
string s, pt;
cin >> s >> pt;
int m = pt.size();

string t = pt + "@" + s;
int sz = t.size();
vector<int> lpp = FailureFunction(t);

vector<int> frq(m + 1);
for (int i = m + 1; i < sz; i++) frq[lpp[i]]++;
for (int i = m; i; i--) frq[lpp[i - 1]] += frq[i];
```

___

**Unique Prefixes in a String**
To determine the unique prefixes in a string $s$ of length $n$, we can use that face that the value of the failure function at index $i$ can increase by at most $1$, more formally:
$$
f(i) \le f(i - 1) + 1
$$
So, if the value of the failure function at index $i$ equals to some positive integer $x$, this implies that all non-negative values less than $x$ will be found before $i$, which means that all prefixes with size less than or equal to $x$ are not unique.
```c++
string s; cin >> s;  
int n = s.size();  
vector<int> lpp = FailureFunction(s), unique(n, 1);  
for (int i = 0; i < n; i++) if (lpp[i]) unique[lpp[i] - 1] = 0;
```

___

**Number of Distinct Substrings in $O(n^2)$**
The number of distinct substrings $\alpha$ in a string $s$ of length $n$ equals to the total number of unique prefixes of each prefix reverse, more formally:
$$
\alpha = \sum_{i=0}^{n-1} g(rev(s_i))
$$
where $g(x)$ denotes the number of distinct prefixes in string $x$.
```c++
string s; cin >> s;  
int n = s.size();  
  
auto cnt = [&](string x) {  
    vector<int> lpp = FailureFunction(x);  
    int invalid = *max_element(lpp.begin(), lpp.end());  
    return x.size() - invalid;  
};  
  
int ans = 0;  
for (int i = 0; i < n; i++) {  
    string cur = s.substr(0, i + 1);  
    reverse(cur.begin(), cur.end());  
    ans += cnt(cur);  
}  
cout << ans << '\n';
```

___

**DP with KMP**
Given a string $s$ of lowercase English letters and question marks and a string $t$ of lowercase English letters, we want to replace all the questions marks with small letters that maximizes the number of occurrences of $t$ in $s$ (occurrences of $t$ in $s$ can overlap).

To achieve this using dynamic programming, lets define a $dp$ array with two dimensions (states): index in $s$ ($i$) and index in $t$ ($k$). When replacing a questions mark with a small letter, we will utilize the KMP automaton to determine the next state of $k$ in the $dp$.
```c++
string s, t;  
cin >> s >> t;  
int n = s.size(), m = t.size();  
  
vector<vector<int>> instant = kmp_automaton(t);  
vector<vector<int>> dp(n, vector<int>(m + 1, -1));  
function<int(int, int)> solve = [&](int idx, int k) -> int {  
    if (idx == n) return 0;  
    int &ret = dp[idx][k];  
    if (~ret) return ret;  
  
    ret = 0;  
    for (char ch = 'a'; ch <= 'z'; ch++) {  
        if (s[idx] != '?' && ch != s[idx]) continue;  
        int nxt_k = instant[k][ch - 'a'];  
        ret = max(ret, solve(idx + 1, nxt_k) + (nxt_k == m));  
    }  
    return ret;  
};  
cout << solve(0, 0) << '\n';
```