# Partial Dependence Plot (PDP)

<https://christophm.github.io/interpretable-ml-book/pdp.html>

The partial dependence plot (short PDP or PD plot) shows the marginal effect one or two features have on the predicted outcome of a machine learning model (J. H. Friedman 200127). A partial dependence plot can show whether the relationship between the target and a feature is linear, monotonic or more complex. For example, when applied to a linear regression model, partial dependence plots always show a linear relationship.

![](https://i.imgur.com/V3yQP5a.png)
![](https://i.imgur.com/AcmsX30.png)

An assumption of the PDP is that the features in C are not correlated with the features in S. If this assumption is violated, the averages calculated for the partial dependence plot will include data points that are very unlikely or even impossible (see disadvantages).

- examples
  - numerical
  
  ![](https://i.imgur.com/3k6S2lc.png)

  - categorical

  ![](https://i.imgur.com/rRpuAwb.png)

  - two features

  ![](https://i.imgur.com/wj9PtiE.png)

- Advantages
  - The computation of partial dependence plots is intuitive
  - In the uncorrelated case, the interpretation is clear
  - Partial dependence plots are easy to implement.
  - The calculation for the partial dependence plots has a causal interpretation.
- Disadvantages
  - The realistic maximum number of features in a partial dependence function is two. This is not the fault of PDPs, but of the 2-dimensional representation (paper or screen) and also of our inability to imagine more than 3 dimensions.
  - Some PD plots do not show the feature distribution. Omitting the distribution can be misleading, because you might overinterpret regions with almost no data. This problem is easily solved by showing a rug (indicators for data points on the x-axis) or a histogram.
  - When the features are correlated, we create new data points in areas of the feature distribution where the actual probability is very low (for example it is unlikely that someone is 2 meters tall but weighs less than 50 kg).
  - One solution to this problem is [Accumulated Local Effect plots](https://christophm.github.io/interpretable-ml-book/ale.html#ale) or short ALE plots that work with the conditional instead of the marginal distribution.
  - Heterogeneous effects might be hidden because PD plots only show the average marginal effects. Suppose that for a feature half your data points have a positive association with the prediction -- the larger the feature value the larger the prediction -- and the other half has a negative association -- the smaller the feature value the larger the prediction. The PD curve could be a horizontal line, since the effects of both halves of the dataset could cancel each other out. You then conclude that the feature has no effect on the prediction. By plotting the [individual conditional expectation curves](https://christophm.github.io/interpretable-ml-book/ice.html#ice) instead of the aggregated line, we can uncover heterogeneous effects.
- Software and Alternatives
 In Python, partial dependence plots are built into scikit-learn and you can use PDPBox.
