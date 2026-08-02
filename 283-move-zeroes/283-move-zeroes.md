# Move Zeroes

- 問題：https://leetcode.com/problems/move-zeroes/description/
- 言語：C++

## Step 1
- 問題の概要：
  -  整数の配列が与えられる。0以外の要素の位置は変えずに、すべての0を末尾に移動させる。移動させる際は、配列のコピーをせずに行う
  
- 先頭から見ていき、0をよける。最後に0を入れていく
- 2重ループになるが、0をよけてから、ずらしいく方法が簡単そう
- 時間計算量：O(n^2)
- 空間計算量：O(1)
- 実行時間は 10 ^ 8 / 10 ^ 8 = 1 s 程度
  
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int i = 0;
        int length = nums.size();

        while (i < length) {
            if (nums[i] != 0) {
                ++i;
                continue;
            }
            for (int j = i; j < nums.size() - 1; ++j) {
                swap(nums[j], nums[j + 1]);
            }
            --length;
        }
    }
};
```

- O(n)で済む方法が思いつかないので、GPTに聞いてみた
- GPTによると2ポインタの解き方があるらしい
- 非0要素を置く位置と0の位置を管理する
- 時間計算量：O(n)
- 空間計算量：O(1)
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int write_index = 0;

        for (int read_index = 0; read_index < nums.size(); ++read_index) {
            if (nums[read_index] == 0) {
                continue;
            }

            if (write_index != read_index) {
                swap(nums[write_index], nums[read_index]);
            }
            ++write_index;
        }
    }
};
```

## Step 2
- https://github.com/dxxsxsxkx/leetcode/pull/54/changes#diff-5b9e45f908557e06344fdf5e09ff5019ba4a0d64edf624433b57cce1dc461735
- まえに0以外の要素をつめてから、あとで後ろに0を入れる
- insert_positionほかにはzero_positionとか？いやでも、最初以外は0じゃないから変か
- swapのときはzero positionが合ってそう
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int insert_position = 0;

        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != 0) {
                nums[insert_position] = nums[i];
                ++insert_position;
            }
        }

        while (insert_position < nums.size()) {
            nums[insert_position] = 0;
            ++insert_position;
        }
    }
};
```

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int zero_position = 0;

        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != 0) {
                swap(nums[zero_position], nums[i]);
                ++zero_position;
            }
        }
    }
};
```

- https://github.com/potrue/leetcode/pull/54/changes
- iter_swap()を使っても解ける　(https://cpprefjp.github.io/reference/algorithm/iter_swap.html)
  
- https://github.com/dxxsxsxkx/leetcode/pull/54/changes#diff-32fac8ef50e0b8433ff1364beae464da20779dbbe785956ff29e70fb26dfb13d
- remove()とfill()で解けるらしい
- https://github.com/t9a-dev/LeetCode_arai60/pull/54/changes#r2981136585
- > 自分が面接官としてこの問題を出題するとしたら、 Erase-remove idiom を知っていて書けるかどうかを主題とすると思います。
- Erase-remove idiomとはコンテナから特定の条件を満たす要素を削除するテクニック（参照 : https://en.wikipedia.org/wiki/Erase%E2%80%93remove_idiom）
- remove() は指定された要素を削除し、残りは前に移動させる。コンテナのサイズは変わらないのでerase()で削除するというイディオム。
- 今回はfill()で残りを0で満たす
- 試してみたが、remove()で移動させた後の要素にもアクセスできるみたい。
  - > 有効な要素を範囲の前方に集める処理には、ムーブが使用される.
    > 取り除いた要素の先頭を指すイテレータを ret とし、範囲 [ret, last) の各要素には、有効な要素からムーブされた値が設定される。それらの値は、「有効だが未規定な値」となる
  - remove()は取り除いているより削除対象ではない要素を前に詰めるメソッドなのか
  - 本当に削除したい場合は、erase()を使う必要がある

  
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        fill(remove(nums.begin(), nums.end(), 0), nums.end(), 0);
    }
};
```

## Step 3
- remove()とfill()を使うのがシンプルで良い
- step2の最初の実装もremove()してからfill()するので、そちらを3回
```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int insert_position = 0;

        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != 0) {
                nums[insert_position] = nums[i];
                ++insert_position;
            }
        }

        while (insert_position < nums.size()) {
            nums[insert_position] = 0;
            ++insert_position;
        }
    }
};
```
