# Search Insert Position
- 問題：https://leetcode.com/problems/search-insert-position/
- 言語：C++

## Step1

- 問題の概要
  - ソート済みのリストから対象の数があるかを調べる。リスト内に値があったらインデックスを返して、ない場合は値を挿入したときのインデックスを返す。

- 二部探索を使えばlog nでできる
- n : nums.length
- 時間計算量：O(log n)
- 空間計算量：O(1)

- numsが空の場合は、-1を返すようにした。
- targetより小さい場合は、middle + 1 ~ rightの範囲で探索、大きい場合は探索範囲をleft ~ middle - 1とする。
- left > rightになるまで探索し、ターゲットの値がない場合は、leftが挿入したときのインデックスになるのでleftを返す。

```cpp
class Solution {
public:
    int searchInsert(const vector<int>& nums, int target) {
        if (nums.empty()) {
            return -1;
        }
        int left = 0;
        int right = nums.size() - 1;
        
        while (left <= right) {
            int middle = (left + right) / 2;
            if (nums[middle] == target) {
                return middle;
            }
            if (nums[middle] < target) {
                left = middle + 1;
            }
            if (nums[middle] > target) {
                right = middle - 1;
            }
        }

        return left;
    }
};
```

## Step2

- https://github.com/akmhmgc/arai60/pull/36/changes step1
- left以上、right未満の範囲で探索する。target以上ならleft以上middle未満で探索する（right = middle）。target未満ならmiddle + 1以上right未満で探索する（left = middle + 1）。
- このとき、leftより左はtargetより小さくて、right以上はtarget以上になる。
- 探索の終了はleft == rightでそのときのインデックスが答えになる。
- leftより左はtargetより小さく、rightを含めて右はtarget以上になるので、left == rightのとき、targetがあったらそこが答えになるし、targetがなくてもそこに挿入すればいい

- 上記の書き方で解いてみた
- geminiのstep1のコードレビューで、リストが空の場合は0を返すべき、middleがオーバーフローになると指摘されたのでそれを基に改善した

```cpp
class Solution {
public:
    int searchInsert(const vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        return left;
    }
};
```

- https://github.com/Yoshiki-Iwasa/Arai60/pull/35#discussion_r1699552857
- https://discord.com/channels/1084280443945353267/1252267683731345438/1253724345369624596
- https://github.com/YukiMichishita/LeetCode/pull/7#discussion_r1561132164
  > 二分探索は、コードのパターンで覚えて書くのではなく、探索範囲が閉区間・半開区間・開区間のどれなのかを意識し、範囲を狭めるときに、範囲の端をどこまで狭めればよいかを意識して書くとよい
- step1では探索範囲を意識していなかった。探索範囲を狭めていく際の端などを良く考えずに書いていた。
  
  > 二分探索の自分の大まかな理解は、配列の各要素に対してtrue or falseを返すクエリを使って、 [false, false, ..., false, true, true, ..., true] の配列を作りfalseとtrueのtrue側の切れ目を探してる、というものです。
- この説明は結構腑に落ちた。これをtrueの中で繰り返していくイメージかな
  > まず最初に問題の設定についてですが、1. [false, false, ..., false, true, true, ..., true] の切れ目を探す。 2. [false, false, ..., false, true, true, ..., true] のうち一番初めの true を探す。
    といった設定が多いと思います。
- 今回の問題は切れ目を探す問題か。
  
- step1は閉区間で考えて、狭めるさいに左右ともにmiddleを含めない書き方をしていたのか。
- 範囲を狭めるときの条件でも書き方が変わってくるか

- https://github.com/nanae772/leetcode-arai60/pull/40/changes
- 開区間の場合
```cpp
class Solution {
public:
    int searchInsert(const vector<int>& nums, int target) {
        int left = -1;
        int right = nums.size();

        while (left + 1 < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle;
            }
        }

        return right;
    }
};
```

- https://github.com/akmhmgc/arai60/pull/36/changes step2
- 切り上げの場合やnums[middle] > targetの場合、動作が大分変ってくる。
- [left ... right)で、middleを切り上げると範囲外のアクセスが生じたりする。
- nums[middle] > targetにすると、target以上ではなくtargetよりも大きいのを探索することになる。
- 重複しているリストの場合、[1, 2, 2, 2, 3] target = 2 最初の2が出てくるインデックスを返すが、条件を変えると2よりも大きい3のインデックスを返す動作になる。

- 閉区間の場合
```cpp
class Solution {
public:
    int searchInsert(const vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] >= target) {
                right = middle - 1;
            } else {
                left = middle + 1;
            }
        }

        return left;
    }
};
```

## Step3
```cpp
class Solution {
public:
    int searchInsert(const vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] >= target) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }

        return left;
    }
};
```
