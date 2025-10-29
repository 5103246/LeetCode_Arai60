# Longest Increasing Subsequence
* 問題：https://leetcode.com/problems/longest-increasing-subsequence
* 言語：C++

## Step1

- 増加していくサブ配列の最長の長さを返す問題
- 値が増加しているかと、順番は合っているかが大事なので、インデックスをリストやmapで管理するのかな。
- 値を昇順にソートした方が、サブ配列を作りやすそう。インデックスも知りたいので、mapで管理する。その際、同じ値が複数出現することも考慮してインデックスはリストで管理
- 入力リストを入れたmapの先頭から順に取り出して、前のインデックスより大きかったら、リストに追加する。最後にリストの長さを返す。
- コードを提出してみたが、[1,3,6,7,9,4,10,5,6]のケースが間違っていた。
- この解き方だと、[1,3,4,5,6]で長さ5を返してしまう。
- なんとか、改良できないか、試したが上手くいかなかった。

- なんとなく、最長共通文字列と似た感じだったので、アルゴリズムを調べてみた。
- 調べていたら、どうやら、longest increasing subsequence（最長増加部分列）は有名な問題らしい。
- リストの中身を更新したら、部分列じゃなくないか？と思ったが、長さ自体は変わらないからよいのか。
- 参考：https://ibako-piyo.hatenablog.com/entry/2022/03/14/224920
- vectorを用意して、numsの先頭を入れておく。
- numsを先頭から見て、用意したリストの最後尾より大きかったら、追加する。
- 最後尾より大きくなかったら、リストを先頭から見て、自分の値より大きいのがあったら更新する
- とりあえず愚直に実装してみた。
- 時間計算量：O(n^2)
- 空間計算量：O(n)

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> subsequence{nums[0]};
        for (auto num : nums) {
            if (num > subsequence.back()) {
                subsequence.push_back(num);
                continue;
            }
            for (int i = 0; i < subsequence.size(); ++i) {
                if (subsequence[i] == num) {
                    break;
                }
                if (num < subsequence[i]) {
                    subsequence[i] = num;
                    break;
                }
            }
        }
        return subsequence.size();
    }
};
```

- lower_boundを使うと、時間計算量がO(n * log n)になる
- 変数名が少し適当かも
- lower_bound : https://en.cppreference.com/w/cpp/algorithm/lower_bound.html
- 内部実装を読んでみたかったのでgccで探してみた。https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/stl_algobase.h
- cppreferenceにも載っていた。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> subsequence{nums[0]};
        for (auto num : nums) {
            if (num > subsequence.back()) {
                subsequence.push_back(num);
                continue;
            }
            auto it = lower_bound(subsequence.begin(), subsequence.end(), num);
            *it = num;
        }
        return subsequence.size();
    }
};
```

## Step2
- https://github.com/colorbox/leetcode/pull/45/files#diff-9b5a29ecc5f6f85f4ebca7124750d32df099c24d4b0f711e2df2513f6b931a28
- lower_boundの返り値が.end()だったら、pushしてそれ以外なら値を更新すれば、もっとシンプルになるな
- https://github.com/irohafternoon/LeetCode/pull/34/files
- 変数名はlis_elementsが良さげに感じた。
- lisで省略するのは分かりづらいかもしれない。

```cpp
class Solution {
public:
    int lengthOfLIS(const vector<int>& nums) {
        vector<int> lis_elements;
        for (auto num : nums) {
            auto it = lower_bound(lis_elements.begin(), lis_elements.end(), num);
            if (it == lis_elements.end()) {
                lis_elements.push_back(num);
            } else {
                *it = num;
            }
        }
        return lis_elements.size();
    }
};
```

- https://github.com/olsen-blue/Arai60/pull/31/files
- pythonのbisect_leftを実装していたので、自分も参考に実装
- lower_boundはイテレータを返すが、インデックスでもやることは同じ
```cpp
class Solution {
public:
    int lengthOfLIS(const vector<int>& nums) {
        vector<int> lis_elements;
        for (auto num : nums) {
            int insert_index = LowerBound(lis_elements, num);
            if (insert_index == lis_elements.size()) {
                lis_elements.push_back(num);
            } else {
                lis_elements[insert_index] = num;
            }
        }
        return lis_elements.size();
    }
private:
    int LowerBound(const vector<int>& nums, int num) {
        int left = 0;
        int right = nums.size();
        while (left < right) {
            int middle = (left + right) / 2;
            if (num <= nums[middle]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return left;
    }
};
```

- イテレータを使う場合のlower_boundを実装した。
- イテレータを引数とした関数は書いたことがなかったので勉強になった。
- テンプレートを使う方が良いと思うが、テンプレートの学習がまだなので一旦これで
- イテレータを中央まで飛ばして、値を比較
- valueより小さかったら、それより右を探索するためにイテレータをずらして、要素数を更新する。
- value以上なら要素数を更新
- インデックスでやる方が分かりやすい

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> lis_elements;
        for (auto num : nums) {
            auto it = LowerBound(lis_elements.begin(), lis_elements.end(), num);
            if (it == lis_elements.end()) {
                lis_elements.push_back(num);
            } else {
                *it = num;
            }
        }
        return lis_elements.size();
    }
private:
    vector<int>::iterator LowerBound(vector<int>::iterator first, 
                                     vector<int>::iterator last,
                                     int value) {
        vector<int>::iterator it;
        typename iterator_traits<vector<int>::iterator>::difference_type count, step;
        count = distance(first, last);

        while (count > 0) {
            it = first;
            step = count / 2;
            advance(it, step);

            if (*it < value) {
                first = ++it;
                count -= step + 1;
            } else {
                count = step;
            }
        }
        return first;
    }
};
```

## Step3

```cpp
class Solution {
public:
    int lengthOfLIS(const vector<int>& nums) {
        vector<int> lis_elements;
        for (auto num : nums) {
            auto it = lower_bound(lis_elements.begin(), lis_elements.end(), num);
            if (it == lis_elements.end()) {
                lis_elements.push_back(num);
            } else {
                *it = num;
            }
        }
        return lis_elements.size();
    }
};
```
