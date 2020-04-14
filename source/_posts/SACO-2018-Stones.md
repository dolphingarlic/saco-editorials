---
title: SACO 2018 Stones
date: 2020-04-14 11:53:19
tags:
    - Fenwick tree
    - Brute force
    - Geometry
categories:
    - 2018
    - Round 3
mathjax: true
---

## Solution

In this problem, we simply want to iterate until the minimum $i$ such that the sum of the number of stones in a rectangle with maximum Chebyshev distance $i$ satisfies the condition.

Since we want to know the number of stones in a rectangle and each update is a point update, an ideal data structure to use for this problem is a 2d Fenwick tree.

## Complexity

Time: $O(QS  \log{N}  \log{M})$

Memory: $O(NM)$

## Code

```cpp
#include <bits/stdc++.h>
#pragma GCC optimize("O3")
#define FOR(i, x, y) for (ll i = x; i < y; i++)
typedef long long ll;
using namespace std;

ll bit[1001][1001], n, m, q;

void update(ll x, ll y, ll val) {
    for (ll i = x; i <= n; i += (i & (-i))) {
        for (ll j = y; j <= m; j += (j & (-j))) {
            bit[i][j] += val;
        }
    }
}

ll query(ll x, ll y) {
    ll sum = 0;
    for (ll i = x; i > 0; i -= (i & (-i))) {
        for (ll j = y; j > 0; j -= (j & (-j))) {
            sum += bit[i][j];
        }
    }
    return sum;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    cin >> n >> m >> q;
    FOR(i, 1, n + 1) {
        FOR(j, 1, m + 1) {
            ll x;
            cin >> x;
            update(i, j, x);
        }
    }
    FOR(qry, 0, q) {
        ll t, a, b, s;
        cin >> t >> a >> b >> s;
        a++;
        b++;
        if (t) {
            update(a, b, -(query(a, b) + query(a - 1, b - 1) - query(a - 1, b) - query(a, b - 1)));
            update(a, b, s);
        } else {
            ll cost = 0, prev = 0;
            for (ll i = 0;
                 s > 0 && (a + i <= n || b + i <= m || a - i > 0 || b - i > 0);
                 i++) {
                ll block = query(min(n, a + i), min(m, b + i)) +
                           query(a - i - 1, b - i - 1) -
                           query(min(n, a + i), b - i - 1) -
                           query(a - i - 1, min(m, b + i)) - prev;

                if (block > s)
                    cost += s * i * 2, s = 0;
                else
                    cost += block * i * 2, s -= block;

                prev += block;
            }
            if (s == 0)
                cout << cost << '\n';
            else
                cout << "-1\n";
        }
    }
    return 0;
}
```