# Word Break
- 問題：https://leetcode.com/problems/word-break/
- 言語：C++

## Step1

- 問題の概要
  - 文字列と辞書が与えられる。文字列が辞書内の単語で分割できるかを判定する。辞書内の単語は複数回使用できる。

- 文字列sの中から辞書の単語を空文字などに置換して、最終的に全部からになればtrueを変えずようにすればよさそう。
- 単に先頭から順にやったら、以下のケースで詰まる
- s = "catsandog", wordDict = ["cat","dog","sand","an","and","cats"]
- 先頭から順に置換していくと、
- catsandog -> sandog -> og -> false
- catで置換するか、catsで置換するかで変わるので、wordごとに置換した文字列を用意した方がよさそう。
- cat:sandog, dog:catsan, sand:cat og, an:cats dog, and:cats og, cats:andog
- 上記のmapを作ってから、置換後の文字列を空文字などでさらに置換していく。
- ただ、この方法だと時間計算量が指数時間になりそうだな。

- 手が止まったので、他の方の解答を見る
- https://github.com/nanae772/leetcode-arai60/pull/38/files のstep1を参考

- 文字列sが分割可能とは、s[i:i + len(word)]が辞書内の単語であり、かつ残りの文字列s[len(word):len(s)]が分割可能であるということ
- 後ろから文字列を見ていき、分割可能かをテーブルにメモする。
- 辞書内に単語があって、かつ残りの文字列が分割可能だったらテーブルに分割可能とメモ
- テーブルの一番最後(.back)は、空文字なので分割可能
- 最終的に、テーブルの先頭に結果が入る
- なんで後ろからなのかと思ったが、分割可能の条件を考えると納得いった
- テーブル[i]が分割可能だったらbreakしてよかったか
- kTrueよりも、分割成功のkSuccess、kFailの方がよかったかも

```cpp
class Solution {
public:
    bool wordBreak(const string& s, const vector<string>& word_dict) {
        vector<char> can_segment(s.size() + 1, kFalse);
        can_segment.back() = kTrue;

        for (int i = s.size() - 1; i >= 0; --i) {
            for (auto word : word_dict) {
                if (i + word.size() > s.size()) {
                    continue;
                }
                if (s.substr(i, word.size()) != word) {
                    continue;
                }
                can_segment[i] = can_segment[i] || can_segment[i + word.size()];
            }
        }

        return can_segment.front() == kTrue;
    }

private:
    static constexpr char kFalse = 0;
    static constexpr char kTrue = 1;
};
```

## Step2

- step1のコードの改良
- 辞書内の単語と一致して、かつ残りの文字列が分割可能ならbreakする
```cpp
class Solution {
public:
    bool wordBreak(const string& s, const vector<string>& word_dict) {
        vector<char> can_segment(s.size() + 1, kFail);
        can_segment.back() = kSuccess;

        for (int i = s.size() - 1; i >= 0; --i) {
            for (auto word : word_dict) {
                if (i + word.size() > s.size()) {
                    continue;
                }
                if (s.substr(i, word.size()) == word && can_segment[i + word.size()]) {
                    can_segment[i] = kSuccess;
                    break;
                }
            }
        } 

        return can_segment.front() == kSuccess;
    }

private:
    static constexpr char kFail = 0;
    static constexpr char kSuccess = 1;
};
```

- Geminiのレビュー
  - > 文字列のコピーコストの削減
    > * s.substr(i, word.size()) は、呼び出されるたびに新しい std::string オブジェクトをヒープ領域に作成（コピー）します。 C++17以降であれば、std::string_view を使うことで、コピーを発生させずに部分文字列を比較できます。
    > * インデックスの型
    > * s.size() は size_t (符号なし) を返します。int i と比較するとコンパイル警告が出る場合があるため、型を合わせるか、符号付きにキャストするのが一般的です。
    - strign_viewがわからなかったので調べてみた。
    > * std::basic_string_view は、文字列の所有権を保持せず、文字列のコピーを持つのではなく参照をして、参照先の文字列を加工して扱うクラスである。
    >   (https://cpprefjp.github.io/reference/string_view/basic_string_view.html)
    - 所有権とは、あるリソース（主に動的メモリ）を誰が責任をもって管理・解放するかを明確にする概念らしい。
    - つまり、string_viewは文字列を読み取るだけで、所有権を持った誰かが文字列を消したら見れなくなる。
    > * なぜコピーがなくなるのか？
    > * std::string::substr の場合 : s.substr(i, len) を実行すると、元の文字列から指定範囲のデータを新しくメモリ確保してコピーし、新しい std::string オブジェクトを生成します。
    > * コスト: メモリ確保 + 文字列のコピー
    > * std::string_view::substr の場合 : view.substr(i, len) を実行しても、データのコピーは一切行われません。単に 「ポインタの位置を i 進めて、長さを len に書き換えた新しい『窓』」 を作るだけです。
    > * コスト: ポインタと整数の数値計算のみ = 極めて軽い (O(1))
    > * substrはデータを切り取るのではなく、参照位置と参照サイズを変更するだけ
    >   (https://cpprefjp.github.io/reference/string_view/basic_string_view/substr.html)
    
- std::string_viewで改良したコード
```cpp
class Solution {
public:
    bool wordBreak(const string& s, const vector<string>& word_dict) {
        string_view s_view(s);
        vector<char> can_segment(s_view.size() + 1, kFail);
        can_segment.back() = kSuccess;

        for (int i = s_view.size() - 1; i >= 0; --i) {
            string_view suffix = s_view.substr(i);
            for (auto word : word_dict) {
                if (suffix.starts_with(word) && can_segment[i + word.size()]) {
                    can_segment[i] = kSuccess;
                    break;
                }
            }
        }

        return can_segment.front() == kSuccess;
    }

private:
    static constexpr char kFail = 0;
    static constexpr char kSuccess = 1;
};
```

- https://github.com/potrue/leetcode/pull/39/files
- https://github.com/potrue/leetcode/pull/39/files
- 前からやるならこう書けるかな。
- 後は、setを使ってさらに効率よく解いている人もいた。
- setを使わないと、時間計算量がO(nml)で、set使うとO(n^2 * l)  (n : s.length, m : word_dict.length, l : word_dict[i].length)
- それぞれの実行時間は、300 * 1000 * 20 / 10 ^ 8 = 60 ms、300 ^ 2 * 20 / 10 ^ 8 = 18 ms
- ただ、setを使っても実行時間は1000 / 300 = 約3倍くらいしか差がつかないので使わなくてもいいかも。
- 辞書の単語数がさらに多くなれば、setを使った方がよいかな。
```cpp
class Solution {
public:
    bool wordBreak(const string& s, const vector<string>& word_dict) {
        string_view s_view(s);
        vector<char> can_segment(s_view.size() + 1, kFail);
        can_segment.front() = kSuccess;

        for (int i = 1; i <= s_view.size(); ++i) {
            for (auto word : word_dict) {
                if (i < word.size()) {
                    continue;
                }
                string_view prefix = s_view.substr(i - word.size());
                if (prefix.starts_with(word) && can_segment[i - word.size()]) {
                    can_segment[i] = kSuccess;
                    break;
                }
            }
        }

        return can_segment.back() == kSuccess;
    }

private:
    static constexpr char kFail = 0;
    static constexpr char kSuccess = 1;
};
```

- https://github.com/katsukii/leetcode/pull/11/files
- Trieを使った解き方もある。
- 実装が大変そうだったので、後でやろう。

## Step3

- 後ろからDPの方が気に入ったので、そちらで3回
```cpp
class Solution {
public:
    bool wordBreak(const string& s, const vector<string>& word_dict) {
        string_view s_view(s);
        vector<char> can_segment(s_view.size() + 1, kFail);
        can_segment.back() = kSuccess;

        for (int i = s_view.size() - 1; i >= 0; --i) {
            string_view suffix = s_view.substr(i);
            for (auto word : word_dict) {
                if (suffix.starts_with(word) && can_segment[i + word.size()]) {
                    can_segment[i] = kSuccess;
                    break;
                }
            }
        }

        return can_segment.front() == kSuccess;
    }

private:
    static constexpr char kFail = 0;
    static constexpr char kSuccess = 1;
};
```
