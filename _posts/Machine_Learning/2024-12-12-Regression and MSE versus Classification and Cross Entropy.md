---
layout: post
title: Regression and MSE versus Classification and Cross Entropy.md
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


# Introduction to Linear Regression

formula

$y=wx+b$

or



$\hat{y}=\mathbf{[w_1, w_2, \ldots, w_n]}\begin{bmatrix}
           x_{1} \\
           x_{2} \\
           \vdots \\
           x_{n}
         \end{bmatrix} + b$

if x is a vector of row the formula of linear regression just like dot product plus bias

$\hat{y}=\mathbf{w} \cdot \mathbf{x} + b= w_1x_1 + w_2x_2 + \cdots + w_nx_n + b$

if we got muilti predictors we got

$\hat{y}=w_1x_1+w_2x_2+...+w_nx_n+b$

Linear regression is a fundamental technique in statistics and machine learning used to model relationships between variables. It is particularly useful for predicting a continuous output based on one or more input features.

Suppose we want to predict house prices based on size and number of bedrooms. A possible linear regression model could be This equation predicts the price based on linear relationships between features and the target.

if we want to measure how far between predict and fact we could MSE(Mean Squared Error) 

$loss = \frac{1}{n}\sum_{i=1}^n(y_i-\hat{y}_i)^2$

$y_i$ is fact number or target number, $\hat{y} is predict number$


# Why Linear Regression Fails for Classification Tasks and logistic regression

Problem with Linear Regression in Classification:
- Suppose all spam emails(spam email as 1, not spam email as 0) in the training set are strongly correlated with a feature (e.g., "contains the word 'discount'"). The linear regression model may predict extreme values for  (e.g., ), making classification unstable.

we define predict value between 0 and 1, if predict value is under 0.5, it not spam email( as 0), if it is upper 0.5 it is spam email( as 1), it's seem  work, but it fail in extreme case, like x is 1000001 y is 0.6, it change regression line much! and make regression line fail to predict, so we need Logistic Regression

To address the shortcomings of linear regression for classification tasks, logistic regression was developed. It maps the continuous output of a linear function to a probability value between 0 and 1 using the sigmoid function.

the formula logistics regression is

$\sigma(x) = \mathbf{w} \cdot \mathbf{x} + b$

$logistics =  \frac{1}{1 + e^{-\sigma(a)}}$

the logistic regression model maps a linear function to a value between 0 and 1, which represents a probability


# Cross Entropy Loss

we could use Cross Entropy Loss the measure how good our logistic regression is, but not MSE

- MSE in classfication or ogistic regression is not convex
- Directly penalizes incorrect predictions based on their probability, aligning with the probabilistic output of logistic regression.
- Yields gradients that guide the model to make sharper probability predictions.

we could prove the cross entropy loss 

first we could give Bernoulli distribution

$P(y_i|x_i)=\hat{y}_i^{y_i}(1-\hat{y}_i)^{1-y_i}$

cause
when $y_i$ is 1 then $P(1|x_i)= \hat{y}_i $ ,  when $y_i$ is 0 then $P(0|x_i)= 1-y_i $ , $\hat{y}_i$ is logistics value(predict) we saw above,

give data set $D={(x_1, y_1), (x_2, y_2), ... , (x_n, y_n)}$ each $y_i$ is i.i.d, then we get

$P(y|X)=\prod_{i=1}^nP(y_i|x_i)=\prod_{i=1}^n\hat{y}_i^{y_i}(1-\hat{y}_i)^{1-y_i}$

then we use log each pair

$logP(y|X)=\sum_{i=1}^n[y_ilog\hat{y}_i+(1-y_i)log(1-\hat{y}_i)]$

the  loss is $logP(y|X)$


