# Find Minimum in Rotated Sorted Array

- 問題：https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/
- 言語：C++

## Step1

- 問題の概要
  - 昇順の配列を1 ~ n回転した配列が与えられる。この配列の最小値を求める問題。配列の中身は重複していなくて、時間計算量O(log n)で記述する。

- O(log n)が制約なのでnループは使えない。二分探索で考える。
- もともとの配列は昇順のため、回転したら増加しない箇所が生まれる。この切れ目を探索すれば最小値も求まる。
- 隣の要素同士の変化分で分ける？ -> マイナスになるのは1か所のみなので分けれない

- 先頭の要素より大きければ、左側は無視して右側を見る。先頭要素より小さければ、値を更新して、左側を探索する。
- 更新が終わった値が最小値になる。昇順の配列なら更新なしで先頭要素が最小値となる。
- targetを求める最小値として、先頭要素を初期値にする。
- leftより左側はtarget以上、rightを含む右側はtarget以上
- middleの要素がtarget以上ならleftをmiddle+1に更新し、要素がtarget未満ならtargetを更新し、rightをmiddleにする。
- left == rightがループの終了条件

- n：nums.length
- 時間計算量：O(n)
- 空間計算量：O(1)
- 解き方は10分程で出てきたが、二分探索の書き方を整理しながら実装したら30分程かかった。

```cpp
class Solution {
public:
    int findMin(const vector<int>& nums) {
        int min_target = nums.front();
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] >= min_target) {
                left = middle + 1;
            } else {
                min_target = nums[middle];
                right = middle;
            }
        }

        return min_target;
    }
};
```

## Step2

- https://github.com/t9a-dev/LeetCode_arai60/pull/42/changes
- 他の人のコードを見たら、targetを使わない書き方が多かったので、書いてみた。
  
```cpp
class Solution {
public:
    int findMin(const vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] >= nums[right]) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }

        return nums[right];
    }
};
```

- https://github.com/Ryotaro25/leetcode_first60/pull/46/changes/BASE..5cd497a61c1610dfb252de6f0dd2a0823e7b2bec#r1869993674
  - > Q1「2で割る処理がありますがこれは切り捨てでも切り上げでも構わないのでしょうか。」
    - A1 切り上げになると無限ループになるのでだめ。
  - > Q2「nums[middle] >= nums[right] とありますが、これは > でもいいですか。」
    - A2 middleが切り捨てならmiddleとrightが重なることはないからどちらも変わらない。
  - > Q3「nums[right] は、nums[nums.size() - 1] でもいいですか。」
    - A3 試してみたが、どちらでも大丈夫だった。
  - > Q4「right の初期値は nums.size() でもいいですか。」
    - A4 配列外を参照してしまうのでよくない。ただ、nums[middle] > nums.back()の場合は、オッケー。nums[middle] >= nums.back()の場合、leftがnums.size()まで来てしまい、インデックスエラーとなる。

- 右端と比較する方法
- leftより左は最小値よりも大きく、rightを含む右側は最小値以上になる。なので探索終了時のleft, rightが最小値となる。
- 最後尾の要素よりmiddleが大きい場合、middleを含む左側には最小値がないことがわかるのでleftをmiddle + 1にする。
- middleが右端以下なら、昇順に並んでいることから、middleを含む左側に最小値があることがわかる。なので、rightをmiddleに更新する。

```cpp
class Solution {
public:
    int findMin(const vector<int>& nums) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] > nums.back()) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }

        return nums[right];
    }
};
```

- https://github.com/nanae772/leetcode-arai60/pull/41/changes#diff-59a1a3ad1a3ae61f3d06b5dd35db0022bbea64931f758a4d4d7ff1bb4c461611
- 左端で比較する方法
- leftより左側は左端以上で、rightより右側は左端未満。left == right + 1がループの停止条件。探索終了時のleft、right + 1が最小値となる。
- left == rightをループの停止条件にすると、[0,1,2,3,4]と[1,2,3,4,0]の区別がつかなくなる。
- leftより左側は左端以上で、rightより右側は左端未満なので、[front, ..., right, left, ...]となるとちょうどleftが最小値になる。なのでleft == right + 1になるまで探索する必要がある。

## Step3

```cpp
class Solution {
public:
    int findMin(const vector<int>& nums) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;

            if (nums[middle] <= nums.back()) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        return nums[right];
    }
};
```
