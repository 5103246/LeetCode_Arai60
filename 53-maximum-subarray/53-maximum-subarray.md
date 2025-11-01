# Maximum Subarray
* 問題：https://leetcode.com/problems/maximum-subarray/
* 言語：C++

## Step1

- まず、累積和を連想した。
- 累積和のリストを作成し、累積和の中で最大値と最小値を求めて、最大値 - 最小値を求めれば解けそうと思ったが、
- [-2, 1] や [-2, 0, -1]のケースが解けない
  
- 累積和に追加しながら、累積和と最小値の差を見ていく。
- ループを回すごとに、最小値を更新する。
- その差が最大のものを返す。
- 他の解き方は思いつかなかった。
- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        vector<int> cumulative_sums{0};
        int cumulative_min = 0;
        int max_sum = nums[0];
        for (auto num : nums) {
            int sum = cumulative_sums.back() + num;
            cumulative_sums.push_back(sum);
            max_sum = max(max_sum, sum - cumulative_min);
            cumulative_min = min(cumulative_min, sum);
        }
        return max_sum;
    }
};
```

- 累積和のリストはいらなかった。
- 時間計算量：O(n）
- 空間計算量：O(1)
- 他の解き方を考えてみたが、arrayを半分に分け、左右のsubarray中で、最大のsubarrayを見つけて、くっつけるイメージの解き方くらいしか思いつかなかった。
- あまり自信はないし、実装が少し大変そう。

```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        int cumulative_sum = 0;
        int min_cumulative_sum = 0;
        int max_sum = nums[0];
        for (auto num : nums) {
            cumulative_sum += num;
            max_sum = max(max_sum, cumulative_sum - min_cumulative_sum);
            min_cumulative_sum = min(min_cumulative_sum, cumulative_sum);
        }
        return max_sum;
    }
};
```

## Step2

- https://github.com/fhiyo/leetcode/pull/33/files
- カダネのアルゴリズムというので解いていた。
- https://github.com/sakupan102/arai60-practice/pull/33/files
- https://github.com/sakupan102/arai60-practice/pull/33/files#r1611415355
- https://github.com/olsen-blue/Arai60/pull/32/files#r1963455912
- 標高差を例に考えると、少ししっくりきた。
- 株価？チャートみたいなのをイメージして図にしたら理解できた。
  
|
|
|  /\            
| /--\----------/\------
|      \       /   \
|        \    /      \
|          \/
            ↑ ここを基準(0)にする

```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        int max_sum = nums[0];
        int current_sum = 0;
        for (auto num : nums) {
            current_sum = max(current_sum, 0) + num;
            max_sum = max(max_sum, current_sum);
        }
        return max_sum;
    }
};
```

- わざわざ0に更新しなくてもいい。
- 自分的には、図のイメージが一番しっくり来たので、0に更新するほうが少しわかりやすいかも
- でも、こちらの方がシンプルかな？

```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        int max_sum = nums[0];
        int current_sum = 0;
        for (auto num : nums) {
            current_sum = max(current_sum + num, num);
            max_sum = max(max_sum, current_sum);
        }
        return max_sum;
    }
};
```

- https://github.com/irohafternoon/LeetCode/pull/35/files
- 上がり幅をリストに追加していく解き方。追加する際は、カダネのアルゴリズムのように前の合計値とnumの合計、numの大きい方を追加する。
- 最後に、リストの中での最大値をmax_elementで返す。
  
```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        vector<int> max_sum_so_far(nums.size());
        max_sum_so_far[0] = nums[0];
        for (int i = 0; i < nums.size() - 1; ++i) {
            max_sum_so_far[i + 1] = max(nums[i + 1], max_sum_so_far[i] + nums[i + 1]);
        }
        return *max_element(max_sum_so_far.begin(), max_sum_so_far.end());
    }
};
```

## Step3
- カダネのアルゴリズムで3回

```cpp
class Solution {
public:
    int maxSubArray(const vector<int>& nums) {
        int current_sum = 0;
        int max_sum = nums[0];
        for (auto num : nums) {
            current_sum = max(num, current_sum + num);
            max_sum = max(max_sum, current_sum);
        }
        return max_sum;
    }
};
```
