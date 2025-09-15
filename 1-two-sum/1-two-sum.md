# Two Sum
* 問題: https://leetcode.com/problems/two-sum/
* 言語: C++

## Step1

- O(n^2)の解き方はすぐ出てきた。今回の問題はnums.length <= 10^4だったので、10^8程度に収まるのでこの解き方でも良いと思う。
- n: 入力長
- 時間計算量：O(n^2)
- 空間計算量：O(1)
  
- mapを使った解き方も思いついた。
- リストをループし、target - nums[i]がmapになければ、追加し、mapにあったらインデックスをリストに加えreturn
- 時間計算量：O(n log n)
- 空間計算量：O(n)
- 実行時間は、(10^4) * log(10^4) ≒ 10^5、10^5 / 10^8 = 10^(-3)で、1 ms 程度だと予想される。
- [実行時間の見積もり](https://discordapp.com/channels/1084280443945353267/1262688866326941718/1346304869773869116)
- differenceとかの変数名に悩んで少し時間がかかった。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> two_index;
        map<int, int> visited;
        for (int i = 0; i < nums.size(); i++) {
            int difference = target - nums[i];
            if (visited.contains(difference)) {
                two_index.push_back(i);
                two_index.push_back(visited[difference]);
                break;
            }
            visited.emplace(nums[i], i);
        }
        return two_index;
    }
};
```

## Step2
### 他の人のコードや議論を見てみる

- インクリメントは前置が推奨されるのを始めて知った。後置にすると、コピーが生じて効率が悪くなるため前置がgoogle style guideで推奨されている。
- そこまで気にしてコードを書いていなかった。
- https://github.com/ntanaka1984/leetcode/pull/3/files#r2283615692
- https://github.com/irohafternoon/LeetCode/pull/22#discussion_r2053628221
- 初期化リストが戻り値にも使えることを知らなかった。```return {...}```より```return vector<int>{...}```の方が読み手にとって親切かも？
  - https://github.com/ryosuketc/leetcode_grind75/pull/1/files
  - 戻り値の型が分かるので```return {...}```で十分らしい。
    - https://github.com/maeken4/Arai60/pull/10/files#r2182296739
- ソートして両端から探す方法もあるらしい。
  - https://github.com/locol23/leetcode/pull/1/files#r2233905016
- difference(差)よりcomplement(補完)の方が操作に合っていると感じた。mapの変数名はvisitedじゃ何が入っているかわかない。num_to_indexがいい

Step1の改良
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        map<int, int> num_to_index;
        for (int i = 0; i < nums.size(); ++i) {
            int complement = target - nums[i];
            if (num_to_index.contains(complement)) {
                return {num_to_index[complement], i};
            }
            num_to_index.emplace(nums[i], i);
        }
        return {};
    }
};
```

sortしてから両端から探索
- 時間計算量：O(n log n)
- 空間計算量：O(n)
- 入力がsort済みなら、時間計算量はO(n)、空間計算量はO(1)
- この両端から探索する（two pointers)方法はsliding windowでも使っていたので、パッと思いつけるようにしたい
  - https://www.geeksforgeeks.org/dsa/two-pointers-technique/
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<pair<int, int>> num_to_index;
        for (int i = 0; i < nums.size(); ++i) {
            num_to_index.push_back({nums[i], i});
        }

        sort(num_to_index.begin(), num_to_index.end());
        int left = 0;
        int right = num_to_index.size() - 1;
        while (left != right) {
            int sum = num_to_index[left].first + num_to_index[right].first;
            if (target == sum) {
                return {num_to_index[left].second, num_to_index[right].second};
            } else if (target > sum) {
                ++left;
            } else {
                --right;
            }
        }
        return {};
    }
};
```

## Step3

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        map<int, int> num_to_index;
        for (int i = 0; i < nums.size(); ++i) {
            int complement = target - nums[i];
            if (num_to_index.contains(complement)) {
                return {num_to_index[complement], i};
            }
            num_to_index.emplace(nums[i], i);
        }
        return {};
    }
};
```
