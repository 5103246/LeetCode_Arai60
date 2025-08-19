# Remove Duplicates from Sorted List II
* 問題: https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/description/
* 言語: C++

## Step1
所要時間：15分
隣接ノードが重複していたら、2個先のノードを見る。値が異なっていたら前のノードと2個先のノードをくっつける。
再帰を用いた方法やmapを使った方法など考えてみたが、つまったので他の人の解答を見た。
一応書いてみたが、time limit exceededになった。
```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode dummy(0, head);
        ListNode* current_node = head;
        ListNode* previous_node = &dummy;
        while (current_node) {
            if (current_node->val == current_node->next->val) {
                if (current_node->val != current_node->next->next->val) {
                    previous_node->next = current_node->next->next;
                } else {
                    current_node->next = current_node->next->next;
                }
            } else {
                current_node = current_node->next;
                previous_node = previous_node->next;
            }
        }
        return dummy.next;
    }
};
```
他の人のコードを参考にしたもの
https://github.com/fhiyo/leetcode/pull/4/files

- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode sentinel(0, head);
        ListNode* previous = &sentinel;
        ListNode* current = head;

        while (current && current->next) {
            if (current->val == current->next->val) {
                while (current->next && current->val == current->next->val) {
                    current = current->next;
                }
                current = current->next;
                previous->next = current;
            } else {
                previous = previous->next;
                current = current->next;
            }
        }
        return sentinel.next;
    }
};
```

## Step2
最初に書いたコードだと、入力が[1,1]の場合などに対応できてなかった。2個先のノードを見るよりもループで回す方がやりやすいと感じた。

### 参考
- https://github.com/fhiyo/leetcode/pull/4/files
- https://github.com/Ryotaro25/leetcode_first60/blob/main/RemoveDuplicatesfromSortedListII/step5.cpp

mapを使った方法も書いてみた。

- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        std::map<int, int> count;
        for (ListNode* node = head; node; node = node->next) {
            count[node->val] += 1;
        }

        ListNode sentinel;
        ListNode* tail = &sentinel;
        for (ListNode* node = head; node; node = node->next) {
            if (count[node->val] == 1) {
                tail->next = node;
                tail = tail->next;
            }
        }
        tail->next = nullptr;
        return sentinel.next;
    }
};
```

## Step3
```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode sentinel(0, head);
        ListNode* previous = &sentinel;
        ListNode* current = head;

        while (current && current->next) {
            if (current->val == current->next->val) {
                while (current->next && current->val == current->next->val) {
                    current = current->next;
                }
                current = current->next;
                previous->next = current;
            } else {
                previous = previous->next;
                current = current->next;
            }
        }
        return sentinel.next;
    }
};
```
