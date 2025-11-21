# House Robber II
* 問題：https://leetcode.com/problems/house-robber-ii/
* 言語：C++

## Step1

- 問題の概要
- 前回と同じく、隣の家同士では盗まずに、得られる最大金額を求める問題。ただし、円形に家が並んでいるので、最初と最後の家が隣同士になっている
- [9, 6, 1, 10, 2, 4]の場合、6 + 10 + 4 = 20が最大
- 前回は i - 1より前の合計金額の中で最大のものを探す解き方。今回は、i - 1より前の合計金額で最大のものを探すときに場合分けする
- 先頭を含んだ合計値と先頭を含まない合計値の両方でそれぞれ最大金額を求める
- 上に挙げたケースの場合、i = 2スタートで考える
- 9, 0, 0, 6  {左から順に、2つ前の最大金額(先頭を含む)、2つ前の最大金額(先頭を含まない)、1つ前の合計金額(先頭を含む)、1つ前の合計金額(先頭を含まない)}
- 9, 6, 10, 1 (i = 2)
- 10, 6, 19, 16 (i = 3)
- 19, 16, 12, 8 (i = 4)
- 19, 16, 23, 20 (i = 5)
- 19, 16, 19, 20 (最後尾なので、23 から先頭と最後尾の内小さい方を引く)
- → 20が最大金額

- 時間計算量：O(n)
- 空間計算量：O(1)
- 実行時間は 100 / 10 ^ 8 = 1 µs程度
- 変数を同時に4つ使うので、読む側にとってわかりにくそう。変数を減らせないだろうか。
- 最初の初期化は、自分が考えやすいのにしたが、0スタートにできそう。でも、先頭を含まない変数もあるので微妙かも

```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        if (nums.empty()) {
            return 0;
        }
        if (nums.size() == 1) {
            return nums.front();
        }

        int two_before_max_front = nums.front();
        int two_before_max = 0;
        int one_before_max_front = 0;
        int one_before_max = nums[1];

        for (int i = 2; i < nums.size(); ++i) {
            int total_front = nums[i] + two_before_max_front;
            int total = nums[i] + two_before_max;
            two_before_max_front = max(two_before_max_front, one_before_max_front);
            two_before_max = max(two_before_max, one_before_max);
            one_before_max_front = total_front;
            one_before_max = total;
        }
        one_before_max_front -= min(nums.front(), nums.back());
        
        return max(max(two_before_max_front, two_before_max),
                   max(one_before_max_front, one_before_max));
    }
};
```

## Step2

- https://github.com/docto-rin/leetcode/pull/41/files
- 先頭を除いて求めた最大金額と最後尾を除いた最大金額を別々に求め、最後にmaxで解く方法
- 2周するが、変数の数が少なくて、かなり読みやすかった。
- step1の解き方も最終的には、先頭を除いたなかでの最大金額と最後尾を除いた最大金額が求まることに気づいた。

```cpp
class Solution {
public:
    int rob(const vector<int>& houses) {
        if (houses.empty()) {
            return 0;
        }
        if (houses.size() == 1) {
            return houses.front();
        }

        return max(CaluculateMaxMoney(houses, 0, houses.size() - 1), 
                   CaluculateMaxMoney(houses, 1, houses.size()));
    }

private:
    int CaluculateMaxMoney(const vector<int>& houses, int left, int right) {
        int max_money_with_last = 0;
        int max_money_without_last = 0;

        for (int i = left; i < right; ++i) {
            int total_money = houses[i] + max_money_without_last;
            max_money_without_last = max(max_money_without_last, max_money_with_last);
            max_money_with_last = total_money;
        }
        return max(max_money_with_last, max_money_without_last);
    }
};
```

- https://github.com/irohafternoon/LeetCode/pull/39/files
- DP配列を使った解き方もメモリ制限が緩ければ、読みやすいのでよさそう。
- pythonのデコレータをC++で実装していた。（デコレータ：https://docs.python.org/ja/3/glossary.html#term-decorator）
- 自分も余裕があったらチャレンジしたい。

- https://github.com/nanae772/leetcode-arai60/pull/35/files
- 先頭を取る、取らないで分ける解き方もあった。
- 基本的な考えは変わらないが、先頭を取る場合は、右隣と最後の家を除いた範囲を関数に渡す
- 先頭を除く、もしくは最後尾を除くの方がシンプルで自分は好みかな

## Step3

- 先頭を除く、最後尾を除く場合分けが一番気に入ったので3回

```cpp
class Solution {
public:
    int rob(const vector<int>& houses) {
        if (houses.empty()) {
            return 0;
        }
        if (houses.size() == 1) {
            return houses.front();
        }

        return max(CaluculateMaxMoney(houses, 0, houses.size() - 1),
                   CaluculateMaxMoney(houses, 1, houses.size()));
    }

private:
    int CaluculateMaxMoney(const vector<int>& houses, int start, int end) {
        int max_money_with_last = 0;
        int max_money_without_last = 0;
        for (int i = start; i < end; ++i) {
            int current_total = houses[i] + max_money_without_last;
            max_money_without_last = max(max_money_without_last, max_money_with_last);
            max_money_with_last = current_total;
        }
        return max(max_money_with_last, max_money_without_last);
    }
};
```
