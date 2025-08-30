# Reverse Linked List
* 問題: https://leetcode.com/problems/reverse-linked-list/
* 言語: C++

## Step1
### 所要時間：約15分

- 入力を変更していいのか悩んだ。変更したら呼び出し側が困りそうなので変更せず、新しいリストを生成する方法で解いた。
- 入力リストの値をスタックに積んでいく。積み終わったら、トップ要素から新しいノードを作成し、繋げる。最後にpopする。

- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
#include <stack>

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* forward_node = head;
        ListNode dummy(0);
        ListNode* reverse_node = &dummy;
        std::stack<int> value_stack;

        while (forward_node) {
            value_stack.emplace(forward_node->val);
            forward_node = forward_node->next;
        }
        while (!value_stack.empty()) {
            reverse_node->next = new ListNode(value_stack.top());
            reverse_node = reverse_node->next;
            value_stack.pop();
        }
        return dummy.next;
    }
};
```

入力リストを変更するコード。

```cpp
#include <stack>

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode dummy(0);
        ListNode* node = &dummy;
        std::stack<ListNode*> node_stack;

        while (head) {
            node_stack.emplace(head);
            head = head->next;
        }
        while (!node_stack.empty()) {
            node->next = node_stack.top();
            node = node->next;
            node_stack.pop();
        }
        node->next = nullptr;
        return dummy.next;
    }
};
```

## Step2

- 繋ぎ変える方法
  - 繋ぎ変えの考え方はいくつかあったが、自分は新しいリストの先頭に繋げる考え方がしっくりきた。
  - https://github.com/fhiyo/leetcode/pull/7/files
  - https://github.com/irohafternoon/LeetCode/pull/9#discussion_r2020110286
- 時間計算量：O(n)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* reverse_list_head = nullptr;
        ListNode* node = head;
        while (node) {
            ListNode* next_node = node->next;
            node->next = reverse_list_head;
            reverse_list_head = node;
            node = next_node;
        }
        return reverse_list_head;
    }
};
```
- ヒープにメモリを確保すると読み手の負荷が高くなるのを考えずにコーディングしていた。
  - https://github.com/Ryotaro25/leetcode_first60/pull/8/files#r1617095525

- 再帰を使った解き方
  - リンクの向きを逆向きにして、逆向きにしたリストの先頭を返す
  - ```node->next->next = node```は最初わからなかったが、図にしたら理解できた。```node->next = nullptr```と合わせるとリンクを逆向きにできる。
  - https://github.com/fhiyo/leetcode/pull/7/files
- 時間計算量：O(n)
- 空間計算量：O(n)
　 

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* node) {
        if (!node || !node->next) return node;
        ListNode* reverse_head = reverseList(node->next);
        node->next->next = node;
        node->next = nullptr;
        return reverse_head;
    }
};
```

##Step3

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* reverse_list_head = nullptr;
        ListNode* node = head;
        while (node) {
            ListNode* next_node = node->next;
            node->next = reverse_list_head;
            reverse_list_head = node;
            node = next_node;
        }
        return reverse_list_head;
    }
};
```
