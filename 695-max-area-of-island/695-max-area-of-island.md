# Max Area of Island
* 問題: https://leetcode.com/problems/max-area-of-island/
* 言語: C++

## Step1

- 前回の問題とほとんど変わらないと思うので、同じようにDFSかBFSで解いていく。
- 最短経路を求めるものでもないので、比較的書きやすいDFSで解いていく。
- voidかintかで悩んだ。
- 面積を求める際にvisitedを変更しているから、voidにした方がよいのかなと思った。
- でも、gridを変更するわけでもないから、普通に面積を返すでよさそう。
- int型でも書いてみたが、こちらの方が何をしたいかがわかりやすい。

```cpp
class Solution {
public:
    int maxAreaOfIsland(const vector<vector<int>>& grid) {
        int num_row = grid.size();
        int num_col = grid[0].size();
        vector<vector<uint8_t>> visited(num_row, vector<uint8_t>(num_col, 0));
        int max_area_island = 0;

        for (int row = 0; row < num_row; ++row) {
            for (int col = 0; col < num_col; ++col) {
                if (grid[row][col] == kLand && !visited[row][col]) {
                    int area = 1;
                    TraverseIsland(grid, num_row, num_col, row, col, area, visited);
                    max_area_island = max(max_area_island, area);
                }
            }
        }
        return max_area_island;
    }

private:
    static constexpr int kSea = 0;
    static constexpr int kLand = 1;
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    void TraverseIsland(const vector<vector<int>>& grid,
                        int num_row, int num_col,
                        int row, int col,
                        int& area,
                        vector<vector<uint8_t>>& visited) {
        visited[row][col] = 1;
        for (auto [dr, dc] : kDirections) {
            int next_row = row + dr;
            int next_col = col + dc;
            if (!(0 <= next_row && next_row < num_row && 0 <= next_col && next_col < num_col)) {
                continue;
            }
            if (grid[next_row][next_col] == kSea || visited[next_row][next_col]) {
                continue;
            }
            ++area;
            TraverseIsland(grid, num_row, num_col, next_row, next_col, area, visited);
        }
    }
};
```

- BFS
```cpp
class Solution {
public:
    int maxAreaOfIsland(const vector<vector<int>>& grid) {
        int num_row = grid.size();
        int num_col = grid[0].size();
        vector<vector<uint8_t>> visited(num_row, vector<uint8_t>(num_col, 0));
        queue<pair<int, int>> island_positions;
        int max_area_island = 0;

        for (int row = 0; row < num_row; ++row) {
            for (int col = 0; col < num_col; ++col) {
                if (grid[row][col] == kLand && !visited[row][col]) {
                    int area = CaluculateAreaOfIsland(grid, num_row, num_col, row, col, island_positions, visited);
                    max_area_island = max(max_area_island, area);
                }
            }
        }
        return max_area_island;
    }

private:
    static constexpr int kSea = 0;
    static constexpr int kLand = 1;
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    int CaluculateAreaOfIsland(const vector<vector<int>>& grid,
                        int num_row, int num_col,
                        int start_row, int start_col,
                        queue<pair<int, int>>& island_positions,
                        vector<vector<uint8_t>>& visited) {
        island_positions.push({start_row, start_col});
        visited[start_row][start_col] = 1;
        int area = 1;

        while (!island_positions.empty()) {
            auto [row, col] = island_positions.front();
            island_positions.pop();
            for (auto [dr, dc] : kDirections) {
                int next_row = row + dr;
                int next_col = col + dc;
                if (!(0 <= next_row && next_row < num_row && 0 <= next_col && next_col < num_col)) {
                    continue;
                }
                if (grid[next_row][next_col] == kSea || visited[next_row][next_col]) {
                    continue;
                }
                ++area;
                island_positions.push({next_row, next_col});
                visited[next_row][next_col] = 1;
            }
        }
        return area;
    }
};
```

## Step2 

- https://github.com/colorbox/leetcode/pull/32/files
- https://github.com/mura0086/arai60/pull/22/files
- > 非再帰的な方法は スタックオーバーフローのリスクを減らすが、ヒープ領域に割り当てられるメモリ量には依然注意が必要
- stackを使ったDFSで解いている人もいた。
- 4方向への配列を使わない場合は、範囲チェックとスタックへの追加処理をまとめた関数やラムダを書いていた。
- ラムダを書く場合、コードの量も配列を使った場合と変わらなくなるので、自分は配列を使った解き方の方が拡張しやすくていいと感じた。


```cpp
class Solution {
public:
    int maxAreaOfIsland(const vector<vector<int>>& grid) {
        int num_rows = grid.size();
        int num_cols = grid[0].size();
        vector<vector<uint8_t>> visited(num_rows, vector<uint8_t>(num_cols, 0));
        int max_area = 0;

        for (int row = 0; row < num_rows; ++row) {
            for (int col = 0; col < num_cols; ++col) {
                if (!(grid[row][col] == kLand && !visited[row][col])) {
                    continue;
                }
                int area = CaluculateAreaOfIsland(grid, num_rows, num_cols, row, col, visited);
                max_area = max(max_area, area);
            }
        }
        return max_area;
    }

private:
    static constexpr int kSea = 0;
    static constexpr int kLand = 1;
    static constexpr array<pair<int, int>, 4> kDirections = {{
        {1, 0}, {-1, 0}, {0, 1}, {0, -1}
    }};

    int CaluculateAreaOfIsland(const vector<vector<int>>& grid,
                                int num_rows, int num_cols,
                                int start_row, int start_col,
                                vector<vector<uint8_t>>& visited) {
        stack<pair<int, int>> island_positions;
        island_positions.push({start_row, start_col});
        visited[start_row][start_col] = 1;
        int area = 1;

        while (!island_positions.empty()) {
            auto [row, col] = island_positions.top();
            island_positions.pop();
            for (auto [delta_row, delta_col] : kDirections) {
                int next_row = row + delta_row;
                int next_col = col + delta_col;
                if (!(0 <= next_row && next_row < num_rows && 0 <= next_col && next_col < num_cols)) {
                    continue;
                }
                if (grid[next_row][next_col] == kSea || visited[next_row][next_col]) {
                    continue;
                }
                island_positions.push({next_row, next_col});
                visited[next_row][next_col] = 1;
                ++area;
            }
        }
        return area;
    }
};
```

- https://github.com/Ryotaro25/leetcode_first60/pull/19/files#diff-ae4b13c0679b53a12f53d9ee1112be7a3d179949a3785cee92a29d92256296c4
- https://github.com/irohafternoon/LeetCode/pull/20/files
- UnionFindで解いている人もいた。
- 集合をvectorで管理している人もいたが、自分はmapで管理した方が親子関係が分かりやすく感じた。
- 集合をくっつけた後に、UnionFindの親のサイズを返す関数を使えば、島がないケースにもシンプルに対応できる。
- UnionFindでは、右と下方向しか使わないので、範囲外をチェックする条件文も短くなる。
- https://github.com/irohafternoon/LeetCode/pull/20/files
- 仕事では、多分ヘッダーファイルとソースファイルに分割すると思う。

```cpp
struct Position {
    int row;
    int column;

    auto operator <=> (const Position&) const = default;
};

class UnionFind {
public:
    UnionFind(int num_rows, int num_columns) : num_rows_(num_rows), num_columns_(num_columns) {
        for (int row = 0; row < num_rows_; ++row) {
            for (int column = 0; column < num_columns_; ++column) {
                parents_[{row, column}] = {row, column};
                size_[{row, column}] = 1;
            }
        }
    }

    Position Find(const Position& position) {
        if (parents_[position] == position) {
            return position;
        }
        parents_[position] = Find(parents_[position]);
        return parents_[position];
    }

    void Union(const Position& position1, const Position& position2) {
        auto parent1 = Find(position1);
        auto parent2 = Find(position2);
        if (parent1 == parent2) {
            return;
        }
        if (size_[parent1] < size_[parent2]) {
            swap(parent1, parent2);
        }
        parents_[parent2] = parent1;
        size_[parent1] += size_[parent2];
    }

    int GetSizeOfParent(const Position& position) {
        auto parent = Find(position);
        return size_[parent];
    }

private:
    const int num_rows_;
    const int num_columns_;
    map<Position, Position> parents_;
    map<Position, int> size_;
};

class Solution {
public:
    int maxAreaOfIsland(const vector<vector<int>>& grid) {
        int num_rows = grid.size();
        int num_columns = grid[0].size();
        UnionFind island_group(num_rows, num_columns);
        int max_area = 0;
        for (int row = 0; row < num_rows; ++row) {
            for (int column = 0; column < num_columns; ++column) {
                if (grid[row][column] == kSea) {
                    continue;
                }
                TraverseIsland(grid, num_rows, num_columns, row, column, island_group);
                int area = island_group.GetSizeOfParent({row, column});
                max_area = max(max_area, area);
            }
        }
        return max_area;
    }

private:
    static constexpr int kSea = 0;
    static constexpr array<Position, 2> kDirections = {{
        {1, 0}, {0, 1}
    }};

    void TraverseIsland(const vector<vector<int>>& grid,
                        int num_rows, int num_columns,
                        int row, int column,
                        UnionFind& island_group) {
        for (auto [delta_row, delta_column] : kDirections) {
            int next_row = row + delta_row;
            int next_column = column + delta_column;
            if (!(next_row < num_rows && next_column < num_columns)) {
                continue;
            }
            if (grid[next_row][next_column] == kSea) {
                continue;
            }
            island_group.Union({row, column}, {next_row, next_column});
        }
    }
};
```

## Step3

- 練習のためUnionFindで解いた。
- 量が多いので、タイピングミスが多かった。
- コードのロジックは理解できていて、タイピングミスしかなかったので、2回書いて終了

```cpp
struct Position {
    int row;
    int column;

    auto operator<=>(const Position&) const = default;
};

class UnionFind {
public:
    UnionFind(int num_rows, int num_columns) : num_rows_(num_rows), num_columns_(num_columns) {
        for (int row = 0; row < num_rows_; ++row) {
            for (int column = 0; column < num_columns_; ++column) {
                child_to_parent_[{row, column}] = {row, column};
                size_[{row, column}] = 1;
            }
        }
    }

    Position Find(const Position& position) {
        if (child_to_parent_[position] == position) {
            return position;
        }
        child_to_parent_[position] = Find(child_to_parent_[position]);
        return child_to_parent_[position];
    }

    void Union(const Position& position1, const Position& position2) {
        auto parent1 = Find(position1);
        auto parent2 = Find(position2);
        if (parent1 == parent2) {
            return;
        }
        if (size_[parent1] < size_[parent2]) {
            swap(parent1, parent2);
        }
        child_to_parent_[parent2] = parent1;
        size_[parent1] += size_[parent2];
    }

    int GetSizeOfUnion(const Position& position) {
        auto parent = Find(position);
        return size_[parent];
    }

private:
    const int num_rows_;
    const int num_columns_;
    map<Position, Position> child_to_parent_;
    map<Position, int> size_;
};

class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int num_rows = grid.size();
        int num_columns = grid[0].size();
        UnionFind island_group(num_rows, num_columns);
        int max_area = 0;

        for (int row = 0; row < num_rows; ++row) {
            for (int column = 0; column < num_columns; ++column) {
                if (grid[row][column] == kSea) {
                    continue;
                }
                TraverseIsland(grid, num_rows, num_columns, row, column, island_group);
                int area = island_group.GetSizeOfUnion({row, column});
                max_area = max(max_area, area);
            }
        }
        return max_area;
    }

private:
    static constexpr int kSea = 0;
    static constexpr array<Position, 2> kDirections = {{
        {1, 0}, {0, 1}
    }};

    void TraverseIsland(const vector<vector<int>>& grid,
                        int num_rows, int num_columns,
                        int row, int column,
                        UnionFind& island_group) {
        for (auto [row_offset, column_offset] : kDirections) {
            int next_row = row + row_offset;
            int next_column = column + column_offset;
            if (!(next_row < num_rows && next_column < num_columns)) {
                continue;
            }
            if (grid[next_row][next_column] == kSea) {
                continue;
            }
            island_group.Union({row, column}, {next_row, next_column});
        }        
    }
};
```
