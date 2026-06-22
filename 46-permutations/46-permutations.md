# Permutations
- 問題：https://leetcode.com/problems/permutations/
- 言語：C++

## Step 1

- 問題の概要
  - 異なる整数からなる配列 nums が与えられたとき、考えられるすべての順列を返してください。答えの順序は問いません。
  - 順列とは、配列のすべての要素を並べ替えたものです。

- numsのサイズが6なので、O(n!)でも大丈夫

- ナイーブな解き方も意外と思いつかない

- 手作業でやろうと考えたが、手が止まったので答えを見る
- 参考：https://github.com/dxxsxsxkx/leetcode/pull/50/changes#diff-f8cd4f05e981ac3daff6c72392a9f13fd6bec9b38438f264393020f8eeff1577
- 先頭に入れる数とそれ以外に分けて考える。再帰で回す。
- 先頭に入れる数を除いた配列を渡す。再帰で返ってきた配列の先頭に残しておいた数を挿入する

```cpp
class Solution {
public:
    vector<vector<int>> permute(const vector<int>& nums) {
        if (nums.size() == 1) {
            return {nums};
        }

        vector<vector<int>> permutations;

        for (int i = 0; i < nums.size(); ++i) {
            int head = nums[i];
            vector<int> remaining = nums;
            remaining.erase(remaining.begin() + i);

            auto permutations_remain = permute(remaining);

            for (auto p : permutations_remain) {
                p.insert(p.begin(), head);
                permutations.push_back(p);
            }
        }

        return permutations;
    }
};
```

## Step 2
- step1の解き方の末尾ver
- あんま変わってない
- 途中の配列のコピーが気になる

```cpp
class Solution {
public:
    vector<vector<int>> permute(const vector<int>& nums) {
        if (nums.size() == 1) {
            return {nums};
        }

        vector<vector<int>> permutations;

        for (int i = 0; i < nums.size(); ++i) {
            int last = nums[i];
            vector<int> remaining = nums;
            remaining.erase(remaining.begin() + i);

            for (auto p : permute(remaining)) {
                p.push_back(last);
                permutations.push_back(p);
            }
        }

        return permutations;
    }
};
```

- https://github.com/dxxsxsxkx/leetcode/pull/50/changes#diff-8a1802033781d0d31a529cb20b9d3b2b9967a6173a4d31b629de2acf24a681ac
- バックトラッキング：とりあえず進んで、条件を満たしていなかったら戻る みたいな方法
- swapを使った書き方より下の書き方の方がわかりやすかった
- バックトラッキングよりもstep1の方がより直感的だが、step1は無駄なコピーが発生してしまうのが難点
- https://github.com/Yuto729/leetcode/pull/54/changes

```cpp
class Solution {
public:
    vector<vector<int>> permute(const vector<int>& nums) {
        vector<vector<int>> permutations;
        vector<int> path;
        vector<uint8_t> visited(nums.size(), 0);

        find_permutations(nums, path, visited, permutations);
        return permutations;
    }

private:
    void find_permutations(const vector<int>& nums,
                           vector<int>& path,
                           vector<uint8_t>& visited,
                           vector<vector<int>>& permutations) {

        if (nums.size() == path.size()) {
            permutations.push_back(path);
            return;
        }

        for (int i = 0; i < nums.size(); ++i) {
            if (visited[i]) {
                continue;
            }

            path.push_back(nums[i]);
            visited[i] = 1;

            find_permutations(nums, path, visited, permutations);

            path.pop_back();
            visited[i] = 0;
        }
    }

};
```

- stackを使用したバックトラッキング
```cpp
class Solution {
public:
    vector<vector<int>> permute(const vector<int>& nums) {
        vector<vector<int>> permutations;
        stack<vector<int>> candidates;
        candidates.push({});

        while (!candidates.empty()) {
            auto candidate = candidates.top();
            candidates.pop();
            if (nums.size() == candidate.size()) {
                permutations.push_back(candidate);
                continue;
            }

            for (int i = 0; i < nums.size(); ++i) {
                if (find(candidate.begin(), candidate.end(), nums[i]) != candidate.end()) {
                    continue;
                }
                candidate.push_back(nums[i]);
                candidates.push(candidate);
                candidate.pop_back();
            }
        }

        return permutations;
    }

};
```

## Step 3
```cpp
class Solution {
public:
    vector<vector<int>> permute(const vector<int>& nums) {
        vector<int> candidate;
        vector<uint8_t> visited(nums.size(), 0);
        vector<vector<int>> permutations;

        FindPermutations(nums, candidate, visited, permutations);
        return permutations;
    }

private:
    void FindPermutations(const vector<int>& nums,
                          vector<int>& candidate,
                          vector<uint8_t>& visited,
                          vector<vector<int>>& permutations) {
        if (nums.size() == candidate.size()) {
            permutations.push_back(candidate);
            return;
        }

        for (int i = 0; i < nums.size(); ++i) {
            if (visited[i]) {
                continue;
            }
            candidate.push_back(nums[i]);
            visited[i] = 1;

            FindPermutations(nums, candidate, visited, permutations);
            candidate.pop_back();
            visited[i] = 0;
        }
    }
};
```
