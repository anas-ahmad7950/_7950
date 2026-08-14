Template 1
```cpp
const int N = 1e5 + 5, LG = 19, SQ = 300;
int par[N][LG], id[2 * N], in[N], out[N];
bool vis[2 * N];
vector<int> adj[N];

struct Query {
    int l, r, i, lca;
    Query(int l, int r, int i, int lca) : l(l) , r(r), i(i), lca(lca) {}

    bool operator<(const Query &oth) const {
        if (l / SQ != oth.l / SQ) return l/SQ < oth.l / SQ;

        if (l / SQ % 2 == 0) return r < oth.r;
        return r > oth.r;
    }
};

void check(int i) {
    if (vis[i]) { // remove
        // ...
    }
    else if (!vis[i]) { // add
        // ...
    }
    vis[i] ^= 1;
}

int getAns() {
    int ans{};
    // ...
    return ans;
}

vector<int> mo(vector<Query> &v) {
    vector<int> ret(v.size());
    sort(v.begin(), v.end());
    
    int curL = 0,curR = -1;
    for (auto &[l, r, i, lc] : v) {
        while (curR < r)
            check(id[++curR]);
        while (curL > l)
            check(id[--curL]);

        while (curR > r)
            check(id[curR--]);
        while (curL < l)
            check(id[curL++]);
        
        int u = id[curL], v = id[curR];
        if (lc != u and lc != v) check(lc);
        
        ret[i] = getAns();

        if (lc != u && lc != v) check(lc);
    }
    return ret;
}

void preDFS(int node, int p) { // preDFS(1, 0)
    static int timer = 0;
    id[timer] = node;
    in[node] = timer++;
    par[node][0] = p;
    
    for (int i = 1; i < LG; i++) {
        if (par[node][i-1] == 0) break;
        par[node][i] = par[par[node][i-1]][i-1];
    }

    for (auto child : adj[node]) if (child != p) preDFS(child, node);
    
    id[timer] = node;
    out[node] = timer++;
}

bool isPar(int a,int b) { return (in[a] <= in[b] && out[a] >= out[b]); }

int lca(int a,int b) {
    if (isPar(a, b)) return a;
    if (isPar(b, a)) return b;

    for (int i = LG - 1;i >= 0;i--) {
        if (par[a][i] != 0 && !isPar(par[a][i], b)) a = par[a][i];
    }
    return par[a][0];
}

// pushing the queries into the vector<Query>
/*
for (int i = 0; i < q; i++) {
    int u, v;
    cin >> u >> v;
    int lc = lca(u, v);
    
    if (in[u] > in[v]) swap(u, v);
    
    Query query(0, 0, i, lc);
    if (lc == u) query.l = in[u], query.r = in[v];
    else query.l = out[u], query.r = in[v];

    queries.push_back(query);
}
*/
```