---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---





# RSA暗号

## フェルマーの小定理
$p$が素数のとき、$\mathbf{Z}_p^*=\{1,2,...,p-1\}$である。  
任意の$a \in \mathbf{Z}_p^*$に対して

$$
a^{p-1} = 1 \quad mod\space p

$$

が成り立つ。



## ポーリック-ヘルマン暗号

以下のような共通鍵暗号方式を**ポーリック-ヘルマン暗号方式**という。

- 鍵生成

1. 入力$1^n=1...1$ に対して$n$ビットの素数$p$をランダムに生成する。
2. $gcd(e,p-1)=1$となる$e$をランダムに選ぶ。
3. 拡張ユークリッドの互除法を用いて$ed = 1 \quad mod\space (p-1)$となる$d>0$を求める
4. 秘密鍵を$(p,e,d)$とする。

- 暗号化  

平文$m \in \mathbf{Z_p}^*$に対し、暗号文$c$を

$$
c = m^e\quad  mod\space p

$$

とする。

- 復号化  

暗号文$c \in \mathbf{Z}_p^*$に対し平文$m$を

$$
m = c^d \quad mod\space p
$$

と求める。

この復号化手順は以下のように正当性を確かめることが出来る。

鍵生成手順の3番目の式、$ed = 1 \quad mod \space (p-1)$より、任意の整数を$r$とすると

$$
ed = 1 + r(p-1)

$$
と書くことが出来る。復号化手順とフェルマーの小定理より

$$
\begin{equation}
\nonumber
\begin{aligned}
c^d &= (m^e)^d \quad mod\space p\\
&= m^{ed} \quad mod\space p\\
&=m^{1+r(p-1)}\quad mod\space p\\
&=m(m^{p-1})^r\quad mod \space p\\
&=m \quad mod\space p
\end{aligned}
\end{equation}

$$

これより復号化の正当性が確認できた。  
この暗号は共通鍵暗号方式なので鍵の公開は行わない。しかし、もし$(e,p)$を公開出来たら公開鍵暗号方式として機能しそうである。ただし、今のままだと拡張ユークリッドの互除法を用いると誰でも簡単に秘密鍵$d$を計算できてしまうので、何らかの工夫が必要である。

--- 

## RSA暗号
RSA暗号はポーリック-ヘルマン暗号の素数$p$を合成数$N=pq$に置き換えたものである。
もし、$p,q$が$3,7$など小さい数である場合は$N=21$から簡単に$p,q$を素因数分解することが出来てしまうが、巨大な素数の組である場合は$N$から$p,q$を計算することは難しい。これによって合成数$N$を公開鍵として公開しても、秘密鍵を計算することが難しい仕組みを作ることが出来る。  
RSA暗号のアルゴリズムは以下のようになる。

- 鍵生成
1. 入力$1^n$に対し、$n$ビットの素数の組$(p,q)$をランダムに生成し$N=pq$とする。
2. $gcd(e,(p-1)(q-1)) = 1$となる$e$をランダムに選ぶ
3. 拡張ユークリッドの互除法を用いて$ed=1\quad mod\space (p-1)(q-1)$となる$d>0$を求める。
4. 公開鍵を$(N,e)$、秘密鍵を$d$とする。

- 暗号化  
公開鍵$(N,e)$と平文$m\in \mathbf{Z}_n^*$から暗号文$c$を

$$
c = m^e\quad mod\space N
$$

とする。

- 復号化  
秘密鍵$d$と暗号文$c\in \mathbf{Z}_N^*$から平文$m$を

$$
m = c^d \quad mod\space N
$$
と計算する。

## オイラー関数とオイラーの定理
RSAの復号の正当性を解説する前に、オイラーの定理を紹介したい。  
### オイラーの定理
$n$を正の整数、$a$を$n$と互いに素な正の整数とする。このとき、

$$
a^{\phi(n)} = 1 \quad mod \space n
$$

が成立する。これを**オイラーの定理**と呼ぶ。  
この定理はフェルマーの小定理を合成数$n$に拡張したものである。  
この定理で使われる$\phi(n)$を**オイラー関数**と呼ぶ。

### オイラー関数
オイラー関数は正整数$n$に対して1から$n$までの整数のうち$n$と互いに素なものの個数を返す関数である。例えば$n=9$とすると、$\phi(9) = \#\{1,2,4,5,7,8\}=6$となる。  
$n$が素数の場合、1から$n-1$までの整数は全て互いに素になるため$\phi(n)=n-1$となる。  
また、互いに素な自然数$k,l$が存在したとき、$\phi(kl) = \phi(k)\phi(l)$となる。

実際にオイラー関数を計算してくれる[サイト](https://keisan.casio.jp/exec/user/1511790555)もあるので適当な数字で遊んでみると面白い。

## RSA復号の正当性
ポーリック-ヘルマン暗号と同じように確認することが出来る。

$$
\begin{equation}
\nonumber
\begin{aligned}
c^d &= (m^e)^d\quad mod\space N \\
&= m^{1+r(p-1)(q-1)} \quad mod \space N\\
\end{aligned}
\end{equation}
$$

ここで、$p,q$は素数であり、当たり前だが互いに素なので$\phi(N) =\phi(p)\phi(q)= (p-1)(q-1)$より

$$
\begin{equation}
\nonumber
\begin{aligned}
m^{1+r(p-1)(q-1)} \quad mod \space N &= m^{1+r\phi(N)} \quad mod \space N\\
&=m(m^{\phi(N)})^r\quad mod \space N\\
&=m
\end{aligned}
\end{equation}
$$

となり復号の正当性が確認できた。







