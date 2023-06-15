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

# 楕円曲線暗号（その2）

## 楕円曲線上の離散対数問題
ある楕円曲線$E(F_p)$上の有理点$P$を考える．（この基準にする点$P$のことをベースポイントと呼ぶ．）今，$P$の整数倍$2P,3P,4P...$を計算していくと，いずれは$nP=O$となり無限遠点$O$たどり着く．そこで$\langle P \rangle=\{P,2P,3P...,O\}$を$P$によって生成される巡回群（$E(F_p)$の巡回部分群）であるとすると，任意の要素$ Q \in \langle P \rangle$に対して

$$
Q = aP
$$
を満たす$a\in \{0,...,n-1\}$がただ一つ存在する．この$a$を離散対数と呼ぶ．
今，$a,P$から$Q$を計算することは楕円曲線その1で紹介した楕円加算を用いれば簡単であることがわかる．しかし，$Q$から$a$を探し出すことは難しい．**楕円曲線上の離散対数問題問題** はこの性質を利用して，「$P,Q$が与えられたとき，離散対数$a$を求める問題」と定義される．  
$P$の位数に巨大な素数が含まれるとき，この問題を準指数時間で解く方法は現時点では見つかっていない．つまりこれは完全指数時間を要する困難な問題であると言える．

--- 

## ECDH
Diffie-Hellman(DH)の鍵配送法は有限体上の離散対数問題を用いたものだった．
これを楕円曲線上の離散対数問題に拡張したものがECDH方式である．

素数位数$p$と使用する楕円曲線$E(F_p)$，ベースポイント$P\in E(F_p)$をあらかじめ決めておく． 

- 鍵生成
1. AliceとBobはそれぞれ乱数$a,b$を選んで自身の秘密鍵とする．
2. Aliceは$A=aP$を計算しこれを公開する
3. Bobは$B=bP$を計算しこれを公開する

- 鍵共有
1. Alice は$K_A=aB=abP$を計算する．
2. 同様に$K_B=aA=abP$を計算する．

$K=K_A=K_B=abP$とするとAliceとBobは鍵を共有できる．  
もし攻撃者が$A,B$を知ったとする．このとき，離散対数問題の難しさから$a,b$を求めることは難しく，共有された鍵$K$を求めることはできない．  
このプロトコルを用いて以下の二つの問題を定義することもある．

- ECCDH問題：$A=aP,B=bP$から$abP$を求める問題．
- ECDDH問題：$A=aP,B=bP$から任意に選んだ楕円曲線上の点$Q\in E(F_p)$が$abP$に等しいか判定する問題．

## ECDH 実装
楕円曲線暗号その１と同様にSagemath上でECDHを実装した．  
前に定義したECCクラスとECC_pointクラスをそのまま使用しているので，実行の際は各自で補ってほしい．実際に使用する際は位数の大きい楕円曲線とベースポイントを用いることが推奨されている．  
仮想通貨などで実際に使用されるパラメータとしては[secp256k1](https://en.bitcoin.it/wiki/Secp256k1)などがあり，今回はこれを利用している．

```python
from random import randint

class ECDH:
    def __init__(self,P,n):
        #base point
        self.P = P
        #order
        self.n = n
    
    def A_key(self):
        self.a = randint(1,self.n-1)
        return P*self.a
    
    def B_key(self):
        self.b = randint(1,self.n-1)
        return P*self.b
    
    def Share_key_A(self,yb):
        return yb*self.a
    
    def Share_key_B(self,ya):
        return ya*self.b


q = ZZ('0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F')

EC = ECC(0,7,q)
P = ECC_point(EC,55066263022277343669578718895168534326250603453777594175500187360389116729240,32670510020758816978083085130507043184471273380659243275938904335757337482424,False) 
n = ZZ('0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141')#order

ecdh = ECDH(P,n)
A = ecdh.A_key()
B = ecdh.B_key()
K_A = ecdh.Share_key_A(B)
K_B = ecdh.Share_key_B(A)


if K_A.x == K_B.x and K_A.y == K_B.y:
    print("Success, K is :",K_A)

```
--- 

## ECElGamal暗号
上と同様に，ElGamal暗号に対しても楕円曲線を用いることができる．  

素数位数$p$と使用する楕円曲線$E(F_p)$，ベースポイント$P\in E(F_p)$をあらかじめ決めておく． 
- 鍵生成
1. Bobは乱数$b$を選び，$B=bP$を計算し，公開鍵を$(E,F_p,P,B)$とする．

- 暗号化
1. Aliceは乱数$a$を選び$A=aP$を計算する．
2. メッセージ$M$を楕円曲線上の点mに対応付け，$C=m+aB$ を計算して暗号文$(A,C)$をBobに送る．

- 復号化
1. Bobは$(A,C)$を受け取り，$bA=abP$を計算する．
2. $m=C-bA$で復号する．

復号化の正当性は以下の通りである．

$$
C-bA = m+aB-bA = m + abP -abP = m
$$

3. 楕円曲線上の点$m$からメッセージ$M$を復元する．

## ECElGamal暗号実装
実装中...

---
## ECDSA
ECDSAはElliptic Curve Digital Signature Algorithmの略で楕円曲線を用いた署名方式である．
Aliceが自分の公開鍵をつかってディジタル署名を行う際は以下のように行う．  

素数位数$p$と使用する楕円曲線$E(F_p)$，素位数$l$の巡回群の生成元$P\in E(F_p)$をあらかじめ決めておく．

- 鍵生成
1. Aliceは乱数$a\in F_l^*$を選び秘密鍵とする．
2. Alice は$A=aP$を計算して自分の公開鍵とする．

- 署名
1. メッセージ$m$とハッシュ関数$h(x)，乱数$t \in F_l^*$を用いて署名文$\{m,T,s\}$を以下のように計算する．

$$
T = tP = (x,y),\quad s = t^{-1}(h(m)+ax)\mod l
$$

- 検証
1. $m,s,T,A$を用いて以下の計算を行う． 
$$
u = s^{-1}h(m)\mod l,\quad v = s^{-1}(h(m)+ax)\mod l
$$

2. $uP + vA$が$T$と一致するかを検証する．


正当な署名ならば

$$
uP + vA = s^{-1}h(m)P + s^{-1}xA = s^{-1}(h(m) + ax)P = tP=T
$$

となるため$T$と一致するか否かを見ることで署名の正当性を確認できる．

## ECDSA実装
実装中...





