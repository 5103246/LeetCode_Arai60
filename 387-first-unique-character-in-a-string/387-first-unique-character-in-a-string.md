# First Unique Character in a String
* 問題: https://leetcode.com/problems/first-unique-character-in-a-string/
* 言語: C++

## Step1

- mapとsetを使う解き方
- mapとsetに値がなければ、mapに値とインデックスを、setに値を追加
- mapとsetに値があれば、mapを削除
- mapになくてsetにあれば、continue
- 最後にmapが空じゃなければ、先頭のvalueを返し、空であれば-1を返す
- 時間計算量：O(n * log n)
- 空間計算量：O(n)
- mapのvalueはソートできないので上の解き方はできなかった。
- とりあえず、地道にループで最小のindexを探して返すようにした。

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        set<char> unique_letters;
        map<char, int> letter_to_index; 
        for (int i = 0; i < s.size(); ++i) {
            if (!unique_letters.contains(s[i]) && !letter_to_index.contains(s[i])) {
                unique_letters.insert(s[i]);
                letter_to_index[s[i]] = i;
                continue;
            }
            if (unique_letters.contains(s[i]) && letter_to_index.contains(s[i])) {
                letter_to_index.erase(s[i]);
                continue;
            }
        }

        if (letter_to_index.empty()) {
            return -1;
        }
        
        int first_unique_letter_index = letter_to_index.begin()->second;
        for (auto [_, index] : letter_to_index) {
            if (first_unique_letter_index > index) {
                first_unique_letter_index = index;
            }
        }

        return first_unique_letter_index;
    }
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/17/files
- mapで出てきた文字をカウントし、カウントが1の文字を返す解き方でやっていた
- setを使わずに済むし、if文が少なく済むのでカウントする解き方の方が良いなと感じた
- カウントする方が読みやすい
- mapやsetに入れるのは最大で26字なので、Step1の計算量の見積もりが間違っていた。
- 正しくは、時間計算量がO(n)で、空間計算量がO(1)　（n : 入力文字列の長さ）
```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        map<char, int> letter_to_frequency;
        for (const char letter : s) {
            ++letter_to_frequency[letter];
        }
        for (int i = 0; i < s.size(); ++i) {
            if (letter_to_frequency[s[i]] == 1) {
                return i;
            }
        }
        return -1;
    }
};
```
- queueで解く方法もあった。
  - mapでカウントして、queueに文字とインデックスのペアを入れる。先頭の文字のカウントが2以上ならpopしていく
  - 先頭の文字のインデックスを返す
```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        map<char, int> letter_to_frequency;
        queue<LetterAndIndex> letters;
        for (int i = 0; i < s.size(); ++i) {
            ++letter_to_frequency[s[i]];
            letters.push({s[i], i});
            while (!letters.empty() && letter_to_frequency[letters.front().letter] >= 2) {
                letters.pop();
            }
        }
        if (letters.empty()) {
            return -1;
        }
        return letters.front().index;
    }
private:
    struct LetterAndIndex {
        char letter;
        int index;
    };
};
```

- https://github.com/Ryotaro25/leetcode_first60/pull/16/files
- vectorでカウントしている解き方もあった。こちらは、文字-'a'で各文字のインデックスを決めてカウントしていた。
- アルファベット順じゃない処理系もあるし、大文字を含める場合に対応しづらいのでmapでカウントする方が良いかな。
- https://github.com/colorbox/leetcode/pull/29/files
- LRUを応用した解き方もあった。
- 双方向リストの実装：https://github.com/irohafternoon/LeetCode/pull/17#r2039457825
- LRUの実装問題：https://github.com/mura0086/arai60/pull/19/files#r2040652351

- 参照の実装をしている人がいた：https://github.com/Ryotaro25/leetcode_first60/pull/16/files#diff-b49c0a6da3e04ac083e21058c3ad3d73b971ed60ecea5fa95f1389e59f709ab1
- LRUの実装など、時間があったらStep4とかでやりたい

## Step3
```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        map<char, int> letter_to_frequency;
        for (const char letter : s) {
            ++letter_to_frequency[letter];
        }
        for (int i = 0; i < s.size(); ++i) {
            if (letter_to_frequency[s[i]] == 1) {
                return i;
            }
        }
        return -1;
    }
};
```
