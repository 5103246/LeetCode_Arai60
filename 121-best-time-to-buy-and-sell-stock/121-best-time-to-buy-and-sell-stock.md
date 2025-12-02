# Best Time to Buy and Sell Stock
* 問題：https://leetcode.com/problems/best-time-to-buy-and-sell-stock/
* 言語：C++

## Step1

- 問題の概要
  - 株の価格の配列が与えられる。配列の中から一つ買って、その後で売る。そのとき得られる最大利益を求める問題。利益がない（マイナスなら）0を返す。

- 最小価格を更新していく方法
- [7, 1, 5, 3, 6]
- [0, -6] まず、7 - 7 = 0が先頭に入り、次に1 - 7 = -6が入る。1の方が小さいので売る株を更新する。
- [0, -6, 4, 2, 5]　順に、差額を求めると、5が最大利益となる。
- 配列で考えたが、必要なのは、最小価格と最大利益の2変数で済む。
- n : prices.length
- 時間計算量：O(n)
- 空間計算量：O(1)
- 実行時間は、10 ^ 5 / 10 ^ 8 = 1 ms程度になる。
- 他の解き方は特に思い浮かばなかったので次に進む。

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int buy_price = prices.front();
        int max_profit = 0;
        for (auto price : prices) {
            int profit = price - buy_price;
            max_profit = max(max_profit, profit);
            buy_price = min(buy_price, price);
        }
        return max_profit;
    }
};
```

## Step2

- https://github.com/docto-rin/leetcode/pull/42/files
　 - https://discord.com/channels/1084280443945353267/1196472827457589338/1196473519689703444
    - > 本質的には、
      > - scanl min でその日までの最安の値。
      > - zipWith (-) で利益。
      > - max を取る。
  - pythonではitertoolsを使えば、関数型風に書けるらしい。
  - Chat GPTにC++ではどのように書くか聞いてみた
  - https://cpprefjp.github.io/reference/numeric/partial_sum.html
  - https://cpprefjp.github.io/reference/ranges/transform_view.html
  - https://cpprefjp.github.io/reference/ranges/zip_view.html
  - C++だと複雑になって読みにくいかな
  - 関数型についてよくわからなかったのでGPTに聞いてみたが、変数の再代入をしないや状態が変化しない特徴があるらしい。（参照透過性）
  - 状態が変化しないので共有データを変更しない→ロックが不要→並列処理に強い  などの特徴がある（https://ja.wikipedia.org/wiki/%E9%96%A2%E6%95%B0%E5%9E%8B%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0）
  - 関数型の考えや機能は他の言語で取り込まれている。（C++のlamda, rangesなど）
    
```cpp
#include <vector>
#include <ranges>
#include <algorithm>
#include <numeric>

class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        vector<int> prefix(prices.size());
        partial_sum(prices.begin(), prices.end(), prefix.begin(),
                    [](int acc, int x){ return min(acc, x); });

        auto profit = views::zip(prices, prefix)
                    | views::transform([](auto pair) {
                        auto [price, min_price] = pair;
                        return price - min_price;
                    });
        
        return *ranges::max_element(profit);
    }
};
```

- https://github.com/olsen-blue/Arai60/pull/37/files
- https://github.com/nanae772/leetcode-arai60/pull/36/files
- buy_priceをmath.infから始めている人もいた。
- prices[0]を使うなら、ループはi = 1から始めた方が自然かもしれない？
- そこを変えても読みやすくなるわけでもないので、変えなくてもいいか。

## Step3

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int max_profit = 0;
        int min_price = prices.front();
        for (auto price : prices) {
            max_profit = max(max_profit, price - min_price);
            min_price = min(min_price, price);
        }
        return max_profit;
    }
};
```
