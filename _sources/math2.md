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

# 暗号に必要な数学2



## ユークリッドの互除法

$a$を$b$で割った余りを$r$とすると、$(a,b)$の最大公約数は$(b,r)$の最大公約数と等しいという性質を用いて以下のステップを行うことで2つの自然数の最大公約数を求める方法。  

1. $a$を$b$で割り、余りを$r$とする。
2. $a$を$b$とし、$b$を$r$とする。
3. 1,2ステップを繰り返す。
4. 余りが0になったらそのときの$b$(つまり除数)が最大公約数である。

以上をPythonで実装すると以下のようになる。

```{code-cell} 
#Euclid の互除法
def Euclid(a,b):
  r=a%b
  if r==0:
    return b
  else:
    return Euclid(b,r)
  
Euclid(768,320)
```

## 拡張ユークリッドの互除法

$ax + by = gcd(a,b) $ の不定方程式を解く。  
ユークリッドの互除法を用いて、逆算をすると


```math
#ユークリッドの互除法
768÷320 = 2 余り128
320÷128 = 2 余り64
128÷64=2 余り0
```

```math
#式変形
128 = 768 - 320*2
64 = 320 -128*2

```

```math
#逆算
320 - 128*2 = 64
320 - (768 - 320*2)*2 = 64
768*(-2) + 320*5 = 64
```

以上より$(x,y) = (-2,5)$と求めることが出来た。  
このようにして一次不定方程式の解を求めるアルゴリズムを**拡張ユークリッドの互除法**という。
拡張ユークリッドの互除法をアルゴリズムで表記すると以下のようになる。

1. $a$を$b$で割り、商とあまりを$q,r$とする。
2. $a=qb+r$とし、$ax+by=gcd(a,b)$に代入、$b(qx+y) + rx = gcd(a,b)$を得る
3. $s=qx+y,t = x$として$bs+rt=gcd(a,b)$についてステップ1から再帰的に繰り返す。
4. $b=0$となったら終了。$x,y$を返す。

以上をPythonで記述したものがこちら。

```{code-cell}
def Ex_Euclid(a,b):
  if b == 0:
    return 1,0,a
  else:
    q = a//b
    r = a%b
    s,t,c = Ex_Euclid(b,r)
    x = t
    y = s-q*t
    return x,y,c

Ex_Euclid(768,320)

```

## 拡張ユークリッド互除法の活用(逆元の求めかた)
拡張ユークリッドの互除法は$\mathbf{Z}_q$内の逆元を求める際に有効。
例えば，$5 \in \mathbf{Z}_7$の逆元$5^{-1}$を求めることを考える。

拡張ユークリッドの互除法を用いて，$7x+5y=1$の$x,y$を求めると

```{code-cell}
Ex_Euclid(7,5)
```
より$7*(-2) + 5*3 = 1$が得られる。  
ここで，$mod7$とすると$5*3=1 \pmod{7}$となり$5^{-1}$は$3$であるとわかる。  
このようにして，逆元を求めたい数と法を$(a,b)$として拡張ユークリッドの互除法を適応することで逆元を求めることができる。






