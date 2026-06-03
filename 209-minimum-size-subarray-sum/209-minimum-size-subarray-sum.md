# Minimum Size Subarray Sum

- 問題：https://leetcode.com/problems/minimum-size-subarray-sum/
- 言語：C++

## Step 1

- 問題の概要
  - targetと配列が与えられる。和がtarget以上となる部分配列の最小の長さを返す。ない場合は、0を返す

- 解き方はスライディングウィンドウ、累積和がぱっと出てきた
- スライディングウィンドウは時間計算量がO(n), 空間計算量がO(1)
- スライディングウィンドウが簡単そうなので、実装
  

```cpp
class Solution {
public:
    int minSubArrayLen(int target, const vector<int>& nums) {
        int left = 0;
        int right = 0;
        int total = 0;
        int min_length = nums.size() + 1;

        while (left < nums.size()) {
            while (right < nums.size() && total < target) {
                total += nums[right];
                ++right;
            }

            if (total >= target) {
                min_length = min(min_length, right - left);
            }

            total -= nums[left];
            ++left;
        }

        if (min_length == nums.size() + 1) {
            return 0;
        }
        return min_length;
    }
};
```

- O(n log n)の解き方も考えてみる
- これは累積和を使ったらできそう
  - 累積和をリストに保存しておく
  - 前から見ていき、targetとの差を二分探索で探す
  - これは時間計算量がO(n log n)、空間計算量はO(n)
  
```cpp
class Solution {
public:
    int minSubArrayLen(int target, const vector<int>& nums) {
        vector<int> prefix(nums.size() + 1, 0);
        for (int i = 0; i < nums.size(); ++i) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int min_length = prefix.size();
        for (int i = nums.size(); i > 0; --i) {
            int diff = prefix[i] - target;
            if (diff < 0) {
                break;
            }

            min_length = min(min_length, i - BinarySearch(0, i, diff, prefix));
        }

        if (min_length == prefix.size()) {
            return 0;
        }
        return min_length;

    }

private:
    int BinarySearch(int left, int right, int target, const vector<int>& prefix) {
        while (left <= right) {
            int middle = left + (right - left) / 2;

            if (prefix[middle] == target) {
                return middle;
            } else if (prefix[middle] > target) {
                right = middle - 1;
            } else {
                left = middle + 1;
            }
        }

        return right;
    }
};
```

## Step2
- 分割統治法を使った解き方
  - https://github.com/naoto-iwase/leetcode/pull/50/changes#r2483829525
  - 中央を跨ぐ場合と跨がない場合で分類して考えるのが難しい
- https://github.com/naoto-iwase/leetcode/pull/50/changes step1
  - for-whileで書いていた
  - forで右を進めて、targetを超えたら左を進める流れだった。
  - 余計なif文がいらないのでこっちのほうがよさそう
  - totalよりもsubarray_sumの方が情報量があっていい
    
- 改良版
```cpp
class Solution {
public:
    int minSubArrayLen(int target, const vector<int>& nums) {
        int left = 0;
        int subarray_sum = 0;
        int min_length = nums.size() + 1;

        for (int right = 0; right < nums.size(); ++right) {
            subarray_sum += nums[right];

            while (subarray_sum >= target) {
                min_length = min(min_length, right - left + 1);
                subarray_sum -= nums[left];
                ++left;
            }
        }

        if (min_length == nums.size() + 1) {
            return 0;
        }
        return min_length;
    }
};
```

## Step 3
```cpp
class Solution {
public:
    int minSubArrayLen(int target, const vector<int>& nums) {
        int left = 0;
        int subarray_sum = 0;
        int min_length = nums.size() + 1;

        for (int right = 0; right < nums.size(); ++right) {
            subarray_sum += nums[right];

            while (subarray_sum >= target) {
                min_length = min(min_length, right - left + 1);
                subarray_sum -= nums[left];
                ++left;
            }
        }

        if (min_length == nums.size() + 1) {
            return 0;
        }
        return min_length;
    }
};
```
