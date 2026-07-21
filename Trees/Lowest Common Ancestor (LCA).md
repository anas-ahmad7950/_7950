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