---
layout: post
title: Sample Distribution and Confidence Intervals
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---
# Preface
Parameter:A parameter is a fixed value or characteristic used to describe a population, like population mean population standard deviation

Statistical: by contrast Statistical is an adjective used to describe sample, like sample mean sample sd

Standard Deviation(Population):$\sqrt{\frac{\sum_{i=1}^{i=n}(x_i - \mu)^2}{N}}$

>$\mu$ is population mean, $N$ is population size

Standard Deviation(Sample):$\sqrt{\frac{\sum_{i=1}^n(x_i-\bar{x}^2)}{n-1}}$
> $n$ is sample size, $\bar{x}$ is sample mean 

Standard Error:$\frac{\sigma}{\sqrt{n}}$
> $\sigma$ is sample standard deviation, $n$ is sample size

>SE is the **standard deviation** of its sampling distribution, in other word SE is for multi sample

Z-Score(**one** sample): $\frac{x-\mu}{\sigma}$
>x is obvious value, $\mu$ is sample mean, $\sigma$ is sample standard deviation

Z-Score(**multi** sample): $\frac{x-\mu}{SE}$
>x is obvious samples statistic like sample mean(highligh the multi sample), $\mu$ is sample mean, SE is sample of standard error

# sample distribution basice
A lot of data drawn and used are actually samples rather than populations. A sample is a subset of a population. Put simply, a sample is a smaller part of a larger group. As such, this smaller portion is meant to be representative of the population as a whole.

Sampling distributions (or the distribution of data) are statistical metrics that determine whether an event or certain outcome will take place. This distribution depends on a few different factors, including the sample size, the sampling process involved, and the population as a whole. There are a few steps involved with sampling distribution. These include:
- Choosing a random sample from the overall population
- Determine a certain statistic from that group, which could be the standard deviation, median, or mean
- Establishing a frequency distribution of each sample
- Mapping out the distribution on a graph



Sample distribution refers to the distribution of values within a sample taken from a larger population, a sample is a subset of individuals or items from a larger population, that is studied to gain insights about the population itself

let's look a example sample distribution of mean(sample mean)

```r
#population
population <- c(0,32)

#repeat count for loop
R <- 10^5

#vector of sample_means
sample_means <- c(NA, R)

for (i in 1:R){
    samp <- sample(population, size = 2, replace = TRUE)
    sample_means[i] <- mean(samp)
}

hist(sample_means, breaks = 3, xlim = c(0, 32))
```
we got population vector from 0 to 32, then we count the sample 10^5 times

# Z-Score

z-score (also called a standard score) gives you an idea of how far from the mean a data point is, In multi-sample z-score describe the standard deviation of those sample means (the standard error)

but in multiple sample. 

# confidence intervals