Decomposes any path on a tree into $O(log(n))$ segments.
```cpp
struct HLD {
    vector<vector<int>> &graph;
    vector<int> sz, heavy, depth, parent; // DFS
    vector<int> in, head, flat; // flatten
    // vector<int> rev;
    int timer;

    HLD(int _n, vector<vector<int>> &g) : graph(g) {
        sz = heavy = depth = parent = in = head = vector<int>(_n + 1);
        // rev = vector<int>(_n + 1);
        timer = 0;

        DFS(1);
        head[1] = 1;
        flatten(1);
    }

    void DFS(int node) {
        sz[node] = 1;
        for (int &ch : graph[node]) {
            if (ch == parent[node]) continue;
            depth[ch] = depth[node] + 1;
            parent[ch] = node;
            DFS(ch);
            sz[node] += sz[ch];
            if (sz[ch] > sz[heavy[node]]) heavy[node] = ch;
        }
    }

    void flatten(int node) {
        in[node] = timer++;
        // rev[timer - 1] = node;
        // flat.push_back(0);

        if (heavy[node]) {
            head[heavy[node]] = head[node];
            flatten(heavy[node]);
        }

        for (int &ch : graph[node]) {
            if (ch == parent[node] || ch == heavy[node]) continue;
            head[ch] = ch;
            flatten(ch);
        }
    }

    vector<pair<int, int>> decompose(int u, int v) {
        vector<pair<int, int>> ret;
        while (true) {
            if (head[u] == head[v]) {
                if (depth[u] > depth[v]) swap(u, v); // u is the LCA
                ret.emplace_back(in[u], in[v]);
                // if (in[u] + 1 <= in[v]) ret.emplace_back(in[u] + 1, in[v]);
                return ret;
            }

            if (depth[head[u]] > depth[head[v]]) swap(u, v);
            ret.emplace_back(in[head[v]], in[v]);
            v = parent[head[v]];
        }
    }
};
```

```cpp
vector<pair<int, int>> decompose(int u, int v) {
	vector<pair<int, int>> r_u, r_v;
	while (true) {
		if (head[u] == head[v]) {
			if (depth[u] > depth[v]) r_u.emplace_back(in[v], in[u]);
			else r_v.emplace_back(in[u], in[v]);
			break;
		}

		if (depth[head[u]] > depth[head[v]]) {
			r_u.emplace_back(in[head[u]], in[u]);
			u = parent[head[u]];
		}
		else {
			r_v.emplace_back(in[head[v]], in[v]);
			v = parent[head[v]];
		}
	}

	while (!r_v.empty()) r_u.push_back(r_v.back()), r_v.pop_back();
	return r_u;
}
```