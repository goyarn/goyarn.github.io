---
title: "BOJ-1128 : 피보나치 냅색"
excerpt: "백준 온라인 저지 1128 문제풀이 및 코드"

categories:
  - PS
tags:
  - algorithm
  - BOJ
last_modified_at: 2020-06-25T11:40:00+09:00
use_math: true
---

<https://boj.kr/1128>

## 문제
유명한 냅색 문제이지만, 무게의 범위가 $10^{16}$ 까지입니다. 따라서 무게를 인자로 사용하는  DP로는 해결이 안 됩니다.

이때 물건의 무게가 모두 피보나치 수라는 특별한 조건을 준 문제입니다.

## 풀이
피보나치에 집중하다 보면, 물건의 무게가 피보나치 숫자이기 때문에 활용 가능한 두 가지 특성을 발견할 수 있습니다.

1. 피보나치 수에는 $fibo(x+2) - 1 = \sum\limits_{k=0}^x fibo(k)$ 라는 공식이 있습니다.
   중요한 건 공식 자체가 아니라 '가장 큰 원소 1, 2개가 나머지 다른 원소들 합만큼 크다' 는 점입니다.
2. 물건 50개 중 중복되는 무게가 생기기 쉽습니다. 주어진 무게 범위 안의 피보나치 숫자는 아마 80개가 안 될 것입니다.

사실 2번 특성 같은 경우는 '반드시 중복이 생길 것이다' 가 아니라, 특성 1을 활용한 최적화가 잘 적용되지 않는 케이스는 중복을 포함한 케이스이기 때문에 저렇게 언급했습니다.

이 특성들을 이용하면 아주 잘 들어맞는 최적화를 두 가지 해줄 수 있습니다.

### 특성 1을 이용한 최적화
현재 가방의 남은 무게에, 지금 보고 있는 물건부터 마지막(=가장 가벼운) 물건까지 모두 넣어줄 수 있는지 체크합니다.
만약 그것이 가능하다면 물건을 모두 넣고 탐색을 종료하고, 아니라면 그대로 탐색을 진행하면 됩니다.

이로 인해 탐색해야 하는 남은 물건들 중 가장 무거운 물건을 1~2개만 가방에 넣지 않으면 단숨에 해당 케이스의 탐색이 종료됩니다.

### 특성 2를 이용한 최적화
정렬을 했으므로 같은 무게를 연속하여 가방에 넣을지 말지 판단하게 될 것입니다.

이때 같은 무게인데 더 높은 가격의 물건 A를 가방에 넣지 않기로 했다면, 같은 무게인데 더 낮은 가격의 물건 B는 반드시 가방에 넣지 않아야 합니다.

따라서 이런 방식으로 연속한 같은 무게의 물건이 k개 있다면, 이 k개를 백트래킹할 때의 시간복잡도는 $O(2^k)$가 아니라 O(k) 가 됩니다.


## 코드
메모이제이션이나 부분합도 적용하지 않고 언급한 솔루션만 적용한 코드입니다.


```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef vector<vector<int>> vvi;

int n;

ll weight[51], price[51];
ll max_price;
ll c;

void backtrack(ll cur_price, ll left_weight, int i)
{
    max_price = max(max_price, cur_price);
    if (i >= n || left_weight < 0)
    {
        return;
    }
    ll pSum = 0, wSum = 0;
    int nextWeightIdx = i;
    for (int k = i; k < n; ++k)
    {
        pSum += price[k];
        wSum += weight[k];
        if (weight[nextWeightIdx] == weight[i]
            && weight[nextWeightIdx] != weight[k])
        {
            nextWeightIdx = k;
        }
    }
    // case 1 : 남은 모든 물건을 챙길 수 있는 경우
    if (wSum <= left_weight)
    {
        max_price = max(max_price, cur_price + pSum);
        return;
    }
    // case 2 : 현재 물건 무게를 먹지 않는 경우
    if (i < nextWeightIdx)
        backtrack(cur_price, left_weight, nextWeightIdx);
    // case 3 : 현재 무게를 먹는 경우
    if (weight[i] <= left_weight)
        backtrack(cur_price + price[i], left_weight - weight[i], i + 1);
    return;
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    cin >> n;
    auto vp = vector<pair<ll, ll>>(n, pair<ll, ll>(0, 0));
    for (int i = 0; i < n; ++i)
    {
        cin >> vp[i].first >> vp[i].second;
    }
    sort(vp.begin(), vp.end());
    reverse(vp.begin(), vp.end());
    for (int i = 0; i < n; ++i)
    {
        weight[i] = vp[i].first;
        price[i] = vp[i].second;
    }
    cin >> c;
    max_price = 0;
    backtrack(0, c, 0);
    cout << max_price;
    return 0;
}
```

