Given a rooted tree and two nodes $u$ and $v$, it finds the lowest node $lc$ that is considered a parent to both $u$ and $v$.

Template 1: pure LCA.
```c++
struct LCA {  
    vector<vector<int>> table;  
    vector<int> in, out;  
  
    LCA(int n, vector<vector<int>> &graph) {  
        table.resize(n + 1, vector<int>(20));  
        in.resize(n + 1);  
        out.resize(n + 1);  
        DFS(1, 1, graph);  
    }  
  
    void DFS(int node, int parent, vector<vector<int>> &graph) {  
        static int timer = 0;  
        in[node] = timer++;  
  
        table[node][0] = parent;  
        for (int j = 1; j < 20; j++) table[node][j] = table[table[node][j - 1]][j - 1];  
  
        for (int &child : graph[node]) {  
            if (child == parent) continue;  
            DFS(child, node, graph);  
        }  
        out[node] = timer++;  
    }  
  
    bool isParent(int u, int v) { return (in[u] <= in[v] && out[u] >= out[v]); }  
  
    int lca(int u, int v) {  
        if (isParent(u, v)) return u;  
        for (int j = 19; ~j; j--) if (!isParent(table[u][j], v)) u = table[u][j];  
        return table[u][0];  
    }  
};
```

Template 2: get the sum, and, or, min, max and gcd on the simple path between nodes $u$ and $v$
```c++
const ll INF = 2e18;  
struct Node {  
    ll mn, mx;  
    ll GCD;  
    ll AND, OR;  
    ll sum;  
  
    Node() { mn = INF; mx = -INF; GCD = 0; AND = -1; OR = 0; sum = 0; }  
    Node(ll val) { mn = mx = GCD = AND = OR = sum = val; }  
  
    static Node merge(const Node &a, const Node &b) {  
        Node ret;  
        ret.mn = min(a.mn, b.mn);  
        ret.mx = max(a.mx, b.mx);  
        ret.GCD = std::gcd(a.GCD, b.GCD);  
        ret.AND = (a.AND & b.AND);  
        ret.OR = (a.OR | b.OR);  
        ret.sum = a.sum + b.sum;  
        return ret;  
    }  
};  
  
struct LCA {  
    vector<vector<int>> table;  
    vector<vector<Node>> values;  
    vector<int> in, out;  
  
    LCA(int n, vector<vector<pair<int, ll>>> &graph) {  
        table.resize(n + 1, vector<int>(20));  
        values.resize(n + 1, vector<Node>(20));  
        in.resize(n + 1);  
        out.resize(n + 1);  
        DFS(1, 1, 0, graph);  
    }  
  
    void DFS(int node, int parent, ll weight, vector<vector<pair<int, ll>>> &graph) {  
        static int timer = 0;  
        in[node] = timer++;  
  
        table[node][0] = parent;  
        values[node][0] = Node(weight);  
  
        for (int j = 1; j < 20; j++) {  
            int p = table[node][j - 1];  
            table[node][j] = table[p][j - 1];  
            values[node][j] = Node::merge(values[node][j - 1], values[p][j - 1]);  
        }  
  
        for (auto &[child, w] : graph[node]) {  
            if (child == parent) continue;  
            DFS(child, node, w, graph);  
        }  
        out[node] = timer++;  
    }  
  
    bool isParent(int u, int v) { return (in[u] <= in[v] && out[u] >= out[v]); }  
  
    int lca(int u, int v) {  
        if (isParent(u, v)) return u;  
        for (int j = 19; ~j; j--) if (!isParent(table[u][j], v)) u = table[u][j];  
        return table[u][0];  
    }  
  
    Node helper(int lc, int v) {  
        if (lc == v) return Node();  
  
        Node ret = Node();  
        for (int j = 19; ~j; j--) {  
            if (!isParent(table[v][j], lc)) {  
                ret = Node::merge(ret, values[v][j]);  
                v = table[v][j];  
            }  
        }  
        return Node::merge(ret, values[v][0]);  
    }  
  
    Node query(int u, int v) {  
        int lc = lca(u, v);  
        return Node::merge(helper(lc, u), helper(lc, v));  
    }  
};
```

Template 3: LCA + Hashing (characters on nodes)
```cpp
struct LCA {
    vector<vector<int>> &graph;
    string &str;
    int n, timer;
    vector<vector<int>> table;
    vector<vector<HashValue>> up, down;
    vector<int> in, out, depth;

    LCA(int _n, vector<vector<int>> &g, string &s) : graph(g), str(s) {
        n = _n, timer = 0;

        table.resize(n + 1, vector<int>(20));
        up.resize(n + 1, vector<HashValue>(20));
        down.resize(n + 1, vector<HashValue>(20));

        in.resize(n + 1);
        out.resize(n + 1);
        depth.resize(n + 1);

        DFS(1, 1);
    }

    void DFS(int node, int par) {
        in[node] = timer++;

        table[node][0] = par;
        up[node][0] = down[node][0] = HashValue::create(str[node]);
        for (int j = 1; j < 20; ++j) {
            int p = table[node][j - 1];

            table[node][j] = table[p][j - 1];
            up[node][j] = up[node][j - 1] + up[p][j - 1];
            down[node][j] = down[node][j - 1] / down[p][j - 1];
        }

        for (const int &ch : graph[node]) {
            if (ch == par) continue;
            depth[ch] = depth[node] + 1;
            DFS(ch, node);
        }
        out[node] = timer++;
    }

    bool isParent(int u, int v) {
        return (in[u] <= in[v] && out[u] >= out[v]);
    }

    int lca(int u, int v) {
        if (isParent(u, v)) return u;
        for (int j = 19; ~j; --j) if (!isParent(table[u][j], v)) u = table[u][j];
        return table[u][0];
    }

    int kthAncestor(int u, int k) {
        for (int j = 19; ~j; --j) if (k >> j & 1) u = table[u][j];
        return u;
    }

    int dist(int u, int v) {
        return depth[u] + depth[v] - 2 * depth[lca(u, v)];
    }

    int kthOnPath(int u, int v, int k) {
        int lc = lca(u, v);
        if (k <= dist(u, lc)) return kthAncestor(u, k);
        // else
        return kthAncestor(v, dist(v, lc) - k + dist(u, lc));
    }

    HashValue helper(int u, int lc, bool dir) {
        HashValue ret;
        if (u == lc) return ret;
        for (int j = 19; ~j; --j) {
            if (isParent(table[u][j], lc)) continue;
            if (dir) ret = ret + up[u][j];
            else ret = ret / down[u][j];
            u = table[u][j];
        }

        if (dir) ret = ret + up[u][0];
        else ret = ret / down[u][0];
        return ret;
    }

    HashValue prefixHash(int u, int v, int idx) {
        int x = kthOnPath(u, v, idx);
        if (isParent(x, u)) return helper(u, x, true) + HashValue::create(str[x]);
        // else
        int lc = lca(u, v);
        return helper(u, lc, true) + HashValue::create(str[lc]) + helper(x, lc, false);
    }

    int compare(int a, int b, int c, int d) {
        int d1 = dist(a, b);
        int d2 = dist(c, d);

        int low = 0, high = min(d1, d2), idx = -1;
        while (low <= high) {
            int mid = (low + high) / 2;

            HashValue h1 = prefixHash(a, b, mid);
            HashValue h2 = prefixHash(c, d, mid);
            if (h1 == h2) {
                low = mid + 1;
            }
            else {
                idx = mid;
                high = mid - 1;
            }
        }

        if (idx == -1) {
            if (d1 == d2) return 0;
            return (d1 > d2 ? 1 : 2);
        }
        int x = kthOnPath(a, b, idx);
        int y = kthOnPath(c, d, idx);
        return (str[x] > str[y] ? 1 : 2);
    }
};
```

Template 4: LCA + Hashing (characters on edges)
```cpp
struct LCA {
    vector<vector<pair<int, char>>> &graph;
    int n, timer;
    vector<vector<int>> table;
    vector<vector<HashValue>> up, down;
    vector<int> in, out, depth;
    vector<char> s;

    LCA(int _n, vector<vector<pair<int, char>>> &g) : graph(g) {
        n = _n, timer = 0;

        table.resize(n + 1, vector<int>(20));
        up.resize(n + 1, vector<HashValue>(20));
        down.resize(n + 1, vector<HashValue>(20));

        in.resize(n + 1);
        out.resize(n + 1);
        depth.resize(n + 1);
        s.resize(n + 1);

        DFS(1, 1, 0);
    }

    void DFS(int node, int par, char c) {
        in[node] = timer++;
        s[node] = c;

        table[node][0] = par;
        if (node != 1) up[node][0] = down[node][0] = HashValue::create(c);
        for (int j = 1; j < 20; ++j) {
            int p = table[node][j - 1];

            table[node][j] = table[p][j - 1];
            up[node][j] = up[node][j - 1] + up[p][j - 1];
            down[node][j] = down[node][j - 1] / down[p][j - 1];
        }

        for (const auto &[ch, w] : graph[node]) {
            if (ch == par) continue;
            depth[ch] = depth[node] + 1;
            DFS(ch, node, w);
        }
        out[node] = timer++;
    }

    bool isParent(int u, int v) {
        return (in[u] <= in[v] && out[u] >= out[v]);
    }

    int lca(int u, int v) {
        if (isParent(u, v)) return u;
        for (int j = 19; ~j; --j) if (!isParent(table[u][j], v)) u = table[u][j];
        return table[u][0];
    }

    int kthAncestor(int u, int k) {
        for (int j = 19; ~j; --j) if (k >> j & 1) u = table[u][j];
        return u;
    }

    int dist(int u, int v) {
        return depth[u] + depth[v] - 2 * depth[lca(u, v)];
    }

    int kthOnPath(int u, int v, int k) {
        int lc = lca(u, v);
        if (k <= dist(u, lc)) return kthAncestor(u, k);
        // else
        return kthAncestor(v, dist(v, lc) - k + dist(u, lc));
    }

    HashValue helper(int u, int lc, bool dir) {
        HashValue ret;
        if (u == lc) return ret;
        for (int j = 19; ~j; --j) {
            if (isParent(table[u][j], lc)) continue;
            if (dir) ret = ret + up[u][j];
            else ret = ret / down[u][j];
            u = table[u][j];
        }

        if (dir) ret = ret + up[u][0];
        else ret = ret / down[u][0];
        return ret;
    }

    HashValue prefixHash(int u, int v, int idx) {
        if (!idx) return {};

        int x = kthOnPath(u, v, idx);
        if (isParent(x, u)) return helper(u, x, true);
        // else
        int lc = lca(u, v);
        return helper(u, lc, true) + helper(x, lc, false);
    }

    char getChar(int u, int v, int k) {
        int a = kthOnPath(u, v, k - 1);
        int b = kthOnPath(u, v, k);
        return (depth[a] > depth[b] ? s[a] : s[b]);
    }

    int compare(int a, int b, int c, int d) {
        int d1 = dist(a, b);
        int d2 = dist(c, d);

        int low = 1, high = min(d1, d2), idx = -1;
        while (low <= high) {
            int mid = (low + high) / 2;

            HashValue h1 = prefixHash(a, b, mid);
            HashValue h2 = prefixHash(c, d, mid);
            if (h1 == h2) {
                low = mid + 1;
            }
            else {
                idx = mid;
                high = mid - 1;
            }
        }

        if (idx == -1) {
            if (d1 == d2) return 0;
            return (d1 > d2 ? 1 : 2);
        }
        char cx = getChar(a, b, idx);
        char cy = getChar(c, d, idx);
        return (cx > cy ? 1 : 2);
    }
};
```