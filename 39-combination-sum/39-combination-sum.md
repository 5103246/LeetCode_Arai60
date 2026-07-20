# Combination Sum
- 問題：https://leetcode.com/problems/combination-sum/description/
- 言語：C++

## Step 1
- 問題の概要
  - 整数のリストが与えられる。要素の和がtargetになるすべての組み合わせを返す。要素は何回でも使用できる。
  - 1 <= target <= 40
  - 2 <= candidates[i] <= 40
  - 1 <= candidates.length <= 30
  - 例：[2, 3, 5] target = 8 answer = [[2,2,2,2], [2,3,3], [3,5]]
  - 例：[2] target = 1 answer = []

- バックトラッキングでいけそう
  - 昇順にソートしてから、先頭から組み合わせを考えていく
  - 合計値がtargetを超えたら、それ以降は調べる必要がないのでreturnする
- 一応書けたが、 引数が多いのが気になる

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(const vector<int>& candidates, int target) {
        vector<int> sorted_candidates = candidates;
        sort(sorted_candidates.begin(), sorted_candidates.end());
        vector<vector<int>> result;
        vector<int> comb;
        
        FindCombinationSum(sorted_candidates, target, 0, 0, comb, result);
        return result;
    }

private:
    void FindCombinationSum(const vector<int>& candidates,
                            int target,
                            int sum,
                            int start,
                            vector<int>& comb,
                            vector<vector<int>>& result) {
        if (sum == target) {
            result.push_back(comb);
            return;
        }

        for (int i = start; i < candidates.size(); ++i) {
            if (sum + candidates[i] > target) {
                return;
            }
            comb.push_back(candidates[i]);
            sum += candidates[i];
            FindCombinationSum(candidates, target, sum, i, comb, result);
            comb.pop_back();
            sum -= candidates[i];
        }
    }
};
```

- while + stack 版
```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(const vector<int>& candidates, int target) {
        vector<int> sorted_candidates = candidates;
        sort(sorted_candidates.begin(), sorted_candidates.end());

        stack<CombinationState> states;
        states.emplace(vector<int>{}, 0, 0);
        vector<vector<int>> result;

        while (!states.empty()) {
            auto [comb, sum, start] = states.top();
            states.pop();

            if (sum == target) {
                result.push_back(comb);
                continue;
            }
        
            for (int i = start; i < sorted_candidates.size(); ++i) {
                if (sum + sorted_candidates[i] > target) {
                    break;
                }
                vector<int> next_comb = comb;
                next_comb.push_back(sorted_candidates[i]);
                states.emplace(next_comb, sum + sorted_candidates[i], i);
            }
        }

        return result;
    }

private:
    struct CombinationState {
        vector<int> comb;
        int sum;
        int start;
    };
};
```

## Step 2
- https://github.com/fhiyo/leetcode/pull/52/changes#r1690161771
- https://github.com/naoto-iwase/leetcode/pull/53/changes
 - > [A, A] まで使うことが確定していて B 以降しか使ってはいけないという状況下で、
      > - B を一つ使うか、C 以降しか使ってはいけないか、に分岐する。
      > - B の使う数を列挙して分岐し、C 以降しか使ってはいけないに遷移する。
      > - 次の1個が、B, C, D, E, F... である場合に分岐する。
- 自分の実装は3つ目のパターンか
- https://github.com/naoto-iwase/leetcode/pull/53/changesの実装1と実装3が他のパターン
- パターン2つ目は結構わかりやすかった

- step 1の実装を少し改善してみた
- remainで引数を減らしたので少し読みやすくはなっている

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(const vector<int>& candidates, int target) {
        vector<int> sorted_candidates = candidates;
        sort(sorted_candidates.begin(), sorted_candidates.end());
        vector<int> comb;
        vector<vector<int>> result;

        FindCombinations(sorted_candidates, target, 0, comb, result);
        return result;
    }

private:
    void FindCombinations(const vector<int>& candidates,
                          int remaining,
                          int start,
                          vector<int>& comb,
                          vector<vector<int>>& result) {
        if (remaining == 0) {
            result.push_back(comb);
            return;
        }

        for (int i = start; i < candidates.size(); ++i) {
            if (remaining < candidates[i]) {
                return;
            }
            comb.push_back(candidates[i]);
            FindCombinations(candidates, remaining - candidates[i], i, comb, result);
            comb.pop_back();
        }
    }
};
```

- https://github.com/dxxsxsxkx/leetcode/pull/52/changes#r2971638946
- > 引数がやや多いように感じました。その内3個は毎回同じなので、lambdaを使うか、変わらない部分はstructでまとめるとかあたりですかね。
- lambdaで実装してみた
- だいぶすっきりしてていい

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(const vector<int>& candidates, int target) {
        vector<int> sorted_candidates = candidates;
        sort(sorted_candidates.begin(), sorted_candidates.end());
        vector<int> comb;
        vector<vector<int>> result;

        auto find_combinations = [&](auto&& self, int remaining, int start) -> void {
            if (remaining == 0) {
                result.push_back(comb);
                return;
            }

            for (int i = start; i < sorted_candidates.size(); ++i) {
                int candidate = sorted_candidates[i];
                if (remaining < candidate) {
                    break;
                }

                comb.push_back(candidate);
                self(self, remaining - candidate, i);
                comb.pop_back();
            }
        };

        find_combinations(find_combinations, target, 0);
        return result;
    }
};
```

## Step 3
- lambdaの練習として3回

```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(const vector<int>& candidates, int target) {
        vector<int> sorted_candidates = candidates;
        sort(sorted_candidates.begin(), sorted_candidates.end());

        vector<int> comb;
        vector<vector<int>> result;

        auto find_combinations = [&](auto&& self, int remaining, int start) -> void {
            if (remaining == 0) {
                result.push_back(comb);
                return;
            }

            for (int i = start; i < sorted_candidates.size(); ++i) {
                int candidate = sorted_candidates[i];
                if (remaining < candidate) {
                    break;
                }
                comb.push_back(candidate);
                self(self, remaining - candidate, i);
                comb.pop_back();
            }
        };

        find_combinations(find_combinations, target, 0);
        return result;
    }
};
```
