# On The Variance Of The Adaptive Learning Rate And Beyond

作者：Liyuan Liu, Haoming Jiang, Pengcheng He, Weizhu Chen, Xiaodong Liu, Jianfeng Gao, Jiawei Han

## 貢獻

這篇論文

1. 點出了 adaptive learning rate (Adam 為例) 的問題：在訓練早期階段 Adam 的 variance 會很大，如此不穩定造成 loss 下降不如預期。
2. 提出 Rectified Adam (RAdam) 來矯正 variance 的發散，並具有理論解釋與實驗支持。
3. 為 warm up 技巧為什麼會 work 提出解釋。

## 介紹

Adam 與 SGD 是目前主流的 optimizer，但有時候可能會收斂到 local 解而不是最佳解， 為了改善這個問題大家會使用 warm up 的技巧，也就是在訓練初期使用較小的 learning rate 並逐步攀升，接著再回復正常的下降。這個方法確實能夠使收斂更加穩定，但目前還沒有理論的支持，而且也沒有明確的指引如何做 warm up，使得我們必須用不同設定去嘗試。

作者透過假設 gradients 的分配來近似出 adaptive learning rate 的 variance，證明它是一個遞減的函數並用來矯正 adaptive learning rate (RAdam)。

## adaptive learning rate

![](https://i.imgur.com/GpKctmN.png)

上圖是 Generic adaptive optimization method setup，以 Adam 來說，

$$\phi_t(g_1,...,g_t)=\frac{(1-\beta_1)\sum ^t_{i=1}\beta_1^{t-i}g_i}{1-\beta_1^t}$$
$$\psi_t(g_1,...,g_t)=\sqrt{\frac{1-\beta_2^t}{(1-\beta_2)\sum ^t_{i=1}\beta_2^{t-i}g_i^2}}$$

因為 momentum 是對 $g$ 做 exponential moving average (EMA)，而 adaptive lr 是對 $g^2$ 做 EMA 然後倒數開根號，而($1-\beta^t$) 項是為了讓 weight 合為 1。

## variance of $\psi(g_1,...,g_t)$

#### scaled inverse chi-square distribution

If $X=1/S^2$ where $S^2=\frac{1}{\nu}\sum^\nu_1Z_i^2$ and $Z_i\sim N(0,\sigma^2)$，then $X\sim scale-inv-\chi^2(\nu,\tau^2)$，$\tau^2=1/\sigma^2$。

* $f(x;\nu,\tau^2)=\frac{(\tau^2\nu/2)^{\nu/2}}{\Gamma(\nu/2)}\frac{exp[\frac{-\nu\tau^2}{2x}]}{x^{1+\nu/2}}$
* $E[X]=\frac{\nu\tau^2}{\nu-2}$ for $\nu>2$
* $Var[X]=\frac{2\nu^2\tau^4}{(\nu-2)^2(\nu-4)}$ for $\nu>2$

### intuition

當 t=1，$\psi_1(g_1)=\sqrt{\frac{1}{g_1^2}}$，假設 $\{g_i\}_{i=1}^t\sim N(0,\sigma^2)$，則 $\frac{1}{g_1^2}\sim sale-inv-\chi^2(1,1/\sigma^2)$，因此 $Var[\sqrt{\frac{1}{g_1^2}}]$ 發散。

由上述的推論作者提出本論文的核心假設：
:::success
Due to the lack of samples in the early stage, the adaptive learning rate has an undesirably large variance, which leads to suspicious/bad local optima.
:::
藉由兩個 Adam 的變形 --- Adam-2k 與 Adam-eps 驗證初期的 variance 發散確實影響到最佳化表現。
![](https://i.imgur.com/iSmhObB.png)

### distribution of adaptive learning rate

有了一些直覺之後，考慮 t>1 的情況。雖然 $\psi_t^2(.)=\frac{1-\beta_2^t}{(1-\beta_2)\sum ^t_{i=1}\beta_2^{t-i}g_i^2}$ 是藉由 EMA 而不是直接平均 $g_i^2$ 得到 (simple moving average, SMA)，但是在訓練初期 t 不大的時候，exponential weights 之間的差距相對較小 (up to $1-\beta_2^{t-1}$)，我們用 SMA 來近似 EMA，又因為$\{g_i\}_{i=1}^t\sim N(0,\sigma^2)$，

$\frac{t}{\sum ^t_{i=1}g_i^2}\sim sale-inv-\chi^2(t,1/\sigma^2)$，

所以我們可以假設

$$\psi^2_t(.)=\frac{1-\beta_2^t}{(1-\beta_2)\sum ^t_{i=1}\beta_2^{t-i}g_i^2}\sim scale-inv-\chi^2(\rho,1/\sigma^2)$$ for some $\rho$。

---

![](https://i.imgur.com/iTsKsLO.png)

---

有了這個定理，只要我們得到 $\rho$ 便可以量化 $Var[\psi_t(.)]$，接下來的任務就是估計 $\rho$。

### estimation of $\rho$

實務中，EMA 可以表現為 SMA 的近似 (Robert Nau. Forecasting with moving averages. 2014)，

$$p(\frac{(1-\beta_2)\sum ^t_{i=1}\beta_2^{t-i}g_i^2}{1-\beta_2^t})\approx p(\frac{\sum ^{f(t,\beta_2)}_{i=1}g_{t+1-i}^2}{f(t,\beta_2)})$$

其中 $f(t,\beta_2)$ 使得 EMA 與 SMA 有相同的**質心**，也就是

$$\frac{(1-\beta_2)\sum ^t_{i=1}\beta_2^{t-i}i}{1-\beta_2^t}=\frac{\sum ^{f(t,\beta_2)}_{i=1}(t+1-i)}{f(t,\beta_2)},$$

求解方程式得到

$$f(t,\beta_2)=\frac{2}{1-\beta_2}-1-\frac{2t\beta_2^t}{1-\beta_2^t}$$

並將 $f(t,\beta_2)$ 視為 $\rho$ 在 t 時的估計量 $\rho_t$，$\rho_{\infty}=\lim_{t\to\infty}f(t,\beta_2)=\frac{2}{1-\beta_2}-1$。

### rectify the variance (RAdam)

因為 Theorem 1. 我們可以估計出 $Var[\psi_t(.)]=\tau^2(\frac{\rho_t}{\rho_t-2}-\frac{\rho_t2^{2\rho_t-5}}{\pi}B(\frac{\rho_t-1}{2},\frac{\rho_t-1}{2})^2)$，

又因為 $Var[\psi_t(.)]$ 單調遞減，可知最小值發生在 $\rho_{\infty}$，定義

$$C_{var}:=Var[\psi_t(.)]|_{\rho_t=\rho_{\infty}}$$

如此一來就可以利用 $C_{var}$ 來矯正 $Var[\psi_t(.)]$：

$$Var[r_t\psi_t(.)]=C_{var}$$ where $r_t=\sqrt{\frac{C_{var}}{Var[\psi_t(.)]}}$。

因為數值計算穩定，作者使用 first-order 展開近似 $Var[\psi_t(.)]$：

$$\sqrt{\psi^2(.)}\approx\sqrt{E[\psi^2(.)]}+\frac{1}{2E[\psi^2(.)]}(\psi^2(.)-E[\psi^2(.)])，$$
兩邊取 Var 可得

$$Var[\psi_t(.)]\approx\frac{Var[\psi_t^2(.)]}{4E[\psi^2(.)]}=\frac{\rho_t}{2(\rho_t-2)(\rho_t-4)\sigma^2}=O(\frac{1}{\rho_t})。$$

因此 $$r_t=\sqrt{\frac{(\rho_t-4)(\rho_t-2)\rho_{\infty}}{(\rho_{\infty}-4)(\rho_{\infty}-2)\rho_t}}，$$實務上如果 $\rho_t<4$ 則不執行 adaptive learning rate。

作者將這個演算法命名為 Rectified Adam (RAdam)。

![](https://i.imgur.com/nNJUHNz.png)

:::info
可以把 Warm up 也可以看成是 $\frac{min(t,T)}{T}$，$T$ 是暖身時長，如此與 $r_t$ 有異曲同工之妙，但 RAdam 不需調參數 $T$。
:::

## 實驗

### 與 vanilla Adam, SGD 比較

![](https://i.imgur.com/U3A7ZoU.png)

* RAdam 較不受不同 learning rate 影響，更加穩健

### 驗證兩個假設

![](https://i.imgur.com/JW5tJrR.png)

#### Figure 8

* 近似誤差在尺度上相對低

#### Figure 9

模擬 $\{g_i\}_{i=1}^t\sim N(\mu,1)$

* 不論 $\mu$ 如何變化都可以看到 $r_t$ 穩定了 variance
