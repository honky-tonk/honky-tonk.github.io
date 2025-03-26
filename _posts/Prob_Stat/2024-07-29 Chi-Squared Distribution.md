---
layout: post
title: Chi-Squared Distribution
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

# Introduction
The Chi-Squared Distribution is Key tool for compare two Parameter if relative, like if My algo trade return relative normal distribution? if gender relative product kind? if education relative salary?

So now let's take a look how Chi-squared distribution work!

**Define**

if $Z_1,Z_2,...,Z_n$ are **independent standard normal random variable**, the Chi-squared Statistic(X) is $X = Z_1^2 + Z_2^2 +...+ Z_n^2$, the Z is $\frac{Observe - Estimate}{Estimate}$， Observe is Observe parameter, Estimate is Estimate paramater, then we comapre X with chi-square table to identify whither Oberve and Estimate is relative


# Example
Relationship Between Advertising Method and Purchasing Behavior

Suppose we have data on different advertising methods (TV, online, newspaper) and whether consumers purchased the product. We want to test if there is a significant association between the advertising method and purchasing behavior.

Data
|Advertising Method	|Purchase |No Purchase	 |Total |
|-|-|-|-|
|TV|50 |30 |80 |
|Online |30 |40 |70 |
|Newspaper |20 |30 |50 |
|Total |100 |100 |200 |

**Step 1: Calculate Expected Frequencies**

The expected frequency for each cell is calculated using the formula:
- For TV and Purchase:

    $E_{11}=\frac{80 * 100}{200}=40$

- For TV and No Purchase:

    $E_{12}=\frac{80 * 100}{200}=40$

- For Online Purchase:

    $E_{21}=\frac{70 * 100}{200}=35$

- For Online No Purchase: 

     $E_{22}=\frac{70 * 100}{200}=35$  

- For Newspaper Purchase:

    $E_{31}=\frac{50 * 100}{200}=25$     

- For Newspaper no Purchase:  

    $E_{32}=\frac{50 * 100}{200}=25$  

Step 2: Calculate the Chi-square Statistic

$\chi^2=E_{11}+E_{12}+...+E_{32}=8.428$

Step 3:  find degree of freedom and find the critial value which correspond  degree of freedom in chi-square distribution table

$df=(r-1)(c-1)=(3-1)(2-1)=2$

$critial value(2) = 5.991$

conclude: $\chi^2$ is greater than critial value so, we reject the null hypothesis. This indicates that there is a significant association between advertising method and purchasing behavior.