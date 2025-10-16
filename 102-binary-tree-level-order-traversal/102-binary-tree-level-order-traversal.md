# Binary Tree Level Order Traversal
* 問題: https://leetcode.com/problems/binary-tree-level-order-traversal/
* 言語: C++

## Step1

- 今回の問題はBFSが解きやすそう
- DFS（再帰）で解くとすると、二次元配列を返すので、再帰で書くとスタックフレームのサイズが大きくなりそう。
- DFSでやろうとするよりも、BFSの方が素直に書けそうなのでBFSで解く。
- キューでも書けそうだが、二次元配列を返すので、vectorを使って解いた方がやりやすそう。
- レベルごとにvectorを作成して、for文でノードの値をvectorに入れていく。
- 最後に返す二次元配列の良い名前が思い浮かばなかったので、resultにした。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        vector<TreeNode*> level{root};
        while (!level.empty()) {
            vector<int> level_values;
            vector<TreeNode*> next_level;
            for (auto node : level) {
                if (!node) {
                    continue;
                }
                level_values.emplace_back(node->val);
                next_level.emplace_back(node->left);
                next_level.emplace_back(node->right);
            }
            level.swap(next_level);
            if (level_values.empty()) {
                continue;
            }
            result.emplace_back(level_values);
        }
        return result;
    }
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/29/files
- 直接、二次元配列に値を入れて解いていた。
- 深さをカウントする。
- 二次元配列に空のvectorを作成し、深さをインデックスとして直接、値を入れていく。
- 自分は数値を入れるためにvectorを作成していたが、直接入れた方が効率がよいので自分もそうする。
- ノードをチェックせずに入れると、最後に空のvectorが残るので、それを消す必要がある。
- ノードをチェックすれば、削除する必要はなくなる。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (!root) {
            return {};
        }
        vector<vector<int>> level_ordered_vals;
        vector<TreeNode*> nodes{root};
        int depth = 0;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            level_ordered_vals.emplace_back();
            for (auto node : nodes) {
                if (!node) {
                    continue;
                }
                level_ordered_vals[depth].emplace_back(node->val);
                next_nodes.emplace_back(node->left);
                next_nodes.emplace_back(node->right);
            }
            if (level_ordered_vals[depth].empty()) {
                level_ordered_vals.pop_back();
            }
            nodes.swap(next_nodes);
            ++depth;
        }
        return level_ordered_vals;
    }
};
```

- https://github.com/irohafternoon/LeetCode/pull/29/files
- DFSでも解いていた。
- 二次元配列と引数、深さとノードを引数で渡す。
- 深さとサイズが等しかったら、二次元配列の中に空のvectorを作成する。
- 深さをインデックスとして、ノードの値を入れていく。
- DFSでも解けるらしいが、二次元配列を引数として再帰をするのは怖い気がする。
- DFSよりもBFSで解く方が、わかりやすいし、読みやすいと思う。

- https://github.com/irohafternoon/LeetCode/pull/29/files#r2061836454
- キューでも解いていた。
- キューならrootが空の場合もまとめて処理できるし、ノードをチェックせずに入れても空のvectorは作られない。
- キューの方が楽に解けるのと、コンパクトに書けるのがいい。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> level_ordered_values;
        queue<pair<TreeNode*, int>> nodes_and_levels;
        nodes_and_levels.emplace(root, 0);
        while (!nodes_and_levels.empty()) {
            auto [node, level] = nodes_and_levels.front();
            nodes_and_levels.pop();
            if (!node) {
                continue;
            }
            if (level == level_ordered_values.size()) {
                level_ordered_values.emplace_back();
            }
            level_ordered_values[level].emplace_back(node->val);
            ++level;
            nodes_and_levels.emplace(node->left, level);
            nodes_and_levels.emplace(node->right, level);
        }
        return level_ordered_values;
    }
};
```

- https://github.com/colorbox/leetcode/pull/40/files
- DFSで解いていた。
- こちらは、左ノード、右ノードで二次元配列を作成して、返ってきた二次元配列を同じレベルで調整する解き方。
- 少し、わかりにくかった。
- 書くのは楽そうな感じもするが、読み手が少し辛い気もするので、自分はBFSの解き方がいい。
- キューを使ったBFSの方が楽に考えれて、読みやすそうなのでstep3ではキューで解く。

## Step3

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> level_ordered_values;
        queue<pair<TreeNode*, int>> nodes_and_levels;
        nodes_and_levels.emplace(root, 0);
        while (!nodes_and_levels.empty()) {
            auto [node, level] = nodes_and_levels.front();
            nodes_and_levels.pop();
            if (!node) {
                continue;
            }
            if (level == level_ordered_values.size()) {
                level_ordered_values.emplace_back();
            }
            level_ordered_values[level].emplace_back(node->val);
            ++level;
            nodes_and_levels.emplace(node->left, level);
            nodes_and_levels.emplace(node->right, level);
        }
        return level_ordered_values;
    }
};
```
