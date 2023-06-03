# Accumulated Local Effects (ALE) Plot

<https://christophm.github.io/interpretable-ml-book/ale.html>
<https://arxiv.org/abs/1612.08468>
Accumulated local effects describe how features influence the prediction of a machine learning model on average. ALE plots are a faster and unbiased alternative to [partial dependence plots](https://hackmd.io/Pi-9mjLVToSqs0UtJrqwzw) (PDPs).

- Motivation and Intuition
  - If features of a machine learning model are correlated, the partial dependence plot cannot be trusted. The computation of a partial dependence plot for a feature that is strongly correlated with other features involves averaging predictions of artificial data instances that are unlikely in reality.

    ![](https://i.imgur.com/3VXrplS.png)

  - We could average over the conditional distribution of the feature, meaning at a grid value of x1, we average the predictions of instances with a similar x1 value. The solution for calculating feature effects using the conditional distribution is called Marginal Plots, or M-Plots (confusing name, since they are based on the conditional, not the marginal distribution).

    ![](https://i.imgur.com/oSViyqh.png)

  - M-Plots do not solve our problem. If we average the predictions of all houses of about 30 m2, we estimate the combined effect of living area and of number of rooms, because of their correlation.
  - ALE plots solve this problem by calculating -- also based on the conditional distribution of the features -- differences in predictions instead of averages. For the effect of living area at 30 m2, the ALE method uses all houses with about 30 m2, gets the model predictions pretending these houses were 31 m2 minus the prediction pretending they were 29 m2. This gives us the pure effect of the living area and is not mixing the effect with the effects of correlated features. The use of differences blocks the effect of other features.
  
    ![](https://i.imgur.com/Q4d8dZF.png)

- Theory
  - Partial dependence plots average the predictions over the marginal distribution.
    $$\begin{align*}\hat{f}_{x_S,PDP}(x_S)&=E_{X_C}\left[\hat{f}(x_S,X_C)\right]\\&=\int_{x_C}\hat{f}(x_S,x_C)\mathbb{P}(x_C)d{}x_C\end{align*}$$
  - M-plots average the predictions over the conditional distribution.
    $$\begin{align*}\hat{f}_{x_S,M}(x_S)&=E_{X_C|X_S}\left[\hat{f}(X_S,X_C)|X_S=x_s\right]\\&=\int_{x_C}\hat{f}(x_S,x_C)\mathbb{P}(x_C|x_S)d{}x_C\end{align*}$$
  - ALE plots average the changes in the predictions and accumulate them over the grid.
    $$\begin{align*}\hat{f}_{x_S,ALE}(x_S)=&\int_{z_{0,1}}^{x_S}E_{X_C|X_S}\left[\hat{f}^S(X_s,X_c)|X_S=z_S\right]dz_S-\text{constant}\\=&\int_{z_{0,1}}^{x_S}\int_{x_C}\hat{f}^S(z_s,x_c)\mathbb{P}(x_C|z_S)d{}x_C{}dz_S-\text{constant}\end{align*}$$
    where
    $$\hat{f}^S(x_s,x_c)=\frac{\delta\hat{f}(x_S,x_C)}{\delta{}x_S}$$

- Estimation
  $$\hat{\tilde{f}}_{j,ALE}(x)=\sum_{k=1}^{k_j(x)}\frac{1}{n_j(k)}\sum_{i:x_{j}^{(i)}\in{}N_j(k)}\left[f(z_{k,j},x^{(i)}_{\setminus{}j})-f(z_{k-1,j},x^{(i)}_{\setminus{}j})\right]$$
  - The difference in prediction is the **Effect** the feature has for an individual instance in a certain interval
  - This average in the interval is covered by the term **Local** in the name ALE.
  - The left sum symbol means that we accumulate the average effects across all intervals. The word **Accumulated** in ALE reflects this.
  - This effect is centered so that the mean effect is zero.
    $$\hat{f}_{j,ALE}(x)=\hat{\tilde{f}}_{j,ALE}(x)-\frac{1}{n}\sum_{i=1}^{n}\hat{\tilde{f}}_{j,ALE}(x^{(i)}_{j})$$
  - ALE plots for the interaction of two features

    ![](https://i.imgur.com/Byg7TtQ.png)

  - ALE for categorical features
    - To compute an ALE plot for a categorical feature we have to somehow create or find an order.
    - One solution is to order the categories according to their similarity based on the other features. The distance between two categories is the sum over the distances of each feature. The feature-wise distance compares either the cumulative distribution in both categories, also called Kolmogorov-Smirnov distance (for numerical features) or the relative frequency tables (for categorical features).
    - Once we have the distances between all categories, we use multi-dimensional scaling to reduce the distance matrix to a one-dimensional distance measure. This gives us a similarity-based order of the categories.
- Examples
  - simulation case

    ![](https://i.imgur.com/fyceEFo.png)
    ![](https://i.imgur.com/BANCHK8.png)

  - number of rented bikes

    ![](https://i.imgur.com/Uhu81BU.png)
    ![](https://i.imgur.com/kDBaXCV.png)
    ![](https://i.imgur.com/tp8FTKE.png)

- Advantages
  - ALE plots are unbiased, which means they still work when features are correlated.
  - ALE plots are faster to compute than PDPs and scale with O(n), since the largest possible number of intervals is the number of instances with one interval per instance.
  - The interpretation of ALE plots is clear: Conditional on a given value, the relative effect of changing the feature on the prediction can be read from the ALE plot. ALE plots are centered at zero.
  - The 2D ALE plot only shows the interaction: If two features do not interact, the plot shows nothing.
- Disadvantages
  - ALE plots can become a bit shaky (many small ups and downs) with a high number of intervals. In this case, reducing the number of intervals makes the estimates more stable, but also smooths out and hides some of the true complexity of the prediction model.
  - There is no perfect solution for setting the number of intervals.
  - ALE plots are not accompanied by ICE curves.
  - Second-order ALE estimates have a varying stability across the feature space, which is not visualized in any way. The reason for this is that each estimation of a local effect in a cell uses a different number of data instances. As a result, all estimates have a different accuracy (but they are still the best possible estimates).
  - Second-order effect plots can be a bit annoying to interpret, as you always have to keep the main effects in mind.
  - Even though ALE plots are not biased in case of correlated features, interpretation remains difficult when features are strongly correlated. Because if they have a very strong correlation, it only makes sense to analyze the effect of changing both features together and not in isolation.
- Implementation and Alternatives
  ALE plots are implemented in R and Python, once in the ALEPlot R package by the inventor himself and once in the iml package and in the [ALEPython](https://github.com/blent-ai/ALEPython) package.
