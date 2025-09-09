# Top K Frequent Elements
* 問題: https://leetcode.com/problems/top-k-frequent-elements/
* 言語: C++

## Step1

- まず、mapを使う方法を思いついた
  - mapで出現頻度をカウントしていくが、出現頻度順でソートする必要がある？
  - mapに入れ終えたら、キーと値を交換したものを別のmapに入れることで出現頻度順にソートしてくれそう
  - 最後に、mapをvectorにして返す
- 時間計算量はO(n * log n)、空間計算量はO(n)
- 出現頻度が同じものを考慮していなかった。重複を許可するmultimapを使用。(https://cpprefjp.github.io/reference/map/multimap.html)
  - leetcodeのジャッジシステムでレバガチャしてしまった。(https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ofzx0kw1dsrd)
    - 範囲for文とイテレータでのアクセスの違いが分かっていなかった。
      - 範囲for文(参照不使用）では、変数にコンテナの要素をコピーするので it.first でアクセスできるが、イテレータは要素へのポインタのようなものなので、(*it).first, it->firstでアクセスする。

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::unordered_map<int, int> count_frequency;
        for (int num : nums) {
            count_frequency[num]++;
        }

        std::multimap<int, int, std::greater<int>> sort_frequency;
        for (auto it : count_frequency) {
            sort_frequency.emplace(it.second, it.first);
        }

        std::vector<int> top_k_frequent_elements;
        auto iter = sort_frequency.begin();
        for (int i = 0; i < k; i++) {
            top_k_frequent_elements.push_back(iter->second);
            iter++;
        }
        return top_k_frequent_elements;
    }
};
```

## Step2

他の人の解答を見ると、mapでカウントしたものをvectorに入れて、sortするものや、priority_queueに入れるものがあった。
クイックセレクトは初耳だったが、常識範囲内らしい。

- https://github.com/Ryotaro25/leetcode_first60/pull/10/files
- https://discord.com/channels/1084280443945353267/1183683738635346001/1185972070165782688

辞書の変数名はx_to_yがよいらしい
- https://github.com/irohafternoon/LeetCode/pull/11/files#r2024946024

ソートしないからunordered_mapを使ったが、デフォルトで使わない方がよいらしい
- https://github.com/Ryotaro25/leetcode_first60/pull/10/files#r1623243488

カウントしたものをvectorに入れてsortする解き方
```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::map<int, int> num_to_frequency;
        for (int num : nums) {
            num_to_frequency[num]++;
        }

        std::vector<std::pair<int, int>> frequency_to_num;
        for (auto [num, frequency] : num_to_frequency) {
            frequency_to_num.push_back({frequency, num});
        }
        std::sort(frequency_to_num.begin(), frequency_to_num.end(), std::greater<std::pair<int, int>>());

        std::vector<int> k_frequent_nums;
        for (int i = 0; i < k; i++) {
            k_frequent_nums.push_back(frequency_to_num[i].second);
        }

        return k_frequent_nums;
    }
};
```

カウントしたものをpriority_queueに入れる解き方

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::map<int, int> num_to_frequency;
        for (int num : nums) {
            num_to_frequency[num]++;
        }

        std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, std::greater<std::pair<int, int>>> frequency_and_nums;
        for (auto [num, frequency] : num_to_frequency) {
            frequency_and_nums.push({frequency, num});
            if (frequency_and_nums.size() > k) {
                frequency_and_nums.pop();
            }
        }

        std::vector<int> top_k_frequent_nums;
        for (int i = 0; i < k; i++) {
            top_k_frequent_nums.push_back(frequency_and_nums.top().second);
            frequency_and_nums.pop();
        }
        return top_k_frequent_nums;
    }
};
```

- pair型は中の値がわかりずらいから独自の構造体を作った方がいい　https://github.com/maeken4/Arai60/pull/9/files#r2161184486
- 自分も書いていて、わかりずらいなと感じたのと、書くのが少し大変だった
- vector, priority_queue, multimapのうちだったら、pair型を使わずコンパクトに書けるmultimapがよいなと感じた


- クイックセレクトを使った解き方
- 下記のコードを見ながら、実装
- https://github.com/Ryotaro25/leetcode_first60/pull/10/files#diff-77ec9897daa7ec6a8f8802b39d01e34e00d44a8d722e0b7d51776b246b964364
- コードを理解するのも、書くのも大変だった。何も見ずに書ける自信はない。

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::vector<std::pair<int, int>> nums_and_frequency = CountFrequency(nums);
        QuickSelect(nums_and_frequency, 0, nums_and_frequency.size() - 1, nums_and_frequency.size() - k);

        std::vector<int> top_k_nums;
        for (int i = nums_and_frequency.size() - k; i < nums_and_frequency.size(); i++) {
            top_k_nums.push_back(nums_and_frequency[i].first);
        }
        return top_k_nums;
    }

private:
    std::vector<std::pair<int, int>> CountFrequency(std::vector<int>& nums) {
        std::map<int, int> num_to_frequency;
        for (int num : nums) {
            num_to_frequency[num]++;
        }
        std::vector<std::pair<int, int>> nums_and_frequency(num_to_frequency.begin(), num_to_frequency.end());
        return nums_and_frequency;
    }

    void QuickSelect(std::vector<std::pair<int, int>>& nums_and_frequency, int left, int right, int kth) {
        while (left < right) {
            int pivot = left + std::rand() % (right - left + 1);
            pivot = Partition(nums_and_frequency, left, right, pivot);

            if (kth == pivot) {
                return;
            } else if (kth < pivot) {
                right = pivot - 1;
            } else {
                left = pivot + 1;
            }
        }
    }

    int Partition(std::vector<std::pair<int, int>>& nums_and_frequency, int left, int right, int pivot) {
        int pivot_frequency = nums_and_frequency[pivot].second;
        std::swap(nums_and_frequency[pivot], nums_and_frequency[right]);
        int sorted_index = left;
        for (int i = left; i < right; i++) {
            if (nums_and_frequency[i].second < pivot_frequency) {
                std::swap(nums_and_frequency[sorted_index], nums_and_frequency[i]);
                sorted_index++;
            }
        }
        std::swap(nums_and_frequency[right], nums_and_frequency[sorted_index]);
        return sorted_index;
    }
};
```

## Step3
```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        std::map<int, int> num_to_frequency;
        for (int num : nums) {
            num_to_frequency[num]++;
        }
        std::multimap<int, int, std::greater<int>> frequency_to_num;
        for (auto [num, frequency] : num_to_frequency) {
            frequency_to_num.emplace(frequency, num);
        }
        std::vector<int> top_k_nums;
        auto iter = frequency_to_num.begin();
        for (int i = 0; i < k; i++) {
            top_k_nums.push_back(iter->second);
            iter++;
        }
        return top_k_nums;
    }
};
```
