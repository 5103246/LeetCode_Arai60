# Unique Email Addresses
* 問題: https://leetcode.com/problems/unique-email-addresses/
* 言語: C++

## Step1
- 最初、multimapを使う解き方を考えた。
- keyにlocalを、valueにdomainを入れる。
- メールをlocalとdomainに分け、multimapのlocalの値とdomainが異なっていたら、multimapに追加
- 最終的にmultimapのサイズを返す。
- mapを使って、valueにdomainのリストを作るのでもできそう。
- しかし、multimapの要素アクセスが面倒くさそうだったので、別の方法を考えた。

- 次に、setに処理したメールアドレスを入れる方法を考えた。
- メールをlocalとdomainに分け、localに処理をしてからdomainとくっつけてsetに入れる。
- 最後はsetのサイズを返す。
- この解き方だと、mapのようにlocalが同じでdomainが異なるケースを考えずに済む。
- n : 入力サイズ、m : メールアドレスの長さ
- 時間計算量が大体O(m * n)で空間計算量がO(n)くらいありそう
- setとmapで速度の差はそれほどなさそうだが、setの方がシンプルに掛けそうなのでsetで解く。

```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> unique_emails;
        for (string& email : emails) {
            int i = email.find('@');
            string domain = email.substr(i);
            email.erase(i);
            string local = SkipDotsAndPlus(email);
            if (!unique_emails.contains(local + domain)) {
                unique_emails.insert(local + domain);
            }
        }
        return unique_emails.size();
    }

private:
    string SkipDotsAndPlus(const string& email) {
        string processed;
        for (char ch : email) {
            if (ch == '.') continue;
            if (ch == '+') break;
            processed += ch;
        }
        return processed;
    }
};
```

- 関数名SkipDotsAndPlusは微妙かな

## Step2

- https://github.com/akmhmgc/arai60/pull/11/files
- 正規化しているからcanonicalizeとかが適している
- normalizeは冗長削減のための正規化、canonicalizeは比較のための正規化（https://it.sifr.me/study/the-difference-between-normalize-and-canonicalize/）
- > 私の感覚、この問題は「書ける」ことが採点のポイントではないと感じます。
  > RFC にメールアドレスの仕様があること、かなり複雑であること、正規表現を知っていること、フォローアップで validation の必要性は状況によると思われるがどのような場合は特に必要か、などを聞きたいのだろうと思います。
- RFCという用語すら知らなかった。RFC(Request for Comments)は、「インターネットで用いられるさまざまな技術の標準化や運用に関する事項など幅広い情報共有を行うために公開される文書シリーズ」らしい。（https://www.nic.ad.jp/ja/newsletter/No24/090.html）
- https://www.rfc-editor.org/rfc/rfc5321
- 正規表現は聞いたことがあったが、よく理解してなかった。メタ文字について学んだ。（https://userweb.mnet.ne.jp/nakama/#no-four）
- validationの必要性などに関しては、そもそもユースケースを考えずに書いていた。他の方は、不正なメールアドレスだったりを考えていた。
  
- https://github.com/seal-azarashi/leetcode/pull/14#discussion_r1676988400
- 問題の入力で動くかどうかしか考えてなかった。
- 今までの問題も、問題文のルールや制約に合っているかしか気にしていなかった。問題を解くことに意識が向きすぎていたので反省したい。
- 問題を解き続けていると、現実との繋がりへの意識が薄くなっていくので、次からは、最初に「問題文～のような仕事を頼まれました。」を考えるようにする。
  
- https://github.com/Yoshiki-Iwasa/Arai60/pull/13#discussion_r1649832719
  - https://github.com/Yoshiki-Iwasa/Arai60/pull/13/files#r1676820615
  - メールアドレス宛に送信するというケースなら、誤ったメールアドレスがあっても、処理は止めずに続けたい。webサイトの会員情報を入力しているケースなら、後続処理を止めて、正確なアドレスを入力させる。
  - そもそも誤ったメールアドレスとは何か
  - メールアドレスの長さが254字を超えているもの（https://zenn.dev/be_the_light/articles/email_validation_rfc）
  - ""で囲まれていないlocal nameに@がある場合
  - @がないメールアドレス
- https://github.com/plushn/SWE-Arai60/pull/14#discussion_r2051724883
- 今回は、送信時と考えて、不正なメールアドレスがあっても処理は続けるようにしてみる。

- https://github.com/colorbox/leetcode/pull/28/files
- 上の話を参考に入力を変更するコードと、コピーして返すコードを書いた。正規化する際にfor文を使わない方法で書いた。
```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> unique_emails;
        for (string& email : emails) {
            Canonicalize(&email);
            unique_emails.emplace(move(email));
        }
        return unique_emails.size();
    }
private:
    void Canonicalize(string* email) {
        const auto at_mark_position = email->rfind('@');
        string local = email->substr(0, at_mark_position);
        const auto plus_position = local.find('+');
        local = local.substr(0, plus_position);
        erase(local, '.');
        email->replace(0, at_mark_position, local);
    }
};
```
- バリデーションチェックも入れてみた。
- メールアドレスの長さと、@が含まれているかのみしかチェックしてない。
- localやdomainそれぞれのチェックもしたかったが、かなり大変そうに感じたので今回は簡単なチェックに留めた。
- 不正なアドレスがあったら、素通りして次のアドレスの処理をするようにした。
  
```cpp
class Solution {
public:
    int numUniqueEmails(const vector<string>& emails) {
        set<string> unique_emails;
        for (const string& email : emails) {
            if(!IsValidAddress(email)) {
                continue;
            }
            unique_emails.emplace(Canonicalize(email));
        }
        return unique_emails.size();
    }
private:
    bool IsValidAddress(const string& email) {
        if (email.size() > 254) {
            return false;
        }
        if (!email.contains('@')) {
            return false;
        }
        return true;
    }
    string Canonicalize(const string& email) {
        const auto at_mark_position = email.rfind('@');
        const string domain = email.substr(at_mark_position);
        string local = email.substr(0, at_mark_position);
        const auto plus_position = local.find('+');
        local = local.substr(0, plus_position);
        erase(local, '.');
        return local + domain;
    }
};
```
- step1の改良コード
- constにしてない箇所やif文を変更

```cpp
#include <regex>

class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> unique_emails;
        for (const string& email : emails) {
            unique_emails.emplace(Canonicalize(email));
        }
        return unique_emails.size();
    }
private:
    string Canonicalize(const string& email) {
        const auto at_mark_position = email.rfind('@');
        const string domain = email.substr(at_mark_position);
        string local = email.substr(0, at_mark_position);
        string canonicalized_local;
        for (const char ch : local) {
            if (ch == '.') {
                continue;
            }
            if (ch == '+') {
                break;
            }
            canonicalized_local += ch;
        }
        return canonicalized_local + domain;
    }
};

```

- 正規表現を使ったコード
- 正規表現を使って正規化をしたが、バリデーションチェックをするときにも使える。
- regexを使うと、明らかに実行時間が長かった。gptに聞いたら、regexが重いライブラリであることが原因らしい。
- staticをつけて、再利用することで幾らか改善した。

```cpp
#include <regex>

class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> unique_emails;
        for (const string& email : emails) {
            unique_emails.emplace(Canonicalize(email));
        }
        return unique_emails.size();
    }
private:
    string Canonicalize(const string& email) {
        static const regex plus(R"(\+.*)");
        static const regex dots(R"(\.)");
        
        const auto at_mark_position = email.rfind('@');
        string local = email.substr(0, at_mark_position);
        string domain = email.substr(at_mark_position);
        local = regex_replace(local, plus, "");
        local = regex_replace(local, dots, "");
        return local + domain;
    }
};
```

- 感想
- 入力を変更する方法、入力を変更せず、変更したものをコピーで返す方法、正規表現を使った方法、for文で地道に回す方法、findやsubstrで解く方法
- 自分的には、入力を変更せず、コピーを返す方が、呼び出し側が安心して使えるのでよいなと感じた。
- 正規表現は、複雑になると読み手がつらくなるので、あまり使いたくない
- 自分はfor文で回す方がわかりやすいかなと感じたが、文字列操作に慣れるためにもstep3では、substrやeraseで解く


## Step3
```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> unique_emails;
        for (const string& email : emails) {
            unique_emails.emplace(Canonicalize(email));
        }
        return unique_emails.size();
    }
    
private:
    string Canonicalize(const string& email) {
        const auto at_mark_position = email.rfind('@');
        const string domain = email.substr(at_mark_position);
        string local = email.substr(0, at_mark_position);
        const auto plus_position = local.find('+');
        local = local.substr(0, plus_position);
        erase(local, '.');
        return local + domain;
    }
};
```
