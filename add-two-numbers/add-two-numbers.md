# Add Two Numbers
* 問題: https://leetcode.com/problems/add-two-numbers/
* 言語: C++

## Step1
### 所要時間：50分

まず、①2つのリストを2つの数字に変換してから合計値を求め、合計値をリストにする方法と、②2つのノードの値を桁ごとに計算していく方法を思いついた。
①の方法を試したが、unsigned long long型の範囲を超える場合を考慮してなかったので、桁数が大きい時にエラーになった。


```cpp
 //#include <cmath>

class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        unsigned long long sum = createNumberFromList(l1) + createNumberFromList(l2);
        ListNode* head = new ListNode;
        ListNode* node = head;
        unsigned digit = 0;

        while (true) {
            unsigned long long num = sum / std::pow(10, digit) ;
            node->val = num % 10;

            if (num / 10 == 0) break;

            ListNode* next_node = new ListNode;
            node->next = next_node;
            node = node->next;
            digit++;
        }
        
        return head;

    }

private:
    unsigned long long createNumberFromList(ListNode* head) {
        ListNode* node = head;
        unsigned digit = 0;
        unsigned long long sum = 0;
        while (node) {
            sum += node->val * std::pow(10, digit);
            node = node->next;
            digit++;
        }
        return sum;
    }
};
```

②の方法はあまり時間がかからなかった。こちらの考えの方が自然かなと感じた。
変数名```x```、```y```はもっと良い名前がありそう。

- 時間計算量：O(n)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode head;
        ListNode* node = &head;
        int carry = 0;

        while (l1 || l2 || carry) {
            int x = (l1 ? l1->val : 0);
            int y = (l2 ? l2->val : 0);
            int sum = x + y + carry;

            node->next = new ListNode(sum % 10);
            carry = sum / 10;
            node = node->next;

            if (l1) l1 = l1->next;
            if (l2) l2 = l2->next;
        }
        return head.next;
    }
};
```

## Step2
### 参考
- https://github.com/fhiyo/leetcode/pull/5/files
- https://github.com/kazukiii/leetcode/pull/6/files

再帰バージョン
```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        return addTwoNumbersHelper(l1, l2, 0);
    }

private:
    ListNode* addTwoNumbersHelper(ListNode* l1, ListNode* l2, int carry) {
        if (!l1 && !l2 && !carry) return nullptr;
        
        int sum = carry;
        if (l1) {
            sum += l1->val;
            l1 = l1->next;
        }
        if (l2) {
            sum += l2->val;
            l2 = l2->next;
        }
        ListNode* node = new ListNode(sum % 10);
        node->next = addTwoNumbersHelper(l1, l2, sum / 10);
        return node;
    }
};
```

## Step3
```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy_head(-1);
        ListNode* node = &dummy_head;
        int carry = 0;

        while (l1 || l2 || carry) {
            int sum = carry;
            if (l1) {
                sum += l1->val;
                l1 = l1->next;
            }
            if (l2) {
                sum += l2->val;
                l2 = l2->next;
            }
            carry = sum / 10;
            node->next = new ListNode(sum % 10);
            node = node->next;
        }
        return dummy_head.next;
    }
};
```
