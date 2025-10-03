# Word Ladder
* 問題: https://leetcode.com/problems/word-ladder/
* 言語: C++

## Step1

- 最短変換シーケンスを求めたいのでBFSを使いそう。
- 一文字違うかを見分けるのは、一文字ずつ比べて、異なったらカウントしていく。カウントが1のものをキューに入れていく。
- 見た単語はsetに入れていく。
- 下記のコードだと、各ステップごとにwordListを走査しているので  Time Limit Exceededになった
- n : word_list.length , L : begin_word.length
- 時間計算量：O(n^2 * log n * L)
- 空間計算量：O(n)
- 実行時間は、5000^2 * log(5000) * 10 / 10^8 ≒ 25 s 程度

```cpp
class Solution {
public:
    int ladderLength(const string& begin_word, const string& end_word, const vector<string>& word_list) {
        map<string, int> word_to_distance;
        queue<string> candidates;
        candidates.push(begin_word);
        word_to_distance.emplace(begin_word, 1);

        while(!candidates.empty()) {
            string word = candidates.front();
            candidates.pop();

            for (int i = 0; i < word_list.size(); ++i) {
                if (word_to_distance.contains(word_list[i]) || !IsOneLetterDifferent(word, word_list[i])) {
                    continue;
                }
                candidates.push(word_list[i]);
                int distance = word_to_distance[word] + 1;
                word_to_distance.emplace(word_list[i], distance);
                if (word_list[i] == end_word) {
                    break;
                }
            }
        }
        return word_to_distance[end_word];
    }

private:
    bool IsOneLetterDifferent(const string& begin_word, const string& word) {
        int count = 0;
        for (int i = 0; i < begin_word.size(); ++i) {
            if (begin_word[i] != word[i]) {
                ++count;
            }
            if (count >= 2) {
                return false;
            }
        }
        return true;
    }
};
```

- 一文字異なるときのパターンとそれに対応するwordをmapで管理するようにする。
- 例：hit → *it, h*t, hi*
- パターンの管理に悩んで、gptに聞いた。
- パターンの文字列をkeyとして、それに対応する単語をvectorに入れていく。
- queueで候補となる単語と、距離を管理する。
- この解き方は、passした。
- n : word_list.length , L : begin_word.length , p : pattern_to_words[pattern].size
- 時間計算量：O(n * L * p)
- 空間計算量：O(n * L)
- ネストが深くなってしまったのが気になる。
- distanceだと距離だから、合ってないかも。
- 文字の長さは10文字以下なのでconst参照にしなくてもいいかと思ったが、文字の長さを増やしていくケースも考えてconst参照にした。


```cpp
class Solution {
public:
    int ladderLength(const string& begin_word, const string& end_word, const vector<string>& word_list) {
        map<string, vector<string>> pattern_to_words;
        SetPatternToWords(word_list, pattern_to_words);

        int length = begin_word.size();
        queue<pair<string, int>> candidate_and_distance;
        candidate_and_distance.push({begin_word, 1});
        set<string> visited{begin_word};

        while (!candidate_and_distance.empty()) {
            auto [candidate, distance] = candidate_and_distance.front();
            candidate_and_distance.pop();
            
            for (int i = 0; i < length; ++i) {
                string pattern = candidate;
                pattern[i] = '*';
                if (!pattern_to_words.contains(pattern)) {
                    continue;
                }
                for (const string& word : pattern_to_words[pattern]) {
                    if (visited.contains(word)) {
                        continue;
                    }
                    if (word == end_word) {
                        return distance + 1;
                    }
                    candidate_and_distance.push({word, distance + 1});
                    visited.insert(word);
                }
            }
        }
        return 0;
    }

private:
    void SetPatternToWords(const vector<string>& word_list, map<string, vector<string>>& pattern_to_words) {
        int length = word_list[0].size();
        for (const string& word : word_list) {
            for (int i = 0; i < length; ++i) {
                string pattern = word;
                pattern[i] = '*';
                pattern_to_words[pattern].push_back(word);
            }
        }
    }
};
```

## Step2

- https://github.com/nktr-cp/leetcode/pull/21/files
- 隣接リストを作成して解いていた。
- 隣接リストを作るのに二重ループが必要なので、大変そう。
- step3のコードは隣接リストを作る処理とqueueに入れる処理が一緒になっていたので読みづらかった。
- https://github.com/irohafternoon/LeetCode/pull/22/files
- アルファベットを使う方法と、単語を前半と後半に分ける方法もやっていた。
- アルファベットを使う方法は結構読みやすく感じたが、文字の種類が増えると管理が大変そう。
- 前半と後半に分ける方法はStep1の2つ目のコードと考えが似ている。前半と後半に分ける処理は関数にしたほうが読みやすいと感じた。
- step3のBFSの各探索ステップで、available_wordの中から次に進めるものを見つける方法は自分がStep1でアクセプトされなかった解き方と同じだと思う。

- アルファベットを使う解き方
- 結構書きやすいし、読みやすいと感じた。
- ただ、最初に、setに入れるのが無駄に感じたのと、速度もパターン化する方法が速いので、自分はパターン化する方が好み。

```cpp
class Solution {
public:
    int ladderLength(const string& begin_word, const string& end_word, const vector<string>& word_list) {
        set<string> available_words(word_list.begin(), word_list.end());
        set<string> seen_words{begin_word};
        queue<pair<string, int>> candidate_and_level;
        candidate_and_level.push({begin_word, 1});

        while (!candidate_and_level.empty()) {
            auto [candidate, level] = candidate_and_level.front();
            candidate_and_level.pop();

            for (int i = 0; i < begin_word.size(); ++i) {
                string word = candidate;
                for (char ch = 'a'; ch <= 'z'; ++ch) {
                    word[i] = ch;
                    if (seen_words.contains(word)) {
                        continue;
                    }
                    if (!available_words.contains(word)) {
                        continue;
                    }
                    if (word == end_word) {
                        return level + 1;
                    }
                    candidate_and_level.push({word, level + 1});
                    seen_words.insert(word);
                }
            }
        }
        return 0;
    }
};
```

## Step3

- パターン化する解き方で3回

```cpp
class Solution {
public:
    int ladderLength(const string& begin_word, const string end_word, const vector<string>& word_list) {
        map<string, vector<string>> pattern_to_words;
        BuildPatternToWords(word_list, pattern_to_words);

        queue<pair<string, int>> candidate_and_distance;
        candidate_and_distance.emplace(begin_word, 1);
        set<string> seen_words{begin_word};
        int length = begin_word.size();
        while (!candidate_and_distance.empty()) {
            auto [candidate, distance] = candidate_and_distance.front();
            candidate_and_distance.pop();

            for (int i = 0; i < length; ++i) {
                string pattern = candidate;
                pattern[i] = kWildCardChar;
                for (const string& word : pattern_to_words[pattern]) {
                    if (word == end_word) {
                        return distance + 1;
                    }
                    if (seen_words.contains(word)) {
                        continue;
                    }
                    candidate_and_distance.emplace(word, distance + 1);
                    seen_words.insert(word);
                }
            }
        }
        return 0;
    }

private:
    static constexpr char kWildCardChar = '*';

    void BuildPatternToWords(const vector<string>& word_list, map<string, vector<string>>& pattern_to_words) {
        int length = word_list[0].size();
        for (const string& word : word_list) {
            for (int i = 0; i < length; ++i) {
                string pattern = word;
                pattern[i] = kWildCardChar;
                pattern_to_words[pattern].push_back(word);
            }
        }
    }
};
```
