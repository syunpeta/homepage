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


+++

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
  
Euclid(423,258)
```
