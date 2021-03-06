---
title: SACO 2017 Stargazing
date: 2020-04-14 12:08:18
tags:
    - Segment tree
    - Geometry
    - DSU
    - Line sweep
    - Data structure
categories:
    - 2017
    - Round 3
mathjax: true
---

## Solution

<!-- more -->

Firstly, a rhombus is inconvenient, so apply the transformation $(x, y) \rightarrow (x + y, x - y)$ to rotate everything by 45 degrees. Manhattan distance becomes Chebyshev distance and the query is on a square now.

Next, notice for each star $i$, we only need to consider "critical stars". These are the 4 stars in each quadrant of star $i$'s query square that have lowest discovery time. This is because if those stars are connected to $i$, so is every other star in the 4 quadrants.

We can use a segment tree and a line sweep to find these critical stars.

After that, we simply use DSU to create connected components and so we can easily query each component's size.

## Complexity

Time: $O(N \log N)$

Space: $O(N)$

## Code

```cpp
#include <bits/stdc++.h>
#define FOR(i, x, y) for (int i = x; i < y; i++)
typedef long long ll;
using namespace std;

struct Point {
    int x, y, t;
};

const int MAXN = 2000002;

set<int> active[MAXN];
int seg[4 * MAXN];
Point stars[100001];
pair<Point, int> events[200001];
vector<int> graph[100001];

int component[100001], sz[100001];

bool cmp(pair<Point, int> a, pair<Point, int> b) {
    if (a.first.y == b.first.y) {
        if (a.second == b.second) return a.first.x < b.first.x;
        return a.second > b.second;
    }
    return a.first.y < b.first.y;
}

int find(int a) {
    while (a != component[a]) component[a] = component[component[a]], a = component[a];
    return a;
}
void onion(int a, int b) {
    if (find(a) != find(b)) {
        sz[find(b)] += sz[find(a)];
        component[find(a)] = component[find(b)];
    }
}

void update(int pos, int val, int node = 1, int l = 1, int r = MAXN) {
    if (l == r) {
        seg[node] = val;
    } else {
        int mid = (l + r) / 2;
        if (pos <= mid) update(pos, val, node * 2, l, mid);
        else update(pos, val, node * 2 + 1, mid + 1, r);
        seg[node] = min(seg[node * 2], seg[node * 2 + 1]);
    }
}
int query(int a, int b, int node = 1, int l = 1, int r = MAXN) {
    if (l > b || r < a) return INT_MAX;
    if (l >= a && r <= b) return seg[node];
    int mid = (l + r) / 2;
    return min(query(a, b, node * 2, l, mid), query(a, b, node * 2 + 1, mid + 1, r));
}

void ins(int pos, int val) {
    active[pos].insert(val);
    update(pos, *(active[pos].begin()));
}
void ers(int pos, int val) {
    active[pos].erase(val);
    update(pos, (active[pos].size() ? *(active[pos].begin()) : INT_MAX));
}

int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    int n, d;
    cin >> n >> d;
    FOR(i, 1, n + 1) {
        int x, y;
        cin >> x >> y;
        // Rotate star by 45 degrees
        stars[i] = {x + y + 1, y - x + 1000001, i};
    }
    FOR(i, 1, n + 1) component[i] = i, sz[i] = 1;
tags:
    - Fenwick tree
    - DP
categories:
    - 2017
    - Round 3
mathjax: true
    FOR(i, 0, 4) {
        FOR(j, 1, n + 1) {
            events[j] = {stars[j], 0}, events[j + n] = {stars[j], 1};
            events[j + n].first.y += d + 1;
        }
        sort(events + 1, events + 2 * n + 1, cmp);

        // Line sweep to find "critical" stars in each quadrant
        FOR(j, 1, 2 * n + 1) {
            if (events[j].second == 1) ers(events[j].first.x, events[j].first.t);
            else {
                int q = query(max(1, events[j].first.x - d), events[j].first.x);
                if (q < events[j].first.t) graph[events[j].first.t].push_back(q);
                ins(events[j].first.x, events[j].first.t);
            }
        }

        // Rotate everything by 90 degrees
        FOR(j, 1, n + 1) stars[j] = {-stars[j].y + 2000002, stars[j].x, stars[j].t};
    }

    FOR(i, 1, n + 1) {
        // DSU to find component sizes
        for (int j : graph[i]) onion(i, j);
        cout << sz[find(i)] << '\n';
    }

    return 0;
}
```
