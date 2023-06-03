# Self-Attention Complexity

## Attention

### Operation

假設 $X_{n\times p}$ 為輸入矩陣， n = sequence length，p = feature dimension，分別乘以 $W^Q$、$W^K$、$W^V$ 得到 $Q_{n\times d}$、$K_{n\times d}$、$V_{n\times d}$，以 n, p, d = 2, 4, 3 為例如，我們可以把 $Q$、$K$、$V$ 寫成
$$Q=\begin{bmatrix}q_1^\top\\q_2^\top\end{bmatrix},K=\begin{bmatrix}k_1^\top\\k_2^\top\end{bmatrix},V=\begin{bmatrix}v_1^\top\\v_2^\top\end{bmatrix},$$
其中 $q_i$、$k_i$、$v_i$ 為 $3\times 1$ 向量，代表每個 word 的 query、key 與 value。

![](https://i.imgur.com/bRfWXOn.png)

首先拿 query $q_i$ 分別與 key $k_j$, $j=1,...,n$ 去做內積，接著除以 $\sqrt{d}$ 之後過一層 softmax 得到 feature score $a_{i,j}$，Attention 運算輸出 $z_i$ 即為 n 個 feature scores $\{a_{i,j}\}_{j=1}^n$ 與 values $\{v_j\}_{j=1}^n$ 的線性組合
$$z_i=\sum_{j=1}^n a_{i,j} v_j$$
矩陣表示法如下圖

![](https://i.imgur.com/jZV5uk5.png)

改寫一下對照比較有感
$$Attention(Q,K,V)=softmax(\frac{QK^\top}{\sqrt d})V\\=softmax(\frac{\begin{bmatrix}q_1^\top\\q_2^\top\end{bmatrix}\begin{bmatrix}k_1&k_2\end{bmatrix}}{\sqrt d})V\\=softmax(\frac{\begin{bmatrix}q_1^\top k_1&q_1^\top k_2\\q_2^\top k_1&q_2^\top k_2\end{bmatrix}}{\sqrt d})\begin{bmatrix}v_1^\top\\v_2^\top\end{bmatrix}\\=\begin{bmatrix}a_{1,1}&a_{1,2}\\a_{2,1}&a_{2,2}\end{bmatrix}\begin{bmatrix}v_1^\top\\v_2^\top\end{bmatrix}\\=\begin{bmatrix}a_{1,1}v_1^\top+a_{1,2}v_2^\top\\a_{2,1}v_1^\top+a_{2,2}v_2^\top\end{bmatrix}$$

### Complexity

:::info
矩陣$M^1_{a\times b}$、$M^2_{b\times c}$相乘運算的複雜度為 $a\times b\times c$
:::
$$Attention(Q,K,V)=softmax(\frac{QK^\top}{\sqrt d})V$$

- $Q_{n\times d}\times(K^\top)_{d\times n}:n\times d\times n=n^2d$
- 除以 $\sqrt d$ 與 softmax 運算 $:n\times n$
- $softmax(.)_{n\times n}\times V_{n\times d}:n\times n\times d=n^2d$
- 最後會再過一個 activation $:n\times d$

|                      | Ops    | Activations |
| -------------------- | ------ | ----------- |
| Attention (dot-prod) | $n^2d$ | $n^2+nd$    |

## Multi-Head Attention

### operation

$Q$、$K$、$V$ 各別先過一層$d\times \frac{d}{h}$ 的矩陣 $W_i^Q$、$W_i^K$、$W_i^V$ 變成 $\frac{d}{h}$ 維，$i=1,...,h$，再進行 Attention 運算，最後 concat 起來

$$MultiHead(Q,K,V)=Concat(head_1,...,head_h)\\
where\quad head_i=Attention(QW_i^Q,KW_i^K,VW_i^V)$$

### complexity

- h 是 constant，忽略不計
- $Q\times W_i^Q:n\times d\times \frac{d}{h}$ (有 3h 個)
- $head_i$ 矩陣乘積 $:n^2\frac{d}{h}$ (有 h 個)
- $head_i$ softmax $:n^2$ (有 h 個)
- 最後的 activation $:n\times d$

|                      | Ops         | Activations |
| -------------------- | ----------- | ----------- |
| Multi-Head Attention | $n^2d+nd^2$ | $n^2+nd$    |

## Reference

- <http://www.chenjianqu.com/show-47.html>
- <https://zhuanlan.zhihu.com/p/264749298>
- <https://www.reddit.com/r/LanguageTechnology/comments/9gulm9/complexity_of_transformer_attention_network/>
