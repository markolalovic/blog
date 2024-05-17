---
layout: post
title: "Multicollinearity effect on OLS regression"
author: "Marko Lalovic"
subtitle: "The cause of inflation effect of highly correlated regressors on ordinary least squares estimator."
nav_order: 1
date: "2019-07-14"
keywords: regression, correlated variables
---
In ordinary least squares (OLS) regression, for given $X: \mathbb{R}^{p} \rightarrow \mathbb{R}^{n}$ and $y \in \mathbb{R}^{n}$, we minimize over $\beta \in \mathbb{R}^{p}$ the sum of squared residuals

$$
\newcommand{\norm}[1]{\left\lVert#1\right\rVert}
S(\beta) = \norm{ y - X\beta }_{2}^{2}
$$

This blog post illustrates the problem (<a href="#figure1">Figure 1</a>) of using OLS method to estimate the unknown parameters $\beta$ in the case of highly correlated regressors on a simple example using R.

<div id="figure1" class="imgcap-nbd">
  <img src="/blog/assets/posts/ols-regression/drawing-ols.svg" width="90%" class="invertImg">
  <div class="thecap">
    Figure 1: As the correlation between regressors increases, the OLS method becomes unstable.
  </div>
</div>

Suppose we have a model

$$
y \sim \beta_{0} + \beta_{1} x_{1} + \beta_{2} x_{2}
$$

where

$$
\beta_{0} = 3, \quad \beta_{1} = \beta_{2} = 1.
$$

Let the sample contain 100 elements:

```r
n <- 100
```

and let's introduce some highly correlated regressors:

```r
set.seed(42)
x1  <- rnorm(n)
x2  <- rnorm(n, mean = x1, sd = 0.01)
```

with correlation coefficient almost 1:
```r
cor(x1, x2)
[1] 0.999962365268769
```

We can run the OLS method 1000 times to get a sense of the effect of highly correlated regressors:
```r
x <- as.matrix(cbind(intr = 1, x1, x2))
nsim  <- 1000
betas  <- sapply(1:nsim, function(i) {
    y  <- rnorm(n, mean = 3 + x1 + x2, sd = 1)
    xx <- solve(t(x) %*% x)
    beta.ols <- as.vector(xx %*% t(x) %*% y)
    return(beta.ols)
})
```

The estimator for $\beta$, obtained by the OLS method, is still unbiased
```r
round(apply(betas, 1, mean), 3)
[1] 2.996 1.006 0.993
```

But the variance is large
```r
round(apply(betas, 1, sd), 3)
[1]  0.101 11.110 11.103
```

The estimated coefficients can be very large, some can even have the wrong sign:
```r
round(betas[, 1], 3)
[1]  3.002 -7.673  9.529
```

The problem can be seen by drawing a contour plot (shown in <a href="#figure1">Figure 1</a>) of the objective function $S(\beta)$:
```r
ssr <- function(b) {
    return(sum((y - x %*% b)^2))
}

xlen <- 100
ylen <- 100
xgrid <- seq(-10.1, 10.1, length.out = xlen)
ygrid <- seq(-10.1, 10.1, length.out = ylen)
zvals <- matrix(NA, ncol = xlen, nrow = ylen)
for (i in 1:xlen) {
  for (j in 1:ylen) {
    zvals[i, j] <- ssr(c(3, xgrid[i], ygrid[j]))
  }
}

contour(x = xgrid, y = ygrid, z = zvals,
        levels = c(1e3, 3e3, 6e3, 1e4))
```

As the correlation between regressors increases, matrix $X$ becomes nearly singular and OLS method becomes unstable. In the limit

$$
|corr(x_{i}, x_{j})| \rightarrow 1
$$

the dimension of the column space decreases

$$
\text{rank}(X) < p.
$$

The objective function $S(\beta)$ is no longer strictly convex, and there are infinitely many solutions of OLS.

Intuitively, we would like to estimate the coefficient $\beta_{1}$ as the influence of $x_{1}$ on $y$ without the influence of $x_{2}$. Since the regressors $x_{1}$ and $x_{2}$ are highly correlated, they vary together and the coefficient $\beta_{1}$ is difficult to estimate. The OLS method does not accomplish this, as it only minimizes the sum of squared residuals, i.e. the objective function $S(\beta)$.

## Why does this happen?
In general, we have a linear model

\begin{equation}
\label{eq: model}
y = X \beta + \epsilon
\end{equation}

and let for errors $\epsilon$ hold the assumption (Gauss–Markov):

$$
\mathbb{E}[\epsilon \epsilon^{T}] = \sigma^{2} I.
$$

The estimator according to the OLS method is

\begin{equation}
\label{eq: betahat}
\hat{\beta} = (X^{T}X)^{-1} X^{T} y.
\end{equation}

From \ref{eq: model} and \ref{eq: betahat}, we get

$$
\hat{\beta} - \beta = (X^{T}X)^{-1} X^{T} \epsilon
$$

Therefore, the covariance matrix for $\hat{\beta}$ is

$$
\begin{align}
\mathbb{E}[(\hat{\beta} - \beta) (\hat{\beta} - \beta)^{T}] \nonumber
&= \mathbb{E}[\left((X^{T}X)^{-1} X^{T} \epsilon\right) \left((X^{T}X)^{-1} X^{T} \epsilon\right)^{T}] \nonumber \\[1em]
&= \mathbb{E}[(X^{T}X)^{-1} X^{T} \epsilon \epsilon^{T} X (X^{T}X)^{-1}] \nonumber \\[1em]
&= (X^{T}X)^{-1} X^{T} \mathbb{E}[\epsilon \epsilon^{T}] X (X^{T}X)^{-1} \nonumber \\[1em]
&= \sigma^{2} (X^{T}X)^{-1}
\label{eq: variance}
\end{align}
$$

In the derivation, we took into account that the matrix $X^{T}X$ is symmetric and assumed that it is not stochastic and it is independent of $\epsilon$. The variance for the coefficient $\hat{\beta}_{k}$ is the $(k, k)$-th element of the covariance matrix.

The average distance between the estimator $\hat{\beta}$ and the actual $\beta$ is

$$
\begin{align}
\mathbb{E}[(\hat{\beta} - \beta)^{T} (\hat{\beta} - \beta)] \nonumber
&= \mathbb{E}[((X^{T}X)^{-1} X^{T} \epsilon)^{T} ((X^{T}X)^{-1} X^{T} \epsilon)] \nonumber \\[1em]
&= \mathbb{E}[\epsilon^{T} X (X^{T}X)^{-1} (X^{T}X)^{-1} X^{T} \epsilon] \nonumber \\[1em]
&= \mathbb{E}[\text{tr}(\epsilon^{T} X (X^{T}X)^{-1} (X^{T}X)^{-1} X^{T} \epsilon)] \nonumber \\[1em]
&= \mathbb{E}[\text{tr}(\epsilon \epsilon^{T} X (X^{T}X)^{-1} (X^{T}X)^{-1} X^{T} )] \nonumber \\[1em]
&= \sigma^{2} \text{tr}( X (X^{T}X)^{-1} (X^{T}X)^{-1} X^{T} )] \nonumber \\[1em]
&= \sigma^{2} \text{tr}( X^{T} X (X^{T}X)^{-1} (X^{T}X)^{-1})] \nonumber \\[1em]
&= \sigma^{2} \text{tr}((X^{T}X)^{-1})
\label{eq: distance}
\end{align}
$$

In the derivation, we used that the distance is a scalar, so its expected value will be equal to its trace. We then used the fact that the trace is invariant with respect to cyclic permutations:

$$
\text{tr}(ABCD) = \text{tr}(DABC).
$$

From \ref{eq: variance} and \ref{eq: distance}, we see that both the variance of the estimator and the distance of the estimator from the actual $\beta$ depend on the matrix $(X^{T}X)^{-1}$.

The reason why the variance of the estimator and the distance of the estimator from the actual $\beta$ become large, can be shown conveniently using singular value decomposition. Let

$$
X = U \Sigma V^{T}
$$

be the singular value decomposition of $X$ where $\Sigma$ contains all the singular values:

$$
\sigma_{1} \geq \sigma_{2} \geq \dots \geq \sigma_{p} > 0.
$$

Then

$$
\begin{align}
(X^{T}X)^{-1} &= (V \Sigma^{T} \Sigma V^{T})^{-1} \nonumber \\[1em]
&= (V^{T})^{-1} (\Sigma^{T} \Sigma)^{-1} V^{-1} \nonumber \\[1em]
&= V (\Sigma^{T} \Sigma)^{-1} V^{T} \nonumber \\[1em]
&= \sum_{j = 1}^{p} \frac{1}{\sigma_{j}^{2}} v_{j} v_{j}^{T}
\label{eq: svd}
\end{align}
$$

In the limit

$$
| corr(x_{i}, x_{j}) | \rightarrow 1
$$

matrix $X$ becomes singular and the smallest singular value vanishes:

$$
\sigma_{p} \rightarrow 0
$$

and from \ref{eq: svd}:

$$
(X^{T}X)^{-1} \rightarrow \infty
$$

Therefore, from \ref{eq: variance} and \ref{eq: distance}, both the variance of the $\hat{\beta}$ and the distance of $\hat{\beta}$ to the actual $\beta$ go to infinity as
$$
| corr(x_{i}, x_{j}) | \rightarrow 1.
$$
