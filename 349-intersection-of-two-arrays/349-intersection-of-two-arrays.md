# Intersection of Two Arrays
* 問題: https://leetcode.com/problems/intersection-of-two-arrays/
* 言語: C++

## Step1

- setを使った解き方
- 片方のリストをsetに入れて、もう片方のリストをループで見て、setを参照していく。
- n : 入力リストの長さ
- 時間計算量：O(n * log n)
- 空間計算量：O(n)
- 実行時間は、1000 * log(1000) / 10^8 ≃ 10^(-5) s、10 µs
- visitedはなんか微妙な気がする。processedもなんか違うかな。
- moveやりすぎな気が...

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int> visited;
        for (int& num1 : nums1) {
            visited.emplace(move(num1));
        }
        vector<int> intersections;
        for (int& num2 : nums2) {
            if (visited.contains(move(num2))) {
                intersections.push_back(move(num2));
                visited.erase(move(num2));
            }
        }
        return intersections;
    }
};
```

- 他の解き方は、ソートしてから二つのリストの要素を見ていく方法が思いついた。
- 2つのリストの要素を比べて、小さい方のインデックスを進める。一致したらリストに追加。
- ソート済みならこの解き方の方が良いかも。
- 時間計算量：O(n * log n)
- 空間計算量：O(n)
- 重複しているかを判定するところで少し時間が掛かった。

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());
        vector<int> intersections;
        int i = 0, j = 0;
        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] == nums2[j]) {
                if (intersections.empty() || intersections.back() != nums1[i]) {
                    intersections.push_back(move(nums1[i]));
                }
                ++i;
                ++j;
            } else if (nums1[i] < nums2[j]) {
                ++i;
            } else {
                ++j;
            }
        }
        return intersections;
    }
};
```

- setとソートを比べてみても、速さに関しては差がなく、発想も自然でわかりやすいかと感じた。
- 入力リストがソート済みなら、ソートを使った解き方を選ぶかな。
- 入力リストが3個ならどうだろうか。
- setを使うなら、2つ目のリストと一致したsetの値を別のsetに入れて、3つ目のリストと比較する。
- ソートを使うなら、同様に1つ目と2つ目で一致したリストと3つ目を比較する。
- 他に良い解き方はあるかな。

## Step2

- step1のsetのコードだと、サイズが大きいのと小さいのが入力されたときに、大きいリストをsetに入れたら非効率になってしまう。
- なので、最初にサイズを比較して小さい方をset入れるようにする。
- setの中身がなくなったら、ループを終えるようにすると無駄がなくなる。
- setのサイズだけリストのメモリを確保しておくと、余分な動的確保が減りそう。
  - 色々考えたら、別にしなくていいなと思った。例えば、サイズが1000で、一致する要素が1つしかなかったら、残りの999個が無駄になる。
- 今回はintなのでfor文回すときは、参照なくてよかった。あと、moveもしなくて大丈夫だった。
- 改良してみたが、コードが長くなってしまった。
- 三項演算子はあまり使いたくないが、今回のコードでは、使うことによってコンパクトで、わかりやすくなると判断した。
- chat-gptに聞いたら、setにイテレータ範囲を渡したらコンストラクタで構築してくれることを知った。https://cpprefjp.github.io/reference/set/set/op_constructor.html
- 今まで、リファレンスではpushやpopなどのメンバ関数しか見ていなかったが、コンストラクタなどもちゃんと見ていきたい。

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        const vector<int>& smaller = (nums1.size() < nums2.size()) ? nums1 : nums2;
        const vector<int>& larger = (nums1.size() < nums2.size()) ? nums2 : nums1;

        set<int> candidates = BuildCandidatesFromNums(smaller);
        vector<int> intersections;
        intersections.reserve(candidates.size());
        FindIntersection(candidates, intersections, larger);
        
        return intersections;
    }

private:
    set<int> BuildCandidatesFromNums(const vector<int>& nums) {
        return set<int>(nums.begin(), nums.end());
    }
    
    void FindIntersection(set<int>& candidates, vector<int>& intersections, const vector<int>& nums) {
        for (int num : nums) {
            if (candidates.contains(num)) {
                intersections.push_back(num);
                candidates.erase(num);
            }
            if (candidates.empty()) break;
        }
    }
};
```

- https://github.com/fhiyo/leetcode/pull/16/files
- 両方setにする方法で解いている人も多かった。
- 両方setにすることでeraseする必要がなくなる。
- サイズが小さい方で回すことでコストが抑えられ、よりコードも簡潔になるから、この解き方の方が好み。
- 時間計算量はO(max(n1 * log n1, n2 * log n2))
```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int> smaller(nums1.begin(), nums1.end());
        set<int> larger(nums2.begin(), nums2.end());
        if (nums1.size() > nums2.size()) {
            larger.swap(smaller);
        }
        vector<int> intersections;
        for (int num : smaller) {
            if (larger.contains(num)) {
                intersections.push_back(num);
            }
        }
        return intersections;
    }
};
```

- https://github.com/maeken4/Arai60/pull/12/files
- https://cpprefjp.github.io/reference/algorithm/set_intersection.html
- c++にはset_intersectionというSTLがあるらしい。
- コンパクトになっててgood。ただ、思いつかなさそう。setやソートを使う方が発想しやすいかな。
```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int> candidates1(nums1.begin(), nums1.end());
        set<int> candidates2(nums2.begin(), nums2.end());
        vector<int> intersections;
        set_intersection(candidates1.begin(), candidates1.end(), 
                         candidates2.begin(), candidates2.end(),
                         back_inserter(intersections));
        return intersections;
    }
};
```

## Step3
```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int> smaller(nums1.begin(), nums1.end());
        set<int> larger(nums2.begin(), nums2.end());
        if (nums1.size() > nums2.size()) {
            larger.swap(smaller);
        }
        vector<int> intersections;
        for (int num : smaller) {
            if (larger.contains(num)) {
                intersections.push_back(num);
            }
        }
        return intersections;
    }
};
```
