# Remove Duplicates from Sorted List
* 問題: https://leetcode.com/problems/remove-duplicates-from-sorted-list/
* 言語: C++

## Step1
setで重複かを判断し、重複していたら次のノードと連結させる方法を思いついたので、やってみたがWrong Answerだった。
連結リストのつなぎかえがわからなくて、悩んだ。
- 時間計算量：O(n log n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        std::set<ListNode*> visited;
        ListNode* curr = head;
        ListNode dummy;
        ListNode* node = &dummy;

        while (curr) {
            if (visited.contains(curr)) {
                curr = curr->next;
                continue;
            }
            visited.insert(curr);
            node->next = curr;
            curr = curr->next;
            node = node->next;
        }
        node->next = nullptr;
        
        return dummy.next;
    }
};
```
chat gptに聞いてみたところ、visitedに入れているのがノードのポインタだったのが原因だった。

直したのが下記のコード
```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        std::set<int> visited;
        ListNode* curr = head;
        ListNode dummy;
        ListNode* node = &dummy;

        while (curr) {
            if (visited.contains(curr->val)) {
                curr = curr->next;
                continue;
            }
            visited.insert(curr->val);
            node->next = curr;
            curr = curr->next;
            node = node->next;
        }
        node->next = nullptr;

        return dummy.next;
    }
};
```

## Step2
### 参考
- https://github.com/fhiyo/leetcode/pull/3/files
- https://github.com/ryosuketc/leetcode_arai60/pull/3/files

他の人のコードを見ると、隣接ノードを比較していく方法、再帰を用いた方法などがあった。
最初に思いついたアイデアに固執しがちなので、選択肢をできるだけ広げてから解くように意識したい。
https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.3jonchja100w

自分はこの方法が一番、シンプルで良いと感じた。
こちらは、後ろにある重複するノードをスキップしていく。
- 時間計算量：O(n)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* node = head;
        while (node && node->next) {
            if (node->val == node->next->val) {
                node->next = node->next->next;
            } else {
                node = node->next;
            }
        }
        return head;
    }
};
```

再帰の方法は、思いついても実装するのが大変そう。
再帰は、手前にある重複したノードをスキップしていく。
- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return head;
        if (head->val == head->next->val) {
            head = deleteDuplicates(head->next);
        } else {
            head->next = deleteDuplicates(head->next);
        }
        return head;
    }
};
```

## Step3
```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* node = head;
        while (node && node->next) {
            if (node->val == node->next->val) {
                node->next = node->next->next;
            } else {
                node = node->next;
            }
        }
        return head;
    }
};
```
