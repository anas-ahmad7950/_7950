Template 1: normal hashing and reverse hashing
```cpp
const int H = 2;  
typedef array<int, H> Value;  
  
const int MOD = 1e9 + 7;  
int add(ll a, int b) { return (a + b) % MOD; }  
int sub(ll a, int b) { return (a - b + MOD) % MOD; }  
int mul(ll a, int b) { return (a * b) % MOD; }  
int fastpower(int base, int power) {  
    int ret = 1;  
    while (power) {  
        if (power & 1) ret = mul(ret, base);  
        base = mul(base, base);  
        power >>= 1;  
    }  
    return ret;  
}  
int division(int a, int b) { return mul(a, fastpower(b, MOD - 2)); }  
  
const int N = 2e5 + 7;  
int base[H] = {29, 31}, r[H] = {1, 3};  
int power[H][N], inverse[H][N];  
void precompute() {  
    int inv;  
    for (int b = 0; b < H; ++b) {  
        power[b][0] = inverse[b][0] = 1;  
        inv = fastpower(base[b], MOD - 2);  
  
        for (int i = 1; i < N; ++i) {  
            power[b][i] = mul(power[b][i - 1], base[b]);  
            inverse[b][i] = mul(inverse[b][i - 1], inv);  
        }  
    }  
}  
  
struct Hashing {  
    int n;  
    vector<Value> Hash, HashRev;
  
    Hashing(const string &s) {  
        n = s.size();  
        Hash.resize(n + 1);  
        HashRev.resize(n + 2);  
        NHashing(s);  
        RHashing(s);  
    }  
  
    Value getChar(char ch, int idx) {  
        Value ret;  
        for (int b = 0; b < H; ++b) ret[b] = mul(ch - 'a' + 1, power[b][idx]);  
        return ret;  
    }  
  
    void NHashing(const string &s) {  
        for (int i = 1; i <= n; ++i) {  
            Value cur = getChar(s[i - 1], i);  
            for (int b = 0; b < H; ++b) Hash[i][b] = add(Hash[i - 1][b], cur[b]);  
        }  
    }  
  
    void RHashing(const string &s) {  
        for (int i = n; i; --i) {  
            Value cur = getChar(s[i - 1], n - i + 1);  
            for (int b = 0; b < H; ++b) HashRev[i][b] = add(HashRev[i + 1][b], cur[b]);  
        }  
    }  
  
    Value Nquery(int l, int r) {  
        Value ret;  
        for (int b = 0; b < H; ++b) ret[b] = mul(sub(Hash[r][b], Hash[l - 1][b]), inverse[b][l - 1]);  
        return ret;  
    }  
  
    Value Rquery(int l, int r) {  
        Value ret;  
        for (int b = 0; b < H; ++b) ret[b] = mul(sub(HashRev[l][b], HashRev[r + 1][b]), inverse[b][n - r]);  
        return ret;  
    }  
};
```