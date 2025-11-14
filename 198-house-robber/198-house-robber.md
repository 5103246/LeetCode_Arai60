# House Robber
* 問題：https://leetcode.com/problems/house-robber/
* 言語：C++

## Step1

- 与えられたリストの合計値の最大を求める。ただし、隣り合う値は使ってはいけない。
- [2, 7, 9, 3, 1] なら、2 → 9 → 1 で答えは12
- [9, 6, 1, 10, 2]なら、9 → 10 で答えは19
- 合計値と一つ前に足した要素のインデックスがわかれば、解けそう
- ひとつ前に足した要素のインデックスを見て、隣だったら、値を比較し大きい方を足す。隣じゃなかったら普通に足す。
- これだと、[1,2,3,4]のような増加していく部分で、最大値が求まらない
- 直前に足したのが隣の要素で、自分の方が大きかったら、さらに一つ前の要素を足すようにする（ループで探していく？）
- テーブルを使ったほうが無駄が省けそう
  
- 一旦、別の解き方を考える
- 計算結果を配列に保存していく。
- [1, 2, 3, 4]なら、[1, 2, 4, 6]のように、i - 1以前の計算結果の中で最大のものを探してそれの合計値を入れていく。
- [9, 6, 1, 10, 2]なら、[9, 6, 10]となり、nums[3] = 10では作成した配列のi - 1より前を探して9が最大なので[9,6, 10, 19, 12]となる。
- こちらの方がシンプルで書きやすそう
- n : nums.length
- 時間計算量：O(n ^ 2)
- 空間計算量：O(n)
- 実行時間は、10 ^ 4 / 10 ^ 8 = 100 µs程度
- とりあえず、素直に実装した。

```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        if (nums.size() == 1) {
            return nums[0];
        }
        vector<int> sums(nums.size());
        sums[0] = nums[0];
        sums[1] = nums[1];
        for (int i = 2; i < nums.size(); ++i) {
            int total = 0;
            for (int j = 0; j < i - 1; ++j) {
                if (total < sums[j]) {
                    total = sums[j];
                }
            }
            sums[i] = nums[i] + total;
        }
        return max(sums[nums.size() - 2], sums[nums.size() - 1]);
    }
};
```

- 上の解き方では、i - 1より前の最大値を知りたい。
- リストを使わなくても、その都度最大値を更新していけば時間計算量がO(n)で済みそう
- 空間計算量はO(1)で、実行時間も1 µs程度になる。

```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        if (nums.empty()) {
            return 0;
        }
        if (nums.size() == 1) {
            return nums[0];
        }
        int nonadjacent_amount_money = nums[0];
        int adjacent_amount_money = nums[1];
        for (int i = 2; i < nums.size(); ++i) {
            int total_amount = nums[i] + nonadjacent_amount_money;
            nonadjacent_amount_money = max(nonadjacent_amount_money, adjacent_amount_money);
            adjacent_amount_money = total_amount;
        }
        return max(nonadjacent_amount_money, adjacent_amount_money);
    }
};
```

## Step2

- https://github.com/nanae772/leetcode-arai60/pull/34/files#r2419654876
- 2変数の初期値を0にすれば、入力リストを先頭から見ていけるか。あと、空の場合やサイズが1個の場合のif文もなくなる。
- 0, 0 [1, 2, 3, 4]　入力リストの先頭に0を2つ付ける感じ
  
- https://github.com/nanae772/leetcode-arai60/pull/34/files#r2419654876
- https://github.com/Ryotaro25/leetcode_first60/pull/36/files#diff-de9ce60116465f45947d326863a7e35c93e2a833a0d935ef880ed4cd4787230c
- 今いる家を含めた最大金額を考えていく解き方。
- 少し、理解するのに手間取ったが、よりシンプルな書き方だと思う。

- https://github.com/docto-rin/leetcode/pull/40/files
- 変数名が、without_last, with_lastは結構良い気がする
- https://github.com/olsen-blue/Arai60/pull/35/files
- 変数名two_before_max, one_before_max
- https://github.com/nanae772/leetcode-arai60/pull/34/files
- 変数名max_robbed_previous, max_skipped_previoud
- 自分的には、two_before_maxはすっと入ってくる感じだが、without_lastの方がより適していると感じた
- max_money_without_lastでいいかな。
  
- 最大金額を考えていく解き方
```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        int max_money_with_last = 0;
        int max_money_without_last = 0;
        for (auto num : nums) {
            int current_max = max(max_money_with_last, max_money_without_last + num);
            max_money_without_last = max_money_with_last;
            max_money_with_last = current_max;
        }
        return max_money_with_last;
    }
};
```

- step1の解き方
```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        int max_money_with_last = 0;
        int max_money_without_last = 0;
        for (auto num : nums) {
            int total = num + max_money_without_last;
            max_money_without_last = max(max_money_without_last, max_money_with_last);
            max_money_with_last = total;
        }
        return max(max_money_with_last, max_money_without_last);
    }
};
```

- 書いてみた感じ、やはりstep1の方が素直でわかりやすく感じたのでstep3ではそれで書く。

## Step3

```cpp
class Solution {
public:
    int rob(const vector<int>& nums) {
        int max_money_with_last = 0;
        int max_money_without_last = 0;
        for (auto num : nums) {
            int current_total = max_money_without_last + num;
            max_money_without_last = max(max_money_without_last, max_money_with_last);
            max_money_with_last = current_total;
        }
        return max(max_money_with_last, max_money_without_last);
    }
};
```
