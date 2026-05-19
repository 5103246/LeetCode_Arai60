# Longest Substring Without Repeating Characters
- 問題：https://leetcode.com/problems/longest-substring-without-repeating-characters/description/
- 言語：C++

## Step1
- 問題の概要
  - 重複なしの部分文字列の長さで最大のものを返す（部分文字列は空白なしで構成されている連続した文字列）
  - 文字列は英字、数字、記号、スペースで構成されている

- 先頭から文字列を作っていく。
- 文字列の中に含まれていない文字だったら文字列に加える。
- 重複が見つかるまでカウントしていく
- 時間計算量：O(n ^ 2)
- 空間計算量：O(n)
- 実行時間は (5 * 10 ^ 4) ^ 2 / 10 ^ 8 = 25 s程度
- 実際には、部分文字列は最大60字程度くらい？大文字小文字を区別しても100字もないと考えると、実行時間は0.2 s程度
- 良い解き方が出てこなかったので、これで実装する

- スペースを空白と勘違いしていたため、wrong answerとなった
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.empty()) {
            return 0;
        }
        int longest = 1;
        string_view sv = s;
        
        for (int i = 0; i < s.size(); ++i) {
            if (sv[i] == ' ') { 
                continue;
            }
            int count = 1;
            for (int j = i + 1; j < s.size(); ++j) {
                auto substring = sv.substr(i, j - i);
                if (substring.contains(sv[j]) || sv[j] == ' ') {
                    break;
                }

                ++count;
                longest = max(longest, count);
            }
        }

        return longest;
    }
};
```

- 修正して、スペースも文字列に含めたらaccept
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.empty()) {
            return 0;
        }

        int longest = 1;
        string_view sv = s;
        
        for (int i = 0; i < s.size(); ++i) {
            int count = 1;
            for (int j = i + 1; j < s.size(); ++j) {
                auto substring = sv.substr(i, j - i);
                if (substring.contains(sv[j])) {
                    break;
                }

                ++count;
                longest = max(longest, count);
            }
        }

        return longest;
    }
};
```
- 実装した後に気づいたが、containsを使っているのでO(n ^ 3)だった
- n ^ 3でも、5 * 10 ^ 4 * 100 ^ 2 = 5 * 10 ^ 8くらいなので、timeoutしなかったのか。

- 他に良い解き方がないか考えてみる
- 1文字ずつずらしていたが、重複している文字より前はカウントしても無駄になる
　- 重複していたら、そこより前は切り離せばいい。切り離した次の位置の文字からカウントしていく。
  - 重複している文字の位置をどう調べるか
  - mapで文字と位置を管理する
  - 時間計算量：O(n log w) (n: 5 * 10 ^ 4, w: 128)
  - 空間計算量：O(w)

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.empty()) {
            return 0;
        }

        map<char, int> letter_to_index = {{s[0], 0}};
        int longest = 1;
        int start = 0;

        for (int i = 1; i < s.size(); ++i) {
            if (letter_to_index.contains(s[i]) && start <= letter_to_index[s[i]]) {
                start = letter_to_index[s[i]] + 1;
            }
            letter_to_index[s[i]] = i;
            longest = max(longest, i - start + 1);
        }

        return longest;
    }
};
```

- 文字と記号がasciiのみならvectorでも解けるな
- 長さ128のvectorを作って、現れた文字のインデックスを書いていく
- 時間計算量：O(n)
- 空間計算量：O(w)

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> last_positions(128, -1);
        int longest = 0;
        int start = 0;

        for (int i = 0; i < s.size(); ++i) {
            int letter = s[i];

            if (start <= last_positions[letter]) {
                start = last_positions[letter] + 1;
            }

            last_positions[letter] = i;
            longest = max(longest, i - start + 1);
        }

        return longest;
    }
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/48/changes
  - mapじゃなくて、setを使った解き方
  - setにあったら、重複した文字がくるまでleftを進めて、前の文字をsetから消していく
- https://github.com/naoto-iwase/leetcode/pull/49/changes
  - startの更新はmaxでできるのか
  - left, rightに人が多い印象を受けた
  - https://github.com/naoto-iwase/leetcode/pull/49/changes#r2485616838
  　- char_to_last_seen_index、last_char_to_indexなどの変数がよかったかな
- setを使った解き方
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        set<char> letter_seen;

        int longest = 0;
        int left = 0;

        for (int right = 0; right < s.size(); ++right) {
            while (letter_seen.contains(s[right])) {
                letter_seen.erase(s[left]);
                ++left;
            }
            letter_seen.insert(s[right]);
            longest = max(longest, right - left + 1);
        }

        return longest;
    }
};
```
- 個人的には、mapを使った方が読みやすかった

## Step 3
- mapで3回
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        map<char, int> letter_to_last_index;
        int longest = 0;
        int left = 0;

        for (int right = 0; right < s.size(); ++right) {
            if (letter_to_last_index.contains(s[right])) {
                left = max(left, letter_to_last_index[s[right]] + 1);
            }

            letter_to_last_index[s[right]] = right;
            longest = max(longest, right - left + 1);
        }

        return longest;
    }
};
```
