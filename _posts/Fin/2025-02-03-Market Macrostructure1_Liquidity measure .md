---
layout: post
title: 2025-02-03-Market Macrostructure1_Liquidity measure  
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---
# background
Why we need measure liqudity?

for trader:
 - liquidity provide a measure of trading cost(when we make a trading how expensive or cheap we could afford, if liquidity is low and we want to sell or buy a stocks , there may no peer to buy or sell this stocks in your expect price, then we could make compromise)

 for regulators(responsibilities for efficiency of market):
 - efficiency is tricky to measure in finacial market: liqudity provides a proxy 
 - lliquid market also seem to be more prone to medium-rum price deviation from fundamentals 
 - lliquidity may be a sign of structural problem in market

 relation to depth: depth measure how much must be traded to mvoe price  by certain amount 

 we have muliti way to measure liqudity, like 
 - Spread Measure: we got quoted spread, effective spread, realized spread
 - Volume-weighted average price: simply using average price
 - price impact: how much price move after a trade 
 - non-trading measure:  

 if trade direction is not available: we could use **Lee-Ready algorithm**

 if quote data are not available: we could use **Roll's measure**

 >quote data mean last price, all Bid/Ask price(order book), volume etc...

 # quoted spread
 In time t quoted spread formula use best ask($a_t$) price minus best bid($b_t$) price

 Quoted spread:

 $$S_t=a_t-b_t$$

you could use normalized quoted spread

$$s_t=\frac{S_t}{m_t}$$
where $m_t$ is midprice $m_t=\frac{a_t+b_t}{2}$

 

 we could combination the trade size $q$

 $$S(q)=\overline{a_t}(q)-\overline{b_t}(q)$$

 where $\overline{a_t}(q)$ and $\overline{b_t}(q)$ are avg exec prices

  >quoted spread has two main defect
  >- 1: you need know best ask/bid price each time, but its not possible in most time
  >- 2: in the initialization, the gap between the best ask and best bid is too large(people need negotiation by the time),   


# Effective spread
Effective spread is eliminate the last question in quoted spread,we got the formula like 
$$S_t^e=d_t(p_t-m_t)$$

$S_t^e$: mean in time t the effective spread is 

$d_t$:mean the trade direct in time t

$p_t$: mean the realized price in time t

$m_t$: mean the mid price


we also could use normalization 
$$s_t^e=\frac{S_t^e}{m_t}$$

$s_t^e$ is normalized effective spread

> in effective spread we could not get the trade direct and mid price, but the performance of effective spread is pretty well

# how we predict direct of trade?
we could use Lee-Ready algorithm

$$
d_t=\Biggl\{^{1\:if\: |p_t-a_t| < |p_t-b_t|}_{-1 \: if \: |p_t-a_t| > |p_t - b_t|}
$$

1 mean the bid initial trade, other word the mid quote close to best bid 

-1 mean the ask initial trade, other word the mid quote close to best ask

> in lee ready algorithm we also miss with trades at the mid quote, and small transactions, transactions in large cap 

 # how to predict quotes data?

 **roll measure**(from 1984) is common used masure to predict quotes data with tansaction
 > The easiest data we could get is transaction data, transaction data and quotes data is different, 
 >
 >the transaction data refer to 
 > - trade price in time t
 > - trade timestamp
 > - trade volume
 > - ...
 >
 >the quoted data refer to the data related to order book, the data like
 > - best bid price in time t
 > - best ask price in time t
 > - bid size in time t
 > - ask size in tine t
 > - spread
 > - ...

 in roll measure, we **suppose** as follow:
 - All trade have the same size, d = 1 mean buy direction, d = -1 mean sell direction
 - Arriving order are **I.I.D**, with $P(d_t=1)=P(d_t=-1)=\frac{1}{2}$
 - Midquote is **random** walk, $m_t=m_{t-1}+\epsilon_t$, the $\epsilon_t$ is noise or stochastic error term, and $\epsilon_t$ is i.i.d $\epsilon_t \sim N(0, \sigma^2) a$, and $E[\epsilon_t]=0$
 > why $E[\epsilon_t]=0$? you could image $\epsilon_t$  that the integral of an odd function over a symmetric interval is zero

 - Spread $S = a_t - b_t$ is contant
 - market order is not informative: $E[d_t\epsilon_t]=E[d_t\epsilon_{t+1}]=0$

 then we could get 
 $$p_t = m_t + \frac{d_tS}{2}$$ 

- $p_t$ is transaction price
- $m_t$ is midquote
- $d_t$ is trade direction and it's i.i.d, $\triangle d_t = d_t-d_{t-1}$ is mean-reverting and other word $d_t$ and $d_{t-1}$ is negative correlation AKA $Cov(\triangle d_t, \triangle d_{t-1})=-1$
  >the formula of correlation is $Cov(X, Y)=\frac{1}{n-1}\sum_{i=1}^n(x_i-\overline{x})(y_i-\overline{y})=E[(X-E[X])(Y-E[Y])]$ 

  >$Cov(\triangle d_t, \triangle d_{t-1})=Cov(d_t-d_{t-1},d_{t-1}-d_{t-2})=Cov(d_t - d_{t-1}, d_{t-1} - d_{t-2}) = Cov(d_t, d_{t-1}) - Cov(d_t, d_{t-2}) - Cov(d_{t-1}, d_{t-1}) + Cov(d_{t-1}, d_{t-2}) \text{  Because  } d_t \text{ is i.i.d so } d_t \text{at different time points have zero correlation. so } Cov(d_t,d_{t-1})=0, Cov(d_t,d_{t-2})=0,Cov(d_{t-1},d_{t-2})=0,\text{  so } Cov(\triangle d_t, \triangle d_{t-1})=-Cov(d_{t-1},d_{t-1})=-Var(d_{t-1})=-\sigma^2$ 

  >why $-Var(d_{t-1})=-\sigma^2=-1$? first d is direction, d always 1 or -1 and $P(d_t=1)=P(d_t=-1)=\frac{1}{2}$ so the $E[d_{t-1}]$ = 0 and $Var(d_{t-1})=1$
- $S$ is spread which best ask minus best bid

the $p_t$ is easiest way to get $d_t$ we could predict by lee-ready algo, then we should estimate $S$ then we could get $m_t$

so how we estimate $S$? first $S$ is spread which best ask minus best bid, and we know $p_t=m_t+\frac{d_tS}{2}$, then we could link $Cov(\triangle p_t, \triangle p_{t-1})$ and $S$ like we estimate direction of trade $Cov(\triangle d_t, \triangle d_{t-1})$, then we could get $$Cov(\triangle p_t, \triangle p_{t-1})=-\frac{S^2}{4}$$

the limit of roll measure 
- roll measure assumption there is no information, $E[d_t\epsilon_t]=E[d_t\epsilon_{t+1}]=0$, but in real market is not possible...,$E[d_t\epsilon_t]=E[d_t\epsilon_{t+1}]=0$ mean $d_t$ and $\epsilon_t$ is no relation, mean no one can know the midquote then change trade direct
- roll measure assumption all order  is I.I.D, in real market the order follow momentum in short time 

so we could use Hasbrouck's Lambda or  Amihud Illiquidity Ratio to prevent The impact of trading volume

# Volume Base Measure
Volume Base Measure basically let you know how good the **broker** exec your order, like broker exec your order straight or use other algo

Volume-Weighted Average Price(VWAP) is most famous way to masure how well broker exec your order $$VWAP=\sum w_ip_i$$

- i mean order count
- $w_i$ mean weight in order i(size of order i /  all order)
- $p_i$ mean exec price in order i  