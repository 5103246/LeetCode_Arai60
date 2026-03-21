# Pow(x, n)
- 問題：https://leetcode.com/problems/powx-n/
- 言語：C++

## Step1

- 問題の概要：x ^ nを返す
  - -100.0 < x < 100.0
  - Either x is not zero or n > 0.
  - -10 ^ 4 <= x ^ n <= 10 ^ 4
  - -2 ^ 31 <= n <= 2 ^ 31-1

- 再帰、ループで書けそう
- ループで一旦考えると、n < 0の場合、xの逆数を変数に保存して、nに-1を掛けてからループで計算する
  - 時間計算量はO(n)、空間計算量はO(1)になる。
  - 時間計算量はO(n)だが、n = 2 ^ 31 - 1 のとき、実行時間は 2147483647 / 10 ^ 8 ≒ 21 s となるので おそらく時間オーバーになる。
  - また、n = -2 ^ 31 のとき、符号を反転するとn = 2 ^ 31になってint型の範囲を超えてしまう

- 手が止まったので、答えを見る
- https://github.com/naoto-iwase/leetcode/pull/46/changes step1
  - 10 ^ 8 = ((10 ^ 2) ^ 2) ^ 2 = みたいに分解してやれば、時間計算量O(log n)で済むのか
  - nの絶対値を取ってからシフト演算で分解していく
  - resultとpowerに分けて、シフト結果が偶数ならpowerを自乗していき、奇数ならresultにpowerを掛ける

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        long long exp = n;

        if (exp < 0) {
            x = 1.0 / x;
            exp = -exp;
        }

        double result = 1.0;
        double power = x;

        while (exp > 0) {
            if (exp % 2 == 1) {
                result *= power;
            }
            power *= power;
            exp /= 2;
        }

        return result;
    }
};
```

- ループの解き方を参考に、再帰で書いてみた。
- 個人的には再帰の方が直感的でわかりやすい

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        if (x == 0.0) {
            return 0.0;
        }
        
        long long exp = n;

        if (exp < 0) {
            return 1 / myPowHelper(x, -exp);
        }
        return myPowHelper(x, exp);
    }

private:
    double myPowHelper(double x, long long n) {
        if (n == 0) {
            return 1.0;
        }

        double half = myPowHelper(x, n / 2);

        if (n % 2 == 0) {
            return half * half;
        } else {
            return half * half * x;
        }
    }
};
```

## Step2

- https://github.com/Ryotaro25/leetcode_first60/pull/48/changes#r1978642684
  - floatは32ビットで、符号が1ビット、指数が8ビット、仮数が23ビット
  - doubleは64ビット、符号が1ビット、指数が11ビット、仮数が52ビット
  - C/C++では、具体的な仕様はコンパイラ依存
- https://github.com/potrue/leetcode/pull/45/files#r2272676005
  - > CPU の割り算の命令は一般的に遅いです。そのため、コンパイラーは等価なビット演算に変換できる場合は、それらに置き換える場合があります
  - compiler explorerで最適化オプションをつけて確認したら、符号なしの場合は割り算と右シフトで同じ命令が生成された。
    符号ありの場合は、少し違ったがシフト演算の命令で割り算が行われていた。
  - x86_64では、符号あり右シフトがsar命令、符号なし右シフトがshr命令、左シフトがsal, shl命令

## Step3
- ループ、再帰それぞれ3回

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        if (x == 0.0) {
            return 0.0;
        }

        long long exp = n;

        if (exp < 0) {
            return 1 / myPowHelper(x, -exp);
        }
        return myPowHelper(x, exp);
    }

private:
    double myPowHelper(double base, long long exp) {
        if (exp == 0) {
            return 1.0;
        }

        double half = myPowHelper(base, exp / 2);

        if (exp % 2 == 0) {
            return half * half;
        } else {
            return half * half * base;
        }
    }
};
```

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        if (x == 0.0) {
            return 0.0;
        }

        long long exp = n;

        if (exp < 0) {
            x = 1.0 / x;
            exp = -exp;
        }

        double result = 1.0;
        double x_powered = x;

        while (exp > 0) {
            if (exp & 1) {
                result *= x_powered;
            }
            x_powered *= x_powered;
            exp >>= 1;
        }

        return result;
    }
};
```
