---
layout: post
title: Neural network and implement
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---

# Preface
Neural network is widely used in LLM and Image Processing... 

# Base neural network
A neural network is a computational model inspired by the way biological neural networks in the human brain process information. It consists of a series of algorithms that attempt to recognize underlying relationships in a set of data through a process that mimics the way the human brain operates.

Neural networks are composed of neurons, which are organized into layers. The three most common types of layers in a neural network are:
- **Input Layer**: This layer receives the initial data. Each neuron in this layer represents a feature in the input dataset.
- **Hidden Layer(s)**: These layers perform most of the computations required by the network. There can be one or multiple hidden layers, depending on the complexity of the model.
- **Output Layer**: This layer produces the final result of the network. Each neuron in this layer represents a possible output of the model.

![single neural](https://honky-tonk.github.io/assets/img/neural/single_ne.jpg)

Single neuron in hidden layer and one input neuron one output neuron in input/output layer struct


![single neural](https://honky-tonk.github.io/assets/img/neural/ne_with_one_input_and_two_input.jpg)

Single neuron in hidden layer and two input neuron one output neuron in input/output layer struct

then we got a sample neuron network, In graph above neural 1 have 1 paramater of input data, Initially paramater is random number, we should input multiple training data and fix the paramater, generally in single neural, this neural contain one paramater(p) and one bias(b), and compute ```y=xp+b```. x is input value, y is predict output

How we fix the paramater of input data? first data is a pair which a input data and the "right" result of the input data, so we could input the data and minus the “right” data and square, then  we get the **loss**, if Y is "right" output, y is predict output we just use ```Loss= (Y-y)^2```or you can write with ```Loss(p,b)=(Y-y(p,b))^2```, our gole is make loss decreasing, how we make loss decresing? we could use gradient descent

The base ideal of gradient descent is sample, we compute the derivatives of loss $$lim_{\triangle x->0}\frac{Loss(p+\triangle x, b)-Loss(p,b)}{\triangle x}$$ for bias we could use $$lim_{\triangle x->0}\frac{Loss(p, b+\triangle x)-Loss(p,b)}{\triangle x}$$
$\triangle x$ is a very very samll number we could represent with ```1e-10``` in C

# Why activation function matter?
It's perfect we got a sample neural network, but if we input data set like XOR problem

| Input x1   | Input x2   | Output y   |
|-------|-------|-------|
| 0   | 0 | 0 |
| 0   | 1 | 1 |
| 1   | 0 | 1 |
| 1   | 1 | 0 |

Take a look  table above, we could not use linear model like $y=w1x1+w2x2+b$ to solve the problem, so we need activation convert linear model to non-linear model
> most relation of this world is non-linear

We could prove like below 
> if we have one neural(x) in input layer one neural(h) in hidden layer one neural(y) in output layer, so we got formula
>$$h=W_1x+b_1$$
>$$y=W_2h+b_2$$
>than 
>$$y=W_2(W_1x+b_1)+b_2=W_1W_2x+W2b_1+b_2=(W_1W_2)x+(W2B_1+b_2)$$
>it's linear relation again..

so we need import activation function like sigmoid and ReLu

sigmoid:$$\sigma(z)=\frac{1}{1+e^{-z}}$$

which $z$ is $h=W_1x+b_1$ or $y=W_2h+b_2$ example above
> The output range of the sigmoid function is between 0 and 1.


RELu activation is more simple:$$ReLU(z)=max(0,z)$$

if $z=y=W_2h+b_2=1$ then $ReLU=z=1$

if $z=y=W_2h+b_2=-1$ then $ReLU=z=-1$

if $z=y=W_2h+b_2=-0$ then $ReLU=z=-0$

so we got new neural figure

![single neural](https://honky-tonk.github.io/assets/img/neural/neurol_network_with_activation_function.jpg)

# sigmoid vs ReLu

# RNN
## LSTM
