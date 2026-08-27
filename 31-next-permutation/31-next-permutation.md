# Next Permutation

- 問題：https://leetcode.com/problems/next-permutation/
- 言語：C++

## Step 1

- 問題の概要
  - numsの次の順列を求める。配列を作らず、渡された配列を書き換える必要がある
  - 例：nums = [2, 8, 3]の場合、{[2,3,8],[2,8,3],[3,2,8],[3,8,2],[8,2,3],[8,3,2]}
  - 答：[3, 2, 8]
    
- [2,2,3,4,5]のケースで順列を考えてみる
- [2,2,3,4,5], [2,2,3,5,4], [2,2,4,3,5], [2,2,4,5,3], [2,2,5,3,4], [2,2,5,4,3], [2,3,2,4,5], [2,3,2,5,4], [2,3,4,2,5], [2,3,4,5,2], [2,3,5,2,4], [2,3,5,4,2], [2,4,2,3,5] ...
- 後ろから見て、nums[i] < nums[i+1]を探す。
- 後ろからnums[i]より大きい要素を探す。nums[i]と入れ替える。
- i+1以降の要素を反転する
- n : nums.length
- 時間計算量：O(n)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        for (int i = nums.size() - 2; i >= 0; --i) {
            if (nums[i] < nums[i + 1]) {
                for (int j = nums.size() - 1; j > i; --j) {
                    if (nums[i] < nums[j]) {
                        swap(nums[i], nums[j]);
                        reverse(nums.begin() + i + 1, nums.end());
                        return;
                    }
                }
            }
        }

        reverse(nums.begin(), nums.end());
    }
};
```

- 一旦書いてみたが、ネストが深くて読みづらいのでリファクタする
- numsが降順のとき、最後に全部反転する。最後まで読まないとわからないのがダメだな

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int n = nums.size();
        int i = n - 2;

        while (i >= 0 && nums[i] >= nums[i + 1]) {
            --i;
        }
        if (i >= 0) {
            int j = n - 1;
            while (nums[i] >= nums[j]) {
                --j;
            }
            swap(nums[i], nums[j]);
        }

        reverse(nums.begin() + i + 1, nums.end());
    }
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/58/changes#diff-bb5a6b8b163a5fd3407269aba54da1f45bc4f8a22f79dc80101fc048442bd135
- iterで解く方法
- is_sorted_until(https://cpprefjp.github.io/reference/algorithm/is_sorted_until.html)
  - ソート済みか判定し、ソートされていない箇所を返す
- upper_bound, iter_swapでswapするものを探して入れ替える。
```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        auto it = is_sorted_until(nums.rbegin(), nums.rend());

        if (it != nums.rend()) {
            auto next_it = upper_bound(nums.rbegin(), it, *it);

            iter_swap(it, next_it);
        }

        reverse(nums.rbegin(), it);
    }
};
```

- https://github.com/kazuki-official/leetcode/pull/59/changes
- https://github.com/potrue/leetcode/pull/58/changes

- c++にはnext_permutation() が標準であるみたい
- https://cpprefjp.github.io/reference/algorithm/next_permutation.html
- 次の順列があればtrueを返す。同時に中身も変換する
- 内部実装：https://cppreference.com/cpp/algorithm/next_permutation
  - step2の実装と同じ
  - 変数名はleft, rightにしている

- https://github.com/potrue/leetcode/pull/58/changes#r2303516173
- itじゃなくて、left, pivotのように場所やインデックスを示す名前が良いのではという話
- どっちがいいだろうか。it, next_itよりはleft, rightの方が情報量がありそう。
- 最後、reverse(nums.rbegin(), left) になって少し変な感じもするが..
  - 個人的にはleft, rightの方が良い気がする
    

## Step 3
```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        auto left = is_sorted_until(nums.rbegin(), nums.rend());

        if (left != nums.rend()) {
            auto right = upper_bound(nums.rbegin(), left, *left);
            iter_swap(left, right);
        }

        reverse(nums.rbegin(), left);
    }
};
```
