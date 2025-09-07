# Kth Largest Element in a Stream
* 問題: https://leetcode.com/problems/kth-largest-element-in-a-stream/
* 言語: C++

## Step1

- 優先度付きキューを使うのはすぐ思いついた（正直に言うと、問題のカテゴリーがheap, priority queueだったのを知っていたから思いついた感がある）
- k番目にアクセスするのを思いつくのに時間がかかった
- 昇順にして、k番目までpopすることで、k番より前の値をpopせずにアクセスできる
- 参考：https://cpprefjp.github.io/reference/queue/priority_queue.html
  
```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) 
        : kth(k), scores()
    {
        for (int num : nums) {
            scores.push(num);
        }  
    }
    
    int add(int val) {
        scores.push(val);
        for (std::size_t i = scores.size(); i > kth; i--) {
            scores.pop();
        }
        return scores.top();
    }

private:
    int kth;
    std::priority_queue<int, std::vector<int>, std::greater<int>> scores;
};
```

## Step2

### 参考
- https://github.com/fhiyo/leetcode/pull/10/files
- https://google.github.io/styleguide/cppguide.html#Variable_Names
- 他の人はpriority_queueのサイズがkを超えたときにpopすることで常にk番目がtopにあるようにしていた
- step1の解き方だと、空間計算量がO(nums.length)だが、下記のコードだとO(k)になる。
- kがnums.lengthより十分小さい場合などを考えるとこちらの解き方が良いと感じた。
- 他の人のコードでは、メンバ変数の変数名の末尾に_が付いていたのが疑問だったが、google c++ style guideで、メンバ変数は末尾に_を付けることが推奨されていた。


```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : kth_(k), kth_largest_scores_() {
        for (int num : nums) {
            add(num);
        }
    }
    
    int add(int val) {
        kth_largest_scores_.push(val);
        if (kth_largest_scores_.size() > kth_) {
            kth_largest_scores_.pop();
        }
        return kth_largest_scores_.top();
    }

private:
    int kth_;
    std::priority_queue<int, std::vector<int>, std::greater<int>> kth_largest_scores_;
};
```

- mapを使って解いている人もいたので、自分も書いてみた。
- 参考：https://github.com/Ryotaro25/leetcode_first60/pull/9/files#diff-3f03b62c631983c6e1f688fff584b4ef4a0e0c77ef08b3a8984ebadecc5617d6
- mapを使った方法だと、時間計算量は O(logn + k) (nは異なる値の種類数）で、priority_queueは O(logk)
- leetcode上のRuntimeだが、mapは約2000ms, priority_queueだと約10msで、200倍ほど差があった。
- このくらい速さが違うと、今回の問題はpriority_queueを使った方法が良いと思う。

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : kth_(k), kth_largest_values_() {
        for (int num : nums) {
            add(num);
        }
    }
    
    int add(int val) {
        kth_largest_values_[val]++;

        int count = 0;
        for (auto it : kth_largest_values_) {
            count += it.second;
            if (count >= kth_) {
                return it.first;
            }
        }
        return -1;
    }

private:
    int kth_;
    std::map<int, int, std::greater<int>> kth_largest_values_;
};
```

- heapを実装している人が結構いたので、自分もStep4で実装してみようと思う。

## Step3

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : kth_(k), kth_largest_scores_() {
        for (int num : nums) {
            add(num);
        }
    }
    
    int add(int val) {
        kth_largest_scores_.push(val);
        if (kth_largest_scores_.size() > kth_) {
            kth_largest_scores_.pop();
        }
        return kth_largest_scores_.top();
    }

private:
    int kth_;
    std::priority_queue<int, std::vector<int>, std::greater<int>> kth_largest_scores_;
};
```

## Step4-map

- mapを使った解き方を改良した。
- priority_queueを使った解き方と同じように、常にk番目の値が先頭にあるようにした。
  - 昇順のmapに対して、スコアをkeyに、同じスコアの個数をvalueに入れていく。
  - mapのvalueをカウントしていき、kを超えたら先頭のvalueを‐1する。
  - 先頭のvalueが0になったら、削除する
  - 最後に先頭を返す
- nは入力長
- 時間計算量：O(n * log k)
- 空間計算量：O(k)

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : count_(0), kth_(k), kth_largest_scores_() {
        for (int num : nums) {
            add(num);
        }
    }
    
    int add(int val) {
        kth_largest_scores_[val]++;
        count_++;
        if (count_ > kth_) {
            auto it = kth_largest_scores_.begin();
            it->second--;
            count_--;
            if (it->second == 0) {
                kth_largest_scores_.erase(it);
            }
        }
        return kth_largest_scores_.begin()->first;
    }

private:
    int count_;
    int kth_;
    std::map<int, int> kth_largest_scores_;
};
```

- multisetを使った方法
- multisetは重複を許可するset（https://cpprefjp.github.io/reference/set/multiset.html）
- chat gptに提案されるまで知らなかったが、multisetを使ったコードの方がmapのコードよりシンプルになっている。
- 初見で解くときは、mapで解く方法が思いつきやすそう。
- 時間計算量：O(n * log k)
- 空間計算量：O(k)

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : kth_(k), kth_largest_scores_() {
        for (int num : nums) {
            add(num);
        }
    }
    
    int add(int val) {
        kth_largest_scores_.insert(val);
        if (kth_largest_scores_.size() > kth_) {
            kth_largest_scores_.erase(kth_largest_scores_.begin());
        }
        return *kth_largest_scores_.begin();
    }

private:
    int kth_;
    std::multiset<int> kth_largest_scores_;
};
```
