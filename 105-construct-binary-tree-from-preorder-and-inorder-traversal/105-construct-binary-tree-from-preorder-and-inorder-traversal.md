# Construct Binary Tree from Preorder and Inorder Traversal
* 問題：https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/
* 言語：C++

## Step1

- preorderとinorderから二分木を作成する問題
- inorderの最初の値は一番左のノードで、最後の値が一番右のノードになる
- preorderで木を作りながら、inorderの最初の値が来たらinorderに切り替えるのか？
- 固有の値しかないので、setかmapを使いそう
- 15分以上考えたが、どう解けばいいかわかなかったので、他の人の解き方を見てみる
  
- https://github.com/irohafternoon/LeetCode/pull/32/files のstep1

1. valueとそれがinorderのどこにあるかをmapで対応付ける（value : inorder_index）
2. DFS関数を呼ぶ。渡すものは、preorder、区間[left, right]、map, preorderのindex
3. preorderのindexがリストのサイズを超えたらnullptrを返す。
4. preorderの値に対応するinorder_indexが[left,right]内になかったらnullptr
5. preorderの値でnewして、左ノードと右ノードで再帰する
6. preorder_indexを+1し、左ノードは[left,inorder_index-1]、右ノードは[inorder+1, right]を渡す。

```cpp
class Solution {
public:
    TreeNode* buildTree(const vector<int>& preorder, const vector<int>& inorder) {
        map<int, int> value_to_inorder_index;
        for (int i = 0; i < inorder.size(); ++i) {
            value_to_inorder_index[inorder[i]] = i;
        }
        int preorder_index = 0;
        return BuildTreeHelper(preorder,value_to_inorder_index, 0, inorder.size() - 1, preorder_index);
    }
private:
    TreeNode* BuildTreeHelper(const vector<int>& preorder, 
                              const map<int, int>& value_to_inorder_index,
                              int left, int right,
                              int& preorder_index) {
        if (!(preorder_index < preorder.size())) {
            return nullptr;
        }
        int value = preorder[preorder_index];
        int inorder_index = value_to_inorder_index.at(value);
        if (!(left <= inorder_index && inorder_index <= right)) {
            return nullptr;
        }
        TreeNode* node = new TreeNode(value);
        ++preorder_index;
        node->left = BuildTreeHelper(preorder, value_to_inorder_index, left, inorder_index - 1, preorder_index);
        node->right = BuildTreeHelper(preorder, value_to_inorder_index, inorder_index + 1, right, preorder_index);
        return node;
        }
};
```

## Step2

- https://github.com/kazukiii/leetcode/pull/30/files　arai60/construct-binary-tree-from-preorder-and-inorder-traversal/step2.cpp (※)
- preorderの値をinorderで探して、その場所から左を左部分木、右を右部分木にする解き方。
- 時間計算量がO(n^2)だが、考え方はわかりやすいかも
- https://github.com/Ryotaro25/leetcode_first60/pull/31/files　105.ConstructBinaryTreefromPreorderandInorderTraversal/hash_map_step2.cpp
- step1の解き方と同じだが、(※)のリンクを見た後に読んだら結構理解できた。
- step1のコードを書いたときは、[left, right]の意味がよく分かっていなかった。
- やっていることは、preorderを先頭から見ていき、該当するinorderの場所を見つけて、その場所から左を左部分木、右を右部分木にするだけ
- mapを作るのは、inorder_indexをいちいち.find()で探していたら効率が悪いため
- step1では閉区間でやっていたが、リンクの人は[left, right)でやっていた。nullptrを返すときのif文が軽くなるのかな？
  
- https://github.com/kazukiii/leetcode/pull/30#discussion_r1821506570
- preorderの頭からツリーを作る。
- 親ノードをスタックに積んでいき、スタックから親ノードを出して、左か右に繋げる。
- どちらに繋げるか判断するための範囲もスタックに積んでおく [left, right]
- 取り出した親ノードのinorder_indexより、inorder_indexが小さければ、左に繋げる。
- 右ノードの場合は、スタックから、inorder_index < rightとなる親ノードを探して、右につける。

- https://github.com/irohafternoon/LeetCode/pull/32/files step2
- preorder, inorderの双方の範囲を限定し、両方を満たすinorderの一番先頭の値をノードに入れる方法
- preorder_indexを使わないので副作用がなくせる
- https://github.com/fhiyo/leetcode/pull/31/files
- 上の人は、部分木のサイズを指定していた。
- preorder_end, inorder_endがいらなくなる。
- 左部分木のサイズはinorderの位置からinorderの開始位置を引いた値
- 右部分木のサイズは木のサイズから自分と左部分木のサイズを引いた値
- 木のサイズが0ならnullptrを返す

- 上記のリンクを参考に書いたもの
- preorder, inorder双方の範囲を指定するよりも、木のサイズを渡す方がわかりやすいと感じた。
- このコードが一番、しっくり来た
```cpp
class Solution {
public:
    TreeNode* buildTree(const vector<int>& preorder, const vector<int>& inorder) {
        map<int, int> inorder_value_to_index;
        for (int i = 0; i < inorder.size(); ++i) {
            inorder_value_to_index[inorder[i]] = i;
        }
        return BuildTreeHelper(preorder, inorder_value_to_index, 0, 0, preorder.size());
    }
private:
    TreeNode* BuildTreeHelper(const vector<int>& preorder,
                              const map<int, int>& inorder_value_to_index,
                              int preorder_start, int inorder_start, int size) {
        if (!size) {
            return nullptr;
        }
        int value = preorder[preorder_start];
        TreeNode* node = new TreeNode(value);
        int inorder_index = inorder_value_to_index.at(value);
        int left_tree_size = inorder_index - inorder_start;
        int right_tree_size = size - left_tree_size - 1;
        node->left = BuildTreeHelper(preorder, inorder_value_to_index, preorder_start + 1, inorder_start, left_tree_size);
        node->right = BuildTreeHelper(preorder, inorder_value_to_index, preorder_start + left_tree_size + 1, inorder_index + 1, right_tree_size);
        return node;
    }
};
```

## Step3

- preorder, inorderの開始地点と部分木のサイズを渡す解き方がしっくりきたので3回

```cpp
class Solution {
public:
    TreeNode* buildTree(const vector<int>& preorder, const vector<int>& inorder) {
        map<int, int> inorder_value_to_index;
        for (int i = 0; i < inorder.size(); ++i) {
            inorder_value_to_index[inorder[i]] = i;
        }
        return BuildTreeHelper(preorder, inorder_value_to_index, 0, 0, preorder.size());
    }
private:
    TreeNode* BuildTreeHelper(const vector<int>& preorder,
                              const map<int, int>& inorder_value_to_index,
                              int preorder_start, int inorder_start, int size) {
        if (!size) {
            return nullptr;
        }
        int value = preorder[preorder_start];
        int inorder_index = inorder_value_to_index.at(value);
        int left_tree_size = inorder_index - inorder_start;
        int right_tree_size = size - left_tree_size - 1;
        TreeNode* node = new TreeNode(value);
        ++preorder_start;
        node->left = BuildTreeHelper(preorder, inorder_value_to_index,
                                     preorder_start, inorder_start, left_tree_size);
        node->right = BuildTreeHelper(preorder, inorder_value_to_index,
                                      preorder_start + left_tree_size, inorder_index + 1, right_tree_size);
        return node;
    }
};
```

- 感想
- 結構難しかったので、理解を確認するために再度解きたい
- 余裕があったら、他の解き方も試したい
