# How Does Batch Normalization Help Optimization?

## Introduction

- Currently, the most widely accepted explanation of BatchNorm’s success, as well as its original motivation, relates to so-called internal covariate shift (ICS). Informally, ICS refers to the change in the distribution of layer inputs caused by updates to the preceding layers
- Even though this explanation is widely accepted, we seem to have little concrete evidence supporting it. In particular, we still do not understand the link between ICS and training performance.
- Our point of start is demonstrating that there does not seem to be any link between the performance gain of BatchNorm and the reduction of internal covariate shift. Or that this link is tenuous, at best. In fact, we find that in a certain sense BatchNorm might not even be reducing internal covariate shift.
- Specifically, we demonstrate that BatchNorm impacts network training in a fundamental way: it makes the landscape of the corresponding optimization problem significantly more smooth. This ensures, in particular, that the gradients are more predictive and thus allows for use of larger range of learning rates and faster network convergence.

## Batch normalization and internal covariate shift

- internal covariate shift
  - Ioffe and Szegedy describe ICS as the phenomenon wherein the distribution of inputs to a layer in the network changes due to an update of parameters of the previous layers.
- Does BatchNorm’s performance stem from controlling internal covariate shift?

 ![](https://i.imgur.com/oHT0oYV.png)

- Is BatchNorm reducing internal covariate shift?

 ![](https://i.imgur.com/ihuiR1S.png)

## Why does BatchNorm work?

- The smoothing effect of BatchNorm
  - it reparametrizes the underlying optimization problem to make its landscape significantly more smooth
  - The first manifestation of this impact is improvement in the Lipschitzness2 of the loss function. That is, the loss changes at a smaller rate and the magnitudes of the gradients are smaller too
  - BatchNorm’s reparametrization makes gradients of the loss more Lipschitz too. In other words, the loss exhibits a significantly better “effective” β-smoothness
  - Recall that f is L-Lipschitz if |f(x1) − f(x2)| ≤ L|x1 − x2|, for all x1 and x2.
- Exploration of the optimization landscape

  ![](https://i.imgur.com/5sfrI4y.png)

- Is BatchNorm the best (only?) way to smoothen the landscape?
  - we study schemes that fix the first order moment of the activations, as BatchNorm does, and then normalizes them by the average of their p-norm (before shifting the mean), for p = 1, 2, ∞. Note that for these normalization schemes, the distributions of layer inputs are no longer Gaussian-like

  ![](https://i.imgur.com/77YeuBy.png)
