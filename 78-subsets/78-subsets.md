# Subsets
- 問題：https://leetcode.com/problems/subsets/
- 言語：C++

## Step 1

- 問題の概要
  - 配列の部分集合を返す問題。
  - [1, 2, 3] ->  [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

- バックトラッキングを使用すれば解ける
- 時間計算量：O(2 ^ n)
- 空間計算量：O(n)
- 実行時間は 1024 / 10 ^ 8 = 10 µs程度
  
```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<int> subset;
        vector<vector<int>> result;

        FindSubsets(nums, 0, subset, result);
        return result;
    }

private:
    void FindSubsets(const vector<int>& nums,
                     int start,
                     vector<int>& subset,
                     vector<vector<int>>& result) {
        result.push_back(subset);

        for (int i = start; i < nums.size(); ++i) {
            subset.push_back(nums[i]);
            FindSubsets(nums, i + 1, subset, result);
            subset.pop_back();
        }
    }

};
```

- ループで実装

```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<vector<int>> result;
        stack<SubsetAndIndex> subset_and_index;
        subset_and_index.push({vector<int>{}, 0});

        while (!subset_and_index.empty()) {
            auto [subset, index] = subset_and_index.top();
            subset_and_index.pop();
            if (index == nums.size()) {
                result.push_back(subset);
                continue;
            }
            subset_and_index.push({subset, index + 1});
            
            subset.push_back(nums[index]);
            subset_and_index.push({subset, index + 1});
        }

        return result;
    }

private:
    struct SubsetAndIndex {
        vector<int> subset;
        int index;
    };
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/51/changes#diff-ad01407803e073f539072a743ce608f45e09a7eb23b9d903e6b81ad196ea9c32
- ビットパターンによる解き方
- 各値を入れるか入れないかをビットに対応させる。
- 具体例
- nums = [1, 2, 3]
  - 0 0 0 -> [], 0 0 1 -> [1], 0 1 0 -> [2] 0 1 1 -> [1, 2]...


```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<vector<int>> result;

        for (int flag = 0; flag < (1 << nums.size()); ++flag) {
            vector<int> subset;
            for (int i = 0; i < nums.size(); ++i) {
                if (flag & (1 << i)) {
                    subset.push_back(nums[i]);
                }
            }
            result.push_back(subset);
        }

        return result;
    }
};
```

- https://github.com/dxxsxsxkx/leetcode/pull/51/changes#diff-966940b0933d8bd7099224eb9b5543081c459d94e7244b62080e2837307315e1
- 前に作ったsubsetを更新していく方法
- 自分的には読みやすかった
- current_sizeは変数名として微妙だが、良い名前が思いつかない
  
```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<vector<int>> result = {{}};

        for (auto num : nums) {
            int current_size = result.size();
            for (int i = 0; i < current_size; ++i) {
                vector<int> subset = result[i];
                subset.push_back(num);
                result.push_back(subset);
            }
        }

        return result;
    }
};
```

## Step 3
- バックトラッキングとビットパターンの書き方を確認

```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<int> subset;
        vector<vector<int>> result;

        FindSubsets(nums, 0, subset, result);
        return result;
    }

private:
    void FindSubsets(const vector<int>& nums,
                     int start,
                     vector<int>& subset,
                     vector<vector<int>>& result) {
        result.push_back(subset);

        for (int i = start; i < nums.size(); ++i) {
            subset.push_back(nums[i]);
            FindSubsets(nums, i + 1, subset, result);
            subset.pop_back();
        }
    }
};
```

```cpp
class Solution {
public:
    vector<vector<int>> subsets(const vector<int>& nums) {
        vector<vector<int>> result;

        for (int flag = 0; flag < (1 << nums.size()); ++flag) {
            vector<int> subset;
            for (int shift = 0; shift < nums.size(); ++shift) {
                if (flag & (1 << shift)) {
                    subset.push_back(nums[shift]);
                }
            }
            result.push_back(subset);
        }

        return result;
    }
};
```
