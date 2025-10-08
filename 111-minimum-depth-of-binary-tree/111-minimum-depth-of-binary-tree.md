# Minimum Depth of Binary Tree
* 問題: https://leetcode.com/problems/minimum-depth-of-binary-tree/
* 言語: C++

## Step1

- 前回と同じように、DFSとBFSの両方で解いてみた。
- DFSは左ノードや右ノードがない場合のケースも考慮する必要がある。

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        if (!root->left) {
            return minDepth(root->right) + 1;
        }
        if (!root->right) {
            return minDepth(root->left) + 1;
        }

        return min(minDepth(root->left), minDepth(root->right)) + 1;
    }
};
```

- BFSはqueueでノードと深さを管理する。葉ノードに達したときの深さが最小の深さなのでreturn
- ノードと深さを別にする解き方もあるが、ループが増えるので一緒にqueueで管理した方が楽だと思ったので、一緒にして実装。

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        queue<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.emplace(root, 1);
        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.front();
            node_and_depth.pop();
            if (!node->left && !node->right) {
                return depth;
            }
            if (node->left) {
                node_and_depth.emplace(node->left, depth + 1);
            }
            if (node->right) {
                node_and_depth.emplace(node->right, depth + 1);
            }
        }
        return 0;
    }
};
```

## Step2

- https://github.com/irohafternoon/LeetCode/pull/24/files
- https://github.com/nktr-cp/leetcode/pull/23/files
- 深さごとに管理する方法で解いてみた。
- https://github.com/irohafternoon/LeetCode/pull/24/files#r2052736369
- とりあえずpush_backして、nullptrだったら飛ばすのでもいいのか。
- 無限ループにしたら、末行でreturnしなくて済むのか。
- https://stackoverflow.com/questions/58520107/do-i-need-a-return-statement-after-an-infinite-loop?utm_source=chatgpt.com
- 問題はないそうだが、コンパイラによっては警告を出すらしい。return文を省略しなくてもいい気がする。
  
```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        vector<TreeNode*> current_nodes{root};
        int depth = 1;
        while (true) {
            vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (!node->left && !node->right) {
                    return depth;
                }
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
            }
            current_nodes.swap(next_nodes);
            ++depth;
        }
    }
};
```

- とりあえずキューに入れて、nullptrだったら飛ばす
- root == nullptrだったら、末行で0が返ってくる
- こっちの方がすっきりしていて好み。

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        queue<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.emplace(root, 1);
        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.front();
            node_and_depth.pop();
            if (!node) {
                continue;
            }
            if (!node->left && !node->right) {
                return depth;
            }
            node_and_depth.emplace(node->left, depth + 1);
            node_and_depth.emplace(node->right, depth + 1);
        }
        return 0;
    }
};
```

## Step3

- DFS

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        if (!root->left) {
            return minDepth(root->right) + 1;
        }
        if (!root->right) {
            return minDepth(root->left) + 1;
        }
        return min(minDepth(root->left), minDepth(root->right)) + 1;
    }
};
```
  
- BFS

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        queue<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.emplace(root, 1);
        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.front();
            node_and_depth.pop();
            if (!node) {
                continue;
            }
            if (!node->left && !node->right) {
                return depth;
            }
            node_and_depth.emplace(node->left, depth + 1);
            node_and_depth.emplace(node->right, depth + 1);
        }
        return 0;
    }
};
```
