# Linked List Cycle
* 問題: https://leetcode.com/problems/linked-list-cycle/
* 言語: C++

## Step1
```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        std::set<ListNode *> visited;
        ListNode *node = head;

        while (node) {
            if (visited.contains(node)) {
                return true;
            }
            visited.insert(node);
            node = node->next;
        }

        return false;
    }
};
```

## Step2

### 参考

- https://github.com/t-ooka/leetcode/pull/8/files#diff-0b928fca29c63aad6c26727fa91f2630562e624eb20fe64f21aa08a8d5c034b4
- https://github.com/fhiyo/leetcode/pull/1/files
  

フロイドの循環検出法というのがあるので、書いてみた
```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode* fast = head;
        ListNode* slow = head;

        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;

            if (fast == slow) return true;
        }
        return false;
    }
};
```

## Step3

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode* node = head;
        std::set<ListNode*> visited;
        while (node) {
            if (visited.contains(node)) return true;
            visited.insert(node);
            node = node->next;
        }
        return false;
    }
};
```
