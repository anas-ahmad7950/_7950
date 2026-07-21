Handles range queries on a static array. 

Template 1: get and, or, min, max and gcd in constant time $O(1)$
```c++
const ll INF = 2e18;  
struct Node {  
    ll mn, mx;  
    ll GCD;  
    ll AND, OR;  
  
    Node() { mn = INF; mx = -INF; GCD = 0; AND = -1; OR = 0; }  
    Node(ll val) { mn = mx = GCD = AND = OR = val; }  
};  
  
struct SparseTable {  
    vector<vector<Node>> table;  
    vector<int> logs;  
  
    SparseTable(int n, vector<ll> &arr) {  
        table.resize(n, vector<Node>(20));  
        logs.resize(n + 1);  
  
        logs[1] = 0;  
        for (int i = 2; i <= n; i++) logs[i] = logs[i / 2] + 1;  
        build(n, arr);  
    }  
  
    Node merge(Node a, Node b) {  
        Node ret;  
        ret.mn = min(a.mn, b.mn);  
        ret.mx = max(a.mx, b.mx);  
        ret.GCD = std::gcd(a.GCD, b.GCD);  
        ret.AND = (a.AND & b.AND);  
        ret.OR = (a.OR | b.OR);  
        return ret;  
    }  
  
    void build(int n, vector<ll> &arr) {  
        for (int i = 0; i < n; i++) table[i][0] = Node(arr[i]);  
  
        for (int j = 1; j <= logs[n]; j++)  
            for (int i = 0; i + (1 << j) <= n; i++)  
                table[i][j] = merge(table[i][j - 1], table[i + (1 << (j - 1))][j - 1]);  
    }  
  
    Node query(int l, int r) {  
        int p = logs[r - l + 1];  
        return merge(table[l][p], table[r - (1 << p) + 1][p]);  
    }  
};
```