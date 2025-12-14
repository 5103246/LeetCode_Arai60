# Best Time to Buy and Sell Stock II
- 問題：https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/
- 言語：C++

## Step1

- 問題の概要
  - 株価のリストが与えられ、株の売買で得られる最大利益を求める。1株のみ保有でき、複数回株の売買ができる。


- 先頭から最小値を更新して、最小値より大きい株価なら売る。売ったら最小値を更新して、再度同じ処理を行っていく。
- 最小値と利益、利益のトータルの3変数で済む。
- 時間計算量：O(n)
- 空間計算量：O(1)
- 実行時間は、3 * 10 ^ 4 / 10 ^ 8 ≒ 3 * 10 ^ (-4) 、300 µs程度
- 他の解き方は、テーブルを作ってでも解けそうかな。
- とりあえず、良さげな解き方なので実装してみる。
- 大分コンパクトに書けた。

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int buy_price = prices.front();
        int total_profit = 0;
        for (auto price : prices) {
            if (buy_price < price) {
                total_profit += price - buy_price;
            }
            buy_price = price;
        }

        return total_profit;
    }
};
```

## Step2

- https://github.com/docto-rin/leetcode/pull/43/files
- さらにコンパクトに書けるのか。
- 価格が上がった分だけ、足していく。
- 保有株を更新する必要がなかったのか。

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int total_profit = 0;
        for (int i = 1; i < prices.size(); ++i) {
            total_profit += max(0, prices[i] - prices[i - 1]);
        }

        return total_profit;
    }
};
```

- 株を持っているか、持っていないかの2つの状態がよくわからなかったので、GPTに聞いて書いてみた。
- 株を買う：cash - price、株を売る：hold + price、何もしない：cash → cash, hold → hold
- 入力 : [7, 1, 5, 3, 6, 15]
- cash :  0  0  4  4  7  16
- hold : -7 -1 -1  1  1  1
- 今日の終わりに、株を持っていない状態で最もお金が多いのはいくらかと、株を持っている状態で最もお金が多いのはいくらかを更新していく

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int cash = 0;
        int hold = -prices.front();
        for (auto price : prices) {
            cash = max(cash, hold + price);
            hold = max(hold, cash - price);
        }
        return cash;
    }
};
```

- 個人的には、step1の解き方や上がった分だけ足していく方がわかりやすかった
- ただ、手数料や回数制限など拡張する場合は上の考えの方がよさそう。

## Step3

```cpp
class Solution {
public:
    int maxProfit(const vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }

        int total_profit = 0;
        for (int i = 1; i < prices.size(); ++i) {
            total_profit += max(0, prices[i] - prices[i - 1]);
        }
        return total_profit;
    }
};
```
