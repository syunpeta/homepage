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

# 楕円曲線暗号
RSA暗号やElGamal暗号は非常にわかりやすく使い勝手がいい暗号であるが，安全面では楕円曲線暗号のほうが優れている．楕円曲線暗号はRSAと同じ安全性をもつために必要な鍵長が短い点が大きな特徴である．また，その鍵長の短さゆえに計算も高速に行うことができる．楕円曲線暗号は楕円曲線上の有理点の演算によって成り立つ暗号である．  

## 楕円曲線
$p$を素数（ただし簡単のため$p\neq2,3$）とする．有限体$F(p)$上の楕円曲線$E$とは定数$a,b\in F(p)$を用いて

$$
\begin{equation}
y^2 = x^3 + ax + b \pmod{p}
\end{equation} 
$$

と表すことができる．(1)式を満たす$(x,y)\in \mathbb{Z}_p$の組からなる点$(x,y)$を**有理点** という．また，全ての有理点に**無限遠点** と呼ばれる$O(\infty,\infty)$を加えた集合を$E(F_p)$と表す．  

例として楕円曲線を二つプロットしてみる。
- $y^2 = x^3 -4x + 1$

```{code-cell}
import numpy as np
import matplotlib.pyplot as plt
import math

fig, ax = plt.subplots()
#x,y
delta = 0.1
xrange = np.arange(-10, 10, delta)
yrange = np.arange(-10, 10, delta)
x, y = np.meshgrid(xrange,yrange)
ax.axis([-10, 10, -10, 10])
#Eliptic curve
z = y**2 - x**3 +4*x -1
#plot
ax.contour(x,y,z,[0])
ax.set_xlabel('$x$')
ax.set_ylabel('$y$')
ax.grid()
plt.show()
```

- $y^2 = x^3 -2x + 4$

```{code-cell}
import numpy as np
import matplotlib.pyplot as plt
import math

fig, ax = plt.subplots()
#x,y
delta = 0.1
xrange = np.arange(-10, 10, delta)
yrange = np.arange(-10, 10, delta)
x, y = np.meshgrid(xrange,yrange)
ax.axis([-10, 10, -10, 10])
#Eliptic curve
z = y**2 - x**3 +2*x -4
#plot
ax.contour(x,y,z,[0])
ax.set_xlabel('$x$')
ax.set_ylabel('$y$')
ax.grid()
plt.show()
```

楕円曲線の形は大きく分けて上の二つのような形を取る。$y^2$からも明らかなように、$x$軸に対して対称な形をとる。
