Divide and Conquer (D&C) on trees.

Template 1: plain centroid decomposition template.
```cpp
const int N = 1e5 + 7;
int sz[N], marked[N];
vector<int> graph[N];
vector<pair<int, int>> anc[N];

int calc_sz(int node, int par) {
    sz[node] = 1;
    for (int &ch : graph[node]) {
        if (ch == par || marked[ch]) continue;
        sz[node] += calc_sz(ch, node);
    }
    return sz[node];
}

int get_centroid(int node, int par, int comp_sz) {
    for (int &ch : graph[node]) {
        if (ch == par || marked[ch]) continue;
        if (sz[ch] * 2 > comp_sz) return get_centroid(ch, node, comp_sz);
    }
    return node;
}

void decompose(int u = 1) {
    int cent = get_centroid(u, -1, calc_sz(u, -1));

    // calculate the answer (update(nd, par, .., delta) & get_ans(nd, par, ..))

    marked[cent] = 1;
    for (int &ch : graph[cent]) if (!marked[ch]) decompose(ch);
}

void helper(int cent, int node, int par, int depth) {
    anc[node].emplace_back(cent, depth);
    for (int &ch : graph[node]) {
        if (ch == par || marked[ch]) continue;
        helper(cent, ch, node, depth + 1);
    }
}

void build(int u = 1) {
    int cent = get_centroid(u, -1, calc_sz(u, -1));
    helper(cent, cent, -1, 0);

    marked[cent] = 1;
    for (int &ch : graph[cent]) if (!marked[ch]) build(ch);
}
```

Template 2: given $u$ and $l$, it gets the number of vertices $v$ such that $d(u, v) \le l$.
```cpp
const int N = 1e5 + 7;
int sz[N], marked[N], parent[N];
vector<pair<int, ll>> graph[N];
vector<ll> me[N], up[N];

int table[N][20];
int in[N], out[N];
ll depth[N];
void lca_build(int node, int par, ll dist) {
    static int timer = 0;
    in[node] = timer++;

    depth[node] = dist;
    table[node][0] = par;
    for (int j = 1; j < 20; j++) table[node][j] = table[table[node][j - 1]][j - 1];

    for (auto &[ch, w] : graph[node]) {
        if (ch == par) continue;
        lca_build(ch, node, dist + w);
    }
    out[node] = timer++;
}
bool isParent(int u, int v) {
    return (in[u] <= in[v] && out[u] >= out[v]);
}
int lca(int u, int v) {
    if (isParent(u, v)) return u;
    for (int j = 19; ~j; j--) if (!isParent(table[u][j], v)) u = table[u][j];
    return table[u][0];
}
ll distance(int u, int v) {
    return depth[u] + depth[v] - 2 * depth[lca(u, v)];
}

int calc_sz(int node, int par) {
    sz[node] = 1;
    for (auto &[ch, w] : graph[node]) {
        if (ch == par || marked[ch]) continue;
        sz[node] += calc_sz(ch, node);
    }
    return sz[node];
}

int get_centroid(int node, int par, int comp_sz) {
    for (auto &[ch, w] : graph[node]) {
        if (ch == par || marked[ch]) continue;
        if (sz[ch] * 2 > comp_sz) return get_centroid(ch, node, comp_sz);
    }
    return node;
}

void fill(int node, int par, ll dist, vector<ll> &v) {
    v.push_back(dist);
    for (auto &[ch, w] : graph[node]) {
        if (ch == par || marked[ch]) continue;
        fill(ch, node, dist + w, v);
    }
}

void build(int u = 1, int c_par = -1) {
    int comp_sz = calc_sz(u, -1);
    int cent = get_centroid(u, -1, comp_sz);

    vector<ll> v;
    fill(cent, -1, 0, v);
    sort(v.begin(), v.end());
    me[cent] = v;

    parent[cent] = c_par;

    marked[cent] = 1;
    for (auto &[ch, w] : graph[cent]) {
        if (marked[ch]) continue;

        vector<ll> chv;
        fill(ch, cent, w, chv);
        sort(chv.begin(), chv.end());
        up[get_centroid(ch, -1, calc_sz(ch, -1))] = chv;

        build(ch, cent);
    }
}

ll query(int node, ll l) {
    int ret = 0, lst_nd = -1;

    int cent = node;
    while (~cent) {
        ll d = distance(cent, node);
        ret += upper_bound(me[cent].begin(), me[cent].end(), l - d) - me[cent].begin();
        if (~lst_nd) ret -= upper_bound(up[lst_nd].begin(), up[lst_nd].end(), l - d) - up[lst_nd].begin();

        lst_nd = cent;
        cent = parent[cent];
    }
    return ret;
}
```