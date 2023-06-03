# ALE Introduction

給定一個訓練完畢的黑盒子模型，Accumulated Local Effects (ALE) plot 能夠描述特徵如何影響模型的預測值，並且不受特徵間相關性影響，因此被認為是比 Partial Dependence PLot (PDP) 更好的模型解釋工具。
在介紹 ALE 之前需要先了解一下 PDP 與它的問題。

## Partial Dependence Plot

[詳細介紹看我](https://christophm.github.io/interpretable-ml-book/pdp.html)
PDP 能夠呈現一個特徵對於模型預測值的 marginal effect，也就是說維持其他特徵不變的情況下，改變某一個特徵會對模型預測值有什麼影響。
![](https://i.imgur.com/gxW30Xu.png)
以 california housing dataset 為例，特徵 AveOccup 的 PDP (上圖左) 是一條線，橫軸為所有可能的值 (約 2~5)，縱軸是 partial dependence，以 AveOccup=3 為例, 將資料集每個 AveOccup 都改成 3，帶入模型輸出 #(sample) 個預測值並取平均，即可得到 partial dependence=-0.1。

- PDP 也可以畫兩個特徵，此時兩個軸對應兩個特徵，partial dependence 以顏色表現  (上圖右)
- 除了數值型特徵，ALE 也可以使用在類別型特徵。
- 最好將特徵的分布考慮進去 (上圖黑色 bar)

### PDP 的優缺點

- 優
  1. 直覺
  2. 容易實現
- 缺
  1. 假如兩個特徵具有相關性，修改單一特徵會產生出不符常理的資料點 (例如身高兩公尺卻只有五十公斤的人)
  2. 平均化輸出可能會抹滅資料的異質性 (假如隨著特徵變化，一半的樣本上升，一半下降，最後可能得到一個水平線)

## Accumulated Local Effects Plot

[詳細介紹看我](https://christophm.github.io/interpretable-ml-book/ale.html)
首先詳細說明 PDP 相關性的問題，假設 $x_1$, $x_2$ 兩個特徵具有高度相關性，如下圖，當我們在畫 $x_1$ 的 PDP 時，以 $x_1=0.75$ 為例，因為 PDP 計算所有的樣本，造成 $x_2$ 的分佈如下圖左，但這並不合理，$x_1=0.75$ 時合理的分布應該如下圖右。
一個較為合理的做法是只考慮 0.75 附近的樣本，這個做法叫做 M-plot，但 M-plot 也有它的問題：你不知道測量到的 effect 是來自 $x_1$ 還是 $x_2$。
![](https://i.imgur.com/iPzFKPR.png)
ALE 的做法是將 $x_1$ 切成一段一段的 interval (實務上可用 quantile 來切)，如下圖虛線；以 N(4) 這個區間為例，收集在區間內的點，各別去計算令 $x_1=z_4$ 得到的輸出減掉 $x_1=z_3$ 得到的輸出，接著去算這些差距的平均，就得到 $x_1$ 在區間內的 effect，ALE 的估計即為區間 effect 的向下累積。
![](https://i.imgur.com/JQUNKVv.png)
公式長這樣，中心化後，平均為 0：
$$
\hat{\tilde{f}}_{j,ALE}(x)=\sum_{k=1}^{k_j(x)}\frac{1}{n_j(k)}\sum_{i:x_{j}^{(i)}\in{}N_j(k)}\left[f(z_{k,j},x^{(i)}_{\setminus{}j})-f(z_{k-1,j},x^{(i)}_{\setminus{}j})\right]\\
\hat{f}_{j,ALE}(x)=\hat{\tilde{f}}_{j,ALE}(x)-\frac{1}{n}\sum_{i=1}^{n}\hat{\tilde{f}}_{j,ALE}(x^{(i)}_{j})
$$
其中 $j$ 代表第 $j$ 個特徵，$\backslash j$ 就是排除第 $j$ 個特徵；$k_j(x)$ 代表 $x$ 所在的區間編號，$N_j(k)$ 是區間內的資料點集合。
![](https://i.imgur.com/ZPI4Rfs.png)
上圖使用 10 百分位切分區間，因為是一個區間輸出一個估計值，所以點會畫在區間中間；淺藍色的線是使用蒙地卡羅法重複抽樣繪出，提供誤差參考；圖片下緣為資料分布狀況，因此較為稀疏的地方看看就好。

### 類別型特徵

因為 ALE 是基於區間兩頭的差距來估計 effect，有序型類別變數可以繪製 ALE，無序型類別型就不太適合，如果一定要的話可以嘗試計算 label 間的距離然後使用 [multidimensional scaling](https://en.wikipedia.org/wiki/Multidimensional_scaling) 轉換成一維數值型變數。

### 兩特徵間交互作用 (2-D ALE)

兩維的 ALE 考慮四個邊界，計算 (d-c) - (b-a)，得到的將會是單純交互作用的 effect，而沒有 $x_1$, $x_2$ 的 effect，因此判讀時需要特別小心。
假如兩變數間無交互作用，ALE 會是一片平坦。
![](https://i.imgur.com/OVkYnvo.png)

### ALE 的優缺點 (補)

- 優
  1. 即使變數間有相關性也能得到無偏的估計
  2. 速度快
  3. 解釋明確
- 缺
  1. 沒有完美的區間數目，需要在 smoothness 跟 complexity 之間做取捨
  2. 無法呈現資料異質性
  3. 2-D ALE 解釋不直觀

## Python 實作

[Examples in Colab](https://colab.research.google.com/drive/1f8zGpgYbm5jBfLeFunwyiMYIWzQYfmSf?usp=sharing)

- [ALEPython](https://github.com/blent-ai/ALEPython)
  比較早的實作，repo 已經過一年沒有更新了
  - 使用

    ```python
    import alepython
    alepython.ale_plot(
        model, X, ['RM'], bins=20,
        monte_carlo=True, monte_carlo_rep=50, monte_carlo_ratio=0.6
    )
    ```

    ![](https://i.imgur.com/CrJaEGm.png)

  - 特色
    - 有蒙地卡羅抽樣，反映可信程度
    - 2-D 是用等高線圖呈現
    - 圖形控制不容易

- [PyALE](https://github.com/DanaJomar/PyALE)
  - 使用

    ```python
    import PyALE
    ale_eff = PyALE.ale(
        X=X, model=model, feature=["RM"],
        grid_size=20, include_CI=True, C=0.95
    )
    ```

    ![](https://i.imgur.com/GtwWw2O.png)

  - 特色
    - 有呈現出信賴區間，而不是只取平均
    - 離散的圖還滿漂亮的，比直接當成數值型更make sense，但還是不建議無序型類別使用
    - 2-D 是以 heatmap 呈現
    - 有提供 fig, ax 輸入，容易繪製 subplot

- [Alibi Explain](https://docs.seldon.io/projects/alibi/en/latest/index.html)
  一個解釋工具的組合包
  - 使用

    ```python
    import alibi.explainers as ae
    ale_obj = ae.ALE(
        model.predict, feature_names=X.columns, target_names=[y.name]
    )
    ale_exp = ale_obj.explain(
        X.values, min_bin_points=len(X)//20
    )
    ae.plot_ale(
        ale_exp, features=['RM'], fig_kw={'figwidth':8, 'figheight': 4}
    )
    ```

    ![](https://i.imgur.com/G1OKQN4.png)

  - 特色
    - 使用 min_bin_point 做區間控制，而不是 bins
    - 可以一次畫所有 features，而且 Y 軸是固定的，比較很方便
    - 沒有 2-D 圖
    - 除了 ALE 還有其他工具

## 參考資料

<https://arxiv.org/abs/1612.08468>
<https://christophm.github.io/interpretable-ml-book/ale.html>
<https://christophm.github.io/interpretable-ml-book/pdp.html>
