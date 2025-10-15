# Pat Sum
* 問題: https://leetcode.com/problems/path-sum/
* 言語: C++

## Step1

- DFSとBFSの両方で解いた。
- 最初はBFSで解いたが、葉ノードが条件なのを忘れていて少し時間がかかった。
- DFSは再帰を書くのに時間が少しかかった。

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        queue<pair<TreeNode*, int>> node_and_sum;
        node_and_sum.emplace(root, root->val);
        while (!node_and_sum.empty()) {
            auto [node, sum] = node_and_sum.front();
            node_and_sum.pop();
            if (sum == targetSum && !node->left && !node->right) {
                return true;
            }
            if (node->left) {
                node_and_sum.emplace(node->left, sum + node->left->val);
            }
            if (node->right) {
                node_and_sum.emplace(node->right, sum + node->right->val);
            }
        }
        return false;
    }
};
```

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        if (!root->left && !root->right) {
            return targetSum == root->val;
        }
        int remaining = targetSum - root->val;
        if (hasPathSum(root->left, remaining)) {
            return true;
        }
        if (hasPathSum(root->right, remaining)) {
            return true;
        }
        return false;
    }
};
```

## Step2

- BFSの改良版
- ひとつ前までの合計値と今のノードの値との和がtargetSumと等しかったらtrueを返すようにした。
- キューにとりあえずノードを入れて、取り出したノードがnullptrならスルー
- 最初のif文とループ処理のif文を減らせて、step1よりスッキリした。

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        queue<pair<TreeNode*, int>> nodes_and_sums;
        nodes_and_sums.emplace(root, 0);
        while (!nodes_and_sums.empty()) {
            auto [node, sum] = nodes_and_sums.front();
            nodes_and_sums.pop();
            if (!node) {
                continue;
            }
            if (sum + node->val == targetSum && !node->left && !node->right) {
                return true;
            }
            nodes_and_sums.emplace(node->left, sum + node->val);
            nodes_and_sums.emplace(node->right, sum + node->val);
        }
        return false;
    }
};
```

- DFSの改良版
- 再帰で返ってきた内、どちらかがtrueならtrueを返せばよいので、orでreturnすればいい。
- step1とやっていることは変わらないが、こちらの方が無駄なif文がなく、スッキリしている。

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        if (!root->left && !root->right) {
            return targetSum == root->val;
        }
        int remaining = targetSum - root->val;
        return hasPathSum(root->left, remaining) || hasPathSum(root->right, remaining);
    }
};
```
* 再帰上限について
- https://github.com/irohafternoon/LeetCode/pull/28/files
- https://github.com/ryoooooory/LeetCode/pull/28/files#r1986234552
- https://discord.com/channels/1084280443945353267/1235829049511903273/1236256946403807323
- 再帰上限について考えてなかった。5000くらいだから大丈夫かなーと思っていたが、スタックオーバーフローが起こる可能性があるらしい。
- スタックメモリの上限がおよそ1 MB程度。言語の環境によって変わるらしく、C/C++だと最大10 MB。
- スタックフレーム：https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%BC%E3%83%AB%E3%82%B9%E3%82%BF%E3%83%83%E3%82%AF
- スタックフレームは関数呼び出し1回分の実行状態を保存するための領域
- リターンアドレスやベースポインタ、ローカル関数、引数などが含まれている。
- 再帰は、独立したスタックフレームが作成される。
- 上記の再帰のコードだと、スタックフレームのサイズが大体32 ~ 64 B程度。
- スタックの上限が1 MBと仮定すると、10^6 / 64 ≒ 15000 となるので今回のコードはスタックオーバーフローの心配はなさそう。
- 今回はスタックフレームのサイズが小さいのでよかったが、10000回再帰するのはまずそう。

## Step3

- BFS, DFSどちらも3回

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        if (!root->left && !root->right) {
            return targetSum == root->val;
        }
        int remaining = targetSum - root->val;
        return hasPathSum(root->left, remaining) || hasPathSum(root->right, remaining);
    }
};
```

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        queue<pair<TreeNode*, int>> nodes_and_sums;
        nodes_and_sums.emplace(root, 0);
        while (!nodes_and_sums.empty()) {
            auto [node, sum] = nodes_and_sums.front();
            nodes_and_sums.pop();
            if (!node) {
                continue;
            }
            sum += node->val;
            if (sum == targetSum && !node->left && !node->right) {
                return true;
            }
            nodes_and_sums.emplace(node->left, sum);
            nodes_and_sums.emplace(node->right, sum);
        }
        return false;
    }
};
```
