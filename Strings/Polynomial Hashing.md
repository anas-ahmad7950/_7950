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

Template 2
```cpp
const int H = 2, MOD = 1e9 + 7;
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
int base[H] = {29, 31};
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

struct HashValue {
    int n = 0;
    array<int, H> hv = {0};

    bool operator==(const HashValue &other) const {
        return hv == other.hv;
    }

    HashValue operator+(const HashValue &other) const {
        HashValue ret = {n + other.n, hv};
        for (int b = 0; b < H; ++b) ret.hv[b] = add(ret.hv[b], mul(other.hv[b], power[b][n]));
        return ret;
    }

    HashValue operator/(const HashValue &other) const {
        HashValue ret = {other.n + n, other.hv};
        for (int b = 0; b < H; ++b) ret.hv[b] = add(ret.hv[b], mul(hv[b], power[b][other.n]));
        return ret;
    }

    HashValue operator-(const HashValue &other) const {
        HashValue ret = {n - other.n, hv};
        for (int b = 0; b < H; ++b) ret.hv[b] = mul(sub(ret.hv[b], other.hv[b]), inverse[b][other.n]);
        return ret;
    }

    static HashValue create(const char ch) {
        HashValue ret;
        ret.n = 1;
        ret.hv.fill(ch - 'a' + 1);
        return ret;
    }
};

struct Hashing {
    int n;
    vector<HashValue> Hash, HashRev;

    Hashing(string &s) {
        n = s.size();
        Hash.resize(n + 1);
        HashRev.resize(n + 2);
        NHashing(s);
        RHashing(s);
    }

    void NHashing(const string &s) {
        for (int i = 1; i <= n; ++i) Hash[i] = Hash[i - 1] + HashValue::create(s[i - 1]);
    }

    void RHashing(const string &s) {
        for (int i = n; i; --i) HashRev[i] = HashValue::create(s[i - 1]) / HashRev[i + 1];
    }

    HashValue Nquery(int l, int r) {
        return Hash[r] - Hash[l - 1];
    }

    HashValue Rquery(int l, int r) {
        return HashRev[l] - HashRev[r + 1];
    }
};
```