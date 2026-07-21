Template 1
```cpp
const int B = 30;  
struct XorBasis {  
    int size;  
    array<ll, B> basis;  
  
    XorBasis() {  
        size = 0;  
        for (int i = 0; i < B; i++) basis[i] = 0;  
    }  
  
    ll have(ll x, int i) {  
        return (x >> i & 1LL);  
    }  
  
    void insert(ll x) {  
        for (int i = B - 1; ~i; --i) {  
            if (!have(x, i)) continue;  
            if (!basis[i]) {  
                basis[i] = x;  
                ++size;  
                return;  
            }  
            x ^= basis[i];  
        }  
    }  
  
    ll getDistinct() {  
        return (1LL << size);  
    }  
  
    ll getMax() {  
        ll ret = 0;  
        for (int i = B - 1; ~i; --i) ret = max(ret, ret ^ basis[i]);  
        return ret;  
    }  
  
    bool can(ll x) {  
        for (int i = B - 1; ~i; --i) x = min(x, x ^ basis[i]);  
        return !x;  
    }  
  
    ll kthSmallest(ll k) { // distinct, 1-based  
        if ((1LL << size) < k) return -1;  
  
        ll ret = 0, c = size;  
        for (int i = B - 1; ~i; --i) {  
            if (!basis[i]) continue;  
            --c;  
  
            if (have(ret, i)) {  
                if ((1LL << c) >= k) ret ^= basis[i];  
                else k -= (1LL << c);  
            }  
            else if ((1LL << c) < k) {  
                ret ^= basis[i];  
                k -= (1LL << c);  
            }  
        }  
        return ret;  
    }  
  
    static XorBasis merge(const XorBasis &a, const XorBasis &b) {  
        XorBasis ret = a;  
        for (int i = B - 1; ~i; --i) ret.insert(b.basis[i]);  
        return ret;  
    }  
};
```