Handles range queries and updates on an array in $O(log(n))$ time.

Template 1: point updates
```c++
struct Node { ll value = 0; } neutral;  
  
struct SegmentTree {  
#define mid ((ms + me) >> 1)  
#define L (2 * node)  
#define R (2 * node + 1)  
    int sz;  
    vector<Node> tree;  
  
    SegmentTree(int n, vector<ll> &arr) {  
        sz = 1;  
        while (sz < n) sz *= 2;  
        tree.resize(2 * sz, neutral);  
        sz = n;  
        build(1, 0, sz - 1, arr);  
    }  
  
    Node merge(const Node &a, const Node &b) {  
        Node ret;  
        ret.value = a.value + b.value;  
        return ret;  
    }  
  
    void build(int node, int ms, int me, vector<ll> &arr) {  
        if (ms == me) {  
            tree[node].value = arr[ms];  
            return;  
        }  
  
        build(L, ms, mid, arr);  
        build(R, mid + 1, me, arr);  
        tree[node] = merge(tree[L], tree[R]);  
    }  
  
    void Set(int idx, ll val, int node, int ms, int me) {  
        if (ms == me) {  
            tree[node].value = val;  
            return;  
        }  
  
        if (idx <= mid) Set(idx, val, L, ms, mid);  
        else Set(idx, val, R, mid + 1, me);  
        tree[node] = merge(tree[L], tree[R]);  
    }  
  
    void Set(int idx, ll val) {  
        Set(idx, val, 1, 0, sz - 1);  
    }  
  
    Node query(int l, int r, int node, int ms, int me) {  
        if (ms > r || me < l) return neutral;  
        if (ms >= l && me <= r) return tree[node];  
  
        Node one = query(l, r, L, ms, mid);  
        Node two = query(l, r, R, mid + 1, me);  
        return merge(one, two);  
    }  
  
    Node query(int l, int r) {  
        return query(l, r, 1, 0, sz - 1);  
    }  
#undef mid  
#undef L  
#undef R  
};
```

Template 2: lazy segment tree (range updates)
```c++
struct Node { ll value = 0; } neutral;  
  
struct LazySegmentTree {  
#define mid ((ms + me) >> 1)
#define L (2 * node)  
#define R (2 * node + 1)  
    int sz;  
    vector<Node> tree;  
    vector<ll> lazy;  
  
    LazySegmentTree(int n, vector<ll> &arr) {  
        sz = 1;  
        while (sz < n) sz *= 2;  
        tree.resize(2 * sz, neutral);  
        lazy.resize(2 * sz, 0);  
        sz = n;  
        build(1, 0, sz - 1, arr);  
    }  
  
    Node merge(const Node &a, const Node &b) {  
        Node ret;  
        ret.value = a.value + b.value;  
        return ret;  
    }  
  
    void build(int node, int ms, int me, vector<ll> &arr) {  
        if (ms == me) {  
            tree[node].value = arr[ms];  
            return;  
        }  
  
        build(L, ms, mid, arr);  
        build(R, mid + 1, me, arr);  
        tree[node] = merge(tree[L], tree[R]);  
    }  
  
    void propagate(int node, int ms, int me) {  
        if (!lazy[node]) return;  
  
        tree[node].value += (me - ms + 1) * lazy[node];  
        if (ms != me) {  
            lazy[L] += lazy[node];  
            lazy[R] += lazy[node];  
        }  
        lazy[node] = 0;  
    }  
  
    void update(int l, int r, ll val, int node, int ms, int me) {  
        propagate(node, ms, me);  
        if (ms > r || me < l) return;  
        if (ms >= l && me <= r) {  
            lazy[node] += val;  
            propagate(node, ms, me);  
            return;  
        }  
  
        update(l, r, val, L, ms, mid);  
        update(l, r, val, R, mid + 1, me);  
        tree[node] = merge(tree[L], tree[R]);  
    }  
  
    void update(int l, int r, ll val) {  
        update(l, r, val, 1, 0, sz - 1);  
    }  
  
    Node query(int l, int r, int node, int ms, int me) {  
        propagate(node, ms, me);  
        if (ms > r || me < l) return neutral;  
        if (ms >= l && me <= r) return tree[node];  
  
        Node one = query(l, r, L, ms, mid);  
        Node two = query(l, r, R, mid + 1, me);  
        return merge(one, two);  
    }  
  
    Node query(int l, int r) {  
        return query(l, r, 1, 0, sz - 1);  
    }  
#undef mid  
#undef L  
#undef R  
};
```

### Segment Tree tricks

**Linear Composite Functions Merging**
Given linear functions $g(x) = c_a \cdot x + d_a$ and $h(x) = c_b \cdot x + d_b$. Composite functions $g(h(x))$ and $h(g(x))$ can be calculated as follows:
$$
g(h(x)) = c_a \cdot (c_b \cdot x + d_b) + d_a = c_a c_b \cdot x + c_a d_b + d_a
$$
$$
h(g(x)) = c_b \cdot (c_a \cdot x + d_a) + d_b = c_b c_a \cdot x + c_b d_a + d_b
$$

```cpp
struct Node {
    int c = 1, d = 0, rev_d = 0;
} neutral;

Node merge(const Node &a, const Node &b) {
	Node ret;
	ret.c = mul(a.c, b.c);
	ret.d = add(mul(b.c, a.d), b.d);
	ret.rev_d = add(mul(a.c, b.rev_d), a.rev_d);
	return ret;
}
```
here, `a` denotes $g(x)$, `b` denotes $h(x)$, `merge` function denotes $h(g(x))$ and `rev_d` denotes the absolute term of $g(h(x))$.