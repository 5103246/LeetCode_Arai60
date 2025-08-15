# Linked List Cycle
* 問題: https://leetcode.com/problems/linked-list-cycle-ii/
* 言語: C++

## Step1
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

## Step2

### 参考

- https://github.com/fhiyo/leetcode/pull/2/files
- https://discord.com/channels/1084280443945353267/1183683738635346001/1185264362508795984
- https://github.com/Ryotaro25/leetcode_first60/pull/2/files

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
