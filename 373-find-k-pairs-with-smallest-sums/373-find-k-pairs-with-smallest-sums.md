# Find K Pairs with Smallest Sums
* 問題: https://leetcode.com/problems/find-k-pairs-with-smallest-sums/
* 言語: C++

## Step1

- まず、347の問題と同じ解き方を思いついた。
- 二重ループで、mapのkeyに数の組を、valueにその合計値を入れていく。
- 次に、multimapやpriority_queue、sortを使って、合計値が小さい順に並べ替える。
- 最後に、k個のリストをリストの中に入れる。
- この解き方だと、時間計算量がO(n^2 + n log n)、空間計算量はO(n^2)くらい
- pairを使わずに済むのでmultimapがよさそうかも
- priority_queueの場合は、降順に並べて、サイズがkを超えたらpopしていくと、メモリが節約できる。multimapも同様にできそう。
- priority_queueを使って解いてみる
- 数の組の重複を許すのでmapではなく、multimap
- 以下のコードでは、num.length = 10000, k = 10000のケースで時間制限オーバーだった

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        std::multimap<std::vector<int>, int> pair_to_sum;
        for (int num1 : nums1) {
            for (int num2 : nums2) {
                std::vector<int> pair{num1, num2};
                pair_to_sum.emplace(pair, num1 + num2);
            }
        }

        std::priority_queue<std::pair<int, std::vector<int>>> sum_and_pairs;
        for (auto [pair, sum] : pair_to_sum) {
            sum_and_pairs.push({sum, pair});
            if (sum_and_pairs.size() > k) {
                sum_and_pairs.pop();
            }
        }

        std::vector<std::vector<int>> k_smallest_pairs;
        while (!sum_and_pairs.empty()) {
            k_smallest_pairs.push_back(sum_and_pairs.top().second);
            sum_and_pairs.pop();
        }
        return k_smallest_pairs;
    }
};
```

- 考えたら、今回の問題ではmapを使う必要はなく、そのままpriority_queueに入れてよかった。
- 空間計算量はO(k)になったが、Time Limit Exceedのままだった

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        std::priority_queue<std::pair<int, std::vector<int>>> sum_and_pairs;
        for (int num1 : nums1) {
            for (int num2 : nums2) {
                std::vector<int> pair{num1, num2};
                std::pair<int, std::vector<int>> sum_to_pair{num1 + num2, pair};
                sum_and_pairs.push(sum_to_pair);
                if (sum_and_pairs.size() > k) {
                    sum_and_pairs.pop();
                }
            }
        }

        std::vector<std::vector<int>> k_smallest_pairs;
        while (!sum_and_pairs.empty()) {
            k_smallest_pairs.push_back(sum_and_pairs.top().second);
            sum_and_pairs.pop();
        }
        return k_smallest_pairs;
    }
};
```

- 他の考えが思いつかなかったので、他の人の解答を見る
- https://github.com/fhiyo/leetcode/pull/13/files
- 最初に(0,0)を入れて、popしたものに対して、(i, 0)なら(i + 1, 0)を入れて、次に(i, j + 1)を入れる
- priority_queue(昇順)から先頭をpopして、上の処理をk回行う

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        priority_queue<
            pair<int, pair<int, int>>, 
            vector<pair<int, pair<int, int>>>, 
            greater<pair<int, pair<int, int>>>
        > sum_and_pairs;
        sum_and_pairs.push({nums1[0] + nums2[0], {0, 0}});
        vector<vector<int>> k_smallest_pairs;

        while (k_smallest_pairs.size() < k) {
            auto [i, j] = sum_and_pairs.top().second;
            sum_and_pairs.pop();
            k_smallest_pairs.push_back({nums1[i], nums2[j]});
            if (i < nums1.size() - 1 && j == 0) {
                sum_and_pairs.push({nums1[i + 1] + nums2[j], {i + 1, j}});
            }
            if (j < nums2.size() - 1) {
                sum_and_pairs.push({nums1[i] + nums2[j + 1], {i, j + 1}});
            }
        }
        return k_smallest_pairs;
    }
};
```

## Step2

- priority_queueのcompareが勉強になった。比較するときに合計値を比較するようにすれば、priority_queueに入れるのはインデックスの組だけになる。
- https://github.com/TORUS0818/leetcode/pull/12/files#r1623354548
- indexの組が重複しないように管理するsetが必要だった
- https://github.com/irohafternoon/LeetCode/pull/12/files
- https://github.com/t0hsumi/leetcode/pull/10/files#r1882223848
- 上記のことを参考に改善してみた。
- Step1では、j == 0のとき(i + 1, j)をpushすることで重複を防いでいたが、あまり自然な発想ではないかなと感じた。
- それよりも、setで管理する方が自然かなと思った

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        auto compare = [&](pair<int, int> a, pair<int, int> b) {
            return nums1[a.first] + nums2[a.second] > nums1[b.first] + nums2[b.second];
        };
        priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(compare)> index_pairs(compare);
        index_pairs.push({0, 0});
        set<pair<int, int>> processed;
        vector<vector<int>> k_smallest_pairs;

        while (k_smallest_pairs.size() < k) {
            auto [i, j] = index_pairs.top();
            index_pairs.pop();
            k_smallest_pairs.push_back({nums1[i], nums2[j]});
            if (i < nums1.size() - 1 && !processed.contains({i + 1, j})) {
                index_pairs.push({i + 1, j});
                processed.insert({i + 1, j});
            }
            if (j < nums2.size() - 1 && !processed.contains({i, j + 1})) {
                index_pairs.push({i, j + 1});
                processed.insert({i, j + 1});
            }
        }
        return k_smallest_pairs;
    }
};
```

- pairを使う代わりに独自の構造体を定義して解いている人がいた。
- https://github.com/Ryotaro25/leetcode_first60/pull/11/files#diff-b7081be8bbdcef2c643b1f2207ccf24562efd201b91b9eb884e933aaa30e4222
- https://github.com/kazukiii/leetcode/pull/11/files
- https://github.com/kazukiii/leetcode/pull/11/files
- 自分も構造体を定義して解いてみた
- pairを使う方法と比較すると、読みやすくなっているが、読む量も増えているので構造体が絶対良いとは思えなかった
- 今回のコードの長さくらいならpairを使ってもいいかなと感じた。
- ただ、コードが長くなっていくと、構造体を定義する方が良いと思う。

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        priority_queue<PairOfSumAndIndices> min_sum_pairs;
        set<PairOfIndices> processed;
        vector<vector<int>> k_smallest_pairs;
        min_sum_pairs.push({nums1[0] + nums2[0], 0, 0});

        while (k_smallest_pairs.size() < k) {
            auto sum_and_indices = min_sum_pairs.top();
            min_sum_pairs.pop();
            int i = sum_and_indices.index1;
            int j = sum_and_indices.index2;
            k_smallest_pairs.push_back({nums1[i], nums2[j]});

            if (i < nums1.size() - 1 && !processed.contains({i + 1, j})) {
                min_sum_pairs.push({nums1[i + 1] + nums2[j], i + 1, j});
                processed.insert({i + 1, j});
            }
            if (j < nums2.size() - 1 && !processed.contains({i, j + 1})) {
                min_sum_pairs.push({nums1[i] + nums2[j + 1], i, j + 1});
                processed.insert({i, j + 1});
            }
        }
        return k_smallest_pairs;
    }

private:
    struct PairOfSumAndIndices {
        int sum;
        int index1;
        int index2;

        bool operator<(const PairOfSumAndIndices& other) const {
            return sum > other.sum;
        }
    };

    struct PairOfIndices {
        int index1;
        int index2;

        bool operator<(const PairOfIndices& other) const {
            if (index1 != other.index1) {
                return index1 < other.index1;
            }
            return index2 < other.index2;
        }
    };
};
```

##Step3

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        auto compare = [&](pair<int, int> a, pair<int, int> b) {
            return nums1[a.first] + nums2[a.second] > nums1[b.first] + nums2[b.second];
        };
        priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(compare)> index_pairs(compare);
        index_pairs.push({0, 0});
        set<pair<int, int>> processed;
        vector<vector<int>> k_smallest_pairs;

        while (k_smallest_pairs.size() < k) {
            auto [i, j] = index_pairs.top();
            index_pairs.pop();
            k_smallest_pairs.push_back({nums1[i], nums2[j]});
            if (i < nums1.size() - 1 && !processed.contains({i + 1, j})) {
                index_pairs.push({i + 1, j});
                processed.insert({i + 1, j});
            }
            if (j < nums2.size() - 1 && !processed.contains({i, j + 1})) {
                index_pairs.push({i, j + 1});
                processed.insert({i, j + 1});
            }
        }
        return k_smallest_pairs;
    }
};
```

構造体を定義する方も練習のため3回

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        priority_queue<PairOfSumAndIndices> min_sum_pairs;
        min_sum_pairs.push({nums1[0] + nums2[0], 0, 0});
        set<PairOfIndex> processed;
        vector<vector<int>> k_smallest_pairs;

        while (k_smallest_pairs.size() < k) {
            auto [_, i, j] = min_sum_pairs.top();
            min_sum_pairs.pop();
            k_smallest_pairs.push_back({nums1[i], nums2[j]});
            if (i < nums1.size() - 1 && !processed.contains({i + 1, j})) {
                min_sum_pairs.push({nums1[i + 1] + nums2[j], i + 1, j});
                processed.insert({i + 1, j});
            }
            if (j < nums2.size() - 1 && !processed.contains({i, j + 1})) {
                min_sum_pairs.push({nums1[i] + nums2[j + 1], i, j + 1});
                processed.insert({i, j + 1});
            }
        }
        return k_smallest_pairs;
    }

private:
    struct PairOfIndex {
        int index1;
        int index2;

        bool operator<(const PairOfIndex& other) const {
            if (index1 != other.index1) return index1 < other.index1;
            return index2 < other.index2;
        }
    };

    struct PairOfSumAndIndices {
        int sum;
        int index1;
        int index2;

        bool operator<(const PairOfSumAndIndices& other) const {
            return sum > other.sum;
        }
    };
};
```
