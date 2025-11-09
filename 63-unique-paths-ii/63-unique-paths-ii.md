# Unique Paths II
* 問題：https://leetcode.com/problems/unique-paths-ii/
* 言語：C++

## Step1

- 問題の概要
  - 二次元配列が与えられ、その中の左上隅から右下隅までのルート数（障害物あり）を求める問題

- 基本的な方針は前回のUnique Pathsと同じで、現在のマスまでのルートは上マスへのルート数と左横のマスへのルート数の合計値になるので、それをdpで管理する。
- 一旦二次元DPで考える。
- DPテーブルに入れる際に、そのマスに障害物がないか確かめる。
- 障害物があったら、DPテーブルには0を入れる。
- 障害物がなかったら、上マスがあったら上を足して、左があったら左を足す

- m * n の二次元配列が与えられるとすると（1 <= m, n <= 100）
- 時間計算量：O(m * n)
- 空間計算量：O(m * n)
- 実行時間は、100 * 100 / 10 ^ 8 = 100 µs程度

- 一次元DPでも解けるか
- 空間計算量はO(min(m, n))になる
- スタート地点に障害物があるケースを考えると、DP[0][0]の初期化も障害物があるか確認したほうがよい
- 前回の問題ではnCrで解けたが、今回は無理そう
- 他の解き方が出てこないので、読みやすい二次元DPで解く。

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(const vector<vector<int>>& obstacle_grid) {
        int num_row = obstacle_grid.size();
        int num_column = obstacle_grid[0].size();
        vector<vector<int>> num_paths(num_row, vector<int>(num_column));
        if (obstacle_grid[0][0] == 1) {
            return 0;
        } else {
            num_paths[0][0] = 1;
        }

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (obstacle_grid[row][column] == 1) {
                    num_paths[row][column] = 0;
                    continue;
                }
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
- ここまでで大体20分

- obstacle_grid[row][column] == 1よりもobstacleなどの定数にすれば読みやすくなるかも
- スタートに障害物があるかの確認はループの中でできるため、最初のif文はいらないかも
- 問題では、1 <= m, nとなっているが、m, n = 0のケースで0を返すようにしてもよかった
- num_paths[0][0]より.front()を使った方が意図が明確か

- 一次元DPver
- num_row, num_columnの小さい方をリストのサイズにしたかったが、obstacle_gridにアクセスするのがめんどくさくなるので、素直にnum_columnをリストのサイズに
- 今回は、左端に障害物があるかもしれないので、columnは0スタート
- 空間計算量はO(n)　（n : 列数）

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(const vector<vector<int>>& obstacle_grid) {
        int num_row = obstacle_grid.size();
        int num_column = obstacle_grid.front().size();
        if (num_row == 0 || num_column == 0) {
            return 0;
        }
        vector<int> num_paths(num_column);
        num_paths.front() = 1;

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (obstacle_grid[row][column] == kObstacle) {
                    num_paths[column] = 0;
                    continue;
                }
                if (column > 0) {
                    num_paths[column] += num_paths[column - 1];
                }
            }
        }
        return num_paths.back();
    }

private:
    static constexpr int kObstacle = 1;
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/37/files
- 自分は障害物があったら0を入れていたが、初期値が0なのでスルーするだけでよかったか
- 障害物をスルーする場合は、最初のスタート地点に障害物があるかの確認が必要になる。
- 行数、列数を見る前に、入力リストが空かチェックした方がよかった。
- ある行すべてに障害物がある場合、打ち切りにするコードもあったが、あまりメリットを感じなかった。
  
- https://github.com/Ryotaro25/leetcode_first60/pull/40/files#diff-cbbfa59c72a7b44bf8aed4b6f8a8555288667c670fd9e1f6b5c9020073edc8ff
- 自分はスタート地点に障害物がある場合に早期リターンしていたが、ゴール地点に障害物があるケースでも早期リターンしてよいか

- https://github.com/olsen-blue/Arai60/pull/34/files
- 二次元DPの壁際を最初に初期化しておく解き方
- そういえば、最初に二次元DPを定義する際に、0を明示しておいた方が読みやすいのかな、サイズ指定のみなら0で初期化されるが明示した方が親切か？
- 壁際を初期化するコードも書いてみた
- 二重ループの中の処理がシンプルになるのはいい感じ
- ただ、コード量も多くなるのでstep1のコードより読みにくいかも
- 自分はstep1のやり方の方が好みだな

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(const vector<vector<int>>& obstacle_grid) {
        if (obstacle_grid.empty() || obstacle_grid.front().empty()) {
            return 0;
        }
        int num_row = obstacle_grid.size();
        int num_column = obstacle_grid.front().size();
        vector<vector<int>> num_paths(num_row, vector<int>(num_column, 0));

        for (int row = 0; row < num_row; ++row) {
            if (obstacle_grid[row][0] == kObstacle) {
                break;
            }
            num_paths[row][0] = 1;
        }
        for (int column = 0; column < num_column; ++column) {
            if (obstacle_grid[0][column] == kObstacle) {
                break;
            }
            num_paths[0][column] = 1;
        }
        for (int row = 1; row < num_row; ++row) {
            for (int column = 1; column < num_column; ++column) {
                if (obstacle_grid[row][column] == kObstacle) {
                    continue;
                }
                num_paths[row][column] = num_paths[row - 1][column] + num_paths[row][column - 1];
            }
        }
        return num_paths.back().back();
    }

private:
    static constexpr int kObstacle = 1;
};
```

- 障害物をスルーするなら、スタート地点に障害物があるかのチェックが必要になる
- 障害物があったら0を入れる場合は、スタート地点のチェックは必要なくなる、
- スタート地点とゴールに障害物がある場合、早期リターンした方がいいか
- 早期リターンしても、あまりメリットがない気がする
- 障害物がある地点に0を入れる方が、少しわかりやすそうな気がする

## Step3

- 二次元DP, 一次元DPどちらも3回

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(const vector<vector<int>>& obstacle_grid) {
        if (obstacle_grid.empty() || obstacle_grid.front().empty()) {
            return 0;
        }
        int num_row = obstacle_grid.size();
        int num_column = obstacle_grid.front().size();
        vector<vector<int>> num_paths(num_row, vector<int>(num_column));
        num_paths.front().front() = 1;

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (obstacle_grid[row][column] == kObstacle) {
                    num_paths[row][column] = 0;
                    continue;
                }
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

private:
    static constexpr int kObstacle = 1;
};
```

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(const vector<vector<int>>& obstacle_grid) {
        if (obstacle_grid.empty() || obstacle_grid.front().empty()) {
            return 0;
        }
        int num_row = obstacle_grid.size();
        int num_column = obstacle_grid.front().size();
        vector<int> num_paths(num_column);
        num_paths.front() = 1;

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (obstacle_grid[row][column] == kObstacle) {
                    num_paths[column] = 0;
                    continue;
                }
                if (column > 0) {
                    num_paths[column] += num_paths[column - 1];
                }
            }
        }
        return num_paths.back();
    }

private:
    static constexpr int kObstacle = 1;
};
```
