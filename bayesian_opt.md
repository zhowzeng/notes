# Bayesian Optimization

## 資料收集

- [Hyperparameter tuning in Cloud Machine Learning Engine using Bayesian Optimization](https://cloud.google.com/blog/products/gcp/hyperparameter-tuning-cloud-machine-learning-engine-using-bayesian-optimization)
  - by google cloud
  - GP只有講公式

      ![](https://i.imgur.com/VworLvA.png)

  - acquisition介紹了Exploration-exploitation trade-of
    - Upper confidence bound
    - Probability of improvement

        ![](https://i.imgur.com/dqw1Tpf.png)

    - Expected Improvement（google vizer使用，已驗算）

        ![](https://i.imgur.com/vTZ7Xqs.png)

  - <https://arxiv.org/pdf/1012.2599.pdf>
    - 2010的paper, 有很多圖

- [A Visual Exploration of Gaussian Processes](https://distill.pub/2019/visual-exploration-gaussian-processes/)
  - 超讚互動圖
  - marginalization & conditioning

      ![](https://i.imgur.com/88nDIAw.png)
      ![](https://i.imgur.com/bicTrt0.png)

    - marginalization

          ![](https://i.imgur.com/SYCuSLN.png)

    - conditioning

          ![](https://i.imgur.com/xJCIdJC.png)

  - GP的觀念與kernel
- [Exploring Bayesian Optimization](https://distill.pub/2020/bayesian-optimization/)
  - 有比較圖還有svm的實例

      ![](https://i.imgur.com/kM6dYbC.png)

  - 使用限制

      ![](https://i.imgur.com/kqdtwrO.png)

- [Tutorial #8: Bayesian optimization](https://www.borealisai.com/en/blog/tutorial-8-bayesian-optimization/)
