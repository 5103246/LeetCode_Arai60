# Unique Paths
* 問題：https://leetcode.com/problems/unique-paths
* 言語：C++

## Step1

- まず思いついたのは、DFSで解く方法。
- 右に行く時と、下に行くときに再帰して、範囲外ならreturn
- ゴールに付いたらcountを+1する
- ぱっと思いついたのはcountを参照渡しで数える方法だが、副作用の可能性もあるな。
- 書いてみたが、時間オーバーになった。

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        int count = 0;
        UniquePathsHelper(m, n, 1, 1, count);
        return count;
    }

private:
    void UniquePathsHelper(int m, int n, int row, int col, int& count) {
        if (row == m && col == n) {
            ++count;
            return;
        }
        if (row > m || col > n) {
            return;
        }
        UniquePathsHelper(m, n, row + 1, col, count);
        UniquePathsHelper(m, n, row, col + 1, count);
    }
};
```

- 他の解き方を考えてみる（30分程度）
- 移動回数は、(m - 1) + (n - 1)になるな。
- 考えてみたら、高校数学の組み合わせの問題だな。
- 総移動回数の中から、m - 1の組み合わせを求めればいいのか。たしかそんな感じだったはず。
- 総移動回数 C (m - 1)で階乗を求めればよさそう。
- だが、素直に階乗を求めたら、long longの上限を超えてしまったので何か工夫が必要そう
- 実装に10分程かかった。

- 順番に分子、分母の値を約分していけばよさそうだが、割り切れないケースもあるので、一旦分母、分子を分けて考える
- 順番に分子、分母の数をみて、割り切れたら、割った値を分子に掛ける。
- 割り切れなかったら、分母、分子にそれぞれ掛ける。加えた後に割り切れたら、除算。
- 最終的に、分子 / 分母を返す。
- m, nどちらかが1だったら、1を返す。
- int型だと範囲を超えてしまったので、long longにしてなんとか処理が完了した。
- 時間計算量：O(min(m, n))
- 空間計算量：O(1)

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        int total_moves = (m - 1) + (n - 1);
        int smaller_moves = min(m - 1, n - 1);
        long long denominator = 1;
        long long numerator = 1;
        if (smaller_moves == 0) {
            return 1;
        }
        for (int i = 0; i < smaller_moves; ++i) {
            int next_numerator = total_moves - i;
            int next_denominator = smaller_moves - i;
            if (next_numerator % next_denominator == 0) {
                numerator *= next_numerator / next_denominator;
                continue;
            }
            numerator *= next_numerator;
            denominator *= next_denominator;
            if (numerator % denominator == 0) {
                numerator /= denominator;
                denominator = 1;
            }
        }
        return numerator / denominator;
    }
};
```

- もっと効率化できそうな感じがする。step2で考えてみる。
- 変数名が気になる。分母、分子だとあまり意味がない気がする。
- 再帰のコードの計算量の見積もりがよくわかならっかたのでAIに聞いてみた。
- 再帰の深さがm + n - 2になるので、再帰全体の回数はm + n - 2の二分木に近似できるらしいので、大体O(2^(m + n))と見積もれる。


## Step2

- https://github.com/olsen-blue/Arai60/pull/33/files
- どうやら動的計画法で解けるらしい。
- あるマスへのルート数は上マスへのルート数と左マスへのルート数の合計になる。
- 3 × 3だったら以下のようになる。
1 1 1
1 2 3
1 3 6
- 上の表を作成して、最後に右下の値を返す。
- 時間、空間計算量ともにO(m * n)
- 最初の行と列を1で初期化してから、1行1列以降の表を埋めていくやり方や、最初に全部1で初期化するのもあった。
- https://github.com/irohafternoon/LeetCode/pull/36/files#r2076491170
- 自分はリンク先の、一番左上だけ1にして、上があったら上を足して、左があったら左を足すというやり方がわかりやすいと感じた。

```cpp
class Solution {
public:
    int uniquePaths(int num_rows, int num_columns) {
        vector<vector<int>> num_of_paths(num_rows, vector<int>(num_columns));
        num_of_paths[0][0] = 1;
        for (int row = 0; row < num_rows; ++row) {
            for (int column = 0; column < num_columns; ++column) {
                if (row > 0) {
                    num_of_paths[row][column] += num_of_paths[row - 1][column];
                }
                if (column > 0) {
                    num_of_paths[row][column] += num_of_paths[row][column - 1];
                }
            }
        }
        return num_of_paths.back().back();
    }
};
```

- 1次元DPでも解ける
- 直前の行の結果のみを使うので1次元でも実装できる
- 二次元DPよりわかりづらかったが、空間計算量が節約できるので大規模なときはこちらの方がよいかも
- https://github.com/irohafternoon/LeetCode/pull/36/files
- 1次元DPだと読みづらいので、2行だけ作成するやり方もあった。たしかに1次元DPよりは読みやすいかな。

```cpp
class Solution {
public:
    int uniquePaths(int num_rows, int num_columns) {
        if (num_rows < num_columns) {
            swap(num_rows, num_columns);
        }
        vector<int> num_of_paths(num_columns);
        num_of_paths[0] = 1;
        for (int _ = 0; _ < num_rows; ++_) {
            for (int column = 1; column < num_columns; ++column) {
                num_of_paths[column] += num_of_paths[column - 1];
            }
        }
        return num_of_paths.back();
    }
};
```

- nCrの解き方
- https://github.com/Ryotaro25/leetcode_first60/pull/39/files#diff-569ba3184f70ed482c8f2e0d61bc6e34fda6e2a608d1b13321e4d5770319b512
- 計算順序を工夫すれば、途中で少数にならなくて済むのか
- 途中で割り算して、割り切れなかったらめんどくさいなと思ってstep1のような解き方でやったが、この順序でやれば普通に掛け算割り算すればいいのか。
- https://github.com/Ryotaro25/leetcode_first60/pull/39/files#r1832769798
- pythonなどは多倍長整数を標準で使えるがC++は標準ライブラリにはないらしい。余裕があれば、多倍長整数の実装をしてみたい。

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        return Combination(m + n - 2, min(m - 1, n - 1));
    }

private:
    long long Combination(int n, int r) {
        long long result = 1;
        for (int i = 1; i <= r; ++i) {
            result *= (n - i + 1);
            result /= i;
        }
        return result;
    }
};
```

## Step3

- DPに慣れてないので、2次元DP, 1次元DPを3回ずつ

```cpp
class Solution {
public:
    int uniquePaths(int num_rows, int num_columns) {
        vector<vector<int>> num_paths(num_rows, vector<int>(num_columns));
        num_paths[0][0] = 1;
        for (int row = 0; row < num_rows; ++row) {
            for (int column = 0; column < num_columns; ++column) {
                if (row > 0) {
                    num_paths[row][column] += num_paths[row - 1][column];
                }
                if (column > 0) {
                    num_paths[row][column] += num_paths[row][column - 1];
                }
            }
        }
        return num_paths.back().back();
    }
};
```

```cpp
class Solution {
public:
    int uniquePaths(int num_rows, int num_columns) {
        if (num_rows < num_columns) {
            swap(num_rows, num_columns);
        }
        vector<int> num_paths(num_columns);
        num_paths[0] = 1;
        for (int _ = 0; _ < num_rows; ++_) {
            for (int column = 1; column < num_columns; ++column) {
                num_paths[column] += num_paths[column - 1];
            }
        }
        return num_paths.back();
    }
};
```
