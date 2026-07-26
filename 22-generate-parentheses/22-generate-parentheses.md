# Generate Parentheses

- 問題：https://leetcode.com/problems/generate-parentheses/
- 言語：C++

## Step 1
- 問題の概要：
  - n個の"()"の組み合わせを求める。
  - 例：n = 3
  - 答：[()()(), ((())), (())(), ()(()), (()())]
  - 誤：)))(((, )()()( などは誤り
  - 1 <= n <= 8

- 考え方としては、"()"をどこに入れるかを考えればよさそう
- 例えば、n = 3の場合、
  - "" に入れると、"()"
  - "()" に入れる場合、先頭か間に入れる "()()", "(())"
  - "()()" に入れる場合、先頭か左括弧、右括弧の間 "()()()", "(())()", "()(())"
  - "(())" に入れる場合、"(()())", "((()))"
- バックトラッキングで解けそう
- この方針で実装
- if文の条件がわかりづらいかも.

```cpp
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        string comb = "()";
        vector<string> result;

        auto find_combinations = [&](auto&& self, int start) -> void {
            if (comb.size() == n * 2) {
                result.push_back(comb);
                return;
            }

            for (int i = start; i < comb.size(); ++i) {
                if (comb[i] != ')' && i != 0) {
                    continue;
                }
                comb.insert(i, kParentheses);
                self(self, i);
                comb.erase(i, 2);
            }
        };

        find_combinations(find_combinations, 0);
        return result;
    }

private:
    static constexpr string kParentheses = "()";
};
```


## Step 2
- https://github.com/hemispherium/LeetCode_Arai60/pull/10#discussion_r2618518592
- > 英単語から文字を削って識別子にすると、読み手の認知負荷が上がる場合があります。原則としてフルスペルで記述することをお勧めします。
- combと略すのもあまりよくないらしい
- https://github.com/5103246/LeetCode_Arai60/pull/49#discussion_r3629017004
- C++では、inner functionをラムダ式で書かないらしい

- https://github.com/dxxsxsxkx/leetcode/pull/53/changes#diff-2970b3999f1163434a1d9ce120435a5d535a22d194f21e4f8a8e11b01243c301
  - '('と')'の数をカウントしたやり方。こっちの方がわかりやすいかも
  - https://github.com/dxxsxsxkx/leetcode/pull/53/changes#r2997320181
    - 終了条件の話
    - たしかに、この解き方の場合は気にする必要があるかもしれない。

```cpp
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        string combination;
        vector<string> result;

        FindCombinations(n, 0, 0, combination, result);
        return result;
    }

private:
    void FindCombinations(int n, 
                          int num_open,
                          int num_close,
                          string& combination, 
                          vector<string>& result) {
        if (combination.size() == n * 2) {
            result.push_back(combination);
            return;
        }

        if (num_open < n) {
            combination.push_back('(');
            FindCombinations(n, num_open + 1, num_close, combination, result);
            combination.pop_back();
        }

        if (num_close < num_open) {
            combination.push_back(')');
            FindCombinations(n, num_open, num_close + 1, combination, result);
            combination.pop_back();
        }
    }
};
```

- https://github.com/Ryotaro25/leetcode_first60/pull/58/changes#r1997665903
  - カタラン数というのがあるらしい
  - 括弧を正しく並べる方法や格子状の数え上げなどで使う
  - https://ja.wikipedia.org/wiki/%E3%82%AB%E3%82%BF%E3%83%A9%E3%83%B3%E6%95%B0

- https://github.com/naoto-iwase/leetcode/pull/54/changes
  - バックトラッキングのloop
  - 再帰よりloopのほうが読みやすくていい
  - 終了条件は、open_remain == 0 && close_remain == 0 がわかりやすい
  
```cpp
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        stack<CombinationState> states;
        states.emplace("", n, n);

        while (!states.empty()) {
            auto [combination, open_remain, close_remain] = states.top();
            states.pop();

            if (open_remain == 0 && close_remain == 0) {
                result.push_back(combination);
                continue;
            } 
            if (open_remain > 0) {
                states.emplace(combination + '(', open_remain - 1, close_remain);
            }
            if (close_remain > open_remain) {
                states.emplace(combination + ')', open_remain, close_remain - 1);
            }
        }

        return result;
    }

private:
    struct CombinationState {
        string combination;
        int open_remain;
        int close_remain;
    };
};
```

## Step 3
- loop backtrackingで3回
```cpp
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        stack<CombinationState> states;
        states.emplace("", n, n);

        while (!states.empty()) {
            auto [combination, open_remain, close_remain] = states.top();
            states.pop();

            if (open_remain == 0 && close_remain == 0) {
                result.push_back(combination);
                continue;
            } 
            if (open_remain > 0) {
                states.emplace(combination + '(', open_remain - 1, close_remain);
            }
            if (close_remain > open_remain) {
                states.emplace(combination + ')', open_remain, close_remain - 1);
            }
        }

        return result;
    }

private:
    struct CombinationState {
        string combination;
        int open_reamin;
        int close_remain;
    };
};
```
