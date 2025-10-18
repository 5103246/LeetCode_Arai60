# Binary Tree Zigzag Level Order Traversal
* 問題: https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/
* 言語: C++

## Step1

- 最初はキューを使った解き方で行けそうだと思ったので、キューで解いてみた。
- レベルが偶数なら右ノードから入れていき、奇数なら左ノードから入れていけば解けそう。
- テストケースが通ったので、提出したら[1,2,3,4,null,null,5]のケースで間違っていた。
- 下のコードだと、[[1], [3,2], [5,4]]になってしまう。

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> zigzag_level_ordered;
        queue<pair<TreeNode*, int>> nodes_and_levels;
        nodes_and_levels.emplace(root, 0);
        while (!nodes_and_levels.empty()) {
            auto [node, level] = nodes_and_levels.front();
            nodes_and_levels.pop();
            if (!node) {
                continue;
            }
            if (level == zigzag_level_ordered.size()) {
                zigzag_level_ordered.emplace_back();
            }
            zigzag_level_ordered[level].emplace_back(node->val);
            ++level;
            if (level % 2 == 0) {
                nodes_and_levels.emplace(node->left, level);
                nodes_and_levels.emplace(node->right, level);
            } else {
                nodes_and_levels.emplace(node->right, level);
                nodes_and_levels.emplace(node->left, level);
            }
        }
        return zigzag_level_ordered;
    }
};
```

- 少し、こんがらがったので紙に書きながら考えた。
- キューじゃ解きづらそうに感じたので、階層をvectorに入れて解く方向で考えた。
- for文で回す方向を後ろからにして、レベルが偶数なら左ノードからvectorに入れて、奇数なら右ノードから入れる。
- [1,2,3,4,null,null,5] の場合
- [1] → [2,3]
- [2,3]を後ろから回すので、[[1],[3,2]]で [2,] → [5, null, null, 4] になる。
- [[1],[3,2],[4,5]]になってok
- 最後に、空のvectorが残っているので削除してから返す。
- このコードだと、vectorにノードを加える際にnullチェックをすることもできるが、if文が増えてネストも深くなるので、nullチェックしない。
- ただ、最後に必ず、空のvectorが残るので削除する必要がある。

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if (!root) {
            return {};
        }
        vector<vector<int>> zigzag_level_ordered;
        vector<TreeNode*> nodes{root};
        int level = 0;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            zigzag_level_ordered.emplace_back();
            for (int i = nodes.size() - 1; i >= 0; --i) {
                if (!nodes[i]) {
                    continue;
                }
                zigzag_level_ordered[level].emplace_back(nodes[i]->val);
                if (level % 2 == 0) {
                    next_nodes.emplace_back(nodes[i]->left);
                    next_nodes.emplace_back(nodes[i]->right);
                } else {
                    next_nodes.emplace_back(nodes[i]->right);
                    next_nodes.emplace_back(nodes[i]->left);
                }
            }
            nodes.swap(next_nodes);
            ++level;
        }
        zigzag_level_ordered.pop_back();
        return zigzag_level_ordered;
    }
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/30/files
- vectorを反転した解き方
- 反転する方法として、reverse, deque, rbegin rendなどがあった。
- bool変数を使うなら、レベルを管理する変数はいらないと感じた。
- https://github.com/colorbox/leetcode/pull/41/files
- https://github.com/Ryotaro25/leetcode_first60/pull/29/files
- https://github.com/olsen-blue/Arai60/pull/27/files
- 他の人も大体、リストを反転した解き方をしていた。
- step1の解き方と比べて、反転した方がわかりやすい感じがする。
- step1はreverseしない分、少し効率がよいと思うが、反転する解き方の方が読みやすかった。
- 反転する方法は、dequeやrbeginよりも、reverseの方が意図が伝わりやすいと感じた。

- step1の改良版

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> zigzag_ordered_values;
        vector<TreeNode*> nodes{root};
        bool left_to_right = true;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            zigzag_ordered_values.emplace_back();
            for (int i = nodes.size() - 1; i >= 0; --i) {
                if (!nodes[i]) {
                    continue;
                }
                zigzag_ordered_values.back().emplace_back(nodes[i]->val);
                if (left_to_right) {
                    next_nodes.emplace_back(nodes[i]->left);
                    next_nodes.emplace_back(nodes[i]->right);
                } else {
                    next_nodes.emplace_back(nodes[i]->right);
                    next_nodes.emplace_back(nodes[i]->left);
                }
            }
            nodes.swap(next_nodes);
            left_to_right = !left_to_right;
        }
        zigzag_ordered_values.pop_back();
        return zigzag_ordered_values;
    }
};
```

- 反転する解き方

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if (!root) {
            return {};
        }
        vector<vector<int>> zigzag_ordered_values;
        vector<TreeNode*> nodes{root};
        bool right_to_left = false;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            zigzag_ordered_values.emplace_back();
            for (auto node : nodes) {
                zigzag_ordered_values.back().emplace_back(node->val);
                if (node->left) {
                    next_nodes.emplace_back(node->left);
                }
                if (node->right) {
                    next_nodes.emplace_back(node->right);
                }
            }
            if (right_to_left) {
                reverse(zigzag_ordered_values.back().begin(),
                        zigzag_ordered_values.back().end());
            }
            nodes.swap(next_nodes);
            right_to_left = !right_to_left;
        }
        return zigzag_ordered_values;
    }
};
```

- ノードの値を入れたvectorを経由するコード
- こちらの方が、処理の流れがわかりやすいかもしれない
- 二次元配列の要素を直接reverseするよりも、二次元配列に入れるvectorをreverseした方が読み手にはわかりやすそう。

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if (!root) {
            return {};
        }
        vector<vector<int>> zigzag_ordered_values;
        vector<TreeNode*> nodes{root};
        bool right_to_left = false;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            vector<int> values;
            for (auto node : nodes) {
                values.emplace_back(node->val);
                if (node->left) {
                    next_nodes.emplace_back(node->left);
                }
                if (node->right) {
                    next_nodes.emplace_back(node->right);
                }
            }
            if (right_to_left) {
                reverse(values.begin(), values.end());
            }
            zigzag_ordered_values.emplace_back(values);
            nodes.swap(next_nodes);
            right_to_left = !right_to_left;
        }
        return zigzag_ordered_values;
    }
};
```

## Step3

- 反転する解き方で3回

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        if (!root) {
            return {};
        }
        vector<vector<int>> zigzag_ordered_values;
        vector<TreeNode*> nodes{root};
        bool right_to_left = false;
        while (!nodes.empty()) {
            vector<TreeNode*> next_nodes;
            vector<int> values;
            for (auto node : nodes) {
                values.emplace_back(node->val);
                if (node->left) {
                    next_nodes.emplace_back(node->left);
                }
                if (node->right) {
                    next_nodes.emplace_back(node->right);
                }
            }
            if (right_to_left) {
                reverse(values.begin(), values.end());
            }
            zigzag_ordered_values.emplace_back(values);
            nodes.swap(next_nodes);
            right_to_left = !right_to_left;
        }
        return zigzag_ordered_values;
    }
};
```
