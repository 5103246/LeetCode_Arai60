# Subarray Sum Equals K
* 問題: https://leetcode.com/problems/subarray-sum-equals-k/
* 言語: C++

## Step1

- 二重ループだと、4 * 10^8 / 10^8 = 4 sくらいなので、いけそうかも
- mapを使った解き方がないか考えてみたが、良い方法が思いつかなかったので二重ループで解いてみる。
- 時間計算量：O(n^2)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    int subarraySum(const vector<int>& nums, int k) {
        int count = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] == k) {
                ++count;
            }
            int sum = nums[i];
            for (int j = i + 1; j < nums.size(); ++j) {
                sum += nums[j];
                if (sum == k) {
                    ++count;
                }
            }
        }
        return count;
    }
};
```

## Step2

- https://github.com/Hurukawa2121/leetcode/pull/16/files
- 累積和を使って解いている人が多かった。
- 累積和はある区間の総和を求める処理を高速にできるものらしい。
- sum[j] - sum[i] = k (i < j)となる(i, j)の組を探す問題に変換できる。
- ややこしかったので、まずは単純なケースで考える

- nums[i] > 0のみの場合を考える
  - ループで回して、累積和を求め、setに入れる。
  - 同時に、sum[j] - kがsetにあるかを調べて、あったらcountを1増やす
```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int count = 0;
        int sum = 0;
        set<int> cumulative_sum{0};
        for (int num : nums) {
            sum += num;
            if (cumulative_sum.contains(sum - k)) {
                ++count;
            }
            cumulative_sum.insert(sum);
        }
        return count;
    }
};
```
- 次に、nums[i]が正負の場合を考える
- 正のみだと、累積和は重複しないのでsetでよかったが、負の数があると重複する
- なので、mapでカウントする
- 正のときと同様に、sum[j] - kがmapにあったら、その個数分カウントする
- sum[j] - k = 0のときのために、mapは{0, 1}で初期化する
- mapを調べる前に、mapに累積和を加えると、sum[j] = k = 0のときに1余分になるので、mapに加えるのは最後。

```cpp
class Solution {
public:
    int subarraySum(const vector<int>& nums, int k) {
        int subarray_count = 0;
        int cumulative_sum = 0;
        map<int, int> cumulative_sum_to_count{{0, 1}};
        for (int num : nums) {
            cumulative_sum += num;
            subarray_count += cumulative_sum_to_count[cumulative_sum - k];
            ++cumulative_sum_to_count[cumulative_sum];
        }
        return subarray_count;
    }
};
```
- 解き方の理解に時間がかかったが、問題を変換したり、単純な場合を考えて理解できた
- https://discord.com/channels/1084280443945353267/1233603535862628432/1252232545056063548

- https://github.com/colorbox/leetcode/pull/30/files#r1867764633
- 部分和を求めるexclusive_scanがあるらしい。

## Step3

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int subarray_count = 0;
        int cumulative_sum = 0;
        map<int, int> cumulative_sum_to_count{{0, 1}};
        for (int num : nums) {
            cumulative_sum += num;
            subarray_count += cumulative_sum_to_count[cumulative_sum - k];
            ++cumulative_sum_to_count[cumulative_sum];
        }
        return subarray_count;
    }
};
```
