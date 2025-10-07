# Maximum Depth of Binary Tree
* 問題: https://leetcode.com/problems/maximum-depth-of-binary-tree/
* 言語: C++

## Step1

- 最初、ループで愚直にやろうと考えたが、実装しづらかったので他の方法を考えた。
- スタックを使う方法を思いついた。再帰でも解けそう。
- 一度見たノードはsetに入れることで、同じノードを見なくて済む。
- スタックを使って解いてみる。
- スタックに根ノードを入れる。スタックトップにあるノードの子をスタックとsetに入れながら辿っていく。
- 一番下のノードまで来たら、スタックのサイズがその経路の深さになるのでmax関数で比べる。
- 最後に、スタックの先頭をpopして、別の経路を調べる。
- n : the number of nodes in the tree
- 時間計算量：O(n * log n)
- 空間計算量：O(n)
- スタックにはtreeの経路を入れるので、pathにしようと思ったが、c++にはpathというクラスがあるのでtree_pathにした。
- 子ノードを辿っていくループのif文が、書いていて気になった。ネストを浅くできそうな感じがする。
- すべてのノードを調べ終えたら、スタックを空にするまでpopしてループなどの処理をするのが無駄に感じたが、せいぜい計算量が2倍になる程度なので別にいいか。

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        int max_depth = 1;
        stack<TreeNode*> tree_path;
        tree_path.push(root);
        set<TreeNode*> visited{root};
        
        while (!tree_path.empty()) {
            TreeNode* node = tree_path.top();

            while (node->left || node->right) {
                if (node->left && !visited.contains(node->left)) {
                    node = node->left;
                    tree_path.push(node);
                    visited.emplace(node);
                    continue;
                } else if (node->right && !visited.contains(node->right)) {
                    node = node->right;
                    tree_path.push(node);
                    visited.emplace(node);
                    continue;
                } else {
                    break;
                }
            }
            int depth = tree_path.size();
            max_depth = max(max_depth, depth);
            tree_path.pop();
        }
        return max_depth;
    }
};
```

- やっていることはDFSなので、BFSでも解けそう。
- BFSなら親ノードに戻る操作がいらないので、setがなくて済む。
- BFSの方が、簡潔に書けた。
- こちらの方が好み。

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        int max_depth = 1;
        queue<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.push({root, 1});

        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.front();
            node_and_depth.pop();
            max_depth = max(max_depth, depth);

            if (node->left) {
                node_and_depth.push({node->left, depth + 1});
            } 
            if (node->right) {
                node_and_depth.push({node->right, depth + 1});
            }
        }
        return max_depth;
    }
};
```

## Step2

- https://github.com/nktr-cp/leetcode/pull/22/files
- こんな簡潔に解けるんか
- 再帰を書くのも読むのも苦手な気がする。

- stackを使う解き方の改良版
  
```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        stack<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.emplace(root, 1);
        int max_depth = 1;

        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.top();
            node_and_depth.pop();
            max_depth = max(max_depth, depth);
            if (node->left) {
                node_and_depth.emplace(node->left, depth + 1);
            }
            if (node->right) {
                node_and_depth.emplace(node->right, depth + 1);
            }
        }
        return max_depth;
    }
};
```

- https://github.com/irohafternoon/LeetCode/pull/23/files
- BFSを2つのvectorを使って解いている人がいた。
- 深さごとのノードの配列を作り、それを上から順に見ていくイメージ
- 配列が空でないときに深さを+1することで、最初のdepthを1から始められる。
- 1から始めるためにif文を追加するのは、無駄に感じる。
- 自分はstep1のBFSの方が自然で好み。

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        vector<TreeNode*> current_nodes{root};
        int depth = 1;
        while (!current_nodes.empty()) {
            vector<TreeNode*> next_nodes;
            for (auto node : current_nodes) {
                if (node->left) {
                    next_nodes.push_back(node->left);
                }
                if (node->right) {
                    next_nodes.push_back(node->right);
                }
            }
            current_nodes.swap(next_nodes);
            if (!current_nodes.empty()) {
                ++depth;
            }
        }
        return depth;
    }
};
```

## Step3

- DFS

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

- BFS

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        queue<pair<TreeNode*, int>> node_and_depth;
        node_and_depth.emplace(root, 1);
        int max_depth = 1;

        while (!node_and_depth.empty()) {
            auto [node, depth] = node_and_depth.front();
            node_and_depth.pop();
            max_depth = max(max_depth, depth);
            if (node->left) {
                node_and_depth.emplace(node->left, depth + 1);
            }
            if (node->right) {
                node_and_depth.emplace(node->right, depth + 1);
            }
        }
        return max_depth;
    }
};
```
