Manages prefix-able range queries on an array.
* build: $O(n)$
* query: $O(log(n))$ 

Template 1 (Fenwick Tree): range sum + point updates
```c++
struct FenwickTree {
    int n;
    vector<ll> bit;
    FenwickTree(int _n) { n = _n; bit.resize(_n + 1); }
    FenwickTree(int _n, vector<ll> &arr) {
        n = _n;
        bit.resize(_n + 1);
        
        // build O(N)
        for (int i = 1; i <= n; i++) {
            bit[i] += arr[i - 1];
            int nxt = i + (i & -i);
            if (nxt <= n) bit[nxt] += bit[i];
        }
    }
 
    void update(int idx, ll val) {
        for (; idx <= n; idx += (idx & -idx))
            bit[idx] += val;
    }
 
    ll query(int idx) {
        ll ret = 0;
        for (; idx; idx -= (idx & -idx))
            ret += bit[idx];
        return ret;
    }
    
    ll query(int l, int r) {
	    return query(r) - query(l - 1);
    }
};
```

Template 2 (Range Fenwick Tree): range sum + range updates, **requires Template 1**
```c++
struct RangeFenwick {
    int n;
    FenwickTree f1, f2;
    RangeFenwick(int _n) : n(_n), f1(_n + 1), f2(_n + 1) {}
 
    void update(int l, int r, ll val) {
        f1.update(l, val);
        f1.update(r + 1, -val);
        f2.update(l, val * (l - 1));
        f2.update(r + 1, -val * (r));
    }
 
    ll query(int idx) {
        return f1.query(idx) * idx - f2.query(idx);
    }
 
    ll query(int l, int r) {
        return query(r) - query(l - 1);
    }
};
```