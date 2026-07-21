Manages range queries on an array (like normal Segment Trees) with the ability to save snapshots (versions) of the array at specific points. 
* build: $O(n)$
* point updates: $O(log(n))$
* query: $O(log(n))$

Template 1 (Arrays): range sum + point updates + get $k^{th}$ element in a range
```c++
const int N = 2e5 + 7, MX_NODES = 50 * N;
struct Node {
    int left, right;
    ll value;

    Node() : left(0), right(0), value(0) {}
    static Node merge(Node &a, Node &b) {
        Node ret;
        ret.value = a.value + b.value;
        return ret;
    }
} tree[MX_NODES], neutral;

struct PST {
#define mid ((ms + me) >> 1)
    int sz, ptr;
    vector<int> roots;

    PST(int _n) {
        sz = _n;
        ptr = 0;
        roots.push_back(build(0, sz - 1));
    }

    int newnode(ll val) {
        Node &ret = tree[ptr++];
        ret.left = ret.right = 0;
        ret.value = val;
        return ptr - 1;
    }

    int merge(int l, int r) {
        Node &ret = tree[ptr++];
        Node &a = tree[l];
        Node &b = tree[r];
        ret = Node::merge(a, b);
        ret.left = l;
        ret.right = r;
        return ptr - 1;
    }

    int build(int ms, int me) {
        if (ms == me) return newnode(0);
        return merge(build(ms, mid), build(mid + 1, me));
    }

    int update(int idx, ll val, int node, int ms, int me) {
        if (ms == me) return newnode(val);
        if (idx <= mid) return merge(update(idx, val, tree[node].left, ms, mid), tree[node].right);
        return merge(tree[node].left, update(idx, val, tree[node].right, mid + 1, me));
    }

    void update(int idx, ll val, int ver) {
        roots.push_back(update(idx, val, roots[ver], 0, sz - 1));
    }

    Node query(int l, int r, int node, int ms, int me) {
        if (ms > r || me < l) return neutral;
        if (ms >= l && me <= r) return tree[node];

        Node one = query(l, r, tree[node].left, ms, mid);
        Node two = query(l, r, tree[node].right, mid + 1, me);
        return Node::merge(one, two);
    }

    Node query(int l, int r, int ver) {
        return query(l, r, roots[ver], 0, sz - 1);
    }
    
    int find_kth(int k, int nl, int nr, int ms, int me) {
        if (ms == me) return ms;
        int left_val = tree[tree[nr].left].value - tree[tree[nl].left].value;
        if (left_val >= k) return find_kth(k, tree[nl].left, tree[nr].left, ms, mid);
        return find_kth(k - left_val, tree[nl].right, tree[nr].right, mid + 1, me);
    }

    int find_kth(int k, int verL, int verR) { // verL = version[l - 1]
        return find_kth(k, roots[verL], roots[verR], 0, sz - 1);
    }
#undef mid
};
```