Manages range queries on an array (same as Segment Trees) plus the ability to split, merge and reverse different parts of the array in $O(log(n))$ time.

Template 1 (Without Pointers): range add + reverse
```c++
// this treap template is ZERO-BASED
#define L tree[trp].left  
#define R tree[trp].right  
const int N = 2e5 + 7, MAX_NODES = 7 * N;  
mt19937 rnd(chrono::steady_clock::now().time_since_epoch().count());  
struct TreapNode {  
    ll elem, sum, lazy = 0;  
    int left = -1, right = -1, sub, y, rev = 0;  
} tree[MAX_NODES];  
  
int ptr = 0;  
int newnode(ll val) {  
    TreapNode &ret = tree[ptr++];  
    ret.elem = val;  
    ret.sum = val;  
    ret.sub = 1;  
    ret.y = rnd();  
    return ptr - 1;  
}  
  
int subsize(int trp) {  
    return (trp == -1 ? 0 : tree[trp].sub);  
}  
  
ll getsum(int trp) {  
    return (trp == -1 ? 0 : tree[trp].sum);  
}  
  
void recalc(int trp) {  
    if (trp == -1) return;  
    tree[trp].sub = subsize(L) + subsize(R) + 1;  
    tree[trp].sum = getsum(L) + getsum(R) + tree[trp].elem;  
}  
  
void apply_lazy(int trp, ll val) {  
    if (trp == -1) return;  
    tree[trp].elem += val;  
    tree[trp].sum += val * subsize(trp);  
    tree[trp].lazy += val;  
}  
  
void apply_rev(int trp) {  
    if (trp == -1) return;  
    swap(L, R);  
    tree[trp].rev ^= 1;  
}  
  
void propagate(int trp) {  
    if (trp == -1) return;  
  
    if (tree[trp].lazy) {  
        apply_lazy(L, tree[trp].lazy);  
        apply_lazy(R, tree[trp].lazy);  
        tree[trp].lazy = 0;  
    }  
  
    if (tree[trp].rev) {  
        apply_rev(L);  
        apply_rev(R);  
        tree[trp].rev = 0;  
    }  
}  
  
array<int, 2> split(int trp, int k) {  
    propagate(trp);  
    if (trp == -1) return {-1, -1};  
  
    int cur_sz = subsize(L) + 1;  
    if (cur_sz <= k) {  
        auto p = split(R, k - cur_sz);  
        R = p[0];  
        recalc(trp);  
        return {trp, p[1]};  
    }  
    // else  
    auto p = split(L, k);  
    L = p[1];  
    recalc(trp);  
    return {p[0], trp};  
}  
  
int merge(int tl, int tr) {  
    propagate(tl); propagate(tr);  
    if (tl == -1) return tr;  
    if (tr == -1) return tl;  
    assert(~tl && ~tr);  
  
    if (tree[tl].y > tree[tr].y) {  
        tree[tl].right = merge(tree[tl].right, tr);  
        recalc(tl);  
        return tl;  
    }  
    // else  
    tree[tr].left = merge(tl, tree[tr].left);  
    recalc(tr);  
    return tr;  
}  
  
void reverse(int &trp, int l, int r) {  
    if (trp == -1) return;  
    auto [a, rem] = split(trp, l);  
    auto [b, c] = split(rem, r - l + 1);  
    apply_rev(b);  
    trp = merge(merge(a, b), c);  
}  
  
void cyclic_shift(int &trp, int l, int r, int k, bool left_cyclic_shift = false) { // right cyclic shift by default  
    k %= (r - l + 1);  
    if (trp == -1 || !k) return;  
  
    if (left_cyclic_shift) k = (r - l + 1) - k;  
  
    auto [a, rem] = split(trp, l);  
    auto [b, c] = split(rem, r - l + 1);  
    auto [x, y] = split(b, r - l + 1 - k);  
    // a y x c  
    trp = merge(merge(a, y), merge(x, c));  
}  
  
void update(int &trp, int l, int r, ll val) {  
    if (trp == -1) return;  
    auto [a, rem] = split(trp, l);  
    auto [b, c] = split(rem, r - l + 1);  
    apply_lazy(b, val);  
    trp = merge(merge(a, b), c);  
}  
  
void remove_range(int &trp, int l, int r) { // if l == r: remove index  
    if (trp == -1) return;  
    auto [a, rem] = split(trp, l);  
    auto [b, c] = split(rem, r - l + 1);  
    trp = merge(a, c);  
}  
  
void insert(int &trp, int idx, ll val) {  
    if (trp == -1) return;  
    auto [a, b] = split(trp, idx);  
    int mid = newnode(val);  
    trp = merge(merge(a, mid), b);  
}  
  
ll get(int trp, int idx) { // zero-based indexing
    assert(~trp);  
    propagate(trp);  
    if (subsize(L) == idx) return tree[trp].elem;  
    if (subsize(L) < idx) return get(R, idx - subsize(L) - 1);  
    return get(L, idx);  
}  
  
int build(vector<ll> &arr) {  
    int _n = arr.size();  
    int ret = -1;  
    for (int i = 0; i < _n; i++) {  
        int cur = newnode(arr[i]);  
        ret = merge(ret, cur);  
    }  
    return ret;  
}  
  
void print_array(int trp) {  
    propagate(trp);  
    if (trp == -1) return;  
    print_array(L);  
    cout << tree[trp].elem << ' ';  
    print_array(R);  
}
```