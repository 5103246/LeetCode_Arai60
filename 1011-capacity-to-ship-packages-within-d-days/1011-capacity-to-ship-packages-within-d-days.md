# Capacity To Ship Packages Within D Days
- 問題：https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/
- 言語：C++

## Step1
- 問題の概要
  - 船に乗せる荷物を、指定されたdays日以内に輸送する。荷物の重さのリストとdaysが与えられ、days日以内に輸送するために必要な船の最小積載重量を求める。
  - リストの中身は重複ありで、リストの順番で船に積む。
  - 1 <= days <= weights.length <= 5 * 10^4
  - 1 <= weights[i] <= 500

- いろいろ考えたが、全然解き方が思いつかない。
- max(weights)を基準にして考える？
  - 1日の荷物の重さの合計がmax(weights)を超えたら、作業を終了し、次の日に移る。
  - これだと、1日余ったりする。days日全てを使って積んだ方が積める重さも減るので、日数を余らせちゃダメか。
  - 残りの荷物と残りの日数がわかれば、さらに効率的に積める
  - 残りの荷物の重さも必要そう
    
- 手が止まったので答えを見る
- https://github.com/fhiyo/leetcode/pull/45/changes step1
  - 二分探索を使用する。
  - ある積載重量の船で荷物を運んだときにかかる日数を計算する。
    - 計算した日数がdaysを超えたら、積載重量を減らして、再度かかる日数を計算する。
    - 日数がdays以内なら、積載重量を増やして、できるだけ運ぶ荷物を増やす。再度かかる日数を計算。
  - このような考え方で、積載重量を調節しながら、最小の積載重量を求めていると思う。
  - 積載重量の上限（high）はweightsの合計で、下限（low）がmax(weights)
    - 求める積載重量は、一番小さくてmax(weights)で、1日に全て詰め込むときが最大なのでweightsの合計になる。
  - middleでdays以内に運べたら、middle ~ lowで探索する。middleでdays以内に運べなかったら、積載重量が足りないので high ~ middle + 1 で探索する。
  
  - 求めたいもの : target
  - low : lowより下にはtargetはない
  - high : highより上にはtargetはない
  - low == highが終了条件
  - middleがtargetより下にある場合、lowをmiddle + 1に更新
  - middleがtargetまたはtargetより上にある場合、highをmiddleに更新する
  - m : weights.length, n : weights.sum
  - 時間計算量：O(m log n)
  - 空間計算量：O(1)
  - nは最大で 500 * 5 * 10 ^ 4 = 2.5 * 10 ^ 7 なので、実行時間は、(5 * 10 ^ 4) * log(2.5 * 10 ^ 7) / 10 ^ 8 ≒ 1.2 * 10 ^ -2 = 12 ms程度

```cpp
class Solution {
public:
    int shipWithinDays(const vector<int>& weights, int days) {
        if (weights.empty() || days <= 0) {
            return 0;
        }

        int low = *max_element(weights.begin(), weights.end());
        int high = accumulate(weights.begin(), weights.end(), 0);

        while (low < high) {
            int middle = low + (high - low) / 2;
            int days_with_capacity = CalculateDays(weights, middle);

            if (days_with_capacity <= days) {
                high = middle;
            } else {
                low = middle + 1;
            }
        }

        return low;
    }

private:
    int CalculateDays(const vector<int>& weights, int capacity) {
        int days = 1;
        int total_weight = 0;

        for (auto weight : weights) {
            if (total_weight + weight > capacity) {
                total_weight = 0;
                ++days;
            }
            total_weight += weight;
        }

        return days;
    }
};
```

## Step2
- https://github.com/fhiyo/leetcode/pull/45/changes
- https://github.com/akmhmgc/arai60/pull/39/changes
- かかる日数を計算して返すのではなく、days以内に収まるかを調べて、boolを返す解き方。
```cpp
class Solution {
public:
    int shipWithinDays(const vector<int>& weights, int days) {
        if (weights.empty() || days <= 0) {
            return 0;
        }

        int low = *max_element(weights.begin(), weights.end());
        int high = accumulate(weights.begin(), weights.end(), 0);

        while (low < high) {
            int middle = low + (high - low) / 2;

            if (IsShippable(weights, middle, days)) {
                high = middle;
            } else {
                low = middle + 1;
            }
        }

        return low;
    }

private:
    bool IsShippable(const vector<int>& weights, int capacity, int days) {
        int days_required = 1;
        int total_weight = 0;

        for (auto weight : weights) {
            if (total_weight + weight > capacity) {
                total_weight = 0;
                ++days_required;
            }
            total_weight += weight;
        }

        return days_required <= days;
    }
};
```

- geminiに合計値、最大値の他の求め方を聞いたら下のがすすめられた。
- https://cpprefjp.github.io/reference/algorithm/ranges_max.html
- https://cpprefjp.github.io/reference/algorithm/ranges_fold_left.html
- https://cpprefjp.github.io/reference/numeric/reduce.html
- ranges::maxは.begin(), .end()書かなくて済むからいいな
- ranges::fold_leftは初期値から始めて左から二項演算していく関数。二項演算を指定する必要がある
- reduceは合計値を求める関数で、accumulateと違って足す順番に規定はなくて、並列計算が可能

- https://github.com/nanae772/leetcode-arai60/pull/43/changes#diff-67d6307f8811e751f858c7989992a3a8ba392d2b1a287ec8c0ba4e918bfb4f2c
- lowを含む下の容量にはtargetがない、highより上の容量にはtargetがない でやっていた
- https://github.com/nanae772/leetcode-arai60/pull/43/changes#r2440340345
  - weight = 0やweight = -1のような場合、どうするかを考えていなかったな
  - > 十分小さい値が0に丸められているようなイメージでしょうか。その場合は少なくとも容量1はあるべきと考えて1を返すようにしたほうがよかったかもしれません。
  - 例えば、0.3 kgや0.8 kgとかのイメージか。
  - weight = -1のときは、エラーとして知らせたほうがよさそうかな。それとも、負の値は無視して計算するか。1 トンを誤って-1で入力していたら、船が沈没するかもだからエラーにしたほうがいいか。
  - リストが空のとき、daysが0以下のとき、0を返すようにしているが、どうだろうか
    - 0を返すのは、何も積まないということだから特に問題はないか？　とりあえず、このままでいいか

## Step3

- ranges::maxとranges::fold_leftにして3回書く

```cpp
class Solution {
public:
    int shipWithinDays(const vector<int>& weights, int days) {
        if (weights.empty() || days <= 0) {
            return 0;
        }

        int low = ranges::max(weights);
        int high = ranges::fold_left(weights, 0, plus<>());

        while (low < high) {
            int middle = low + (high - low) / 2;

            if (IsShippable(weights, middle, days)) {
                high = middle;
            } else {
                low = middle + 1;
            }
        }

        return low;
    }

private:
    bool IsShippable(const vector<int>& weights, int capacity, int days) {
        int days_required = 1;
        int total_weight = 0;

        for (auto weight : weights) {
            if (total_weight + weight > capacity) {
                total_weight = 0;
                ++days_required;
            }
            total_weight += weight;
        }

        return days_required <= days;
    }
};
```
