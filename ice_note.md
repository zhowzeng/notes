# Individual Conditional Expectation (ICE)

<https://christophm.github.io/interpretable-ml-book/ice.html#ice>
Individual Conditional Expectation (ICE) plots display one line per instance that shows how the instance's prediction changes when a feature changes.
An ICE plot visualizes the dependence of the prediction on a feature for each instance separately, resulting in one line per instance, compared to one line overall in [partial dependence plots](https://hackmd.io/Pi-9mjLVToSqs0UtJrqwzw). A PDP is the average of the lines of an ICE plot.
A more formal definition: In ICE plots, for each instance in $\{(x_S^{(i)},x_C^{(i)})\}^N_{i=1}$ the curve $\hat{f}_S^{(i)}$ is plotted against $x_S^{(i)}$, while $x_C^{(i)}$ remains fixed.

- Examples

  ![](https://i.imgur.com/7AHTc63.png)
  ![](https://i.imgur.com/L2FRZqO.png)

- Centered ICE Plot
  There is a problem with ICE plots: Sometimes it can be hard to tell whether the ICE curves differ between individuals because they start at different predictions. A simple solution is to center the curves at a certain point in the feature and display only the difference in the prediction to this point. The resulting plot is called centered ICE plot (c-ICE). Anchoring the curves at the lower end of the feature is a good choice.
  - Example

    ![](https://i.imgur.com/PR7Y9BG.png)
    ![](https://i.imgur.com/6xh4KAC.png)

- Advantages
  - Individual conditional expectation curves are even more intuitive to understand than partial dependence plots. One line represents the predictions for one instance if we vary the feature of interest.
  - Unlike partial dependence plots, ICE curves can uncover heterogeneous relationships.
- Disadvantages
  - ICE curves can only display one feature meaningfully, because two features would require the drawing of several overlaying surfaces and you would not see anything in the plot.
  - ICE curves suffer from the same problem as PDPs: If the feature of interest is correlated with the other features, then some points in the lines might be invalid data points according to the joint feature distribution.
  - If many ICE curves are drawn, the plot can become overcrowded and you will not see anything. The solution: Either add some transparency to the lines or draw only a sample of the lines.
  - In ICE plots it might not be easy to see the average. This has a simple solution: Combine individual conditional expectation curves with the partial dependence plot.
- Software and Alternatives
  - In Python, partial dependence plots are built into scikit-learn starting with version 0.24.0.
