# Convert Sorted Array to Binary Search Tree
* 問題: https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/
* 言語: C++

## Step1


- 平衡二分探索木なので赤黒木やAVL木をイメージした。
- 最初は、先頭から順にノードにして大小比較して繋げる解き方が思いついた。
- 根ノードがリストの真ん中の値になることに気づいて、それを繰り返す方法を考えた。
- クイックソートみたいに、リストの真ん中の値をノードにして、左と右をつなげるときに再帰をする。
- n : nums.size
- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return CreateNode(nums, 0, nums.size() - 1);
    }
private:
    TreeNode* CreateNode(vector<int>& nums, int first, int last) {
        if (!(0 <= first && first <= last)) {
            return nullptr;
        }
        int i = (first + last) / 2;
        TreeNode* node = new TreeNode(nums[i]);
        node->left = CreateNode(nums, first, i - 1);
        node->right = CreateNode(nums, i + 1, last);
        return node;
    }
};
```

- 再帰でできるならループでも書けそう。
- 終了条件はfirst > lastのみでよかった。

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return CreateNode(nums, 0, nums.size() - 1);
    }
private:
    TreeNode* CreateNode(vector<int>& nums, int first, int last) {
        if (first > last) {
            return nullptr;
        }
        int i = (first + last) / 2;
        TreeNode* node = new TreeNode(nums[i]);
        node->left = CreateNode(nums, first, i - 1);
        node->right = CreateNode(nums, i + 1, last);
        return node;
    }
};
```

## Step2

- https://github.com/olsen-blue/Arai60/pull/24/files
- コードが短いから一文字変数でもいいかなと思ったが、ちゃんと変数名付けといた方がよさそう。
- インデックスはleft, middle, rightがよさそう。TreeNodeのと被るので left_index, right_indexがいいかも。
- nodeじゃなくてrootの方がしっくり来ている人が多かった。
- 根ノードは親ノードを持たないノードのことなので、この場合はparentがいいかも？
- 自分は単純にnodeでいいと思う。
- https://github.com/irohafternoon/LeetCode/pull/27/files#r2060367845
- 右半開区間だと、最初に渡すときと、左につなげるときに渡すrightインデックスを - 1 しなくていいのか。終了条件は left >= rightになる。
- https://github.com/irohafternoon/LeetCode/pull/27/files#r2061263134
- ダブルポインタを使えば、キュートループで書けるのか。nullptrを入れてから、popしたあとにnewすればメモリリークの心配がない。
- リンク先のコードは、TreeNode* root = new TreeNode() をしているのでメモリリークしている。TreeNode* root = nullptr が良いと感じた。

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(const vector<int>& nums) {
        queue<tuple<TreeNode**, int, int>> nodes_and_ranges;
        TreeNode* root = nullptr;
        nodes_and_ranges.emplace(&root, 0, nums.size());
        while (!nodes_and_ranges.empty()) {
            auto [node_pointer, left_index, right_index] = nodes_and_ranges.front();
            nodes_and_ranges.pop();
            if (left_index >= right_index) {
                continue;
            }
            int middle_index = (left_index + right_index) / 2;
            *node_pointer = new TreeNode(nums[middle_index]);
            TreeNode* node = *node_pointer;
            nodes_and_ranges.emplace(&(node->left), left_index, middle_index);
            nodes_and_ranges.emplace(&(node->right), middle_index + 1, right_index);
        }
        return root;
    }
};
```
- https://github.com/irohafternoon/LeetCode/pull/27/files#r2061259662
- TreeNode* で実装すると、適当にnewして、popしたあとにnode->valを変えるのか。
- 左インデックスがmiddle以下なら左ノードを作成してキューに入れる。右インデックスがmiddleより大きければ右ノードを作成する。
- こちらなら場合分けがあるが、メモリリークの心配がないのがよい。
- 再帰版の関数名だが、ContructBSTは結構良いなと感じた。自分も関数名はこちらにしようと思う。
  
- https://github.com/colorbox/leetcode/pull/38/files#r1939182919
- const付けるの忘れていた。
  

## Step3

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(const vector<int>& nums) {
        return ContructBST(nums, 0, nums.size());
    }
private:
    TreeNode* ContructBST(const vector<int>& nums, int left_index, int right_index) {
        if (left_index >= right_index) {
            return nullptr;
        }
        int middle_index = (left_index + right_index) / 2;
        TreeNode* node = new TreeNode(nums[middle_index]);
        node->left = ContructBST(nums, left_index, middle_index);
        node->right = ContructBST(nums, middle_index + 1, right_index);
        return node;
    }
};
```
