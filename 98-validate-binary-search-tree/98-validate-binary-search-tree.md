# Validate Binary Search Tree
* 問題：http://leetcoe.com/problems/validate-binary-search-tree/
* 言語：C++

## Step1

- 単純に、親ノードの値と比較していけばよいかなと思ったが、右BSTのすべてのノードは自ノードより大きく、左BSTは自ノードより小さい必要があった。
- なんとなく、最大値と最小値を使って比較していくのかなと考えたが、どう実装するかで悩んだ。
- 木のサイズが1 ~ 10000なので、再帰は使わない方が良いかなと思ったので、非再帰のDFSかBFSでの解き方を考えた。
- 30分以上考えたが、わからなかったので他の人のコードを見る。
- https://github.com/irohafternoon/LeetCode/pull/31/files のstep1
- 参考にした人は、step1では再帰で書いていた。
- node->valの範囲がINT_MIN ~ INT_MAXなので、long longを使う必要があるのか
- min_val < node->val < max_val が満たすべき条件で、範囲外ならfalseを返す。

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        return IsValidBSTHelper(root, LLONG_MIN, LLONG_MAX);
    }
private:
    bool IsValidBSTHelper(TreeNode* node, long long min_value, long long max_value) {
        if (!node) {
            return true;
        }
        if (node->val <= min_value || node->val >= max_value) {
            return false;
        }
        return IsValidBSTHelper(node->left, min_value, node->val) &&
                IsValidBSTHelper(node->right, node->val, max_value);
    }
};
```

- 回答を参考にして書いたBFS
- nullチェックはしなくてよかったかも。

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        queue<tuple<TreeNode*, long long, long long>> nodes_and_ranges;
        nodes_and_ranges.emplace(root, LLONG_MIN, LLONG_MAX);
        while (!nodes_and_ranges.empty()) {
            auto [node, min_value, max_value] = nodes_and_ranges.front();
            nodes_and_ranges.pop();
            if (!(min_value < node->val && node->val < max_value)) {
                return false;
            }
            if (node->left) {
                nodes_and_ranges.emplace(node->left, min_value, node->val);
            }
            if (node->right) {
                nodes_and_ranges.emplace(node->right, node->val, max_value);
            }
        }
        return true;
    }
};
```

##Step2

- https://github.com/irohafternoon/LeetCode/pull/31/files
- 再帰か非再帰で解くかは考えたが、in-order, pre-orderなどの選択肢はなかった。
- 番兵の値を使うほかに、ポインタでも書ける
- pre-order （行きがけ順）: 根を調べてから、部分木を調べる。例：親ノード → 左子ノード → 右子ノードの順番
- in-order（通りがけ順）： 片方の部分木を調べて、次に根を調べて、反対の部分木を調べる。例：左子ノード → 親ノード → 右子ノードの順番
- in-orderなら、前に見たノードの値より大きいのかを調べればよい
- post-order（帰りがけ順）： 部分木を調べてから、根を調べる。例：左子ノード → 右子ノード → 親ノードの順番
- in-orderは、少し分かりづらいと感じた。

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        bool result = true;
        long long pre_value = LLONG_MIN;
        IsValidBSTHelper(root, pre_value, result);
        return result;
    }
private:
    void IsValidBSTHelper(TreeNode* node, long long& pre_value, bool& result) {
        if (!result) {
            return;
        }
        if (!node) {
            return;
        }
        IsValidBSTHelper(node->left, pre_value, result);
        if (!(node->val > pre_value)) {
            result = false;
            return;
        }
        pre_value = node->val;
        IsValidBSTHelper(node->right, pre_value, result);
    }
};
```

- in-order 非再帰のコードがあったので、自分も書いてみた。
- 非再帰の方が、コードが簡潔になっていて好み。
- node変数を複数個所で動かしているのが、わかりにくいかもしれない。
- in-orderよりも、pre-order BFSの方が、シンプルな解き方で無難かな。

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*> in_order_nodes;
        long long previous_value = LLONG_MIN;
        TreeNode* node = root;
        while (true) {
            while (node) {
                in_order_nodes.push(node);
                node = node->left;
            }
            if (in_order_nodes.empty()) {
                break;
            }
            node = in_order_nodes.top();
            in_order_nodes.pop();
            if (!(node->val > previous_value)) {
                return false;
            }
            previous_value = node->val;
            node = node->right;
        }
        return true;
    }
};
```

- https://github.com/olsen-blue/Arai60/pull/28/files
- https://github.com/colorbox/leetcode/pull/42/files
- 最小、最大の変数は、low, high, under, upper, lower, lower_boundなどいろいろある。
- 下限と上限の意味がしっくり来たので、lower_bound, upper_boundが良いと感じた。
- 

##Step3

- BFSがしっくり来たので、3回

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        queue<tuple<TreeNode*, long long, long long>> nodes_and_ranges;
        nodes_and_ranges.emplace(root, LLONG_MIN, LLONG_MAX);
        while (!nodes_and_ranges.empty()) {
            auto [node, lower_bound, upper_bound] = nodes_and_ranges.front();
            nodes_and_ranges.pop();
            if (!node) {
                continue;
            }
            if (!(lower_bound < node->val && node->val < upper_bound)) {
                return false;
            }
            nodes_and_ranges.emplace(node->left, lower_bound, node->val);
            nodes_and_ranges.emplace(node->right, node->val, upper_bound);
        }
        return true;
    }
};
```
