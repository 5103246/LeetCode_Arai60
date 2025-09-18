# Group Anagrams
* 問題: https://leetcode.com/problems/group-anagrams/
* 言語: C++

## Step1

- mapのkeyに、昇順にソートした文字列を入れ、valueにアナグラムのリストを入れる。
- ソートした文字列がmapにあったら、valueのリストにpush
- n : 入力長、m : strs[i].length
- 時間計算量：O(n * m log m)
- 空間計算量：O(n)
- 実行時間は、(10^4) * (100) * log(100) ≃ 10^6、 10^6 / 10^8 = 10^(-2) s なので、大体10 ms

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<string, vector<string>> ascending_to_anagram;
        for (const string& word : strs) {
            string ascending = word;
            sort(ascending.begin(), ascending.end());
            if (ascending_to_anagram.contains(ascending)) {
                ascending_to_anagram[ascending].push_back(word);
                continue;
            }
            ascending_to_anagram[ascending] = {word};
        }

        vector<vector<string>> anagrams;
        for (auto [_, anagram] : ascending_to_anagram) {
            anagrams.push_back(anagram);
        }
        return anagrams;
    }
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/14/files
- https://github.com/mura0086/arai60/pull/16/files
- https://github.com/Hurukawa2121/leetcode/pull/12/files
- https://github.com/colorbox/leetcode/pull/26/files
- https://github.com/Ryotaro25/leetcode_first60/pull/13/files
- https://github.com/maeken4/Arai60/pull/11/files

- std::transformを使った書き方もあるらしい。
- std::rangesを使った書き方もある。
- 上の二つの書き方が結構使うのだろうか。自分としては見慣れていないので、わかりにくい感じがする。
- 変数名は、sorted_word_to_anagrams、word_for_sort, grouped_anagramsがいいと感じた。

- 他の解き方
- 各文字の出現頻度をキーとする方法（時間計算量：O(n * m)、空間計算量：O(n)）
  - ソートした方がコードもコンパクトだし、読みやすいと感じた
  - EBCDIC 等、文字コードがアルファベット順に並んでいない文字コード表を扱う処理系の場合、コードが動かなくなるらしい。
  - アルファベット順に並んでいない文字コードがあるということは頭の中に入れといたほうがいい。
  - ソートを使う方法と比べて、時間計算量の差はそれほどないし、採用するうまみがないと思う。
    
```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<vector<int>, vector<string>> frequency_to_anagrams;
        for (const string& word : strs) {
            vector<int> frequency(26);
            for (const char& letter : word) {
                if (letter < 'a' || 'z' < letter) {
                    continue;
                }
                ++frequency[letter - 'a'];
            }
            frequency_to_anagrams[frequency].push_back(move(word));
        }

        vector<vector<string>> anagram_groups;
        for (auto& [_, group] : frequency_to_anagrams) {
            anagram_groups.push_back(move(group));
        }
        return anagram_groups;
    }
};
```

- https://github.com/Ryotaro25/leetcode_first60/pull/13/files#r1636916877
- 上のリンクはstd::moveについての議論。自分もmoveについて理解してなかったので、大変勉強になった。
- 最終的なサイズが分かっているので、reserveを入れることによって、余分な動的メモリ確保とムーブがなくなる。
- leetcode上の計測だが、Step1のコードが約20 ~ 30 msのに対して、下記のコードは約10 ~ 15 msと明らかに速くなっていた。
- そこまでの差ではないが、コピーとムーブによって速さが確実に変わることが実感できた。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<string, vector<string>> sorted_word_to_anagrams;
        for (string& word : strs) {
            string word_for_sort = word;
            sort(word_for_sort.begin(), word_for_sort.end());
            sorted_word_to_anagrams[word_for_sort].push_back(move(word));
        }
        vector<vector<string>> anagram_groups;
        anagram_groups.reserve(sorted_word_to_anagrams.size());
        for (auto& [_, group] : sorted_word_to_anagrams) {
            anagram_groups.push_back(move(group));
        }
        return anagram_groups;
    }
};
```

## Step3
```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<string, vector<string>> sorted_word_to_anagrams;
        for (string& word : strs) {
            string word_for_sort = word;
            sort(word_for_sort.begin(), word_for_sort.end());
            sorted_word_to_anagrams[word_for_sort].push_back(move(word));
        }
        vector<vector<string>> anagram_groups;
        anagram_groups.reserve(sorted_word_to_anagrams.size());
        for (auto& [_, group] : sorted_word_to_anagrams) {
            anagram_groups.push_back(move(group));
        }
        return anagram_groups;
    }
};
```
