# Number of Islands
* 問題: https://leetcode.com/problems/number-of-islands/
* 言語: C++

## Step1

- DFSやBFSを使った解き方を考えてみたが、わからなかったので答えを見る
- https://github.com/Ryotaro25/leetcode_first60/pull/18/files#diff-bce6e87bcb5dbd51123b20f51064ed5eae40110bde5cc87786b4025c21cec1a1
- 自分はBFSで考えていたが、一旦DFSの解答を見てみる。
- gridの位置での場合分けに苦しんでいたが、このように解くのか。境界処理を愚直に場合分けしていたが、移動した先が境界を超えていたらスルーするようにすればよかったのか。
- 見た'1'のマス目は'0'にして、ループを避ける。
- 参考にしたリンクのコードでは、入力をそのまま変更していた。二次元配列のコピーは大変な気もするが、今回は大体、300 * 300 = 90000 バイトで、メモリ的には大丈夫そうだったので事前にコピーして処理した。
- 見たislandをwaterに変えるよりも、islandの位置をsetに入れる方がよさそう。

```cpp
class Solution {
public:
    int numIslands(const vector<vector<char>>& grid) {
        int island_count = 0;
        vector<vector<char>> map = grid;
        int height = map.size();
        int width = map[0].size();

        for (int i = 0; i < height; ++i) {
            for (int j = 0; j < width; ++j) {
                if (map[i][j] == '1') {
                    TraverseIsland(map, i, j);
                    ++island_count;
                }
            }
        }
        return island_count;
    }

private:
    void TraverseIsland(vector<vector<char>>& grid, int i, int j) {
        int height = grid.size();
        int width = grid[0].size();
        if (i < 0 || j < 0 || height <= i || width <= j) {
            return;
        }
        if (grid[i][j] == '0') {
            return;
        }
        grid[i][j] = '0';

        TraverseIsland(grid, i - 1, j);
        TraverseIsland(grid, i, j - 1);
        TraverseIsland(grid, i, j + 1);
        TraverseIsland(grid, i + 1, j);
    }
};
```

## Step2

- https://github.com/nanae772/leetcode-arai60/pull/18/files
- DFS, BFSの他にはUnionFindというのを使った解き方があるらしい。
- islandをwaterにするのではなく、二次元配列の同じ位置にtrue/falseを入れる方法で解いていた。
  - https://github.com/Hurukawa2121/leetcode/pull/17#r1898907043
    - c++では、vector<bool>は特殊化されていて、プロキシクラス経由で値にアクセスすることになるため、速度が落ちる可能性があるといこと。そのため、vector<uint8_t>などの方がよい。
- 見るマス目の位置を変えるときに、あらかじめ上下左右への移動を表す配列を用意していた。
- グリッド上の探索では良く使うらしい。移動ベクトルともいうらしい。（https://qiita.com/drken/items/a803d4fc4a727e02f7ba）

- https://github.com/mura0086/arai60/pull/21/files#r2033460513
- > 他の人のコードは、コメントなどではなくてコードのロジック部分を読んで差異やよしあしを感じ取って欲しいのですが、できていると感じますか。
- https://github.com/mura0086/arai60/pull/21/files
- https://github.com/mura0086/arai60/pull/21/files
- '0'を直接使わず、seaやwaterなどの定数を使うのは良いなと思った。
- 移動ベクトルやseaなどの定数はコンパイル時に決定するのでconstexprをつけてよさそう。
- インデックスをi, jやy, xで管理するよりも、row, colで管理する方がわかりやすかった。
- https://github.com/nktr-cp/leetcode/pull/18/files
- 探索中の場合分けを関数にしている人もいた。
  - グリッド内に収まっているか
  - 島であるか
  - 探索済みであるか
  - 関数化するのもありだと感じたが、あまりメリットが感じられなかったのでまとめて書く。
- https://github.com/colorbox/leetcode/pull/31/files#r1881804172
- グリッドの範囲外であるという条件文よりも「範囲内でない」という方が分かりやすいという意見もあった。
- https://github.com/mura0086/arai60/pull/21/files#r2033346904
- 大小関係をバラバラに書いていた。数直線上に書いた方がわかりやすい。
- https://github.com/irohafternoon/LeetCode/pull/19/files
- https://github.com/nktr-cp/leetcode/pull/18/files
- 関数の引数の順番は入力専用が出力専用より先に配置するのが推奨されている。上記のリンクのコードでは、引数gridが後ろの方にあったが、gridは必須の引数なので先頭に合った方がよいと感じた。
- grid, num_row, num_column, row, colmun, visitedの順番がいいかな。

- DFS
```cpp
class Solution {
public:
    int numIslands(const vector<vector<char>>& grid) {
        int num_row = grid.size();
        int num_column = grid[0].size();
        vector<vector<uint8_t>> visited(num_row, vector<uint8_t>(num_column, 0));
        int island_count = 0;

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (grid[row][column] == kLand && !visited[row][column]) {
                    TraverseIsland(grid, num_row, num_column, row, column, visited);
                    ++island_count;
                }
            }
        }
        return island_count;
    }

private:
    static constexpr char kSea = '0';
    static constexpr char kLand = '1';
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    void TraverseIsland(const vector<vector<char>>& grid,
                        int num_row, int num_column,
                        int row, int column,
                        vector<vector<uint8_t>>& visited) {
        visited[row][column] = 1;
        for (auto [dr, dc] : kDirections) {
            int next_row = row + dr;
            int next_column = column + dc;
            if (!(0 <= next_row && next_row < num_row && 0 <= next_column && next_column < num_column)) {
                continue;
            }
            if (grid[next_row][next_column] == kSea || visited[next_row][next_column]) {
                continue;
            }
            TraverseIsland(grid, num_row, num_column, next_row, next_column, visited);
        }
    }
};
```

- BFS
```cpp
class Solution {
public:
    int numIslands(const vector<vector<char>>& grid) {
        int num_row = grid.size();
        int num_column = grid[0].size();
        queue<pair<int, int>> island_positions;
        vector<vector<uint8_t>> visited(num_row, vector<uint8_t>(num_column, 0));
        int island_count = 0;
        
        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (grid[row][column] == kLand && !visited[row][column]) {
                    island_positions.push({row, column});
                    visited[row][column] = 1;
                    TraverseIsland(grid, num_row, num_column, island_positions, visited);
                    ++island_count;
                }
            }
        }
        return island_count;
    }

private:
    static constexpr char kSea = '0';
    static constexpr char kLand = '1';
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    void TraverseIsland(const vector<vector<char>>& grid,
                        int num_row, int num_column,
                        queue<pair<int, int>>& island_positions,
                        vector<vector<uint8_t>>& visited) {
        
        while (!island_positions.empty()) {
            auto [row, column] = island_positions.front();
            island_positions.pop();
            for (auto [dr, dc] : kDirections) {
                int next_row = row + dr;
                int next_column = column + dc;
                if (!(0 <= next_row && next_row < num_row && 0 <= next_column && next_column < num_column)) {
                    continue;
                }
                if (grid[next_row][next_column] == kSea || visited[next_row][next_column]) {
                    continue;
                }
                island_positions.push({next_row, next_column});
                visited[next_row][next_column] = 1;
            }
        }
    }
};
```

- DFS, BFSどちらも書いてみたが、今回の問題に対しては、どちらで解いても変わんないかなと感じた。
- 若干、DFSの方がコードが短いので、step3ではDFSで書く。
- uionfindは余裕があったら、step4で書いてみようと思う。

## Step3

```cpp
class Solution {
public:
    int numIslands(const vector<vector<char>>& grid) {
        int num_row = grid.size();
        int num_column = grid[0].size();
        vector<vector<uint8_t>> visited(num_row, vector<uint8_t>(num_column, 0));
        int island_count = 0;

        for (int row = 0; row < num_row; ++row) {
            for (int column = 0; column < num_column; ++column) {
                if (grid[row][column] == kLand && !visited[row][column]) {
                    TraverseIsland(grid, num_row, num_column, row, column, visited);
                    ++island_count;
                }
            }
        }
        return island_count;
    }

private:
    static constexpr char kSea = '0';
    static constexpr char kLand = '1';
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    void TraverseIsland(const vector<vector<char>>& grid,
                        int num_row, int num_column,
                        int row, int column,
                        vector<vector<uint8_t>>& visited) {
        visited[row][column] = 1;
        for (auto [dr, dc] : kDirections) {
            int next_row = row + dr;
            int next_column = column + dc;
            if (!(0 <= next_row && next_row < num_row && 0 <= next_column && next_column < num_column)) {
                continue;
            }
            if (grid[next_row][next_column] == kSea || visited[next_row][next_column]) {
                continue;
            }
            TraverseIsland(grid, num_row, num_column, next_row, next_column, visited);
        }
    }
};
```
