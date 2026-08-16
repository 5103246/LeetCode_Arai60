# Is Subsequence

- 問題：https://leetcode.com/problems/is-subsequence/
- 言語：C++

## Step 1
- 問題の概要：
  - 文字列tに部分文字列sが含まれているかを調べる
  - 部分文字列
    
- 一文字ずつ比較して、文字列sが含まれているかを調べる
- 時間計算量：O(t.length)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int s_index = 0;
        for (auto ch : t) {
            if (s_index == s.size()) {
                break;
            }
            if (ch == s[s_index]) {
                ++s_index;
            }
        }

        if (s_index == s.size()) {
            return true;
        }
        return false;
    }
};
```

- > Follow up: Suppose there are lots of incoming s, say s1, s2, ..., sk where k >= 109,
   and you want to check one by one to see if t has its subsequence.
   In this scenario, how would you change your code?
- sの長さが10 ^ 9まである場合、どうするか
- 左右からダブルポインタで同時に比較していくとか？ 10 ^ 9 / 2だからあんまり変わらないか
- O(nlogn)でできないか
- 考えてみたが、止まったので次に進む

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/57/changes
- https://github.com/potrue/leetcode/pull/57/changes
- 辞書型にして二分探索する方法があるみたい
- そもそも、フォローアップを誤解していた
  - sの長さが10 ^ 9じゃなくて、10 ^ 9個のsが来るときどうするかという話だった
- https://github.com/naoto-iwase/leetcode/pull/58/changes
- https://github.com/naoto-iwase/leetcode/pull/58/changes#r2510229524
- この方法なら一回のsの時間計算量はO(s log t)になる。

```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        if (s.empty()) {
            return true;
        }
        if (t.empty()) {
            return false;
        }

        map<char, vector<int>> ch_to_positions;
        for (int i = 0; i < t.size(); ++i) {
            ch_to_positions[t[i]].push_back(i);
        }

        int last_used_position = -1;
        for (auto ch : s) {
            vector<int> positions_in_t = ch_to_positions[ch];
            if (positions_in_t.empty()) {
                return false;
            }
            auto next_position = upper_bound(positions_in_t.begin(), positions_in_t.end(), last_used_position);
            if (next_position == positions_in_t.end()) {
                return false;
            }
            last_used_position = *next_position;
        }

        return true;
    }
};
```

- step1はより簡潔に書ける
```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int s_index = 0;

        for (auto ch : t) {
            if (s_index < s.size() && s[s_index] == ch) {
                ++s_index;
            }
        }

        return s_index == s.size();
    }
};
```

- https://github.com/dxxsxsxkx/leetcode/pull/57/changes
- この書き方のほうがわかりやすいかも
- 手作業として考えたときにこっちのほうが自然な気がする
```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        if (s.empty()) {
            return true;
        }
        if (t.empty()) {
            return false;
        }

        int s_index = 0;
        for (auto ch : t) {
            if (s[s_index] == ch) {
                ++s_index;
            }
            if (s_index == s.size()) {
                return true;
            }
        }

        return false;
    }
};
```

## Step 3
```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        if (s.empty()) {
            return true;
        }
        if (t.empty()) {
            return false;
        }

        int s_index = 0;
        for (auto ch : t) {
            if (s[s_index] == ch) {
                ++s_index;
            }
            if (s_index == s.size()) {
                return true;
            }
        }

        return false;
    }
};
```
