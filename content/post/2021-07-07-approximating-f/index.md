---
title: Estimating f
author: Derek Dixon
date: '2021-07-24'
slug: approximating-f
categories:
  - statistical learning
  - ISL
  - SLRP
tags: []
---

```{r, warning=FALSE, message=FALSE}

library(tidyverse)

```

Here we go, my first substantial blog post!

I want to begin with the conceptual fundamentals of statistical learning in the tradition of Hastie et al in ESL. While there may be a distinction between "statistical learning" and "machine learning", I won't attempt to address that here. I will use the terms interchangeably as schools of thought fundamentally aiming to **estimate f**, where f is some function of a set of predictors X and a response Y.

In theory, any relationship or pattern that exists between a set of predictors and some response (if indeed there is a pattern) would be reflected in the a set of data and can be characterized as such:
Y = f(X) + e 

Y represents our response, or target, or dependent variable. The thing we would like to predict using future values of X.
X represents one or more predictors, or features, or independent variables. The things we suspect might be useful in predicting Y because of their having some unknown relationship with Y. This relationship need not be causal. It could be purely correlative. 
e, typically notated as epsilon in the literature, represents the imperfection associated with our modeling of Y. e represents things like measurement error in the data, the effects of omitted variables having important relationships with Y, or perhaps even the inherent variability in Y itself. If we assume the world works in a perfectly deterministic fashion and we had the perfect set of predictors which all have direct causal relationships with Y, and all of which were measured perfectly without error, then perhaps we could omit e. But barring any of that, here in the real world of error, uncertainty, and perhaps even fundamental randomness (quantum mechanics? Indeterminism?), we are left with adding in epsilon to bridge the gap between our estimate of Y and Y itself. One very important concept in statistical learning is that, given a set of predictors, even if we knew the exact functional form of the relationship between Y and X, we'd still have error due to the various reasons stated. For this reason, epsilon is also often called the "irreducible error".

The ideas of this post draw heavily from the opening chapters of An Introduction to Statistical Learning as well as the companion lecture series. I'm sure I'll opine on these ideas more clearly when I start methodically blogging about ESL in future posts. Hopefully as I'm doing so I won't realize I made egregious errors in this post since I'm working off memory of a past reading of ISL. 

Let's illustrate these ideas using some synthetic data. My preferred didactic approach is to work through the concepts using a single predictor (2 dimensional space), then think about how it would look with 2 predictors (3 dimensional space, what you and I both live in), then just realize that the math and geometry can and does generalize to any arbitrary dimensional space >3. Specifically, in real cases we work in p+1 dimensional space where p is the number of predictors and the +1 is the response. Each predictor equates to another dimension and high dimensions becomes impossible to visualize after 3. 

So let's move on using a single predictor, X.

Suppose the true underlying relationship between X and Y is Y = (X+2)^2. 

```{r, message=FALSE, warning=FALSE}

data <- 
  tibble(
    x = rnorm(10000, 5, 1),
    y = (x + 2)**2,
    y_obs = y + rnorm(10000, 0, 5)
    )

data %>%  
  ggplot() +
  geom_smooth(aes(x = x, y = y), colour = "red") +
  labs(title = "True relationship between X and Y",
       subtitle = "Y = (x+2)**2")

```
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />


This true relationship is unknown to us but we stipulate that it "exists" (if perhaps only in the abstract). We can only observe the data at hand, a set of pairs of (y, x) representing historical examples which will include error (epsilon). If we have enough of these pairs we could have more than one value of Y for each value of X. Let's superimpose the "actual" data over the unknown relationship line, the true f.

```{r}

data %>% 
  ggplot(aes(x = x, y = y_obs)) +
  geom_point(alpha = 0.5) +
  geom_smooth(aes(x = x, y = y), colour = "red") +
  labs(title = "Data Sample")

```
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Because we have so much data (10,000 examples here), it's easy to see how that line fits nicely in the spread of points. This also illustrates the idea that more data is better than less data in the cases where there is an informative relationship between X and Y. 

But remember, in practice, we don't see that red line, the underlying true f. We only see points. And our goal is to estimate that red line from those points.

An intuitive general approach to estimate f is to simply take the mean value of Y at each value of X, plotting those means across the range of X. This is expressed as E(Y|X = x). This would give us the ideal f. In statistical learning this ideal f is known as the regression function. E(Y|X = x) also reflects the idea that Y is treated as a random variable where it's expected value can vary throughout the predictor space. Pick any particular point in the predictor space (that is, pick a value of x) and the corresponding Y will be a random variable with a particular distribution. 

Let's look at how this plays out for one value of X in particular, say X = 5. 

```{r}

data %>% 
  ggplot(aes(x = x, y = y_obs)) +
  geom_point(alpha = 0.5) +
  geom_smooth(aes(x = x, y = y), colour = "red") +
  labs(title = "Data Sample") +
  geom_vline(xintercept = 5, colour = "red")

# Rounding because no value of X exactly equals 5
data %>% 
  filter(round(x,1) == 5) %>% 
  select(y_obs) %>% 
  ggplot(aes(x = y_obs)) +
  geom_histogram(bins = 30) +
  geom_vline(xintercept = mean((data %>% filter(round(x,1) == 5))$y_obs))  +
  scale_x_continuous(breaks = seq(20, 80, 5)) +
  labs(title = "Histogram of Y at X = 5")

data %>% 
  filter(round(x,1) == 5) %>% 
  select(y_obs) %>% 
  ggplot(aes(x = 1, y = y_obs)) +
  ggdist::stat_halfeye(adjust = 0.5, width = 0.5) +
  geom_boxplot(width = 0.08, outlier.shape = NA) +
  ggdist::stat_dots(
    side = "left", 
    dotsize = .8, 
    justification = 1.05, 
    binwidth = .3
  ) +
  scale_y_continuous(breaks = seq(20, 80, 5)) +
  coord_flip(xlim = c(0.8, NA)) +
  scale_x_continuous(labels = NULL) +
  labs(title = "Raincloud of Y at X = 5", x = "")



```
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-2.png" width="672" />
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-3.png" width="672" />

The histogram and rain cloud clearly show the distribution of Y at X = 5 and that E(Y|X=5) = 49, approximately. The variability of Y at X=5 is a result of the irreducible error mentioned earlier. Even if we estimated f perfectly from the data, future predictions would still fail to be exact because Y varies for each value of X and our predicted values of E(Y|X=x) likely won't exactly equal Y in any particular instance. But the regression function, E(Y|X=x), would give us the best predictions on average, in the long run.

This gives us a good first glimpse into the conceptual underpinnings of statistical learning, but there is still much more to consider when we enter the real world of high dimensions and sparsity. I.e. when we have many predictors and only 1 or even no values of Y for each predictor X. In such cases, we cannot simply average the Y values for each X. We have to generalize our approach of taking the average Y not within each particular value of X, but within some neighborhood of each value of X. 

I'll save such discussions for future blog posts, most likely when I go through the opening chapters of The Elements of Statistical Learning.

That's all for now. Thank you for reading and I hope you enjoy the blog.