# Merge Two Binary Trees
* 問題: https://leetcode.com/problems/merge-two-binary-trees/
* 言語: C++

## Step1

- BFSで、順に子ノードを足した新しいノードを作って繋げていく方法をまず思いついた。
- 実装を試したが、結構書くのが大変そうだったのでDFSで解けないか考えてみた。
- newしたノードの左と右につなげるときに再帰をする
- 片方のノードがnullptrならもう片方を返す。
- DFSの実装は上手くできなかった。
- 下のコードだと、ノードを共有されてしまうのが気になる。

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1) {
            return root2;
        }
        if (!root2) {
            return root1;
        }
        TreeNode* merged_node = new TreeNode(root1->val + root2->val);
        merged_node->left = mergeTrees(root1->left, root2->left);
        merged_node->right = mergeTrees(root1->right, root2->right);
        return merged_node;
    }
};
```

## Step2

- https://discord.com/channels/1084280443945353267/1206101582861697046/1334883898378682378
- リンクのコードなら、ノードの共有をせずに済む。ノードがnullptrならdummyノードにしてからnewすることでノードの共有を回避している。

- https://github.com/irohafternoon/LeetCode/pull/26/files#r2056460029
- 分散処理などで使う状況があるのか。
- gitのマージも似た処理を行っていそう。

- https://github.com/irohafternoon/LeetCode/pull/26/files
- BFSはタプルをキューに入れて管理すればよかったのか。3個のキューでやろうとしていた。
- マージノードはデフォルトでnewし、キューから取り出したときに値を変更するのか。自分はノードの合計値をもとめてからマージノードにしようとしていた。

```cpp
class Solution {
public:
    TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        queue<tuple<const TreeNode*, const TreeNode*, TreeNode*>> nodes_and_merged;
        TreeNode* new_root = new TreeNode();
        nodes_and_merged.emplace(root1, root2, new_root);
        while (!nodes_and_merged.empty()) {
            auto [node1, node2, merged_node] = nodes_and_merged.front();
            nodes_and_merged.pop();
            if (!node1) {
                node1 = &kDummy;
            }
            if (!node2) {
                node2 = &kDummy;
            }
            merged_node->val = node1->val + node2->val;
            if (node1->left || node2->left) {
                merged_node->left = new TreeNode();
                nodes_and_merged.emplace(node1->left, node2->left, merged_node->left);
            }
            if (node1->right || node2->right) {
                merged_node->right = new TreeNode();
                nodes_and_merged.emplace(node1->right, node2->right, merged_node->right);
            }

        }
        return new_root;
    }
private:
    const TreeNode kDummy = {};
};
```

- https://github.com/irohafternoon/LeetCode/pull/26/files#r2059368013
- ダブルポインタを使うとif文を減らせる
- とりあえずノードをキューに入れて、無効なノード同士だったら、マージノードをnullptrにしたい。ダブルポインタを使うことで変更可能になる。
- しかし、リンク先のコードでは、nullptrにすると確保していたヒープメモリが残ってしまっていた（メモリリーク）
- そのため、書き換える際に、deleteすることでメモリリークを避けた。
- メモリ管理がわかりづらくなるのでダブルポインタを使わないほうがいいかもしれない。

```cpp
class Solution {
public:
    TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
        queue<tuple<const TreeNode*, const TreeNode*, TreeNode**>> nodes_and_merged;
        TreeNode* new_head = new TreeNode();
        nodes_and_merged.emplace(root1, root2, &new_head);
        while (!nodes_and_merged.empty()) {
            auto [node1, node2, merged_node_ptr] = nodes_and_merged.front();
            nodes_and_merged.pop();

            if (!node1 && !node2) {
                delete *merged_node_ptr;
                *merged_node_ptr = nullptr;
                continue;
            }
            if (!node1) {
                node1 = &kDummy;
            }
            if (!node2) {
                node2 = &kDummy;
            }
            (*merged_node_ptr)->val = node1->val + node2->val;
            (*merged_node_ptr)->left = new TreeNode();
            nodes_and_merged.emplace(node1->left, node2->left, &((*merged_node_ptr)->left));
            (*merged_node_ptr)->right = new TreeNode();
            nodes_and_merged.emplace(node1->right, node2->right, &((*merged_node_ptr)->right));
        }
        return new_head;
    }
private:
    const TreeNode kDummy = TreeNode();
};
```

- https://github.com/colorbox/leetcode/pull/37/files
- スタックを使った解き方もあった。
- 三項演算子を使うとかなりコンパクトになっていいかもしれない。
- リンク先の617/step2_4.cppはダブルポインタを使っているが、スタックにmerged_node（nullptr）を入れている。
- スタックから取り出した後に、nullptrから、新しいノードに変更しているので。このやり方だとメモリリークの心配はなさそう。


## Step3

```cpp
class Solution {
public:
    TreeNode* mergeTrees(const TreeNode* root1, const TreeNode* root2) {
        if (!root1 && !root2) {
            return nullptr;
        }
        if (!root1) {
            root1 = &kDummy;
        }
        if (!root2) {
            root2 = &kDummy;
        }
        TreeNode* node = new TreeNode(root1->val + root2->val);
        node->left = mergeTrees(root1->left, root2->left);
        node->right = mergeTrees(root1->right, root2->right);
        return node;
    }
private:
    const TreeNode kDummy{};
};
```
