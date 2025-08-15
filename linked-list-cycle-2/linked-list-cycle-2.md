# Linked List Cycle II
* 問題: https://leetcode.com/problems/linked-list-cycle-ii/
* 言語: C++

## Step1
所要時間： 5分程度

前回の問題（Linked List Cycle) と同じように、まずはsetを使った方法を思いついた。
フロイドの方法も思いついたが、setを使った方が読み書きがしやすいと判断し、setで記述した。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* node = head;
        std::set<ListNode*> visited;
        while (node) {
            if (visited.contains(node)) return node;
            visited.insert(node);
            node = node->next;
        }
        return node;
    }
};
```

## Step2

### 参考

- https://github.com/fhiyo/leetcode/pull/2/files
- https://discord.com/channels/1084280443945353267/1183683738635346001/1185264362508795984
- https://github.com/Ryotaro25/leetcode_first60/pull/2/files

nullptrにした方が、意図が伝わりやすいと感じた。

## Step3
```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* node = head;
        std::set<ListNode*> visited;
        while (node) {
            if (visited.contains(node)) return node;
            visited.insert(node);
            node = node->next;
        }
        return nullptr;
    }
};
```
