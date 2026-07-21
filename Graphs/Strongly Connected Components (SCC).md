Template 1 (undirected graphs, not necessarily connected): 2-ECC, BCC and BCT
```cpp
struct SCC {
    vector<vector<int>> &graph;
    vector<int> dfs_num, low_link;
    vector<bool> in_stack, art;
    stack<int> st;
    int timer, n, C, A;
    // C = #connected components
    // A = #articulation points

    // Bridge Connected Components (2-ECC)
    vector<int> ecc_id;
    int ecc_nxt;
    // #bridges = ecc_nxt - 1 - C

    // Bi-Connected Components (BCC)
    vector<vector<int>> bcc;
    vector<int> bct_id;
    stack<int> bct_st;
    int bcc_nxt;

    SCC(int _n, vector<vector<int>> &g) : graph(g) {
        n = _n;
        timer = 1;
        A = C = 0;
        dfs_num = low_link = vector<int>(n + 1);
        in_stack = art = vector<bool>(n + 1);

        // 2 Edge Connected Components (2-ECC)
        ecc_id = vector<int>(n + 1);
        ecc_nxt = 1;

        // Bi-Connected Components (BCC)
        bct_id = vector<int>(n + 1);
        bcc_nxt = 1;

        for (int i = 1; i <= n; ++i)
            if (!dfs_num[i])
                tarjan(i, i), ++C;
    }

    void tarjan(int node, int par) {
        dfs_num[node] = low_link[node] = timer++;
        st.push(node);
        in_stack[node] = true;

        bct_st.push(node);

        int c_cnt = 0;
        for (int &ch : graph[node]) {
            if (ch == par) continue;

            if (!dfs_num[ch]) {
                tarjan(ch, node);
                low_link[node] = min(low_link[node], low_link[ch]);

				// if (low_link[ch] > dfs_num[node]), then edge(ch, node) is a bridge
                if (low_link[ch] >= dfs_num[node]) {
                    if (node ^ par) art[node] = true;
                    
                    bcc.push_back({node});
                    while (bcc.back().back() != ch)
                        bcc.back().push_back(bct_st.top()), bct_st.pop();
                }
                ++c_cnt;
            }
            else if (in_stack[ch]) {
                low_link[node] = min(low_link[node], dfs_num[ch]);
            }
        }
        if (c_cnt > 1 && node == par) art[node] = true;
        A += art[node];

        if (dfs_num[node] == low_link[node]) {
            while (true) {
                int x = st.top();
                st.pop();
                in_stack[x] = false;

                ecc_id[x] = ecc_nxt;

                if (x == node) break;
            }
            ++ecc_nxt;
        }
    }

    vector<vector<int>> twoECC() {
        vector<vector<int>> ret(ecc_nxt);
        for (int node = 1; node <= n; ++node) {
            for (int &ch : graph[node]) {
                if (node < ch) continue;

                int u = ecc_id[ch];
                int v = ecc_id[node];
                if (u != v) {
                    ret[u].push_back(v);
                    ret[v].push_back(u);
                }
            }
        }
        return ret;
    }
    
    vector<vector<int>> BCT() {
        vector<vector<int>> ret(2 * n + 1);

        for (int node = 1; node <= n; ++node)
            if (art[node])
                bct_id[node] = bcc_nxt++;

        for (const vector<int> &comp : bcc) {
            for (const int &node : comp) {
                if (art[node]) {
                    int u = bct_id[node];
                    int v = bcc_nxt;

                    ret[u].push_back(v);
                    ret[v].push_back(u);
                }
                else bct_id[node] = bcc_nxt;
            }
            ++bcc_nxt;
        }

        return ret;
    }
};
```

Template 2 (directed graphs)
```cpp
struct SCC {
    vector<vector<int>> &graph;
    vector<int> dfs_num, low_link;
    stack<int> st;
    vector<bool> in_stack;
    int n, timer, C;
    // C = #connected components (NOT STRONGLY CONNECTED COMPONENTS)

    vector<vector<int>> scc;

    // Condensation Graph => DAG
    vector<int> id;
    int cur_id;

    SCC(int _n, vector<vector<int>> &g) : graph(g) {
        n = _n;
        timer = 1;
        C = 0;
        dfs_num = low_link = vector<int>(n + 1);
        in_stack = vector<bool>(n + 1);

        // Condensation Graph
        id = vector<int>(n + 1);
        cur_id = 1;

        for (int i = 1; i <= n; ++i)
            if (!dfs_num[i])
                tarjan(i), ++C;
    }

    void tarjan(int node) {
        dfs_num[node] = low_link[node] = timer++;
        st.push(node);
        in_stack[node] = true;

        for (int &ch : graph[node]) {
            if (!dfs_num[ch]) {
                tarjan(ch);
                low_link[node] = min(low_link[node], low_link[ch]);
            }
            else if (in_stack[ch]) {
                low_link[node] = min(low_link[node], dfs_num[ch]);
            }
        }

        if (dfs_num[node] == low_link[node]) {
            scc.emplace_back();
            while (true) {
                int x = st.top();
                st.pop();
                in_stack[x] = false;

                id[x] = cur_id;
                scc.back().push_back(x);

                if (x == node) break;
            }
            ++cur_id;
        }
    }

    vector<vector<int>> DAG() {
        vector<vector<int>> ret(cur_id);
        for (int node = 1; node <= n; ++node) {
            for (int &ch : graph[node]) {
                int u = id[node];
                int v = id[ch];
                if (u != v) ret[u].push_back(v);
            }
        }
        return ret;
    }
};
```