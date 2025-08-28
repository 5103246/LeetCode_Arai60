# Valid Parentheses
* 問題: https://leetcode.com/problems/valid-parentheses/
* 言語: C++

## Step1
### 所要時間：30分

スタックを使う方法はすぐ思いついたが、調べながらやっていたら時間がかかった。

- 入力文字列の長さが奇数ならfalse
- 先頭から走査していき、開括弧に会ったら、スタックに開括弧を積んでいく。閉括弧に会ったらスタックのトップ要素を見て同じ括弧ならpop、違う括弧ならfalse。
  - 対応する括弧を事前に、mapに保存しておく。
  - 閉括弧とスタックの要素を比較するさい、スタックの長さが0ならfalse
- ループ処理後のスタックに文字が残っていたらfalse

### 調べたこと
- スタック
  - https://cpprefjp.github.io/reference/stack/stack.html
- map
  - https://cpprefjp.github.io/reference/map/map.html
  - https://qiita.com/_EnumHack/items/f462042ec99a31881a81
- stringについて
  - https://zenn.dev/reputeless/books/standard-cpp-for-competitive-programming/viewer/string

nは入力文字列の長さ
- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
#include <stack>

class Solution {
public:
    bool isValid(string s) {
        if (s.size() % 2 != 0) return false;

        std::stack<char> bracket_stack;
        std::map<char, char> correspond;
        correspond[')'] = '(';
        correspond['}'] = '{';
        correspond[']'] = '[';
        
        for (char bracket : s) {
            if (bracket == '(' || bracket == '{' || bracket == '[') {
                bracket_stack.push(bracket);
            } else {
                if (bracket_stack.size() == 0) return false;
                if (bracket_stack.top() == correspond[bracket]) {
                    bracket_stack.pop();
                } else {
                    return false;
                }
            }
        }
        
        if (bracket_stack.size() != 0) return false;
        return true;
    }
};
```

## Step2

mapの[]とat()の違いがわかってなかった。
- https://github.com/konnysh/arai60/pull/6/files

pushの代わりにemplaceを使ってた人もいた。pushは値をコピーもしくはムーブして格納するが、emplaceはコピーや移動操作がなく、直接格納する。
そのため、emplaceの方が処理効率が良い。vector, deque, listでも同様らしい。
- https://github.com/Ryotaro25/leetcode_first60/pull/7/files
- https://en.cppreference.com/w/cpp/container/stack/emplace.html
- https://zenn.dev/melos/articles/01231a07c787be

Step1の修正
```cpp
class Solution {
public:
    bool isValid(const string& s) {
        const std::map<char, char> bracket_pair = {
            {'(', ')'},
            {'{', '}'},
            {'[', ']'}
        };
        std::stack<char> open_bracket;

        for (char bracket : s) {
            if (bracket_pair.contains(bracket)) {
                open_bracket.push(bracket);
                continue;
            }
            if (open_bracket.empty() || 
                bracket_pair.at(open_bracket.top()) != bracket) {
                return false;
            }
            open_bracket.pop();
        }
        return open_bracket.empty();
    }
};
```

この問題を見ると、チョムスキー階層、タイプ-2、文脈自由文法、プッシュダウンオートマトンという単語を連想するらしい。
- https://discord.com/channels/1084280443945353267/1206101582861697046/1216945010189144085
- https://discord.com/channels/1084280443945353267/1201211204547383386/1202541275115425822

正規言語、正規文法、有限オートマトンと対比される
- https://discord.com/channels/1084280443945353267/1201211204547383386/1202604857664475227

- チョムスキー階層
  - https://ja.wikipedia.org/wiki/%E3%83%81%E3%83%A7%E3%83%A0%E3%82%B9%E3%82%AD%E3%83%BC%E9%9A%8E%E5%B1%A4
  - 言語の複雑さを4つの階層に分類したもの。タイプ-2は文脈自由文法で文脈自由言語を生成する。大体のプログラミング言語の文法が
    ここに対応している。括弧の対応やif構文などが例。タイプ-3は正規文法で正規言語を生成する。
  - 正規言語は単純なルールしか表現できない（例：aを一つ以上繰り返す）
- 有限オートマトン
  - 正規文法を認識できる
- プッシュダウンオートマトン
  - https://www.jaist.ac.jp/~uehara/course/2006/ti113/09pda.pdf
  - 文脈自由文法を認識できるスタックを持ったオートマトン。

再帰下降構文解析で解いている人もいた。この問題を見たら、再帰下降も連想するのだろうか。
- https://github.com/fhiyo/leetcode/pull/6/files
- https://discord.com/channels/1084280443945353267/1235829049511903273/1238815737548898346

再帰下降構文解析
```cpp
class Solution {
public:
    bool isValid(const string& s) {
        int i = 0;
        return parseBrackets(s, i) && s.length() == i;
    }

private:
    bool parseBrackets(const string& input, int& index) {
        while (true) {
            if (consume(input, '(', index)) {
                if (!(parseBrackets(input, index) && consume(input, ')', index))) return false;
            } else if (consume(input, '{', index)) {
                if (!(parseBrackets(input, index) && consume(input, '}', index))) return false;
            } else if (consume(input, '[', index)) {
                if (!(parseBrackets(input, index) && consume(input, ']', index))) return false;
            } else {
                return true;
            }
        }
    }
    bool consume(const string& input, const char ch, int& index) {
        if (index < input.length() && input[index] == ch) {
            ++index;
            return true;
        }
        return false;
    }
};
```

## Step3
```cpp
class Solution {
public:
    bool isValid(const string& s) {
        const std::map<char, char> brackets_pair = {
            {'(', ')'},
            {'{', '}'},
            {'[', ']'},
        };
        std::stack<char> open_brackets;

        for (char bracket : s) {
            if (brackets_pair.contains(bracket)) {
                open_brackets.emplace(bracket);
                continue;
            }
            if (open_brackets.empty() ||
                brackets_pair.at(open_brackets.top()) != bracket) {
                return false;
            }
            open_brackets.pop();
        }
        return open_brackets.empty();
    }
};
```
