## BDH Problem

$\mathbb{G}_1,\mathbb{G}_2$を素数位数$q$の群とし、$\hat{e} : \mathbb{G}_1 \times \mathbb{G}_2$.また、$P$を$\mathbb{G}_1$の生成元とする。$\langle \mathbb{G}_1,\mathbb{G}_2,\hat{e}\rangle$でのBDH問題とは、以下の通りである。

$a,b,c \in \mathbb{Z}_q^*$に対して$\langle P,aP,bP,cP\rangle$が与えられたとき、$W=\hat{e}(P,P)^{abc}\in \mathbb{G}_2$を計算する問題


---

## BDH Assumption
randomized algorithm $\mathcal{G}$がパラメータ生成機であるとする。巨大な$k$についてアルゴリズム$\mathcal{A}$がBDH問題をアドバンテージ$\epsilon(k)$で解けるとき、以下のように書ける。

$$
\begin{equation}
\nonumber
Adv_{\mathcal{G},\mathcal{A}(k)} = \left[\mathcal{A}(q,\mathbb{G}_1,\mathbb{G}_2,\hat{e},P,aP,bP,cP)=\hat{e}(P,P)^{abc} |

\begin{aligned}
&\langle q,\mathbb{G}_1,\mathbb{G}_2,\hat{e}\rangle \leftarrow \mathcal{G}\\
&P \leftarrow \mathbb{G}_1,\space a,b,c \leftarrow \mathbb{Z}_q^*
\end{aligned} 
\right]\geq \epsilon(k)
\end{equation}
$$
どんな多項式アルゴリズム$\mathcal{A}$を持っていたとしても、上の確率$Adv_{\mathcal{G},\mathcal{A(k)}}$が無視できる確率のとき、$\mathcal{G}$をBDH仮定を満たすという。つまり、BDH仮定とはBDH問題を解く効率的な多項式時間アルゴリズムが存在しない仮定である。

---

## BDH仮定の使い方
BasicIndentと呼ばれる単純なIBEシステムを紹介する。
このシステムはSetup,Extract,Encrypt,Decryptの4つのアルゴリズムから構成される。

- Setup:セキュリティパラメータを$k\in \mathbb{Z}^+$とする

1. BDHパラメータ生成機$\mathcal{G}$を使い素数$q$と位数$q$の群$\mathbb{G}_1,\mathbb{G}_2$、双線形写像$\hat{e}:\mathbb{G}_1\times\mathbb{G}_2\rightarrow\mathbb{G}_2$を生成する。そして、$P\in\mathbb{G}$を選択する。
2. ランダムな$s\in\mathbb{Z}_q^*$を選択し、$P_pub = sP$とする。
3. ハッシュ関数$H_1 :\{0,1\}^* \rightarrow \mathbb{G}_1^*$




