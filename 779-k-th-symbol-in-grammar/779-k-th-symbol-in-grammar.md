# K th Symbol in Grammar
- 問題：https://leetcode.com/problems/k-th-symbol-in-grammar/
- 言語：C++

## Step 1
- 問題の概要
  - 下記のような表を作る。nとkが与えられるので、n行目のk番目の数字を返す。このとき、前行を確認し、0 -> 01, 1 -> 10に置き換える。最初は0スタート
  - 1 <= n <= 30 , 1 <= k <= 2 ^ (n - 1)
```
1 : 0
2 : 0 1
3 : 0 1 1 0
4 : 0 1 1 0 1 0 0 1
:
n : 0 1 ... 1 0 ... 0 1 ...
```

- シンプルに考えるなら、n行目の数字を全て作ってから、k番目を探す。
  - kの最大値は2 ^ 29 ≒ 5 * 10 ^ 9 なので、速度の制限がなければこれでもよさそう。
- n行目の数字は全部で 2 ^ (n - 1)、表は下のように木構造になっている
  ```
  　　　　　　　　　　0
                   0   1
                 0  1  1   0
               0 1 1 0 1 0 0 1
  ```
- k番目の数字は親の数字がわかれば、求まる
- 二分探索的に解ける
  - left=1, right=2 ^ (n-1) 
  - leftより左にはkがなく、rightより右にもkはない
  - middle >= kなら、right = middle、置き換え後の左の数字を渡す。例：0なら0を渡す
  - middle < kなら、left = middle + 1、置き換え後の右の数字を渡す。例：0なら1、1なら0を渡す。
  - left = rightのときの数字を返す。
- 時間計算量：O(log (2 ^ n -1)) = O(n)
- 空間計算量：O(n)
- 再帰の方が考えやすいので再帰で書いてみる。

```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        return KthSymbol(0, k, 1, pow(2, n - 1));
    }

private:
    int KthSymbol(int symbol, int k, int left, int right) {
        if (left == right) {
            return symbol;
        }

        int next_symbol;
        int middle = left + (right - left) / 2;

        if (middle >= k) {
            if (symbol == 0) {
                next_symbol = 0;
            } else {
                next_symbol = 1;
            }
            right = middle;
        } else {
            if (symbol == 0) {
                next_symbol = 1;
            } else {
                next_symbol = 0;
            }
            left = middle + 1;
        }

        return KthSymbol(next_symbol, k, left, right);
    }
};
```

- if文を簡潔にしたもの

```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        return KthSymbol(0, k, 1, pow(2, n - 1));
    }

private:
    int KthSymbol(int symbol, int k, int left, int right) {
        if (left == right) {
            return symbol;
        }
        int middle = left + (right - left) / 2;

        if (middle >= k) {
            return KthSymbol(symbol, k, left, middle);
        } else {
            return KthSymbol(!symbol, k, middle + 1, right);
        }
    }
};
```

- ループ版
- 時間計算量：O(n)
- 空間計算量：O(1)

```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        int symbol = 0;
        int left = 1; 
        int right = pow(2, n - 1);

        while (left != right) {
            int middle = left + (right - left) / 2;

            if (middle >= k) {
                right = middle;
            } else {
                symbol = !symbol;
                left = middle + 1;
            }
        }

        return symbol;
    }
};
```

## Step 2
- https://github.com/Ryotaro25/leetcode_first60/pull/49/changes#r1894794397
  - pow関数は計算に浮動小数を使用しているので、戻り値が整数になるかは注意が必要
  - 1 << n - 1 の方がよい。(1 << n は1を左にnビットシフトするので2 ^ nがもとまる)

```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        int symbol = 0;

        int left = 1;
        int right = 1 << n - 1;

        while (left != right) {
            int middle = left + (right - left) / 2;

            if (middle >= k) {
                right = middle;
            } else {
                symbol = !symbol;
                left = middle + 1;
            }
        }
        return symbol;
    }
};
```

- https://github.com/olsen-blue/Arai60/pull/47/changes#r2002307405
- https://github.com/5ky7/arai60/pull/46/changes
- どうやら、k - 1のビットカウントの偶奇で答えが求まるらしい
- よくわからんので、aiに聞いてみる。
  - 右に行くと反転するので、根（０）から右に行った回数で答えがもとまる。
  - n = 3, k = 4 の場合、0 -> 1 -> 0 で右に2回移動。偶数回なら0、奇数回なら1
  - 右に行く回数とkがなぜつながるのかがわからん
```
                           0
                        0     1
                       0  1  1   0
                    0  1 1 0 1 0 0  1
```
- n = 4の場合を図にして考える
  - k = 1なら右に行く回数は0なので0が答え
  - k = 2なら最後に1回右に行くので答えは1
  - k = 3なら途中に1回右に行くので1
  - k = 4なら右に2回移動するので答えは0
  - たしかに、k - 1のビットの個数が関係しているな
  - 左に行くを0、右に行くを1とすると、k = 1の場所は、000で行ける
  - k = 2なら、001 k = 3なら、010 k = 6なら、101と、k - 1の2進数と一致する
  - k - 1のビットカウントをする場合、std::popcount()がある
    
```cpp
#include <bit>

class Solution {
public:
    int kthGrammar(int n, int k) {
        return popcount(static_cast<uint32_t>(k - 1)) % 2;
    }
};
```
- ビットカウントを実装するなら、ループで1桁ずつカウントするのが簡単だが、たしかもう少し効率が良いのがあったので実装してみる
- 変数名や、処理はもっとよく書けると思う
- ビット演算のみなので高速、パイプライン化がしやすい
- 32bitを2bitごとのビットカウント->4bitごと->8bit->16bit->32bitごとのビットカウントのように、ビットをグループ化して足すを繰り返す
```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        uint32_t x = k - 1;

        uint32_t tmp1 = x & 0xAAAAAAAA;
        uint32_t tmp2 = x & 0x55555555;
        uint32_t tmp3 = (tmp1 >> 1) + tmp2;

        tmp1 = tmp3 & 0xCCCCCCCC;
        tmp2 = tmp3 & 0x33333333;
        tmp3 = (tmp1 >> 2) + tmp2;

        tmp1 = tmp3 & 0xF0F0F0F0;
        tmp2 = tmp3 & 0x0F0F0F0F;
        tmp3 = (tmp1 >> 4) + tmp2;

        tmp1 = tmp3 & 0xFF00FF00;
        tmp2 = tmp3 & 0x00FF00FF;
        tmp3 = (tmp1 >> 8) + tmp2;

        tmp1 = tmp3 & 0xFFFF0000;
        tmp2 = tmp3 & 0x0000FFFF;
        tmp3 = (tmp1 >> 16) + tmp2;

        return tmp3 % 2;
    }
};
```

- https://github.com/dxxsxsxkx/leetcode/pull/46/changes#diff-cead94aaa0be09245aaf9ec8204cfa11d2998d9a6aef44e8d59f223e5be4a853
- n行目の前半はn - 1行目のコピーで、後半はそれを反転したものという性質を活かした解き方
- コンパクトでよい

```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        if (n == 1 && k == 1) {
            return 0;
        }

        int middle = 1 << (n - 2);
        
        if (middle >= k) {
            return kthGrammar(n - 1, k);
        } else {
            return !kthGrammar(n - 1, k - middle);
        }
    }
};
```
## Step 3
```cpp
class Solution {
public:
    int kthGrammar(int n, int k) {
        if (n == 1 && k == 1) {
            return 0;
        }

        int middle = 1 << (n - 2);

        if (middle >= k) {
            return kthGrammar(n - 1, k);
        } else {
            return !kthGrammar(n - 1, k - middle);
        }
    }
};
```
