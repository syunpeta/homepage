

# 耐量子暗号(格子暗号2)
## Regev暗号
Regev暗号はRegevが2005年に提案した暗号で，LWE問題をベースに作られた公開鍵暗号となっている。以下でそのアルゴリズムおよび実装の紹介を行う。
### 前提

- $n\in \mathbb{Z}$  

- 素数$n^2 \leq q \leq 2n^2 $  
- 任意の定数$\epsilon > 0$
- $m = (1+\epsilon)(n+1)\log q$(実装では$m=1.1n\log qとした$)
- ノイズパラメータ a > 0  (実装では$a=1/\sqrt n\log^2 n とした$)

を選択する。
### 鍵生成
秘密鍵：$sk\in \mathbb{Z}_q^n$を一様ランダムに選択する。  
公開鍵： $i=1,\dots ,m$について$m$個のベクトル$\mathbf{a_1,a_2,\dots,a_m\in \mathbb{Z_q^n}}$を一様ランダムに選択し行列$A= (\mathbf{a_i})^m_{i=1}\in \mathbb{Z}_q^{n\times m}$とする。同時に平均0,標準偏差$a$の離散正規分布$\chi$から誤差$e_1,e_2\dots,e_m\in\mathbb{Z_q}$を選択する。  

公開鍵$pk$は$b = sk\cdot A + e \in \mathbb{Z}_q^m$を用いて$(A,b)$とする。

### 暗号化
メッセージ$M\in \{0,1\}$を選択する．乱数列$r\leftarrow\{0,1\}^m$をランダムに生成する．  
暗号文$c,c'$を

$$
\begin{equation}
c = A\cdot r, ~~~~~~~~~~c' = b\cdot r + M\lfloor \frac{q}{2}\rfloor　
\end{equation}
$$

として受信者に送信する．

### 復号化
復号分$d$を以下のように計算する。
$$
\begin{equation}
d = c' - sk\cdot c\mod q
\end{equation}
$$

$d\approx\lfloor\frac{q}{2}\rfloor$ならば1を出力する。それ以外ならば0を出力する。

### 正当性
復号化において、$d$を展開すると以下のように計算できる。

$$
\begin{equation}
\begin{aligned}
d &= c' - sk\cdot c\mod q\\
&= b\cdot r + M\lfloor \frac{q}{2}\rfloor  -sk\cdot A\cdot r\mod q\\
&= (sk\cdot A +e)\cdot r + M\lfloor \frac{q}{2}\rfloor -sk\cdot A \cdot r\mod q\\
&=e\cdot r + M\lfloor \frac{q}{2} \rfloor\mod q\\ 
\end{aligned}
\end{equation}
$$

ここで、$e\cdot r$は誤差ベクトルと$\{0,1\}$の乱数ベクトルの内積のため非常に小さく、$d$の値は$M\lfloor \frac{q}{2}\rfloor$に依存する。よって$d$が$\lfloor \frac{q}{2}\rfloor$に近いかどうかで$M$の値がわかる。

## Regev暗号　実装
Pythonを用いてRegev暗号を実装した。
以下がソースコードとなる。

```python
import numpy as np
import math

N=230
Q =52901

class Regev():
    def __init__(self,n,q):
        self.n = n
        self.q = q
        self.m = int((1.1)*self.n*math.log(self.q,2))
        self.a = 1/(math.sqrt(self.n)*math.log(self.n,2)**2)
    
    def keygen(self):
        #secret key sk
        self.sk = np.matrix(np.random.randint(0,self.q,self.n))
        #matrix A
        A=np.matrix(np.random.randint(0,self.q,size=(self.n,self.m)))
        
        #error e
        e = np.matrix(np.random.normal(0,self.a,size=self.m))
        #make B
        
        b = (self.sk*A+e)%self.q
        
        return A,b
    
    def encrypt(self,A,b,M):
        
        #random r
        r = np.matrix(np.random.randint(0,2,size=self.m))
        
        #c
        c = np.dot(A,r.transpose())
        #c'
        c_p = (np.dot(b,r.transpose())+M*math.floor(self.q/2))
       
        return c,c_p
        
    def decrypt(self,c,c_p):
        
        #dec d
        d = (c_p - np.dot(self.sk,c))%self.q
        if (d>(self.q/4)) and ( d <3*self.q/4):
            return 1
        else:
            return 0



#main
M=1
tmp = Regev(N,Q)
A,B = tmp.keygen()
C,C_P=tmp.encrypt(A,B,M)
ans = tmp.decrypt(C,C_P)
print(ans)


```


## NTRU暗号
前ぺージでも紹介した格子暗号の一種であるNTRU暗号のアルゴリズムと実装を原著論文にしたがって以下で紹介する．

### 前提
- パラメータ$n ,p ,q\in \mathbb{N}~(gcd(p,q)=1, p<< q)$を設定し、多項式環$R = \mathbb{Z}[x]/(x^n -1)$定義する。 $R$の元$F\in R$は$F=\sum_{i=0}^{n-1} F_i X^i$で表す。  

- $\mathcal{L}(d_1,d_2) $を以下のように定義する。

$$
\begin{equation}
\mathcal{L}(d_1,d_2)  = \left\{ F\in R:
\begin{aligned}
Fはd_1個の係数1を持つ\\
Fはd_2個の係数-1を持つ\\
残りの係数を0とする。
\end{aligned}
\right \}
\end{equation}
$$

- $f,g \in R$に対して$f\star g$は$f$と$g$の畳み込み積を表し、以下で定義される。

$$

\begin{equation}
f\star g = \sum_{i+j = k \mod n} f_i g_i 
\end{equation}
$$

### 鍵生成
受信者は$d_f,d_g \in \mathbb{N}$を設定する。多項式$g\in \mathcal L(d_g,g_g), f\in \mathcal(d_f,d_f-1)$をランダムに生成する。ただし、$f$は以下の式を満たす逆元$f_p,f_q$を持つように生成する。

$$
\begin{equation}
f\star f_q = 1 \mod q,~~~~f\star f_p = 1 \mod p 
\end{equation}
$$

以上を用いて公開多項式$h$を以下のように生成しする。

$$
\begin{equation}
h = f_q \star g \mod q 
\end{equation}
$$

このとき、$(f,fq)$を秘密鍵、$h$を公開鍵とする。


### 暗号化
送信者は平文$m$を用意する。ただし、係数が$-1,0,1$のみの多項式とする。  
$d_\phi \in \mathbb{N}$を設定し、ランダム多項式$\phi\in \mathcal L(d_\phi,d_\phi)$を生成する。これを用いて、暗号文$e$を以下のように計算する。

$$
\begin{equation}
e = p\cdot h \star \phi  + m \mod q
\end{equation}

$$
 
できた暗号文$e$を受信者に送信する。
### 復号化
暗号文$e$と秘密鍵$f$を用いて以下の多項式$a$を生成する。ただし、復号化の中では、mod計算は通常と異なり、$(\frac{-p}{2},\frac{p}{2}]$,$(\frac{-q}{2},\frac{q}{2}]$の範囲で計算する。 

$$
\begin{equation}
a = f\star e \mod q
\end{equation}

$$

その後、以下の計算により復号された平文$m'$が取得できる。

$$
\begin{equation}
m' = f_p \star a \mod p
\end{equation}

$$

### 正当性
上を踏まえて、多項式$a$を展開すると次のようになる。

$$
\begin{equation}
\begin{aligned}
a = f\star e &= f\star p\phi \star h + f\star m \mod q\\
 &= f\star p\phi\star f_q \star g + f\star m \mod q 
\end{aligned}
\end{equation}
$$

ここで、$f\star f_q = 1$より

$$
\begin{equation}
a = p\phi \star g + f \star m \mod q
\end{equation}
$$

この時点で適切なパラメータを選択していた場合は$a$の係数は$(\frac{-q}{2},\frac{q}{2}]$の範囲に収まるため、$\mod q $をそのまま外すことができる。  

つまり、

$$
\begin{equation}
a = p\phi \star g + f\star m
\end{equation}
$$

となる。これに$f_p$をかけ、$\mod p $をとると、

$$

\begin{equation}
\begin{aligned}
f_p \star a &= f_p \star p\phi \star g + f_p \star f \star m \mod p\\
&= pf_p \star\phi \star g + m \mod p\\
&= m\mod p\\
&= m 
\end{aligned}
\end{equation}
$$

となり平文が復元できることがわかる。

ただし、$|f|_{\infty} = max_{0\leq i \leq n-1}f_i - min_{0\leq i \leq n-1} f_i $
としないと、$a = p\phi \star g + f \star m \mod q$式が満たされず復号ができなくなる。
## NTRU 実装
以下では、NTRU暗号の実装を行う。Python(Sagemath)で実装するため、実装結果は[SageCell](https://sagecell.sagemath.org/)等で確認できる。


```python

from random import sample
from fpylll import *

#parameter###################
N=107
p=3
q=2
e=6
df = 15
dg = 12
dphi=5
R.<x> = ZZ[]
#############################

#tools ######################
def random_poly(NN,o,mo):
 
    s=[1]*o+[-1]*mo+[0]*(NN-o-mo)
    shuffle(s)
    return R(s)
def Convolution_in_R(ff,gg,NN):
    return (ff*gg)%(x^NN-1)

def Convolution_in_R_p(ff,gg,NN,pp):
    hh = (ff*gg)%(x^NN-1)
    h1 = list(   (hh[i]%pp)   for i in range(NN)   )
    return  R(h1)  

def Invertmodprime(ff,pp,NN):    
    T = R.change_ring(Integers(pp)).quotient(x^NN-1)
    return R(lift(1/T(ff)))

def Invertmodpowerofprime(ff,qq,ee,NN): 
    q_e = qq^ee
    FF = Invertmodprime(ff,qq,NN)
    if ee == 1:      
        return FF
    nn = 2
    while ee>0:
        temp = Convolution_in_R(FF,ff,NN);
        FF = Convolution_in_R_p(FF,2-temp,NN,qq^nn);
        ee = floor(ee/2)
        nn *=2
    return FF%q_e

def change_ring_Rq(ff,qq,NN):
    T = R.change_ring(Integers(qq)).quotient(x^NN-1)
    return R(lift(T(ff)))

def center_lift(ff,pp):
    flist = ff.list()
    ph = pp//2
    for i in range(len(flist)):
        tmp = int(flist[i])%pp
        if tmp > ph:
            tmp -= pp
        flist[i] = tmp
    return R(flist)


#keygen#######################
def keygen(dff,dgg,qq,ee,NN):
    
    while True:
        f = random_poly(NN,dff,dff-1)
        g = random_poly(NN,dgg,dgg)
        try :
            fq = Invertmodpowerofprime(f,qq,ee,NN)
            break
        except:
            continue
    h = Convolution_in_R_p(fq,g,NN,qq^ee)
    return h,f,g

#encryption###################
def enc(m,qq,ee,d_phi,NN):
    phi = random_poly(NN,d_phi,d_phi)
    e_q = Convolution_in_R_p(h,p*phi,NN,qq^ee)
    e = e_q + m
    return e%(qq^ee)

#decryption##################
def dec(e_q,ff,qq,ee,pp,NN):
    a = Convolution_in_R_p(ff,e_q,NN,qq^ee)
    a_c = center_lift(a,qq^ee)
    fp = Invertmodprime(ff,pp,NN)
    b = Convolution_in_R_p(a_c,fp,NN,pp)
    b = center_lift(b,pp)
    return b

#main#######################
h,f,g = keygen(df,dg,q,e,N)
m = R([randint(-(p-1)/2,(p-1)/2) for i in range(N)])
e_m = enc(m,q,e,dphi,N)
dec_m = dec(e_m,f,q,e,p,N)
if dec_m == m:
    print("OK")
    print("m is :",m)
else:
    print("m is :",m)
    print("enc_m is :",e_m)
    print("Error")

```


```python

OK
m is : -x^106 - x^105 - x^103 + x^101 - x^98 + x^96 - x^95 - x^94 + x^93 - x^92 - x^88 + x^87 - x^86 - x^85 - x^81 + x^79 - x^76 - x^75 + x^72 - x^70 - x^69 + x^68 + x^67 + x^66 + x^65 - x^64 + x^61 - x^59 + x^58 + x^57 - x^54 - x^52 + x^51 + x^50 + x^49 - x^48 - x^47 - x^45 + x^43 + x^42 + x^40 + x^39 + x^37 - x^36 + x^34 - x^33 - x^32 + x^29 + x^27 - x^26 - x^25 + x^24 + x^23 - x^20 + x^19 - x^18 + x^15 + x^14 - x^13 + x^12 - x^11 + x^10 - x^8 - x^7 - x^6 - x^5 - x^4 - x^2 + x - 1
```
