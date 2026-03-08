# Search in Rotated Sorted Array
- 問題：https://leetcode.com/problems/search-in-rotated-sorted-array/
- 言語：C++

## Step1
- 問題の概要
  - 前回と同様に、回転したリストが与えられる。リストからtargetを探索し、リスト内にあったらインデックスを返し、ない場合は-1を返す。時間計算量O(log n)で解くこと。

- 前回のように、一旦切れ目を探索してから、元の昇順のリストにして二分探索する解き方がパッと思いついた。
- 時間計算量もO(log n)で済みそう。
- 切れ目 <= target <= 最後尾ならこの範囲で二分探索して、それ以外なら0 ~ 切れ目 - 1の範囲で探索する

- n : nums.length
- 時間計算量：O(log n)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    int search(const vector<int>& nums, int target) {
        if (nums.empty()) {
            return -1;
        }
        
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] <= nums.back()) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        if (nums[left] <= target && target <= nums.back()) {
            return BinarySearch(nums, target, left, nums.size());
        }

        return BinarySearch(nums, target, 0, left);

    }

private:
    int BinarySearch(const vector<int>& nums, int target, int left, int right) {
        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        if (nums[left] != target) {
            return -1;
        }

        return left;
    }
};
```

- 似た処理が複数できてしまうのが気になった。
- 切れ目を探す二分探索も別の関数にした方が分かりやすいかも。

- 両方とも関数に分けたコード
```cpp
class Solution {
public:
    int search(const vector<int>& nums, int target) {
        if (nums.empty()) {
            return -1;
        }

        int min_index = FindMin(nums);

        if (target <= nums.back()) {
            return BinarySearch(nums, target, min_index, nums.size());
        } else {
            return BinarySearch(nums, target, 0, min_index);
        }
    }

private:
    int FindMin(const vector<int>& nums) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] <= nums.back()) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        return left;
    }

    int BinarySearch(const vector<int>& nums, int target, int left, int right) {
        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        if (nums[left] != target) {
            return -1;
        }

        return left;
    }
};
```

## Step2

- https://github.com/akmhmgc/arai60/pull/38/changes
- https://github.com/huyfififi/coding-challenges/pull/42/changes#diff-aff52e446f95248f073b12b4a41918d96e4efa22d100e14b948a8e226407682a
- 1回の二分探索で済む解き方もあった。
- 2回二分探索する方が個人的にわかりやすくて好み。

```cpp
class Solution {
public:
    int search(const vector<int>& nums, int target) {
        if (nums.empty()) {
            return -1;
        }

        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] == target) {
                return middle;
            }

            if (nums[middle] <= nums[right]) {
                if (nums[middle] < target && target <= nums[right]) {
                    left = middle + 1;
                } else {
                    right = middle - 1;
                }
            } else {
                if (nums[left] <= target && target < nums[middle]) {
                    right = middle - 1;
                } else {
                    left = middle + 1;
                }
            }
        }

        return -1;
    }
};
```

# Step3
- 最小値を見つけてから二分探索する方で3回

```cpp
class Solution {
public:
    int search(const vector<int>& nums, int target) {
        if (nums.empty()) {
            return -1;
        }

        int left = 0;
        int right = nums.size();
        int min_index = FindMinIndex(nums);

        if (target <= nums.back()) {
            left = min_index;
        } else {
            right = min_index;
        }

        return FindTargetIndex(nums, target, left, right);
    }

private:
    int FindMinIndex(const vector<int>& nums) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] <= nums.back()) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        return left;
    }

    int FindTargetIndex(const vector<int>& nums, int target, int left, int right) {
        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        if (nums[left] != target) {
            return -1;
        }

        return left;
    }
};
```
