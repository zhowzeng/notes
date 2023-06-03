# The Graph Neural Network Model

（這篇整理是把報 paper 的簡報貼過來）

paper: <https://persagen.com/files/misc/scarselli2009graph.pdf>

發表於 IEEE Transactions on Neural Networks, vol. 20, no. 1, pp. 61-80.

## Introduction

![](https://i.imgur.com/qAiQ84j.png)

首先介紹圖結構的問題是什麼樣子，有很多結構都可以用 graph 來表示。例如化學式，用 node 代表原子用 edge 代表鍵；或是這張城堡的圖畫，把顏色同質的區域點用一個點來代表它，在圖上是相鄰就用邊連接。

GNN 想解的問題就是產生一個函數，給你整張圖或圖跟一個點，能夠輸出實數，這兩種分別是：

- graph-focused: $\tau$ 跟單一 n 無關，整張圖出一個 output。
    例：化合物會造成某種疾病的機率 or 把這個圖分類成某個物件(城堡)
- node-focused: 跟 n 有關
    例：黑點代表是屬於城堡，把位置定位出來

![](https://i.imgur.com/sRyTBvf.png)

如何來解這個問題? 在這篇之前有這些方法:

- preprocessing
    把圖的結構mapping到比較容易的表示法，例如向量，人為定義出來的。
- RvNN
    以遞迴的方式計算出最終預測，由一個點當代表輸出預測，必須是無環。
- MC
    每個點當成一個狀態，想像從一個狀態到另一個狀態是服從某個機率（固定的），例如 1 這點可以跑到 2346，2 又可以跑到 19，隨機漫步就是想像有一個人在各個狀態游移，一定時間之後，漫步的人在每個狀態的機率會趨於穩定，拿那個機率來算 output（機率不一定是一個數值也可以是向量）。
- GNN
    每個點會更新一個狀態 x，互相交換資訊直到達成平衡，以 x 為基礎計算output ，這個機制受到一些constraint來確保平衡永遠存在。

RvNN跟RW可視為 GNN 的特例

## The model

![](https://i.imgur.com/OAxHJMB.png)

首先介紹一下 notation，graph 由 node 跟 edge 組成

- ne[n]
    與n相接的點
- co[n]
    與n相接的邊

點與邊都有 label，是他們本身的特徵，以城堡為例就像亮度、周長、面積等等。

之後講的圖都假設無向，有環，有向的話就把方向設定在邊的 label 裡。

![](https://i.imgur.com/BqobEG3.png)

GNN model 就是給每個點一個狀態 $x_n$，下標 n 代表點是 n 的狀態，由點 n 與點的周邊 label 來計算 (f，用 NN 來算，會有parameter omega)，再根據自己的 state 跟 label 輸出 output（g，用 NN）。

- f: transition function
- g: output function

小寫local大寫global，global就是所有點一起看(這邊假設每個node共享同樣的f跟g)，假設鄰居沒有順序或重要性分別的話 f 還可以改寫成 h (nonpositional)。

希望這個擴散機制達到平衡，也就是 $x_n$ 不再改變，需要假設 F 是一個 contraction mapping，contraction mapping 就是說兩個點在經過這個 mapping 之前之後會符合這個關係，可以想成限制導數的大小。

接著巴拿赫定理可以保證存在一個不動點 (使得怎麼計算都維持不變) 而且可以透過一直計算來找到它。

![](https://i.imgur.com/Cm18Hwf.png)

GNN 的計算方法就是一直去算 f 更新狀態，一直到平衡，然後經過 g 輸出 output（f 跟 g 都用FNN來實現）。

第三個圖是表示計算的流程，t 從 $t_0$ 一直到 T，以第二個點來看，它需要 1 跟 4 的 state 跟 label 來計算。

過了很多次 f ，想像中應該都要算 gradient 相加，但因為 x 會趨於穩定，這篇論文有提出一個比較有效率的算法。

![](https://i.imgur.com/WwpraN7.png)

L 是 dataset，p 是圖的個數，$q_i$ 是 output 點的個數，node-focused 的話 $q_i$ 可以是 $G_i$ 的點個數；graph-focused的話 $q_i$ = 1；t 是 target。

loss function 定義這邊 $\phi$ 表示取出第 i 個圖第 j 個node 的 output。

learning algorithm 就是 gradient descent 首先不斷更新找到不動點，然後算 gradient，最後更新 weight。

這篇提供兩個定理，一個是 $\phi$ 的可微性一個是 backprop 演算法，它透過定義一個 z，像 x 一樣一直去更新它，證明說 z 也會跑到一個不動點，而且可以用來計算 gradient，可以把 z 理解成是在累積各個時間 t 中 loss 對 x 的微分。

FG都是一個 FNN，之後會講。

![](https://i.imgur.com/nZCkxtd.png)

main 就是一個標準 gradient descent，forward 的 x 跟 backward 的 z 會一直去更新直到與前一次迭代夠靠近。

![](https://i.imgur.com/hw3JuG2.png)

G 沒有什麼限制，F 要服從 contraction mapping 的性質，這篇提供一種 Linear 一種 Nonlinear 的實現方法

- L
    對 x 來說是線性函數，A、b 分別用兩個 FNN 來產生，定義 $\phi$ 是 NN，input 自己的 node label 和鄰居 nodel label 和相接的 edge label，output $s^2$ 維的向量；$\rho$ 也是 NN，input 自己的 node label，output s 維的向量。

    A、b 定義如上，$\Xi$ 把長長的向量變成矩陣，$\mu$ 在 0 到 1 之間，對應到 contraction mapping 的那個 q。

    假設 $\phi$ 的 1-norm 小於 s (只要讓 activation bound 住就好)，則對於所有 $\omega$，F 會是一個 contraction mapping(大 A 是 block matrix，矩陣的 1-norm 定義成沿著 row sum 取最大)。

- NL
    在 loss 後面加一個 penalty，限制它的大小。

實驗顯示NL比L好

## Experiments

![](https://i.imgur.com/pR5XRr1.png)

目的是要定位出 graph G 裡的 subgraph S，是 node-focused 問題，試圖分類每個 node 是否屬於 S ，每個dataset有 600 個graph，平均分成train validation test。

圖產生的方式是每個 node pair 有 0.2 的機率隨機連接，然後強制讓每個圖都含有 S 在裡頭，給每個點一個[0, 10]的整數（s的數字固定），加隨機N(0, 0.25)。

表格為 repeat 5 次的平均表現。

在這個問題上他的表現不是最好最快的，但他說可以應用得最廣ㄏㄏ，所以只跟自己比，FNN 是只看 node 的數字，不考慮連接的關係。

表現來看 NL > L > FNN

可以發現 S 的點個數跟 G 的點個數呈現 1：2 時表現最差，點越少表現越好。

[pytorch implement](https://github.com/zhowzeng/subgraph-matching)

![](https://i.imgur.com/1g2VGn9.png)

230 個硝基芳香族，目標是找出會誘發突變的化合物，graph-focused 問題。

一個化合物平均有26個原子。

GNN 表現最好。
