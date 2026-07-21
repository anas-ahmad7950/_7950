Template 1
```cpp
const int INF = 1e9, MOD = (119 << 23) + 1; // 998244353  
template<typename T> using Function = vector<T>;  
int root = 5;  
  
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

int generator() {
    vector<int> fact;
    int phi = MOD - 1, n = phi;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            fact.push_back(i);
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) fact.push_back(n);

    for (int res = 2; res <= MOD; ++res) {
        bool ok = true;
        for (size_t i = 0; i < fact.size() && ok; i++)
            ok &= fastpower(res, phi / fact[i]) != 1;
        if (ok) return res;
    }
    return -1;
}

void NTT(Function<int> &f, bool inverse) {  
    int n = f.size();  
  
    for (int i = 0, j = 0; i < n; i++) {  
        if (i < j) swap(f[i], f[j]);  
  
        int bit = n / 2;  
        for (; bit & j; bit >>= 1) j ^= bit;  
        j ^= bit;  
    }  
  
    for (int len = 2; len <= n; len <<= 1) {  
        int step = fastpower(root, MOD / len);  
        int g1 = (inverse ? fastpower(step, MOD - 2) : step);  
  
        for (int i = 0; i < n; i += len) {  
            int g = 1;  
            for (int j = 0; j < len / 2; j++) {  
                int tmp = f[i + j];  
                f[i + j] = add(tmp, mul(g, f[i + j + len / 2]));  
                f[i + j + len / 2] = sub(tmp, mul(g, f[i + j + len / 2]));  
                g = mul(g, g1);  
            }  
        }  
    }  
}  
  
Function<int> multiply(const Function<int> &a, const Function<int> &b, int sz_lim = INF) {  
    Function<int> fa(a.begin(), a.end());  
    Function<int> fb(b.begin(), b.end());  
  
    int n = 1;  
    while (n < a.size() + b.size()) n <<= 1;  
    fa.resize(n);  
    fb.resize(n);  
  
    NTT(fa, false);  
    NTT(fb, false);  
  
    for (int i = 0; i < n; i++) fa[i] = mul(fa[i], fb[i]);  
  
    NTT(fa, true);  
  
    int sz = min(sz_lim, n), inv = fastpower(n, MOD - 2);  
    Function<int> ans(sz);  
    for (int i = 0; i < sz; i++) ans[i] = mul(fa[i], inv);
    // while (ans.size() > a.size() + b.size()) ans.pop_back();
    return ans;  
}
```

### NTT Tricks

**Horizontal Polynomial Shift by K Units**
Given a polynomial $f(x)$ of degree $d$, we can compute its horizontal translation by $k$ units in $O(n \cdot log(n))$ time using the binomial theorem.

The polynomial $f(x + k)$ represents a horizontal shift of the original function $f(x)$ by $k$ units.
$$
f(x + k) = a_0 + a_1 (x + k) + a_2 (x + k)^2 + \cdots + a_d (x + k)^d
$$
From the equation above, the value of coefficient $c$ at term $j$ can be computed using binomial theorem:
$$
c_j = \sum_{i=j}^{d} a_i \cdot \binom{i}{j} \cdot k^{i - j} = \sum_{i=j}^{d} a_i \cdot \frac{i!}{j! (i - j)!} \cdot k^{i - j} = \boxed{\frac{1}{j!} \sum_{i=j}^{d} (a_i \cdot i!) \cdot \frac{k^{i - j}}{(i - j)!}}
$$
so we need for every constant $j$, the summation of multiplying every two pairs with index difference equals to $j$.

In order to implement that, we create two polynomials $f_1(x)$ and $f_2(x)$:
$$
f_1(x) = \sum_{i=0}^{d} (a_i \cdot i!) \cdot x^i, f_2(x) = \sum_{i=0}^{d} \frac{k^i}{i!} \cdot x^{-i}
$$
then we apply polynomial multiplication to get our answer (don't forget to multiply by $(j!)^{-1}$).

```cpp
Function<int> poly_shift(Function<int> f, int k) { // f(x + k)
    if (k < 0) k += MOD;
    int n = f.size(), shift = n - 1;
    Function<int> poly1(n); // i
    for (int i = 0; i < n; i++) poly1[i] = mul(f[i], fact[i]);
 
    Function<int> poly2(n); // -z
    for (int z = 0; z < n; z++) poly2[-z + shift] = mul(fastpower(k, z), invfact[z]);
 
    Function<int> result = multiply(poly1, poly2);
    Function<int> ret(n);
    for (int i = 0; i < n; i++) ret[i] = mul(result[i + shift], invfact[i]);
    return ret;
}
```

**Raising a polynomial to a power**
We can use fast power to calculate $(f(x))^n$ in $O(n \cdot log^2(n))$. It is important not to forget to use `sz_lim` parameter.

```cpp
Function<int> poly_pow(Function<int> f, int k, int sz_lim = INF) {
    Function<int> ret = {1};
    while (k) {
        if (k & 1) ret = multiply(ret, f, sz_lim);
        f = multiply(f, f, sz_lim);
        k >>= 1;
    }
    return ret;
}
```