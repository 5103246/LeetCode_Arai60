# Zigzag Conversion

- 問題：https://leetcode.com/problems/zigzag-conversion/description/
- 言語：C++

## Step 1

- 問題の概要：
  - 文字列sを指定された行数のジグザグに並べる。並べた後、上から1行ずつ読んでいった文字列を返す。
  - s = "abcdefghijk" rows = 3
  ```
  a   e   i
  b d f h j
  c   g   k
  ```
  - "aeibdfhjcgk"が答えになる

- 二次元配列で考える
- (1,1) -> (2,1) -> (3,1)
- 最終行まで行ったら、前行に切り返す。切り返すときは、(row - 1, col + 1)に文字を入れる
- (2,2) -> (1,3) -> (2,3) -> (3,3) -> (2,4) -> (1,5) -> (2,5) -> ...
- 最後に文字を連結して返す
- 時間計算量：O(n ^ 2)
- 空間計算量：O(n ^ 2)
- n : s.length
- 実行時間は 1000 ^ 2 / 10 ^ 8 = 10 ms程度

- 二次元配列を使わずに考えてみる
- 最終的に各行を連結していくので、文字列の配列を考えればよさそう
- 最初の行と最後の行に着いたら、切り換えせばいい。列の情報はいらないので行だけ見てればいいか
- 時間計算量：O(n)
- 空間計算量：O(n)
- この方法で行けそうなので実装してみる

- 似たようなif文があるのが気になる
- 良い変数名がありそう
```cpp
class Solution {
public:
    string convert(const string& s, int num_rows) {
        vector<string> zigzag_rows(num_rows);
        int row_index = 0;
        bool down_flag = true;

        for (auto ch : s) {
            zigzag_rows[row_index] += ch;

            if (down_flag && row_index < num_rows - 1) {
                ++row_index;
            } else if (!down_flag && row_index > 0) {
                --row_index;
            }

            if (row_index == 0) {
                down_flag = true;
            } else if (row_index == num_rows - 1) {
                down_flag = false;
            }
        }

        string result = "";
        for (auto row : zigzag_rows) {
            result += row;
        }

        return result;
    }
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/60/changes#diff-8efbd75a7938537415031daf3af653b8b5b406823c807db827b95a7ef4aef9ea
- flagを使わないで実装していた。row_indexを進める方向を変えるだけでいいのか
- 2つ目のif文だけで済むのか
- https://github.com/dxxsxsxkx/leetcode/pull/60/changes#r3050556883
> 以下のように書くことも可能ですね。
> 
>             line += direction;
>             if (line == 0 || line == numRows - 1) {
>                 direction = -direction;  // reverse
>             }
> なお、
> 
>             if (line == 0 || line == numRows - 1) {
>                 direction = -direction;  // reverse
>             }
>             line += direction;
> と書く場合はdirectionの初期値を-1にしないと整合しなくなり、気持ち悪いですね。

- 反転するだけなので、これでもいいのか
- lineを進める処理の位置は気にする必要がある

- https://cpprefjp.github.io/reference/numeric/accumulate.html
- 最後はaccumulateを返すだけでよい
- 行ごとの文字列を作成するときは、一文字ずつ追加するので、+=よりもpush_backの方がわかりやすいかな

```cpp
class Solution {
public:
    string convert(const string& s, int num_rows) {
        if (num_rows == 1 || s.length() <= num_rows) {
            return s;
        }

        vector<string> strings_by_line(num_rows);
        int line = 0;
        int direction = 1;

        for (auto ch : s) {
            strings_by_line[line].push_back(ch);

            line += direction;
            if (line == 0 || line == num_rows - 1) {
                direction = -direction;
            }
        }

        return accumulate(strings_by_line.begin(), strings_by_line.end(), string());
    }
};
```

- https://github.com/potrue/leetcode/pull/60/changes
- indexを直接求める方法もあった
- 1周期の長さを基に各行でindexを計算する
- 切り換えし地点以外は1周期で2回文字を追加するので、first_indexの次に、seconde_indexを計算している
- first_index, second_indexが何を表しているのかがわかりにくかった。
- 変数名を少し変えたが、あまり読みやすくはなってなさそう
```cpp
class Solution {
public:
    string convert(const string& s, int num_rows) {
        if (num_rows == 1 || s.length() <= num_rows) {
            return s;
        }

        string result = "";
        int cycle = (num_rows - 1) * 2;
        for (int row = 0; row < num_rows; ++row) {
            for (int start = 0; start < s.length(); start += cycle) {
                int down_index = start + row;
                if (down_index < s.length()) {
                    result.push_back(s[down_index]);
                }

                if (row == 0 || row == num_rows - 1) {
                    continue;
                }

                int up_index = start + cycle - row;
                if (up_index < s.length()) {
                    result.push_back(s[up_index]);
                }
            }
        }

        return result;
    } 
};
```

## Step 3
- step 2の最初の実装で3回

```cpp
class Solution {
public:
    string convert(const string& s, int num_rows) {
        if (num_rows == 1 || s.length() <= num_rows) {
            return s;
        }

        vector<string> strings_by_line(num_rows);
        int line = 0;
        int direction = 1;
        for (auto ch : s) {
            strings_by_line[line].push_back(ch);
            line += direction;
            if (line == 0 || line == num_rows - 1) {
                direction = -direction;
            }
        }

        return accumulate(strings_by_line.begin(), strings_by_line.end(), string());
    }
};
```
