# Coin Change
- 問題：https://leetcode.com/problems/coin-change/
- 言語：C++

## Step1

- 問題の概要
  - 異なる額面の硬貨を組み合わせて、与えられた合計金額を作る。その際の、硬貨の最小枚数を返す。組み合わせがない場合は-1を返す。効果の枚数はいくら使ってもよい。
  - coins.length : n
  - 1 <= n <= 12
  - 1 <= coins[i] <= 2^31 - 1
  - 0 <= amount <= 10^4

- 大きなコイン順に組み合わせを考えれば解けるかと思ったが、[1, 5, 11] , amout = 15のようなケースがあるか。
- [11, 1, 1, 1, 1]よりも[5, 5, 5]の方が少ない。
- コインを大きい順に並べる。最も大きいコインでamoutを最大枚引く。残りのamoutをそれ以降のコインで組み合わせを求める。求まったら、次にに大きいコインで再度同じ処理を行い、最小枚数を求める。
- [11, 5, 1] amout = 15 count = 0
- 15 - 11 = 4 -> [5,1] amount = 4 count = 1-> [1] amount = 4 -> 4 - 1 * 4 = 0 count = 5
- [5, 1] amout = 15 count = 0
- 15 - 5*3 = 0 count = 3 -> answer = 3
- この方法で実装してみたが、wrong answerとなった。全部を満遍なく使った方が良いケースを考えていなかった。
- 一旦、別の方法で考える。
  
- 手が止まったので、他の人の解答を見る
- https://github.com/docto-rin/leetcode/pull/45/files step1
- 0円から順に、その金額を作る最小枚数をメモしていく解き方
- [1, 5, 11] amout = 15のケースを考える
- 15 - 1 = 14で、14を作る最小枚数+1をすれば、最後に1を使ったときの枚数が求まる
- 15 - 5 = 10で、10を作る最小枚数+1が、5を使ったときの枚数
- 15 - 11 = 4で、4を作る最小枚数+1が、11を使ったときの枚数
- さらに、14 - 1, 14 - 5 ...を考えると、同じ計算が出てくるので、~円を作る最小枚数をメモしていけば無駄が省ける
- 0円からamount円まで順に、金額を作る最小枚数のリストを用意するdp[]
- dp[i] = min(dp[i], dp[i - coin] + 1) でその金額の最小枚数が求まる。
- 最初、動的計画法で考えていたが、自分が考えていたのは動的計画法ではなかった。
- コインを何枚使うかを中心に考えていたが、ゴールから逆算して必要なものや、小さな問題に分解することを考えればよかった。
- n : coins.length, m : amount
- 時間計算量：O(n * m)
- 空間計算量：O(m)
- 実行時間は、12 * 10 ^ 4 / 10 ^ 8 = 1.2 ms程度

```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }

        vector<int> min_coins(amount + 1, numeric_limits<int>::max());
        min_coins.front() = 0;
        for (int i = 1; i <= amount; ++i) {
            for (auto coin : coins) {
                if (coin > i) {
                    continue;
                }
                if (min_coins[i - coin] == numeric_limits<int>::max()) {
                    continue;
                }
                min_coins[i] = min(min_coins[i], min_coins[i - coin] + 1);
            }
        }

        if (min_coins.back() == numeric_limits<int>::max()) {
            return -1;
        }
        return min_coins.back();
    }
};
```

## Step2

- https://github.com/docto-rin/leetcode/pull/45/files step2
- dpは、コインを昇順に並べて、coin > iになったらbreakすれば分岐を減らせる。
- numeric_limits<int>::max()の他には、amount + 1も使える
- DFS, BFSでも解ける。
- treeのノードが各コインを足した際の合計値もしくは残額で、ノードがゼロ、amountになったときの、treeの長さが答えになる。
- ただ、BFSの場合は愚直にやったら被りが出てくるので、計算したところをメモしておく必要がある
- https://github.com/seal-azarashi/leetcode/pull/37/files#r1830469401
 - > (コインの価格が十分に近いと Queue の中身は)、2^n で増えていきます。
   
- BFSを書いたが、最初にamountより大きなコインを除いていなくてruntime errorになった。
- スタックに追加する際にintの上限を超えてしまうのを防ぐためにもamountより大きなコインを除く必要があるのか。
- 参考にしたPythonコードで弾いていなかったのは、Pythonの整数型が多倍長整数だから。intの上限を超えても自動的にメモリを確保してくれるためエラーが起きない。
- 非再帰DFS
```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }
        if (amount == 0) {
            return 0;
        }

        vector<int> min_coins(amount + 1, numeric_limits<int>::max());
        vector<int> sorted_coins;
        for (auto coin : coins) {
            if (coin > amount) {
                continue;
            }
            sorted_coins.push_back(coin);
        }
        sort(sorted_coins.begin(), sorted_coins.end());
        stack<pair<int, int>> coin_and_sum;
        coin_and_sum.push({0, 0});
        
        while (!coin_and_sum.empty()) {
            auto [num_coin, sum] = coin_and_sum.top();
            coin_and_sum.pop();
            if (sum > amount) {
                continue;
            }
            if (num_coin >= min_coins[sum]) {
                continue;
            }
            min_coins[sum] = num_coin;
            for (auto coin : sorted_coins) {
                coin_and_sum.push({num_coin + 1, coin + sum});
            }
        }

        if (min_coins.back() == numeric_limits<int>::max()) {
            return -1;
        }

        return min_coins.back();
    }
};
```

- 再帰版DFS
- 参考：https://github.com/seal-azarashi/leetcode/commit/e63c235e4f484f8662894b816586e1d2da12ac85
- 降順に並べたコインを渡す。先に大きなコインを使うようになるので少し速くなる。
- leetcode上の実行時間だが、非再帰の方は100 ms越えだったが、再帰の方は50 ~ 60 ms程度だった。
- 降順にしたのと、ヒープメモリを使わない分速くなったのかな？
- amount <= 10 ^ 4なので、スタックオーバーフローの可能性もあるので非再帰の方が安全ではある。
```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }

        vector<int> sorted_coins;
        for (auto coin : coins) {
            if (coin > amount) {
                continue;
            }
            sorted_coins.push_back(coin);
        }
        sort(sorted_coins.begin(), sorted_coins.end(), greater<int>());

        vector<int> min_coins(amount + 1, numeric_limits<int>::max());
        CoinChangeHelper(sorted_coins, amount, 0, 0, min_coins);

        if (min_coins.back() == numeric_limits<int>::max()) {
            return -1;
        }
        return min_coins.back();
    }

private:
    void CoinChangeHelper(const vector<int>& coins, int amount, int num_coins, int sum_coins, vector<int>& min_coins) {
        if (sum_coins > amount || num_coins >= min_coins[sum_coins]) {
            return;
        }

        min_coins[sum_coins] = num_coins;

        for (auto coin : coins) {
            CoinChangeHelper(coins, amount, num_coins + 1, sum_coins + coin, min_coins);
        }
    }
};
```

- BFS
- 参考：https://github.com/docto-rin/leetcode/pull/45/files step2
- リストを交換しながらコインの枚数を数えていく。
- 降順に並べた方が、大きなコインを優先して見れるので少し効率がいい
- 交換するリストの良い名前が思いつかなかったので参考にしたコードの名前のままにした。
- 
```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }
        if (amount == 0) {
            return 0;
        }

        vector<int> sorted_coins = coins;
        sort(sorted_coins.begin(), sorted_coins.end(), greater<int>());
        vector<char> visited(amount + 1, 0);
        vector<int> rests = {amount};
        int num_used = 1;

        while (!rests.empty()) {
            vector<int> next_rests;
            for (auto rest : rests) {
                for (auto coin : sorted_coins) {
                    int next_rest = rest - coin;
                    if (next_rest == 0) {
                        return num_used;
                    }
                    if (next_rest < 0) {
                        continue;
                    }
                    if (visited[next_rest]) {
                        continue;
                    }
                    next_rests.push_back(next_rest);
                    visited[next_rest] = 1;
                }
            }
            ++num_used;
            rests.swap(next_rests);
        }

        return -1;
    }
};
```

- queueを使ったBFS
- queueを使った方が、内側のループが一つなくなるので読みやすい。
- visitedには0, 1を入れていたが、true, falseに変えてみた。これはこれでありかな？0, 1か定数を使う方が一般的かも
- amountから引いていくので、残額を意味するremainingにしてみた。
- 一旦、queueに入れた後に、早期リターンするようにした。amount = 0のケースに対応できるのと、queueの初期値が0にできる。
```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }

        vector<int> sorted_coins = coins;
        sort(sorted_coins.begin(), sorted_coins.end(), greater<int>());

        queue<pair<int, int>> coins_and_remaining;
        coins_and_remaining.push({0, amount});
        vector<char> visited(amount + 1, false);

        while (!coins_and_remaining.empty()) {
            auto [num_coins, remaining] = coins_and_remaining.front();
            coins_and_remaining.pop();
            if (remaining == 0) {
                return num_coins;
            }

            for (auto coin : sorted_coins) {
                int next_remaining = remaining - coin;
                if (next_remaining < 0 || visited[next_remaining]) {
                    continue;
                }
                coins_and_remaining.push({num_coins + 1, next_remaining});
                visited[next_remaining] = true;
            }
        }

        return -1;
    }
};
```

## Step3
- DPが苦手なのでDPで3回
- if文が減るのでamount + 1を上限に
- 定数名、変数名は他に良いのがありそう。

```cpp
class Solution {
public:
    int coinChange(const vector<int>& coins, int amount) {
        if (amount < 0) {
            return -1;
        }

        const int kUpperBound = amount + 1;
        vector<int> min_coins(amount + 1, kUpperBound);
        min_coins.front() = 0;

        for (int target = 1; target <= amount; ++target) {
            for (auto coin : coins) {
                if (target < coin) {
                    continue;
                }
                min_coins[target] = min(min_coins[target], min_coins[target - coin] + 1);
            }
        }

        if (min_coins.back() == kUpperBound) {
            return -1;
        }

        return min_coins.back();
    }
};
```
