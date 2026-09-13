# String to Integer (atoi)

- 問題：https://leetcode.com/problems/string-to-integer-atoi/
- 言語：C++

## Step 1

- 問題の概要：
  - 文字列をintに変換する
  - 次のルールに従う
    - 先頭の空白文字は無視する
    - "-"があったら負の整数として変換する
    - 数字以外の文字が現れるまで変換する。数字がなかったら0を返す
    - intの範囲を超えたら丸める
      
      
- 文字列から数字を抽出して、intに変換する順番で考える
- intへの変換はasciiの引き算で行う
- 数字に変換するときにintを超えていないかチェックする
- 他に思いつかないので実装してみる
- とりあえず書けたが、汚いので改善してみる
  
```cpp
class Solution {
public:
    int myAtoi(string s) {
        int index = 0;
        while (isspace(s[index])) {
            ++index;
        }
        
        int sign = 1;
        if (!isdigit(s[index])) {
            if (s[index] == '-') {
                sign = -1;
            } else if (s[index] == '+') {
                sign = 1;
            } else {
                return 0;
            }
            ++index;
        }

        while (s[index] == '0') {
            ++index;
        }

        string num = "";
        while (isdigit(s[index])) {
            if (num.size() > 10) {
                if (sign > 0) {
                    return numeric_limits<int>::max();
                } else {
                    return numeric_limits<int>::min();
                }
            }
            num += s[index];
            ++index;
        }

        long long result = 0;
        for (int i = 0; i < num.size(); ++i) {
            result += sign * (num[i] - '0') * pow(10, num.size() - 1 - i);
        }

        if (result > numeric_limits<int>::max()) {
            return numeric_limits<int>::max();
        } else if (result < numeric_limits<int>::min()) {
            return numeric_limits<int>::min();
        }

        return result;
    }
};
```

- powじゃなくて、桁を一つ増やしてから足せばいいか
  - result * 10 + s[index] - '0'で直接計算する
- 最初の実装は範囲外チェックをしていなかった
  - 今回は数字の抽出を行わないので最後のループでも範囲外チェックが必要

```cpp
class Solution {
public:
    int myAtoi(string s) {
        int index = 0;
        int n = s.size();

        while (index < n && isspace(s[index])) {
            ++index;
        }

        int sign = 1;
        if (index < n && !isdigit(s[index])) {
            if (s[index] == '-') {
                sign = -1;
                ++index;
            } else if (s[index] == '+') {
                sign = 1;
                ++index;
            }
        }

       long long result = 0;
       while (index < n && isdigit(s[index])) {
            int digit = s[index] - '0';
            result = result * 10 + digit;

            if (sign == 1 && result >= numeric_limits<int>::max()) {
                return numeric_limits<int>::max();
            } else if (sign == -1 && -result <= numeric_limits<int>::min()) {
                return numeric_limits<int>::min();
            }
            ++index;
       }

       return sign * result;
    }
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/59/changes#diff-343ca0610d8d8419413f5dd6c460978961c8aaf52f2a16574a8f904122200476
  - long longを使わない方法でやっている
  - numeric_limits<int>::max() / 10を基準に考えるのか。これを超える場合はmax, minを符号によって返す
  - 等しい場合は、最後にdigitを足したときに超えないかチェックしている
  - 読みやすさで言ったら、long longの方が自分は読みやすい。
  - 7で判定するのは初見だとわからなかった
  
```cpp
class Solution {
public:
    int myAtoi(string s) {
        int i = 0;
        int n = s.size();

        while (i < n && isspace(s[i])) {
            ++i;
        }

        int sign = 1;
        if (i < n && (s[i] == '+' || s[i] == '-')) {
            if (s[i] == '-') {
                sign = -1;
            }
            ++i;
        }

        int result = 0;
        while (i < n && isdigit(s[i])) {
            int digit = s[i] - '0';

            if (result > numeric_limits<int>::max() / 10 ||
               (result == numeric_limits<int>::max() / 10 && digit > numeric_limits<int>::max() % 10)) {
                return sign == 1 ? numeric_limits<int>::max() : numeric_limits<int>::min();
            }

            result = result * 10 + digit;
            ++i;
        }

        return sign * result;
    }
};
```

- ロケールという言葉が良く分からなかったので調べた
- ロケール（Locale）とは、プログラムが扱う言語、国、地域、およびそれに伴う文化的な慣習のルールをまとめた設定（データセット）のこと
  - 3桁区切りのカンマの位置や日時の並び順など
- https://github.com/dxxsxsxkx/leetcode/pull/59/changes#r3038920196
  - 符号によって分けるのもいいかもしれない
- https://github.com/Ryotaro25/leetcode_first60/pull/64/changes#r2027104867
  - long longではなく、int64_tの方が安全だったかも
- https://github.com/potrue/leetcode/pull/59/changes
  - 符号を先につけた方が丸め込が楽になりそう

```cpp
class Solution {
public:
    int myAtoi(string s) {
        int i = 0;
        int n = s.size();

        while (i < n && isspace(s[i])) {
            ++i;
        }

        int sign = 1;
        if (i < n && (s[i] == '+' || s[i] == '-')) {
            if (s[i] == '-') {
                sign = -1;
            }
            ++i;
        }

        int result = 0;
        while (i < n && isdigit(s[i])) {
            int digit = sign * (s[i] - '0');

            if (result > numeric_limits<int>::max() / 10 || 
               (result == numeric_limits<int>::max() / 10 && digit > numeric_limits<int>::max() % 10)) {
                return numeric_limits<int>::max();
            } else if (result < numeric_limits<int>::min() / 10 ||
                      (result == numeric_limits<int>::min() / 10 && digit < numeric_limits<int>::min() % 10)) {
                return numeric_limits<int>::min();
            }

            result = result * 10 + digit;
            ++i;
        }

        return result;
    }
};
```

## Step 3
- int64_tの方が読みやすくていいと感じたので3回
```cpp
class Solution {
public:
    int myAtoi(const string& s) {
        int i = 0;
        while (i < s.size() && isspace(s[i])) {
            ++i;
        }

        int sign = 1;
        if (i < s.size() && (s[i] == '+' || s[i] == '-')) {
            if (s[i] == '-') {
                sign = -1;
            }
            ++i;
        }

        int64_t result = 0;
        while (i < s.size() && isdigit(s[i])) {
            int digit = s[i] - '0';
            result = result * 10 + sign * digit;

            if (result > numeric_limits<int>::max()) {
                return numeric_limits<int>::max();
            } else if (result < numeric_limits<int>::min()) {
                return numeric_limits<int>::min();
            }

            ++i;
        }

        return result;
    }
};
```
