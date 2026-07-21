AND, OR and XOR convolutions in $O(n \cdot log(n))$.

Template 1
```cpp
template<typename T> using Function = vector<T>;  
enum Conv { AND = 0, OR = 1, XOR = 2 };
   
void FWHT(Function<__int128> &f, bool inverse, int type) {  
    int n = f.size();  
   
    for (int len = 2; len <= n; len <<= 1) {  
        for (int i = 0; i < n; i += len) {  
            for (int j = 0; j < len / 2; j++) {  
                __int128 u = f[i + j];  
                __int128 v = f[i + j + len / 2];  
   
                if (type == AND) {  
                    if (inverse) f[i + j] = u - v;
                    else f[i + j] = u + v;  
                }  
                else if (type == OR) {  
                    if (inverse) f[i + j + len / 2] = v - u;  
                    else f[i + j + len / 2] = v + u;  
                }  
                else if (type == XOR) {  
                    f[i + j] = u + v;  
                    f[i + j + len / 2] = u - v;  
                }  
            }  
        }  
    }  
}  
   
Function<ll> multiply(const Function<ll> &a, const Function<ll> &b, int type) {  
    Function<__int128> fa(a.begin(), a.end());  
    Function<__int128> fb(b.begin(), b.end());  
   
    int n = 1;  
    while (n < max(a.size(), b.size())) n <<= 1;  
    fa.resize(n);  
    fb.resize(n);  
   
    FWHT(fa, false, type);  
    FWHT(fb, false, type);  
   
    for (int i = 0; i < n; i++) fa[i] *= fb[i];  
   
    FWHT(fa, true, type);  
   
    Function<ll> ans(n);  
    for (int i = 0; i < n; i++) ans[i] = (type == XOR ? fa[i] / n : fa[i]);  
    return ans;  
}
```
Note: in some cases we must use `__int128` to avoid overflows as the maximum value at any arbitrary index during forward FWHT can reach $(\sum a_i)^x \cdot n$.

### FWHT Tricks

**Frequency Domain**
A frequency domain of a polynomial $f$ of degree $d$ is a set $d + 1$ distinct points $(x, y)$ where $f(x) = y$.

It is very important to understand that Inverse FFT/NTT/FWHT are Linear Operations, so if I want to calculate $\sum_{i=1}^{n} (f(x))^i$, we will apply this equation to each value $y$ in the frequency domain of $f$ then applying IFWHT.
$$
\sum_{i=1}^{k} y^i = \frac{y \cdot (y^k - 1)}{y - 1}
$$
Note: this trick is not very helpful in FFT/NTT because, for example, if I want to raise a polynomial $f(x)$ to a power $n$, then the size of $f_a$ will equal to $n \cdot |f|$, so if $n, |f| \le 2 \cdot 10^5$, $f_a$ will become very large and we can't limit this size because it is needed in the FFT/NTT calculations.

AtCoder Problem: https://atcoder.jp/contests/abc212/tasks/abc212_h
```cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
#define ld long double
#define imie(...) " [" << #__VA_ARGS__ << " = " << (__VA_ARGS__) << "] "
#define display(x) cout<<#x<<": ";for(auto itr:x)cout<<itr<<' ';cout<<endl;
#define HERE cout << "HERE" << endl;
template<typename T1, typename T2>
ostream &operator<<(ostream &os, pair<T1, T2> &p) {
    return os << "{" << p.first << ", " << p.second << "}";
}

int n, k;
template<typename T> using Function = vector<T>;
const int MX = 1 << 17, MOD = (119 << 23) + 1;
int add(ll a, int b) { return (a + b) % MOD; }
int sub(ll a, int b) { return (a - b + MOD) % MOD; }
int mul(ll a, int b) { return (a * b) % MOD; }
int fastpower(int base, int power) {
    int ret = 1;
    while (power) {
        if (power & 1) ret = mul(ret, base);
        base = mul(base, base);
        power >>= 1;
    }
    return ret;
}

void FWHT(Function<int> &f, bool inverse) {
    int n = f.size();

    for (int len = 2; len <= n; len <<= 1) {
        for (int i = 0; i < n; i += len) {
            for (int j = 0; j < len / 2; j++) {
                int u = f[i + j];
                int v = f[i + j + len / 2];

                f[i + j] = add(u, v);
                f[i + j + len / 2] = sub(u, v);
            }
        }
    }
}

int calc(int x) {
    if (x == 0) return 0;
    if (x == 1) return n; // don't forget to handle these edge cases
    int up = mul(x, sub(fastpower(x, n), 1));
    int down = fastpower(sub(x, 1), MOD - 2);
    return mul(up, down);
}

Function<int> apply_fhwt_xor(const Function<int> &a) {
    Function<int> fa(a.begin(), a.end());

    int n = 1;
    while (n < a.size()) n <<= 1;
    fa.resize(n);

    FWHT(fa, false);
    for (int i = 0; i < n; i++) fa[i] = calc(fa[i]);
    FWHT(fa, true);

    int inv = fastpower(n, MOD - 2);
    Function<int> ans(n);
    for (int i = 0; i < n; i++) ans[i] = mul(fa[i], inv);
    return ans;
}

void test_case() {
    cin >> n >> k;

    vector<int> arr(k);
    for (int &itr : arr) cin >> itr;

    Function<int> f(MX);
    for (int i = 0; i < k; i++) f[arr[i]]++;

    Function<int> result = apply_fhwt_xor(f);

    int total = 0;
    for (int i = 1; i <= n; i++) total = add(total, fastpower(k, i));
    cout << sub(total, result[0]) << '\n';
}

int32_t main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);

    int tst = 1;
    // cin >> tst;
    while (tst--) { test_case(); }
    return 0;
}
```