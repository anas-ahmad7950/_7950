Used to multiply two polynomials $A$ and $B$ in $O(n \cdot log(n))$.

Template 1
```cpp
const int INF = 1e9;   
const double PI = acos(-1);  
using cd = complex<double>;  
template<typename T> using Function = vector<T>;  
  
void FFT(Function<cd> &f, bool inverse) {  
    int n = f.size();  
  
    for (int i = 0, j = 0; i < n; i++) {  
        if (i < j) swap(f[i], f[j]);  
  
        int bit = n / 2;  
        for (; bit & j; bit >>= 1) j ^= bit;  
        j ^= bit;  
    }  
  
    for (int len = 2; len <= n; len <<= 1) {  
        double angle = 2 * PI / len;  
        if (inverse) angle *= -1.0;  
        cd w1(cos(angle), sin(angle));  
  
        for (int i = 0; i < n; i += len) {  
            cd w(1, 0);  
            for (int j = 0; j < len / 2; j++) {  
                cd u = f[i + j], v = w * f[i + j + len / 2];  
                f[i + j] = u + v;  
                f[i + j + len / 2] = u - v;  
                if (inverse) f[i + j] /= 2, f[i + j + len / 2] /= 2;  
                w *= w1;  
            }  
        }  
    }  
}  
  
Function<ll> multiply(const Function<ll> &a, const Function<ll> &b, int sz_lim = INF) {  
    Function<cd> fa(a.begin(), a.end());  
    Function<cd> fb(b.begin(), b.end());  
  
    int n = 1;  
    while (n < a.size() + b.size()) n <<= 1;  
    fa.resize(n);  
    fb.resize(n);  
  
    FFT(fa, false);  
    FFT(fb, false);  
  
    for (int i = 0; i < n; i++) fa[i] *= fb[i];  
  
    FFT(fa, true);  
  
    int sz = min(n, sz_lim);  
    Function<ll> ans(sz);
    for (int i = 0; i < sz; i++) ans[i] = round(fa[i].real() /* / n */);
    // while (ans.size() > 1 && !ans.back()) ans.pop_back();
    // while (ans.size() > a.size() + b.size()) ans.pop_back();
    return ans;  
}
```

Template 2: **FFT with Arbitrary MOD (MTT)**
```cpp
const int INF = 1e9;  
const double PI = acos(-1);  
using cd = complex<ld>;  
template<typename T> using Function = vector<T>;  
   
void FFT(Function<cd> &f, bool inverse) {  
    int n = f.size();  
   
    for (int i = 0, j = 0; i < n; i++) {  
        if (i < j) swap(f[i], f[j]);  
   
        int bit = n / 2;  
        for (; bit & j; bit >>= 1) j ^= bit;  
        j ^= bit;  
    }  
   
    for (int len = 2; len <= n; len <<= 1) {  
        double angle = 2 * PI / len;  
        if (inverse) angle *= -1.0;  
        cd w1(cos(angle), sin(angle));  
   
        for (int i = 0; i < n; i += len) {  
            cd w(cos(0), sin(0));  
            for (int j = 0; j < len / 2; j++) {  
                cd tmp = f[i + j];  
                f[i + j] = tmp + w * f[i + j + len / 2];  
                f[i + j + len / 2] = tmp - w * f[i + j + len / 2];  
                if (inverse) f[i + j] /= 2, f[i + j + len / 2] /= 2;  
                w *= w1;  
            }  
        }  
    }  
}  
   
Function<ll> multiplyMOD(const Function<ll> &a, const Function<ll> &b, ll MOD, int sz_lim = INF) {  
    int n = 1;  
    while (n < a.size() + b.size()) n <<= 1;  
   
    ll cut = sqrt(MOD);  
    Function<cd> fa(n), fb(n);  
    for (int i = 0; i < a.size(); i++) fa[i] = cd(a[i] / cut, a[i] % cut);  
    for (int i = 0; i < b.size(); i++) fb[i] = cd(b[i] / cut, b[i] % cut);  
   
    FFT(fa, false);  
    FFT(fb, false);  
   
    Function<cd> outl(n), outs(n);  
    for (int i = 0; i < n; i++) {  
        int j = -i & (n - 1);  
   
        cd a1 = (fa[i] + conj(fa[j])) * cd(0.5, 0);  
        cd a0 = (fa[i] - conj(fa[j])) * cd(0, -0.5);  
   
        outl[i] = a1 * fb[i];  
        outs[i] = a0 * fb[i];  
    }  
   
    FFT(outl, true);  
    FFT(outs, true);  
   
    int sz = min(n, sz_lim);  
    Function<ll> ans(sz);  
    for (int i = 0; i < sz; i++) {  
        ll av = round(outl[i].real());  
        ll cv = round(outs[i].imag());  
   
        ll bv = round(outl[i].imag()) + round(outs[i].real());  
   
        av %= MOD;  
        bv %= MOD;  
        cv %= MOD;  
   
        ans[i] = ((av * cut % MOD * cut) % MOD + (bv * cut) % MOD + cv) % MOD;  
        if (ans[i] < 0) ans[i] += MOD;  
    }  
   
    while (ans.size() > a.size() + b.size()) ans.pop_back();  
    return ans;  
}
```
To understand the implementation details of the above template, lets discuss "FFT for two polynomials simultaneously". Let $A(x)$ and $B(x)$ be polynomials with "real" coefficients. Instead of applying FFT on polynomials $A$ and $B$ separately, we combine them into a single complex polynomial $P(x)$ where:
$$
P(x) = A(x) + iB(x)
$$
then we apply FFT on $P(x)$, giving us an array of values: $P(w_n^0), P(w_n^1), \dots, P(w_n^{n-1})$, where $n$ is a power of $2$ and $w_n^k = e^{2 \pi k i/n}$.

To extract the values of $A(w_n^k)$ and $B(w_n^k)$, we will use some properties of complex conjugates:
* $w_n^{n - k}$ is the complex conjugate of $w_n^k$ (i.e. $\overline{w_n^k} = w_n^{n - k}$, reflection on the real axis).
* If $F$ is a polynomial with "purely" real coefficients, then $F(\overline{w_n^k}) = \overline{F(w_n^k)}$.

Lets look at the conjugate of $P(w_n^{n - k})$:
$$
\overline{P(w_n^{n - k})} = \overline{A(w_n^{n - k}) + iB(w_n^{n - k})}
$$
by applying the above properties to the equation:
$$
\overline{P(w_n^{n - k})} = A(w_n^k) - iB(w_n^k)
$$
Note: $P(\overline{w_n^k}) \ne \overline{P(w_n^k)}$ because the coefficients of $P$ are complex (not purely real).

Now we have two equations for any given $k$:
Equation 1: $P(w_n^k) = A(w_n^k) + iB(w_n^k)$
Equation 2: $\overline{P(w_n^{n - k})} = A(w_n^k) - iB(w_n^k)$

By adding the two equations together, the $iB(w_n^k)$ terms cancel out:
$$
A(w_n^k) = \frac{P(w_n^k) + \overline{P(w_n^{n - k})}}{2}
$$

By subtracting the Equation 2 from Equation 1, the $A(w_n^k)$ terms cancel out:
$$
B(w_n^k) = \frac{P(w_n^k) - \overline{P(w_n^{n - k})}}{2i}
$$

Note: by applying Inverse FFT to $P(w_n^k)$, we will be able to get our final coefficients, as the coefficients of $A$ will be the real part of the IFFT result, and the coefficients of $B$ will be the imaginary part of the IFFT result.

___

FFT with mod splits each coefficients into two smaller parts using a constant equals to $\sqrt{MOD}$, which implies that the maximum coefficient in any of the smaller polynomials will not exceed $(\sqrt{10^9})^2 \cdot 10^6 \approx 10^{15}$. lets apply this concept on the two polynomials $A(x)$ and $B(x)$:
* $A(x) = A_1(x) \cdot \text{cut} + A_0(x)$
* $B(x) = B_1(x) \cdot \text{cut} + B_0(x)$

If we multiply polynomials $A$ and $B$ together, the final output will be:
$A \cdot B = (A_1 B_1) \cdot \text{cut}^2 + (A_1 B_0 + A_0 B_1) \cdot \text{cut} + (A_0 B_0)$
Naively, we will need $4$ separate polynomial multiplications (with a lot of FFT calls) before merging their values again under our $\text{MOD}$ space. The above MTT template avoids that by using FFT for two polynomials simultaneously:

First, we create two complex polynomials:
* $f_a(x) = A_1(x) + iA_0(x)$
* $f_b(x) = B_1(x) + iA_0(x)$
then, we apply FFT on both $f_a(x)$ and $f_b(x)$.

Second, we extract the values of $A_1(w_n^k)$ and $A_0(w_n^k)$ by the equations discussed before. Then, we multiply the extracted $A_1(w_n^k)$ part by $f_b(w_n^k)$:
$$
\text{outl} = A_1(w_n^k) \cdot f_b(w_n^k) = A_1(w_n^k) \cdot (B_1(w_n^k) + iB_0(w_n^k)) = A_1(w_n^k) B_1(w_n^k) + iA_1(w_n^k) B_0(w_n^k)
$$
the same applies to $A_0(w_n^k)$:
$$
\text{outs} = A_0(w_n^k) \cdot f_b(w_n^k) = A_0(w_n^k) \cdot (B_1(w_n^k) + iB_0(w_n^k)) = A_0(w_n^k) B_1(w_n^k) + iA_0(w_n^k) B_0(w_n^k)
$$

Finally, be applying IFFT on $\text{outl}$ and $\text{outs}$, we will be able to extract our coefficients and combine them under the $\text{MOD}$ space.
$$
\text{IFFT(outl)} = A_1(x) B_1(x) + iA_1(x) B_0(x)
$$
$$
\text{IFFT(outs)} = A_0(x) B_1(x) + iA_0(x) B_0(x)
$$
Note: `-i & (n - 1)` is equivalent to `(n - i) % n` when $n$ is a power of $2$. The reason we need mod $n$ because $\overline{w_n^0} = w_n^n$ (index $n$ doesn't exist).

### FFT Tricks

**Multiplying a Sequence of Polynomials**
Given $n$ polynomials each of degree $d_i$, and $\sum_{i=1}^{n} d_i \le 2 \cdot 10^5$. We can multiply all of them in $O(N \cdot log(N) \cdot log(n))$, where $N = 2 \cdot 10^5$.

To achieve this, we create a priority queue of pairs $(sz_i, i)$, and select the smallest two polynomials based on $sz_i$, multiply them, and finally store the result at $i$.

This is valid because the time complexity of multiplying polynomials of sizes $A$ and $B$ together is $O(S_i \cdot log(S_i))$, where $S_i = A + B$. So the total time complexity is:
$$
O = \sum_{\text{merge i}} S_i \cdot log(S_i) = log(N) \sum_{\text{merge i}} S_i
$$
since the maximum possible size of any merged polynomial is $N$, therefore, $log(S_i) \le log(N)$.

By choosing the smallest two polynomials every time to multiply them, the maximum value of $\sum_{i} S_i = N \cdot log(n)$, because the priority queue will create a balanced binary tree with height of $log(n)$ and at every single level of this tree, every polynomial is involved at most once, therefore the sum of sizes at each level is $N$ (very similar concept to small-to-large).

```cpp
int n;
cin >> n;

vector<Function<int>> poly(n);
priority_queue<pair<int, int>> pq; // {szi, i}
for (int i = 0; i < n; i++) {
	int deg; cin >> deg;
	Function<int> cur_f(deg + 1);
	for (int j = 0; j <= deg; j++) cin >> cur_f[j];
	poly[i] = cur_f;
	pq.emplace(-deg, i);
}

while (pq.size() > 1) {
	auto [d1, i1] = pq.top();
	pq.pop();

	auto [d2, i2] = pq.top();
	pq.pop();

	Function<int> result = multiply(poly[i1], poly[i2]);
	poly[i1] = result;
	pq.emplace(d1 + d2, i1);
}

auto [d, i] = pq.top();
for (int c = 0; c <= -1 * d; c++) cout << poly[i][c] << ' ';
cout << '\n';
```
Note: don't forget to remove the extra padded zeros from the result of each polynomial multiplication operation.