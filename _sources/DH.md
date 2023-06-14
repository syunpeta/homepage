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

# DH鍵配送問題

## 離散対数問題(DLP)
$G$を位数が大きな素数$q$の巡回群とし、その生成元を$g$とする。このとき、$(g,y) \in G$から

$$
y = g^x\pmod{q}
$$
となる正整数$x$を求める問題を**離散対数問題**という。(加法群の場合も同じように定義される)　　

$x$は$0 \leq x < q$の範囲にある。従って、これを総当たりで解こうとすると$O(q)$の計算量が必要になる。  
また、これを効率的に解くアルゴリズムとしてPollardの$\rho$法やBaby-step Giant-stepなどのアルゴリズムがあるが、何れも計算量$O(\sqrt{q})$のアルゴリズムであるため巨大な位数の離散対数問題を解くことは難しい。

## DH鍵配送法
Diffie-Hellman(DH)の鍵配送法を用いると、事前の相談なしにAliceとBobが秘密鍵を共有することが出来る。

- 鍵生成
1.  大きな素数$q$と小さな整数$g$を決め、AliceとBobで共有しておく(公開しても問題ない。)
2. Alice は秘密鍵$a \in Z_q$をランダムに選び、$y_A = g^a \pmod{q}$を公開する。
3. Bobは秘密鍵$b\in Z_q$ をランダムに選び$y_B = g^b \pmod{q}$を公開する。

- 鍵共有
1. Alice は$K_A = (y_B)^a \pmod{q}$を計算する。
2. 同様にBobは$K_B=(y_A)^b\pmod{q}$を計算する。

$K=g^{ab}$とすると$K=K_B=K_A$となり、Alice,Bobは鍵$K$を共有できた。  

![DH_figure](./_build/html/_images/DH.png)

悪意のある第3者Charlieが公開された$y_A,y_B$を手に入れたとする。このとき、Charlieは離散対数問題の難しさにより秘密鍵$a,b$を計算することが出来ない。よって鍵$K$を得ることが出来ず、安全である。

## DH鍵配送実装

```{code-cell}
from Crypto.Util import number
from random import randint

class DHE:
    def __init__(self,k):
        self.q = number.getPrime(k)
        #generator
        self.g=2
    
    def A_key(self):
        self.a = randint(1,self.q-1)
        return pow(self.g,self.a,self.q)
    
    def B_key(self):
        self.b = randint(1,self.q-1)
        return pow(self.g,self.b,self.q)
    
    def Share_key_A(self,yb):
        return pow(yb,self.a,self.q)
    
    def Share_key_B(self,ya):
        return pow(ya,self.b,self.q)
    

dhe = DHE(128)
a_key = dhe.A_key()
b_key = dhe.B_key()

#make shared key by Alice
s_key_A = dhe.Share_key_A(b_key)
#make shared key by Bob
s_key_B = dhe.Share_key_B(a_key)

print("secret key of Alice:",a_key)
print("secret key of Bob:",b_key)
print("Shared key of Alice:",s_key_A)
print("Shared key of Bob",s_key_B)

```

## DH問題
$(g,g^a,g^b)$から$g^{ab}\pmod{q}$を求める問題を**DH問題**と呼ぶ。  
もし、離散対数問題が解ければDH問題も解ける。しかし、その逆が成り立つかはわかっていない。
$\mathbf{Z}_p^*$の部分群および楕円曲線上に定義される群においては、DH問題を解くことが困難と予想されている。これを**DH仮定**と呼ぶ。